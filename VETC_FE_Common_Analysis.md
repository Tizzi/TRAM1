# Phân tích ứng dụng VETC.FE.Common

## Tổng quan

**VETC.FE.Common** là một thư viện C# (.NET Framework 4.5.2) được phát triển bởi VETC (Vietnam Electronic Toll Collection) cho hệ thống thu phí điện tử. Đây là một phần của hệ thống Front-End (FE) trong kiến trúc thu phí đường bộ.

### Thông tin cơ bản
- **Phiên bản**: 1.0.7466.14703
- **Công ty**: VETC
- **Năm phát triển**: 2016
- **Framework**: .NET Framework 4.5.2
- **Loại**: Class Library

## Kiến trúc hệ thống

### 1. Các Process chính (ProcIDConst)
Hệ thống được chia thành 8 process chính:

```csharp
public enum ProcIDConst
{
    Core,       // Process lõi
    GUI,        // Giao diện người dùng
    Sensor,     // Cảm biến
    BE,         // Backend
    NPR,        // Number Plate Recognition
    PLC,        // Programmable Logic Controller
    MTC         // Manual Toll Collection
}
```

### 2. Loại làn xe (LaneTypeConst)
```csharp
public enum LaneTypeConst
{
    MTC = 1,    // Manual Toll Collection
    ETC,        // Electronic Toll Collection
    MIX         // Hỗn hợp
}
```

### 3. Chế độ hoạt động (OperModeConst)
```csharp
public enum OperModeConst
{
    RUN = 1,            // Chế độ chạy bình thường
    TEST_PLC,           // Test PLC
    TEST_OFFLINE,       // Test offline
    TEST_ETAGLIST       // Test danh sách E-Tag
}
```

## Các thành phần chính

### 1. Hệ thống Message Passing

#### CProcMsg - Process Message Handler
- Quản lý giao tiếp giữa các process
- Sử dụng thư viện MsgConnect
- Hỗ trợ gửi/nhận message giữa các process

#### CHttpMsg - HTTP Message Handler
- Xử lý giao tiếp HTTP
- Hỗ trợ kết nối/ngắt kết nối
- Gửi message qua HTTP protocol

### 2. Hệ thống Queue Management (FEQueue)

#### Tính năng chính:
- **Quản lý hàng đợi xe**: Lưu trữ và quản lý danh sách xe trong hàng đợi
- **Tìm kiếm xe**: Theo số xe, biển số, E-Tag
- **Cập nhật trạng thái**: Cập nhật thông tin xe trong hàng đợi
- **Xử lý ảnh**: Quản lý flag ảnh và thư mục ảnh
- **Thread-safe**: Sử dụng lock để đảm bảo thread safety

#### Các phương thức chính:
```csharp
- Add(CarData car)           // Thêm xe vào hàng đợi
- Remove(CarData car)        // Xóa xe khỏi hàng đợi
- GetTop()                   // Lấy xe đầu hàng đợi
- FindCarByPlate(string)     // Tìm xe theo biển số
- FindEtag(string)           // Tìm xe theo E-Tag
- ClearPicFlag()             // Xóa flag ảnh
- UpdateCarData(CarData)     // Cập nhật thông tin xe
```

### 3. Dữ liệu xe (CarData)

#### Thông tin cơ bản:
- **CarNum**: Số thứ tự xe
- **TransID**: ID giao dịch
- **ETag**: Thẻ điện tử
- **PlateNumReg**: Biển số đăng ký
- **CarType**: Loại xe
- **CheckInResult**: Kết quả check-in
- **CommitResult**: Kết quả commit
- **RollbackResult**: Kết quả rollback

#### Thông tin giao dịch:
- **TicketID**: ID vé
- **TicketType**: Loại vé
- **Price**: Giá tiền
- **VehicleType**: Loại phương tiện
- **PlateType**: Loại biển số
- **ETCDiscount**: Giảm giá ETC

#### Thông tin cảm biến:
- **SensorHis[]**: Lịch sử cảm biến
- **IsLoopDetect**: Phát hiện vòng lặp
- **IsSickDetect**: Phát hiện bệnh
- **IsTagArrive**: Tag đến
- **IsTagDepart**: Tag rời

#### Thông tin ảnh:
- **PicNum**: Số lượng ảnh
- **PicList[]**: Danh sách đường dẫn ảnh
- **PicSubFolder**: Thư mục con chứa ảnh
- **CamPlateTrigger**: Trigger camera biển số

#### Serialization:
- Hỗ trợ serialize/deserialize để lưu trữ và truyền tải dữ liệu
- Sử dụng byte array để tối ưu hiệu suất

### 4. Hệ thống Logging (CLogManager)

#### Tính năng:
- Sử dụng thư viện NLog
- Hỗ trợ logging theo thread
- Các level: Info, Error, Debug
- Queue-based logging để tránh blocking

#### Các phương thức:
```csharp
- Info(string)      // Log thông tin
- Error(string)     // Log lỗi
- Debug(string)     // Log debug
```

### 5. Utilities (FEUtils)

#### Các tiện ích chính:
- **Time utilities**: Tính toán thời gian, retry datetime
- **File operations**: Upload ảnh, copy file
- **String utilities**: Xử lý chuỗi, format
- **HTTP utilities**: Gửi HTTP request
- **Byte utilities**: Chuyển đổi byte array
- **Sensor utilities**: Xử lý tín hiệu cảm biến

## Hệ thống Message ID

### Cấu trúc Message ID:
Hệ thống sử dụng các Message ID được định nghĩa trong `MessageIDConst`:

#### Sensor Messages (131073+):
- SENSORBARREQ: Yêu cầu bar
- SENSORLEDREQ: Yêu cầu LED
- SENSORBUZZREQ: Yêu cầu buzzer
- SENSORLAMPREQ: Yêu cầu đèn
- SENSORNOTETAG: Thông báo E-Tag
- SENSORNOTSIG: Thông báo tín hiệu
- SENSORNOTBARCODE: Thông báo barcode

#### Backend Messages (196609+):
- BECHECKINREQ: Yêu cầu check-in
- BECOMMITREQ: Yêu cầu commit
- BERollbackREQ: Yêu cầu rollback
- BEHCREQ: Yêu cầu heartbeat

#### GUI Messages (16777217+):
- LANEAPPSTARTREQ: Yêu cầu khởi động
- LANEAPPCOMPLETEREQ: Yêu cầu hoàn thành
- LANEAPPCAMRESP: Phản hồi camera
- LANEAPPCHECKOUTMTC: Check-out MTC

#### MTC Messages (17170432+):
- MTCHCREQ: Heartbeat MTC
- MTCLOGINBARREQ: Đăng nhập bar
- MTCLOGINPASSREQ: Đăng nhập pass
- MTCCHECKOUTREQ: Check-out

## Luồng hoạt động chính

### 1. Quy trình xử lý xe:

1. **Xe vào làn** → Sensor phát hiện → Tạo CarData
2. **Check-in** → Gửi BEProcRequest → Nhận BEProcResp
3. **Chụp ảnh** → Camera trigger → Lưu ảnh
4. **Nhận diện biển số** → NPR xử lý → Cập nhật CarData
5. **Thu phí** → MTC/ETC xử lý → Commit/Rollback
6. **Xe ra** → Xóa khỏi queue → Hoàn tất

### 2. Giao tiếp giữa các Process:

```
GUI ←→ Core ←→ Sensor
  ↓      ↓       ↓
  BE ←→ NPR ←→ PLC
  ↓
 MTC
```

### 3. Xử lý lỗi và Recovery:

- **Heartbeat**: Kiểm tra kết nối giữa các process
- **Retry mechanism**: Tự động thử lại khi gặp lỗi
- **Offline mode**: Hoạt động khi mất kết nối
- **Logging**: Ghi log chi tiết để debug

## Cấu hình và Deployment

### Dependencies:
- **MsgConnect.dll**: Thư viện giao tiếp message
- **NLog.dll**: Thư viện logging
- **System.Core**: .NET Framework core
- **System.Xml.Linq**: XML processing

### Cấu hình:
- **Target Framework**: .NET Framework 4.5.2
- **Platform**: AnyCPU
- **Output Type**: Library
- **File Alignment**: 512 bytes

## Đặc điểm kỹ thuật

### 1. Thread Safety:
- Sử dụng lock objects trong FEQueue
- Thread-safe logging với Task queue
- Synchronized access cho shared resources

### 2. Performance:
- Byte array serialization cho hiệu suất cao
- Queue-based message processing
- Async/await pattern cho I/O operations

### 3. Scalability:
- Modular design với các process riêng biệt
- Message-based communication
- Configurable components

### 4. Reliability:
- Comprehensive error handling
- Retry mechanisms
- Offline mode support
- Detailed logging

## Kết luận

VETC.FE.Common là một thư viện được thiết kế tốt cho hệ thống thu phí điện tử với:

- **Kiến trúc modular**: Chia thành các process riêng biệt
- **Giao tiếp linh hoạt**: Hỗ trợ cả process message và HTTP
- **Quản lý dữ liệu hiệu quả**: Queue-based car management
- **Logging chi tiết**: Hỗ trợ debug và monitoring
- **Thread safety**: Đảm bảo hoạt động ổn định trong môi trường đa luồng

Thư viện này đóng vai trò quan trọng trong việc xử lý giao thông và thu phí tự động tại các trạm thu phí Việt Nam. 