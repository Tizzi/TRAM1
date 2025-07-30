# Phân tích Logic Hoạt Động - VET.FE.BEService

## Tổng quan hệ thống

**VET.FE.BEService** là một **Windows Service** được thiết kế để làm cầu nối giao tiếp giữa Frontend (FE) và Backend (BE) trong hệ thống thu phí đường bộ (Electronic Toll Collection - ETC). Phần mềm sử dụng **Topshelf** để chạy như một Windows Service.

## Kiến trúc và luồng hoạt động chính

### 1. Khởi động hệ thống

**Program.cs** - Điểm khởi đầu:
```csharp
private static void Main(string[] args)
{
    try
    {
        ConfigureService.Configure();
    }
    catch (Exception)
    {
    }
}
```

**ConfigureService.cs** - Cấu hình service:
- Sử dụng Topshelf để cấu hình Windows Service
- Tên service: "VETC.FE.BEService"
- Mô tả: "Communication with BE for FE Application"
- Chạy với quyền LocalSystem

### 2. Quản lý tiến trình (ProcMan)

**ProcMan.cs** - Quản lý các tiến trình:
```csharp
public void Start()
{
    try
    {
        CConfigFile configFile = new CConfigFile();
        configFile.LoadFile("LaneServer.config");
        foreach (Dictionary<string, string> laneConfigDict in configFile.allConfigValue)
        {
            BEProcConfig objConfig = new BEProcConfig();
            objConfig.LoadFromDict(laneConfigDict);
            BEProc processs = new BEProc(objConfig.BEProcName, 3, objConfig);
            ProcMan.listProc.Add(processs);
            processs.Run();
        }
    }
    catch (Exception ex)
    {
        ProcMan.logger.Error("ProcMan_Start: Error to start BE process, error message = " + ex.Message);
    }
}
```

**Chức năng:**
- Đọc file cấu hình `LaneServer.config`
- Tạo nhiều instance `BEProc` cho mỗi lane (làn xe)
- Mỗi lane có cấu hình riêng biệt
- Quản lý danh sách các tiến trình đang chạy

### 3. Xử lý giao tiếp BE (BEProc)

**BEProc.cs** là thành phần chính xử lý giao tiếp:

#### a) Kết nối với Backend
```csharp
private async Task<int> ConnectBE()
{
    try
    {
        // Tạo APIClient với thông tin kết nối
        this.TCOCClient = new APIClient(this.ConfigValue.BEIpAddr, this.ConfigValue.BEPort, 
                                       this.ConfigValue.BEHandshake, this.ConfigValue.BETimeout, 
                                       this.ConfigValue.BEAESKey, byIVArray);
        
        // Thực hiện kết nối
        ConnectRequestBody ConnectReq = new ConnectRequestBody(this.ConfigValue.BEUser, 
                                                            this.ConfigValue.BEPass, 
                                                            this.ConfigValue.StationID);
        ConnectResponeBody ConnectResp = await this.TCOCClient.ConnectAsync(...);
        
        // Thực hiện handshake
        SessionResponseBody ShakeResp = await this.TCOCClient.ShakeAsync();
        
        this.ConfigValue.IsConnect = true;
    }
    catch (Exception ex)
    {
        return -1;
    }
    return 0;
}
```

**Đặc điểm:**
- Sử dụng **TCOC API** để kết nối với BE
- Thực hiện authentication và handshake
- Tự động reconnect khi mất kết nối
- Sử dụng timer để quản lý việc kết nối lại

#### b) Socket Server
```csharp
public void Run()
{
    try
    {
        BEProc.logger.Debug("Start socket server");
        this.netServer.Start(IPAddress.Any, this.ConfigValue.BEClientPort);
        Task.Run(delegate
        {
            this.ConnectBEAsync();
        });
    }
    catch (Exception ex)
    {
        BEProc.logger.Error("Error to run process, error message = " + ex.Message);
    }
}
```

**Chức năng:**
- Lắng nghe kết nối từ Frontend qua socket
- Xử lý các message từ FE và chuyển tiếp đến BE
- Quản lý kết nối client với GUID

#### c) Xử lý Message
```csharp
private void OnSocketMessageReceived(object Sender, NetClientReceivedEventArgs<NetObject> Args)
{
    int MessageID = Args.Data.MessageID;
    int FromProc = (int)(((long)MessageID & (long)((ulong)(-16777216))) >> 24);
    int ToProc = (MessageID & 16711680) >> 16;
    int MesseageType = (MessageID & 65280) >> 8;
    int MesseageClass = MessageID & 255;
    
    switch (MessageID)
    {
        case 196608: // Test Connection
            // Xử lý test connection
            break;
        case 196609: // Checkin
            BEProcRequest CheckinReq = (BEProcRequest)Args.Data.Object;
            CTaskCheckinSocketServer taskCheckin = new CTaskCheckinSocketServer(...);
            taskCheckin.Run();
            break;
        case 196610: // Commit
            // Xử lý commit
            break;
        case 196611: // Rollback
            // Xử lý rollback
            break;
        case 196613: // Operation Mode
            // Xử lý thay đổi chế độ
            break;
        case 196614: // Connection Status
            // Trả về trạng thái kết nối
            break;
    }
}
```

**Các loại message được xử lý:**
- **0x30000 (196608)**: Test Connection - Kiểm tra kết nối
- **0x30001 (196609)**: Checkin - Đăng ký xe vào trạm
- **0x30002 (196610)**: Commit - Xác nhận giao dịch
- **0x30003 (196611)**: Rollback - Hoàn tác giao dịch
- **0x30005 (196613)**: Operation Mode - Thay đổi chế độ hoạt động
- **0x30006 (196614)**: Connection Status - Kiểm tra trạng thái kết nối

### 4. Các Task xử lý chính

#### CTaskCheckin
```csharp
private async Task<int> ProcessCheckin()
{
    CTaskCheckin.logger.Debug("Task check in begin");
    try
    {
        // Kiểm tra chế độ hoạt động
        if (this.ConfigVal.OperMode == 3)
        {
            // Chế độ test - trả về dữ liệu giả
            int[] values = new int[] { 35000, 50000, 75000, 150000, 200000 };
            Random random = new Random();
            int randomType = random.Next(values.Length);
            int randomValue = values[randomType];
            
            BEProcResp ProcResp = new BEProcResp
            {
                Status = 0,
                CarNum = this.ProcReq.CarNum,
                TransId = this.ProcReq.TransId,
                Etag = this.ProcReq.EtagNum,
                TicketID = this.ProcReq.CarNum,
                TicketType = 0,
                Price = randomValue,
                VehilceClass = randomType + 1,
                PlateNum = this.ProcReq.CarNum.ToString(),
                PlateType = 1,
                OfflineMode = 0
            };
            
            // Gửi response về FE
            byte[] data = ProcResp.Serialize();
            MCMessage message = new MCMessage();
            message.MsgCode = FEUtils.GetMessageID(3, 0, 128, 1);
            message.Data = data;
            message.DataSize = data.Count<byte>();
            this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message);
            return 0;
        }
        
        // Kiểm tra kết nối BE
        if (!this.ConfigVal.IsConnect)
        {
            this.SendOfflineResp();
            return 0;
        }
        
        // Gọi API BE thực tế
        ChargeRequestBody CheckinReq = new ChargeRequestBody(this.ProcReq.EtagNum, 
                                                           this.ConfigVal.StationID, 
                                                           this.ConfigVal.LaneNum, 
                                                           this.ProcReq.TID, "");
        CheckinResponseBody CheckinResp = await this.TCOCClient.CheckinAsync(this.ProcReq.CarNum, CheckinReq);
        
        // Xử lý response từ BE
        if (CheckinResp == null || CheckinResp.Status > TransactionStatus.Success)
        {
            // Gửi response lỗi
            BEProcResp ProcResp = new BEProcResp();
            ProcResp.Status = (int)CheckinResp.Status;
            ProcResp.CarNum = this.ProcReq.CarNum;
            ProcResp.TransId = this.ProcReq.TransId;
            ProcResp.Etag = CheckinResp.Etag;
            ProcResp.OfflineMode = 0;
            
            byte[] data = ProcResp.Serialize();
            MCMessage message = new MCMessage();
            message.MsgCode = FEUtils.GetMessageID(3, 0, 128, 1);
            message.Data = data;
            message.DataSize = data.Count<byte>();
            this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message);
            return -1;
        }
        
        // Gửi response thành công
        BEProcResp ProcResp2 = new BEProcResp();
        ProcResp2.Status = (int)CheckinResp.Status;
        ProcResp2.CarNum = this.ProcReq.CarNum;
        ProcResp2.TransId = this.ProcReq.TransId;
        ProcResp2.Etag = CheckinResp.Etag;
        ProcResp2.TicketID = CheckinResp.TicketId;
        ProcResp2.TicketType = (int)CheckinResp.TicketType;
        ProcResp2.Price = CheckinResp.Price;
        ProcResp2.VehilceClass = (int)CheckinResp.VehicleType;
        ProcResp2.PlateNum = CheckinResp.Plate;
        ProcResp2.PlateType = (int)CheckinResp.PlateType;
        ProcResp2.OfflineMode = 0;
        
        byte[] data2 = ProcResp2.Serialize();
        MCMessage message2 = new MCMessage();
        message2.MsgCode = FEUtils.GetMessageID(3, 0, 128, 1);
        message2.Data = data2;
        message2.DataSize = data2.Count<byte>();
        this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message2);
    }
    catch (Exception ex)
    {
        CTaskCheckin.logger.Error("TCOC: Error to check in, error message = " + ex.Message);
        if (!this.ConfigVal.IsConnect)
        {
            this.SendOfflineResp();
            return 0;
        }
        // Gửi response lỗi
        byte[] data = new BEProcResp
        {
            Status = -1,
            CarNum = this.ProcReq.CarNum,
            TransId = this.ProcReq.TransId,
            Etag = this.ProcReq.EtagNum
        }.Serialize();
        MCMessage message = new MCMessage();
        message.MsgCode = FEUtils.GetMessageID(3, 0, 128, 1);
        message.Data = data;
        message.DataSize = data.Count<byte>();
        this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message);
        return -1;
    }
    return 0;
}
```

**Chức năng:**
- Xử lý đăng ký xe vào trạm thu phí
- Hỗ trợ chế độ test với dữ liệu giả
- Xử lý offline mode khi không kết nối được BE
- Gửi thông tin xe, eTag, giá vé về FE

#### CTaskCommit
```csharp
private async Task<int> ProcessCommit()
{
    CTaskCommit.logger.Debug("Task commit begin");
    try
    {
        // Kiểm tra chế độ test
        if (this.ConfigVal.OperMode == 3)
        {
            // Trả về response thành công giả
            byte[] data = new BEProcResp
            {
                Status = 0,
                CarNum = this.ProcReq.CarNum,
                TransId = this.ProcReq.TransId,
                Etag = this.ProcReq.EtagNum
            }.Serialize();
            MCMessage message = new MCMessage();
            message.MsgCode = FEUtils.GetMessageID(3, 0, 128, 2);
            message.Data = data;
            message.DataSize = data.Count<byte>();
            this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message);
            return 0;
        }
        
        // Kiểm tra kết nối
        if (!this.ConfigVal.IsConnect)
        {
            this.SendOfflineResp();
            return 0;
        }
        
        // Gọi API commit với BE
        TransactionRequestBody CommitReq = new TransactionRequestBody(this.ProcReq.EtagNum, 
                                                                   this.ConfigVal.StationID, 
                                                                   this.ConfigVal.LaneNum, 
                                                                   this.ProcReq.TicketID, 
                                                                   (PlateStatus)this.ProcReq.PlateMatch, 
                                                                   this.ProcReq.PlateNum, 
                                                                   this.ProcReq.ImageCount, 
                                                                   this.ProcReq.VehLength, 0);
        TransactionResponseBody CommitResp = await this.TCOCClient.CommitAsync(this.ProcReq.CarNum, CommitReq);
        
        // Xử lý response
        if (CommitResp == null || CommitResp.Status > TransactionStatus.Success)
        {
            // Gửi response lỗi
            BEProcResp ProcResp = new BEProcResp();
            ProcResp.Status = (int)CommitResp.Status;
            ProcResp.CarNum = this.ProcReq.CarNum;
            ProcResp.TransId = this.ProcReq.TransId;
            ProcResp.Etag = this.ProcReq.EtagNum;
            
            byte[] data = ProcResp.Serialize();
            MCMessage message = new MCMessage();
            message.MsgCode = FEUtils.GetMessageID(3, 0, 128, 2);
            message.Data = data;
            message.DataSize = data.Count<byte>();
            this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message);
            return -1;
        }
        
        // Gửi response thành công
        BEProcResp ProcResp2 = new BEProcResp();
        ProcResp2.Status = (int)CommitResp.Status;
        ProcResp2.CarNum = this.ProcReq.CarNum;
        ProcResp2.TransId = this.ProcReq.TransId;
        ProcResp2.Etag = this.ProcReq.EtagNum;
        
        byte[] data2 = ProcResp2.Serialize();
        MCMessage message2 = new MCMessage();
        message2.MsgCode = FEUtils.GetMessageID(3, 0, 128, 2);
        message2.Data = data2;
        message2.DataSize = data2.Count<byte>();
        this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message2);
    }
    catch (Exception ex)
    {
        CTaskCommit.logger.Error("TCOC: Error to check in, error message = " + ex.Message);
        // Gửi response lỗi
        byte[] data = new BEProcResp
        {
            Status = -1,
            CarNum = this.ProcReq.CarNum,
            TransId = this.ProcReq.TransId,
            Etag = this.ProcReq.EtagNum
        }.Serialize();
        MCMessage message = new MCMessage();
        message.MsgCode = FEUtils.GetMessageID(3, 0, 128, 2);
        message.Data = data;
        message.DataSize = data.Count<byte>();
        this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message);
        return -1;
    }
    return 0;
}
```

**Chức năng:**
- Xác nhận giao dịch với BE
- Gửi thông tin biển số, hình ảnh, loại xe
- Xử lý kết quả commit và trả về FE

#### CTaskRollback
```csharp
private async Task<int> ProcessRollback()
{
    CTaskRollback.logger.Debug("Task commit begin");
    try
    {
        // Kiểm tra chế độ test
        if (this.ConfigVal.OperMode == 3)
        {
            // Trả về response thành công giả
            byte[] data = new BEProcResp
            {
                Status = 0,
                CarNum = this.ProcReq.CarNum,
                TransId = this.ProcReq.TransId,
                Etag = this.ProcReq.EtagNum
            }.Serialize();
            MCMessage message = new MCMessage();
            message.MsgCode = FEUtils.GetMessageID(3, 0, 128, 3);
            message.Data = data;
            message.DataSize = data.Count<byte>();
            this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message);
            return 0;
        }
        
        // Kiểm tra kết nối
        if (!this.ConfigVal.IsConnect)
        {
            this.SendOfflineResp();
            return 0;
        }
        
        // Gọi API rollback với BE
        RollbackRequestBody RollbackReq = new RollbackRequestBody(this.ProcReq.EtagNum, 
                                                                 this.ConfigVal.StationID, 
                                                                 this.ConfigVal.LaneNum, 
                                                                 this.ProcReq.TicketID, 
                                                                 (PlateStatus)this.ProcReq.PlateMatch, 
                                                                 this.ProcReq.PlateNum, 
                                                                 this.ProcReq.ImageCount, -1);
        RollbackResponseBody RollbackResp = await this.TCOCClient.RollbackAsync(this.ProcReq.CarNum, RollbackReq);
        
        // Xử lý response
        if (RollbackResp == null || RollbackResp.Status > TransactionStatus.Success)
        {
            // Gửi response lỗi
            BEProcResp ProcResp = new BEProcResp();
            ProcResp.Status = (int)RollbackResp.Status;
            ProcResp.CarNum = this.ProcReq.CarNum;
            ProcResp.TransId = this.ProcReq.TransId;
            ProcResp.Etag = this.ProcReq.EtagNum;
            
            byte[] data = ProcResp.Serialize();
            MCMessage message = new MCMessage();
            message.MsgCode = FEUtils.GetMessageID(3, 0, 128, 3);
            message.Data = data;
            message.DataSize = data.Count<byte>();
            this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message);
            return -1;
        }
        
        // Gửi response thành công
        BEProcResp ProcResp2 = new BEProcResp();
        ProcResp2.Status = (int)RollbackResp.Status;
        ProcResp2.CarNum = this.ProcReq.CarNum;
        ProcResp2.TransId = this.ProcReq.TransId;
        ProcResp2.Etag = this.ProcReq.EtagNum;
        
        byte[] data2 = ProcResp2.Serialize();
        MCMessage message2 = new MCMessage();
        message2.MsgCode = FEUtils.GetMessageID(3, 0, 128, 3);
        message2.Data = data2;
        message2.DataSize = data2.Count<byte>();
        this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message2);
    }
    catch (Exception ex)
    {
        CTaskRollback.logger.Error("TCOC: Error to check in, error message = " + ex.Message);
        // Gửi response lỗi
        byte[] data = new BEProcResp
        {
            Status = -1,
            CarNum = this.ProcReq.CarNum,
            TransId = this.ProcReq.TransId,
            Etag = this.ProcReq.EtagNum
        }.Serialize();
        MCMessage message = new MCMessage();
        message.MsgCode = FEUtils.GetMessageID(3, 0, 128, 3);
        message.Data = data;
        message.DataSize = data.Count<byte>();
        this.ProcServer.SendMessage(this.ConfigVal.CoreProcName, message);
        return -1;
    }
    return 0;
}
```

**Chức năng:**
- Hoàn tác giao dịch khi có lỗi
- Hủy bỏ thông tin đã đăng ký
- Xử lý rollback transaction

### 5. Tích hợp với ITD (Intelligent Transportation Data)

#### CTaskCheckEtagITD
```csharp
private int ProcessCheckEtag()
{
    try
    {
        // Kiểm tra URL service
        if (string.IsNullOrEmpty(this.ConfigVal.ITDCheckEtagServiceUrl))
        {
            return 0;
        }
        
        // Tạo web service client
        WebHttpBinding binding = new WebHttpBinding();
        string uri = this.ConfigVal.ITDCheckEtagServiceUrl;
        ICheckEtagService proxy = new ChannelFactory<ICheckEtagService>(binding, uri)
        {
            Endpoint = { Behaviors = { new WebHttpBehavior() } }
        }.CreateChannel();
        
        // Tạo thông tin eTag
        EtagInformation info = new EtagInformation
        {
            EtagId = this.req.EtagNum,
            CheckResult = false,
            CheckTime = DateTime.Now,
            DetectTime = DateTime.Now,
            DenyTime = this.ConfigVal.ITDCheckEtagServiceDenyTime,
            LaneId = this.req.LaneNum.ToString("000")
        };
        
        // Gọi service ITD
        EtagInformation result = proxy.RequestUpdateEtag(info);
        
        // Log kết quả
        object[] objArray = new object[] { result.EtagId, result.CheckResult, result.CheckTime, result.LaneId };
        CTaskCheckEtagITD.logger.Info(string.Format("CTaskCheckEtagITD: UpdateEtag {0}, result {1}, lastTime {2}, Lane {3}", objArray));
    }
    catch (Exception ex)
    {
        CTaskCheckEtagITD.logger.Error("CTaskCheckEtagITD: Error to check etag, error message = " + ex.Message);
        CTaskCheckEtagITD.logger.Error("CTaskCheckEtagITD: Error to check etag, error trace = " + ex.StackTrace);
    }
    return 0;
}
```

**Chức năng:**
- Gọi web service ITD để kiểm tra eTag
- Cập nhật thông tin eTag với hệ thống ITD
- Log kết quả kiểm tra

#### ICheckEtagService Interface
```csharp
[ServiceContract]
public interface ICheckEtagService
{
    [OperationContract]
    [WebInvoke(Method = "GET", ResponseFormat = WebMessageFormat.Xml, BodyStyle = WebMessageBodyStyle.Wrapped, UriTemplate = "xml/{id}")]
    string XMLData(string id);

    [OperationContract]
    [WebInvoke(Method = "GET", ResponseFormat = WebMessageFormat.Json, BodyStyle = WebMessageBodyStyle.Wrapped, UriTemplate = "json/{id}")]
    string JSONData(string id);

    [OperationContract]
    [WebInvoke(Method = "POST", ResponseFormat = WebMessageFormat.Xml, RequestFormat = WebMessageFormat.Xml, BodyStyle = WebMessageBodyStyle.Bare, UriTemplate = "CheckService")]
    ShakeStatus CheckService(ShakeStatus rData);

    [OperationContract]
    [WebInvoke(Method = "POST", ResponseFormat = WebMessageFormat.Xml, RequestFormat = WebMessageFormat.Xml, BodyStyle = WebMessageBodyStyle.Bare, UriTemplate = "RequestCheckEtag")]
    EtagInformation RequestCheckEtag(EtagInformation rData);

    [OperationContract]
    [WebInvoke(Method = "POST", ResponseFormat = WebMessageFormat.Xml, RequestFormat = WebMessageFormat.Xml, BodyStyle = WebMessageBodyStyle.Bare, UriTemplate = "RequestUpdateEtag")]
    EtagInformation RequestUpdateEtag(EtagInformation rData);

    [OperationContract]
    [WebInvoke(Method = "POST", ResponseFormat = WebMessageFormat.Xml, RequestFormat = WebMessageFormat.Xml, BodyStyle = WebMessageBodyStyle.Bare, UriTemplate = "ConnectEtagService")]
    ShakeStatus Connect(ShakeStatus rData);
}
```

### 6. Hỗ trợ Socket Server

Các class `CTaskCheckinSocketServer`, `CTaskCommitSocketServer`, `CTaskRollbackSocketServer` xử lý giao tiếp qua socket thay vì message queue.

**Ví dụ CTaskCheckinSocketServer:**
```csharp
private async Task<int> ProcessCheckin()
{
    CTaskCheckinSocketServer.logger.Debug("Task check in begin");
    try
    {
        // Kiểm tra chế độ test
        if (this.ConfigVal.OperMode == 3)
        {
            // Tạo dữ liệu giả
            int[] values = new int[] { 35000, 50000, 75000, 150000, 200000 };
            Random random = new Random();
            int randomType = random.Next(values.Length);
            int randomValue = values[randomType];
            
            // Kiểm tra version TCOC
            if (this.ConfigVal.BETCOCVer == 2)
            {
                // Sử dụng BEProcResp_v2
                BEProcResp_v2 ProcResp = new BEProcResp_v2();
                ProcResp.Status = 0;
                ProcResp.CarNum = this.ProcReq.CarNum;
                ProcResp.LaneNum = this.ProcReq.LaneNum;
                ProcResp.TransId = this.ProcReq.TransId;
                ProcResp.Etag = this.ProcReq.EtagNum;
                ProcResp.TicketID = this.ProcReq.CarNum;
                ProcResp.TicketType = 0;
                ProcResp.Price = 0;
                ProcResp.VehilceClass = 5;
                ProcResp.PlateNum = "29A" + this.ProcReq.CarNum.ToString() + "T";
                ProcResp.PlateType = 1;
                ProcResp.OfflineMode = 0;
                ProcResp.DiscountType = 1;
                
                NetObject resp = new NetObject();
                resp.MessageID = FEUtils.GetMessageID(3, 0, 128, 1);
                resp.Object = ProcResp;
                this.SocketServer.DispatchTo(this.incomingGuid, resp);
            }
            else
            {
                // Sử dụng BEProcResp thường
                BEProcResp ProcResp = new BEProcResp();
                ProcResp.Status = 0;
                ProcResp.CarNum = this.ProcReq.CarNum;
                ProcResp.LaneNum = this.ProcReq.LaneNum;
                ProcResp.TransId = this.ProcReq.TransId;
                ProcResp.Etag = this.ProcReq.EtagNum;
                ProcResp.TicketID = this.ProcReq.CarNum;
                ProcResp.TicketType = 0;
                ProcResp.Price = 0;
                ProcResp.VehilceClass = 5;
                ProcResp.PlateNum = "29A" + this.ProcReq.CarNum.ToString() + "T";
                ProcResp.PlateType = 1;
                ProcResp.OfflineMode = 0;
                
                NetObject resp = new NetObject();
                resp.MessageID = FEUtils.GetMessageID(3, 0, 128, 1);
                resp.Object = ProcResp;
                this.SocketServer.DispatchTo(this.incomingGuid, resp);
            }
            return 0;
        }
        
        // Kiểm tra kết nối
        if (!this.ConfigVal.IsConnect)
        {
            this.SendOfflineResp();
            return 0;
        }
        
        // Gọi API BE thực tế
        if (this.ConfigVal.BETCOCVer == 2)
        {
            // Sử dụng API v2
            ChargeRequestBody CheckinReq = new ChargeRequestBody(this.ProcReq.EtagNum, 
                                                               this.ConfigVal.StationID, 
                                                               this.ProcReq.LaneNum, 
                                                               this.ProcReq.TID, "");
            CheckinResponseBody_v2 CheckinResp = await this.TCOCClient.CheckinAsync_v2(this.ProcReq.CarNum, CheckinReq);
            
            // Xử lý response v2
            if (CheckinResp == null || CheckinResp.Status > TransactionStatus.Success)
            {
                BEProcResp_v2 ProcResp = new BEProcResp_v2();
                ProcResp.Status = (int)CheckinResp.Status;
                ProcResp.CarNum = this.ProcReq.CarNum;
                ProcResp.LaneNum = this.ProcReq.LaneNum;
                ProcResp.TransId = this.ProcReq.TransId;
                ProcResp.Etag = CheckinResp.Etag;
                ProcResp.OfflineMode = 0;
                ProcResp.DiscountType = 1;
                
                NetObject resp = new NetObject();
                resp.MessageID = FEUtils.GetMessageID(3, 0, 128, 1);
                resp.Object = ProcResp;
                this.SocketServer.DispatchTo(this.incomingGuid, resp);
                return -1;
            }
            
            // Response thành công v2
            BEProcResp_v2 ProcResp2 = new BEProcResp_v2();
            ProcResp2.Status = (int)CheckinResp.Status;
            ProcResp2.CarNum = this.ProcReq.CarNum;
            ProcResp2.LaneNum = this.ProcReq.LaneNum;
            ProcResp2.TransId = this.ProcReq.TransId;
            ProcResp2.Etag = CheckinResp.Etag;
            ProcResp2.TicketID = CheckinResp.TicketId;
            ProcResp2.TicketType = (int)CheckinResp.TicketType;
            ProcResp2.Price = CheckinResp.Price;
            ProcResp2.VehilceClass = (int)CheckinResp.VehicleType;
            ProcResp2.PlateNum = CheckinResp.Plate;
            ProcResp2.PlateType = (int)CheckinResp.PlateType;
            ProcResp2.OfflineMode = 0;
            ProcResp2.DiscountType = CheckinResp.DiscountType;
            
            NetObject resp2 = new NetObject();
            resp2.MessageID = FEUtils.GetMessageID(3, 0, 128, 1);
            resp2.Object = ProcResp2;
            this.SocketServer.DispatchTo(this.incomingGuid, resp2);
        }
        else
        {
            // Sử dụng API v1
            ChargeRequestBody CheckinReq = new ChargeRequestBody(this.ProcReq.EtagNum, 
                                                               this.ConfigVal.StationID, 
                                                               this.ProcReq.LaneNum, 
                                                               this.ProcReq.TID, "");
            CheckinResponseBody CheckinResp = await this.TCOCClient.CheckinAsync(this.ProcReq.CarNum, CheckinReq);
            
            // Xử lý response v1
            if (CheckinResp == null || CheckinResp.Status > TransactionStatus.Success)
            {
                BEProcResp ProcResp = new BEProcResp();
                ProcResp.Status = (int)CheckinResp.Status;
                ProcResp.CarNum = this.ProcReq.CarNum;
                ProcResp.LaneNum = this.ProcReq.LaneNum;
                ProcResp.TransId = this.ProcReq.TransId;
                ProcResp.Etag = CheckinResp.Etag;
                ProcResp.OfflineMode = 0;
                
                NetObject resp = new NetObject();
                resp.MessageID = FEUtils.GetMessageID(3, 0, 128, 1);
                resp.Object = ProcResp;
                this.SocketServer.DispatchTo(this.incomingGuid, resp);
                return -1;
            }
            
            // Response thành công v1
            BEProcResp ProcResp2 = new BEProcResp();
            ProcResp2.Status = (int)CheckinResp.Status;
            ProcResp2.CarNum = this.ProcReq.CarNum;
            ProcResp2.LaneNum = this.ProcReq.LaneNum;
            ProcResp2.TransId = this.ProcReq.TransId;
            ProcResp2.Etag = CheckinResp.Etag;
            ProcResp2.TicketID = CheckinResp.TicketId;
            ProcResp2.TicketType = (int)CheckinResp.TicketType;
            ProcResp2.Price = CheckinResp.Price;
            ProcResp2.VehilceClass = (int)CheckinResp.VehicleType;
            ProcResp2.PlateNum = CheckinResp.Plate;
            ProcResp2.PlateType = (int)CheckinResp.PlateType;
            ProcResp2.OfflineMode = 0;
            
            NetObject resp2 = new NetObject();
            resp2.MessageID = FEUtils.GetMessageID(3, 0, 128, 1);
            resp2.Object = ProcResp2;
            this.SocketServer.DispatchTo(this.incomingGuid, resp2);
        }
    }
    catch (Exception ex)
    {
        CTaskCheckinSocketServer.logger.Error("TCOC: Error to check in, error message = " + ex.Message);
        if (!this.ConfigVal.IsConnect)
        {
            this.SendOfflineResp();
            return 0;
        }
        
        // Gửi response lỗi
        if (this.ConfigVal.BETCOCVer == 2)
        {
            BEProcResp_v2 ProcResp = new BEProcResp_v2();
            ProcResp.Status = -1;
            ProcResp.CarNum = this.ProcReq.CarNum;
            ProcResp.LaneNum = this.ProcReq.LaneNum;
            ProcResp.TransId = this.ProcReq.TransId;
            ProcResp.Etag = this.ProcReq.EtagNum;
            ProcResp.DiscountType = 1;
            
            NetObject resp = new NetObject();
            resp.MessageID = FEUtils.GetMessageID(3, 0, 128, 1);
            resp.Object = ProcResp;
            this.SocketServer.DispatchTo(this.incomingGuid, resp);
        }
        else
        {
            BEProcResp ProcResp = new BEProcResp();
            ProcResp.Status = -1;
            ProcResp.CarNum = this.ProcReq.CarNum;
            ProcResp.LaneNum = this.ProcReq.LaneNum;
            ProcResp.TransId = this.ProcReq.TransId;
            ProcResp.Etag = this.ProcReq.EtagNum;
            
            NetObject resp = new NetObject();
            resp.MessageID = FEUtils.GetMessageID(3, 0, 128, 1);
            resp.Object = ProcResp;
            this.SocketServer.DispatchTo(this.incomingGuid, resp);
        }
        return -1;
    }
    return 0;
}
```

### 7. Chế độ hoạt động

```csharp
private void HandleOperMode(int inOperMode)
{
    try
    {
        switch (inOperMode)
        {
            case 1: // Normal mode
                this.ConfigValue.OperMode = inOperMode;
                if (!this.ConfigValue.IsConnect)
                {
                    this.timerReconnect.Enabled = true;
                }
                break;
            case 2: // Maintenance mode
                break;
            case 3: // Test mode
                this.ConfigValue.OperMode = inOperMode;
                if (!this.ConfigValue.IsConnect)
                {
                    this.timerReconnect.Enabled = false;
                }
                break;
            default:
                BEProc.logger.Debug("HandleOperMode: Do not support operation mode = " + inOperMode.ToString());
                break;
        }
    }
    catch (Exception ex)
    {
        BEProc.logger.Error(ex.Message);
        BEProc.logger.Error(ex.StackTrace);
    }
}
```

**Các chế độ:**
- **Mode 1**: Kết nối bình thường với BE
- **Mode 2**: Chế độ bảo trì
- **Mode 3**: Chế độ test - trả về dữ liệu giả

## Đặc điểm kỹ thuật

### 1. Multi-threading
- Sử dụng `Task.Run()` cho các operation bất đồng bộ
- Xử lý song song nhiều request từ FE

### 2. Error Handling
- Xử lý lỗi comprehensive với logging
- Fallback mechanism khi BE không khả dụng
- Offline mode khi mất kết nối

### 3. Configuration Management
- Đọc cấu hình từ file `LaneServer.config`
- Hỗ trợ nhiều lane với cấu hình riêng biệt
- Cấu hình linh hoạt cho từng trạm

### 4. Logging System
- Sử dụng `CLogManager` để ghi log
- Log chi tiết cho debug và monitoring
- Error tracking và troubleshooting

### 5. Reconnection Mechanism
- Tự động kết nối lại khi mất kết nối BE
- Timer-based reconnection với configurable interval
- Connection status monitoring

### 6. Offline Mode
- Hoạt động offline khi không kết nối được BE
- Cache và queue mechanism
- Graceful degradation

### 7. Version Support
- Hỗ trợ TCOC API v1 và v2
- Backward compatibility
- Version-specific response handling

## Luồng dữ liệu

```
Frontend (FE) → Socket/Message → VET.FE.BEService → TCOC API → Backend (BE)
                ↑                                    ↓
                ← Response ← ← ← ← ← ← ← ← ← ← ← ← ← ←
```

### Chi tiết luồng xử lý:

1. **FE gửi request** qua socket hoặc message queue
2. **VET.FE.BEService** nhận và parse message
3. **Kiểm tra chế độ hoạt động** (test/online/offline)
4. **Gọi API BE** nếu ở chế độ online
5. **Xử lý response** từ BE
6. **Gửi response** về FE
7. **Log kết quả** và cập nhật trạng thái

## Kết luận

Phần mềm **VET.FE.BEService** đóng vai trò như một **middleware** quan trọng trong hệ thống thu phí tự động, đảm bảo giao tiếp ổn định giữa các thành phần FE và BE. Với kiến trúc modular, error handling robust, và hỗ trợ nhiều chế độ hoạt động, phần mềm này có thể đáp ứng các yêu cầu khác nhau của hệ thống thu phí đường bộ.

**Các điểm mạnh:**
- Kiến trúc modular và dễ mở rộng
- Error handling comprehensive
- Hỗ trợ offline mode
- Multi-threading hiệu quả
- Logging system chi tiết
- Configuration linh hoạt

**Ứng dụng:**
- Hệ thống thu phí đường bộ tự động
- Electronic Toll Collection (ETC)
- Intelligent Transportation Systems (ITS)
- Traffic Management Systems 