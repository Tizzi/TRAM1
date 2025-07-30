# VETC.FE.NPRService - Hệ thống Nhận dạng Biển số Xe

## Tổng quan

VETC.FE.NPRService là một Windows Service được thiết kế để nhận dạng biển số xe (License Plate Recognition - NPR) cho hệ thống thu phí điện tử VETC (Vietnam Electronic Toll Collection). Đây là một phần của hệ thống Front-End (FE) cho các làn thu phí MTC.

## Mục đích và chức năng chính

- **Nhận dạng biển số xe**: Tự động nhận dạng biển số từ ảnh chụp xe
- **Xử lý đa luồng**: Hỗ trợ nhiều lane thu phí đồng thời
- **Giao tiếp đa giao thức**: TCP Socket và Message Queue
- **Windows Service**: Chạy như một dịch vụ hệ thống
- **Logging chi tiết**: Ghi log để debug và monitor

## Kiến trúc tổng thể

```
┌─────────────────┐
│   Program.cs    │ ← Điểm khởi đầu
└─────────┬───────┘
          │
┌─────────▼───────┐
│ConfigureService.cs│ ← Cấu hình Windows Service
└─────────┬───────┘
          │
┌─────────▼───────┐
│   ProcMan.cs    │ ← Quản lý tiến trình
└─────────┬───────┘
          │
┌─────────▼───────┐
│   NPRProc.cs    │ ← Xử lý chính
└─────────┬───────┘
          │
    ┌─────┴─────┐
    │           │
┌───▼───┐   ┌───▼───┐
│CTaskHC│   │CTaskReg│
│       │   │PlateNum│
└───────┘   └───────┘
```

## Chi tiết từng thành phần

### 1. Program.cs - Điểm khởi đầu

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

**Chức năng:**
- Điểm khởi đầu ứng dụng
- Gọi ConfigureService.Configure() để thiết lập Windows Service
- Xử lý exception cơ bản

### 2. ConfigureService.cs - Cấu hình Windows Service

```csharp
HostFactory.Run(delegate(HostConfigurator configure)
{
    configure.Service(delegate(ServiceConfigurator<ProcMan> service)
    {
        service.ConstructUsing((HostSettings s) => new ProcMan());
        service.WhenStarted(delegate(ProcMan s) { s.Start(); });
        service.WhenStopped(delegate(ProcMan s) { s.Stop(); });
        service.WhenShutdown(delegate(ProcMan s) { s.Stop(); });
    });
    configure.RunAsLocalSystem();
    configure.SetServiceName("VETC.FE.NPRService.MTC");
    configure.SetDisplayName("VETC.FE.NPRService.MTC");
    configure.SetDescription("recognize plate number for FE Application (MTC lane)");
});
```

**Chức năng:**
- Sử dụng Topshelf để cấu hình Windows Service
- Service chạy với quyền LocalSystem
- Tên service: "VETC.FE.NPRService.MTC"
- Mô tả: nhận dạng biển số cho ứng dụng FE (làn MTC)
- Định nghĩa các event Start, Stop, Shutdown

### 3. ProcMan.cs - Process Manager

**Chức năng chính:**
- Quản lý các tiến trình NPR
- Đọc file cấu hình
- Khởi tạo và quản lý nhiều instance NPRProc

**Logic Start():**
```csharp
public void Start()
{
    try
    {
        // 1. Đọc file cấu hình
        CConfigFile configFile = new CConfigFile();
        configFile.LoadFile("LaneServer.config");
        
        // 2. Duyệt qua từng cấu hình lane
        foreach (Dictionary<string, string> laneConfigDict in configFile.allConfigValue)
        {
            NPRProcConfig objConfig = new NPRProcConfig();
            objConfig.LoadFromDict(laneConfigDict);
            
            // 3. Tách danh sách port TCP
            List<string> listTCPPort = objConfig.NPRServPort.Split(';').ToList();
            
            // 4. Tạo process cho mỗi port
            foreach (string port in listTCPPort)
            {
                int port_num = Convert.ToInt32(port);
                NPRProc processs = new NPRProc(objConfig.NPRProcName, 4, objConfig, port_num);
                ProcMan.listProc.Add(processs);
                processs.Run();
            }
        }
    }
    catch (Exception ex)
    {
        ProcMan.logger.Error("Error to start NPR process: " + ex.Message);
    }
}
```

**Logic Stop():**
```csharp
public void Stop()
{
    try
    {
        // Dừng tất cả các process
        foreach (NPRProc proc in ProcMan.listProc)
        {
            proc.Stop();
        }
        ProcMan.listProc.Clear();
    }
    catch (Exception ex)
    {
        ProcMan.logger.Error(ex.Message);
    }
}
```

### 4. NPRProc.cs - NPR Process

**Chức năng chính:**
- Khởi tạo message queue và socket server
- Xử lý các message từ cả message queue và socket
- Quản lý kết nối client
- Thực hiện nhận dạng biển số

**Constructor:**
```csharp
public NPRProc(string Name, int ID, NPRProcConfig inConfigValue, int inTCPPort)
{
    // 1. Lưu cấu hình
    this.ConfigValue = inConfigValue;
    this.MyProcName = Name;
    this.MyProcID = ID;
    this.TCPPort = inTCPPort;
    
    // 2. Khởi tạo message queue
    NPRProc.ProcServer = new CProcMsg(Name);
    ProcServer.MessageReceived += this.OnProcMessage;
    
    // 3. Khởi tạo socket server
    this.netServer = new NetObjectServer();
    this.netServer.EchoMode = NetEchoMode.None;
    this.netServer.OnReceived += this.OnSocketMessageReceived;
    this.netServer.OnClientConnected += this.OnSocketConnected;
    this.netServer.OnClientDisconnected += this.OnSocketDisconnected;
    
    // 4. Khởi tạo ANPR object
    this.anprObj = new CANPR();
}
```

**Xử lý Socket Messages:**
```csharp
private void OnSocketMessageReceived(object Sender, NetClientReceivedEventArgs<NetObject> Args)
{
    int MessageID = Args.Data.MessageID;
    switch (MessageID)
    {
        case 262144: // 0x40000 - Health Check
            CTaskHC taskHC = new CTaskHC(this.netServer, Args.Guid);
            taskHC.Run();
            break;
            
        case 262145: // 0x40001 - Nhận dạng biển số
            NPRRegPlateReq nprReq = (NPRRegPlateReq)Args.Data.Object;
            CTaskRegPlateNum taskRegPlateNumer = new CTaskRegPlateNum(
                this.netServer, Args.Guid, nprReq, this.anprObj, 
                this.ConfigValue.NPRMaxRetry, this.ConfigValue.NPRDelayTime);
            taskRegPlateNumer.Run();
            break;
            
        case 262146: // 0x40002 - Start Request
            CTaskStartReq taskStartReq = new CTaskStartReq(this.netServer, Args.Guid, this.ConfigValue);
            taskStartReq.Run();
            break;
            
        default:
            NPRProc.logger.Debug("Do not handle message, message id = 0x" + MessageID.ToString("X8"));
            break;
    }
}
```

**Xử lý Process Messages:**
```csharp
private void OnProcMessage(object Sender, ProcMessageArg Args)
{
    int MessageID = Args.MessageCode;
    int FromProc = (MessageID & 0xFF000000) >> 24;
    int ToProc = (MessageID & 0x00FF0000) >> 16;
    int MessageType = (MessageID & 0x0000FF00) >> 8;
    int MessageClass = MessageID & 0x000000FF;
    
    if (ToProc != 4) // Process ID = 4
    {
        NPRProc.logger.Error("Wrong Process ID = 0x" + ToProc.ToString("X8"));
        return;
    }
    
    if (MessageID == 262145) // 0x40001
    {
        NPRRegPlateReq nprReq = new NPRRegPlateReq();
        nprReq.Deserialize(Args.MessageData.Data);
        this.HandleRegPlateNumReq(nprReq);
    }
}
```

### 5. CTaskHC.cs - Health Check Task

**Chức năng:**
- Xử lý yêu cầu kiểm tra sức khỏe hệ thống
- Trả về "Test Connection" để xác nhận service đang hoạt động

**Logic:**
```csharp
private int ProcessHC()
{
    try
    {
        NetObject resp = new NetObject();
        resp.MessageID = 327679; // 0x4FFFF
        resp.Object = "Test Connection";
        
        this.netServer.DispatchTo(this.guid, resp);
    }
    catch (Exception ex)
    {
        CTaskHC.logger.Error("Error to send TCP message: " + ex.Message);
    }
    return 0;
}
```

### 6. CTaskRegPlateNum.cs - Nhận dạng Biển số

**Chức năng chính:**
- Nhận dạng biển số xe từ file ảnh
- Cắt ảnh biển số từ ảnh gốc
- Trả về kết quả nhận dạng

**Logic nhận dạng:**
```csharp
private int RegPlateNum()
{
    try
    {
        // 1. Nhận dạng biển số
        CANPR_Result plate_result = this.anprObj.RecognizeLicensePlate(
            this.reqRegPlate.FilePath, "Vietnam");
        
        if (plate_result != null && plate_result.Plate != "")
        {
            // 2. Tạo response
            NPRRegPlateResp resp = new NPRRegPlateResp
            {
                CarNum = this.reqRegPlate.CarNum,
                LaneNum = this.reqRegPlate.LaneNum,
                TransId = this.reqRegPlate.TransId,
                PlateNum = plate_result.Plate,
                filePath = this.reqRegPlate.FilePath,
                filePath2 = this.reqRegPlate.FilePath2
            };
            
            // 3. Gửi response
            NetObject respSock = new NetObject();
            respSock.MessageID = 67141633; // 0x4004001
            respSock.Object = resp;
            this.netServer.DispatchTo(this.guid, respSock);
            
            // 4. Cắt ảnh biển số
            this.ExtracePlateFromImage(this.reqRegPlate.FilePath, plate_result);
        }
        else
        {
            CTaskRegPlateNum.logger.Error("Cannot Recognize License Plate: " + this.reqRegPlate.FilePath);
        }
    }
    catch (Exception ex)
    {
        CTaskRegPlateNum.logger.Error(ex.Message);
    }
    return 0;
}
```

**Logic cắt ảnh biển số:**
```csharp
private void ExtracePlateFromImage(string filePath, CANPR_Result plateResult)
{
    try
    {
        string newPath = filePath + "_crop.jpg";
        
        using (Bitmap originalImage = new Bitmap(filePath))
        {
            // Tính toán vùng cắt
            int x = Math.Min(plateResult.X1, plateResult.X4);
            int y = Math.Min(plateResult.Y1, plateResult.Y2);
            int x2 = Math.Max(plateResult.X2, plateResult.X3);
            int y2 = Math.Max(plateResult.Y3, plateResult.Y4);
            int width = Math.Abs(x2 - x);
            int height = Math.Abs(y2 - y);
            
            Rectangle crop = new Rectangle(x, y, width, height);
            Bitmap croppedImage = originalImage.Clone(crop, originalImage.PixelFormat);
            
            // Lưu ảnh đã cắt
            croppedImage.Save(newPath, ImageFormat.Jpeg);
            croppedImage.Dispose();
        }
    }
    catch (Exception ex)
    {
        CTaskRegPlateNum.logger.Error(ex.Message);
    }
}
```

### 7. CTaskStartReq.cs - Start Request Task

**Chức năng:**
- Xử lý yêu cầu khởi động từ client
- Trả về thông tin FileServerInfo

**Logic:**
```csharp
private int ProcessStartReq()
{
    try
    {
        // Tạo response với thông tin FileServerInfo
        NPRStartResp nprStartResp = new NPRStartResp
        {
            FileServerInfo = this.ConfigValue.NPRShareFolder
        };
        
        NetObject respStart = new NetObject();
        respStart.MessageID = 67141634; // 0x4004002
        respStart.Object = nprStartResp;
        
        this.netServer.DispatchTo(this.guid, respStart);
    }
    catch (Exception ex)
    {
        CTaskStartReq.logger.Error(ex.Message);
    }
    return 0;
}
```

## Các loại Message

### Socket Messages (TCP)
| Message ID | Hex | Tên | Mô tả |
|------------|-----|-----|-------|
| 262144 | 0x40000 | Health Check | Kiểm tra sức khỏe hệ thống |
| 262145 | 0x40001 | Plate Recognition | Yêu cầu nhận dạng biển số |
| 262146 | 0x40002 | Start Request | Yêu cầu khởi động |

### Response Messages
| Message ID | Hex | Tên | Mô tả |
|------------|-----|-----|-------|
| 327679 | 0x4FFFF | HC Response | Phản hồi health check |
| 67141633 | 0x4004001 | Plate Response | Kết quả nhận dạng biển số |
| 67141634 | 0x4004002 | Start Response | Phản hồi start request |

### Process Messages (Message Queue)
| Message ID | Hex | Tên | Mô tả |
|------------|-----|-----|-------|
| 262145 | 0x40001 | Plate Recognition | Yêu cầu nhận dạng từ process khác |

## Cấu hình hệ thống

### File cấu hình: LaneServer.config
```xml
<LaneConfig>
    <NPRProcName>NPRProc1</NPRProcName>
    <NPRServPort>8080;8081</NPRServPort>
    <CoreProcName>CoreProc</CoreProcName>
    <NPRShareFolder>\\server\share</NPRShareFolder>
    <NPRMaxRetry>3</NPRMaxRetry>
    <NPRDelayTime>100</NPRDelayTime>
</LaneConfig>
```

### Các tham số cấu hình:
- **NPRProcName**: Tên process
- **NPRServPort**: Danh sách port TCP (phân cách bằng dấu chấm phẩy)
- **CoreProcName**: Tên process core để gửi kết quả
- **NPRShareFolder**: Thư mục chia sẻ file
- **NPRMaxRetry**: Số lần thử lại tối đa
- **NPRDelayTime**: Thời gian delay giữa các lần thử (ms)

## Dependencies và thư viện

### Core Dependencies:
- **Topshelf**: Tạo Windows Service
- **MsgConnect**: Giao tiếp giữa các process
- **NetSockets**: Giao tiếp qua TCP socket
- **VETC.FE.Common**: Thư viện chung của hệ thống VETC
- **VETC.FE.RegPlate**: Thư viện nhận dạng biển số

### System Dependencies:
- **System.Drawing**: Xử lý ảnh
- **System.Threading.Tasks**: Xử lý đa luồng
- **System.Collections.Generic**: Cấu trúc dữ liệu

## Luồng hoạt động chi tiết

### 1. Khởi động hệ thống
```
Program.cs → ConfigureService.Configure() → ProcMan.Start()
```

### 2. Đọc cấu hình
```
CConfigFile.LoadFile("LaneServer.config") → Parse config → Create NPRProc instances
```

### 3. Khởi tạo process
```
NPRProc constructor → Initialize message queue → Initialize socket server → Start listening
```

### 4. Xử lý request
```
Client connects → Socket message received → Route to appropriate task → Process → Send response
```

### 5. Nhận dạng biển số
```
Receive image path → ANPR.RecognizeLicensePlate() → Extract plate region → Save cropped image → Send result
```

## Đặc điểm kỹ thuật

### Multi-threading
- Sử dụng Task để xử lý bất đồng bộ
- Mỗi request được xử lý trong thread riêng biệt
- Không block main thread

### Error Handling
- Try-catch bao quanh tất cả các operation
- Logging chi tiết cho debugging
- Graceful degradation khi có lỗi

### Configuration-driven
- Cấu hình qua file config
- Hỗ trợ nhiều lane thu phí
- Dễ dàng thêm/sửa/xóa lane

### Image Processing
- Sử dụng System.Drawing để xử lý ảnh
- Cắt ảnh biển số từ ảnh gốc
- Lưu ảnh đã cắt với định dạng JPEG

### Communication Protocols
- **TCP Socket**: Giao tiếp với client
- **Message Queue**: Giao tiếp giữa các process
- **File System**: Lưu trữ ảnh và kết quả

## Monitoring và Logging

### Log Levels:
- **Debug**: Thông tin chi tiết cho development
- **Info**: Thông tin hoạt động bình thường
- **Error**: Lỗi và exception

### Log Categories:
- Process management
- Socket communication
- Plate recognition
- Image processing
- Error handling

## Deployment

### Requirements:
- Windows Server 2012 trở lên
- .NET Framework 4.5.2
- Quyền LocalSystem để chạy service
- Access đến thư mục chia sẻ file

### Installation:
1. Build project
2. Copy executable và dependencies
3. Install service: `VETC.FE.NPRService.exe install`
4. Start service: `VETC.FE.NPRService.exe start`

### Configuration:
1. Tạo file `LaneServer.config`
2. Cấu hình các tham số
3. Restart service

## Troubleshooting

### Common Issues:
1. **Service không start**: Kiểm tra quyền và dependencies
2. **Socket connection failed**: Kiểm tra port và firewall
3. **Plate recognition failed**: Kiểm tra ANPR library và ảnh input
4. **File access denied**: Kiểm tra quyền truy cập thư mục

### Debug Steps:
1. Kiểm tra Windows Event Log
2. Kiểm tra application log
3. Test socket connection
4. Verify configuration file
5. Check file permissions

## Performance Considerations

### Optimization:
- Sử dụng connection pooling cho socket
- Implement caching cho ANPR results
- Optimize image processing
- Monitor memory usage

### Scalability:
- Hỗ trợ nhiều lane đồng thời
- Load balancing qua multiple ports
- Horizontal scaling với multiple instances

## Security Considerations

### Network Security:
- Validate input data
- Implement authentication nếu cần
- Use secure communication protocols
- Monitor network traffic

### File Security:
- Validate file paths
- Check file permissions
- Implement file access controls
- Secure file storage

## Future Enhancements

### Potential Improvements:
1. **Machine Learning**: Cải thiện độ chính xác nhận dạng
2. **Real-time Processing**: Xử lý video stream
3. **Cloud Integration**: Upload results to cloud
4. **API Interface**: RESTful API cho integration
5. **Dashboard**: Web interface để monitor
6. **Analytics**: Thống kê và báo cáo

### Technical Debt:
1. **Code Refactoring**: Tách logic thành modules nhỏ hơn
2. **Unit Testing**: Thêm test cases
3. **Documentation**: Cải thiện documentation
4. **Error Handling**: Standardize error handling
5. **Configuration**: Move to external config service 