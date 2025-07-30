# Phân tích kiến trúc và logic hoạt động của VETC.FE.SyncService

## 1. Tổng quan kiến trúc

- Ứng dụng là một **Windows Service** sử dụng [Topshelf](https://topshelf-project.com/) để quản lý vòng đời dịch vụ, kết hợp với [Quartz.NET](https://www.quartz-scheduler.net/) để lập lịch các tác vụ định kỳ.
- Chức năng chính:
  - Lấy file từ FTP server về máy nội bộ hoặc upload file từ máy lên FTP.
  - Theo dõi các thư mục nội bộ, khi có file mới sẽ tự động xử lý (import/export dữ liệu, cập nhật DB, ...).
  - Ghi log hoạt động và lỗi bằng NLog.

## 2. Luồng khởi động

- **Program.cs**: Điểm khởi động, gọi `ConfigureService.Configure()`.
- **ConfigureService.cs**: Đăng ký service với Topshelf, chỉ định class `SyncService` làm service chính, ánh xạ các sự kiện Start/Stop/Shutdown.
- **AppSettings.cs**: Đọc các cấu hình từ file cấu hình (app.config), bao gồm thông tin FTP, các tham số job, đường dẫn thư mục, ...

## 3. Logic chính của SyncService

### 3.1. Hàm Start()

- Khởi tạo Quartz Scheduler.
- Đối với mỗi cấu hình trong `AppSettings.GetFileParam`:
  - Nếu `jobInterval > 0`: Tạo job `GetFileJob` (lấy file từ FTP hoặc upload lên FTP) và lập lịch chạy định kỳ.
  - Nếu `jobInterval = 0`: Tạo và chạy ngay job `UploadFileJob` (upload file lên FTP).
- Đối với mỗi cấu hình trong `AppSettings.LoadFileParam`:
  - Gọi `StartLoadFileJob(jobName, jobFilePath)`, khởi tạo các job xử lý file nội bộ (import/export dữ liệu, ...).

### 3.2. Các loại Job chính

- **GetFileJob**: Kết nối FTP, đồng bộ file giữa FTP và thư mục local (download/upload), ghi log kết quả.
- **UploadFileJob**: Theo dõi thư mục local, khi có file mới sẽ tự động upload lên FTP.
- **LoadFileXXX**: Các job xử lý file nội bộ (import vé tháng, vé thu phí, nhân sự, biển số xe, ...). Mỗi job sẽ:
  - Theo dõi thư mục, khi có file mới sẽ kiểm tra và thực hiện import/export dữ liệu vào DB hoặc xử lý nghiệp vụ tương ứng.
  - Ghi log hoạt động và lỗi.

#### Một số job tiêu biểu kế thừa `LoadFileAbstract`:

| Tên class                  | Chức năng chính                                                                 |
|----------------------------|-------------------------------------------------------------------------------|
| **LoadFileCommuterTicket** | Theo dõi thư mục vé tháng, tự động import file CSV vào DB khi có file mới.    |
| **LoadFileTollTicket**     | Theo dõi thư mục vé thu phí, tự động import file CSV vào DB khi có file mới.  |
| **LoadFileEmployee**       | Theo dõi thư mục nhân sự, tự động import/update/delete thông tin nhân sự.     |
| **LoadFileVehiclePlate**   | Theo dõi thư mục biển số xe, tự động import/update/delete thông tin phương tiện.|
| **LoadFileImageHinhChuan** | Theo dõi thư mục ảnh chuẩn, tự động copy ảnh mới vào vị trí chuẩn hóa.        |
| **LoadFileNull**           | Job rỗng, dùng cho cấu hình không hợp lệ hoặc không cần xử lý.                |

**Mô tả hoạt động chung của các job LoadFileXXX:**
- Sử dụng `FileSystemWatcher` để theo dõi thư mục.
- Khi có file mới, kiểm tra hợp lệ, sau đó thực hiện import/update/delete vào DB tùy theo loại file và nghiệp vụ.
- Ghi log chi tiết quá trình xử lý.

### 3.3. Factory và Abstract

- **LoadFileFactory**: Tạo instance các job xử lý file dựa trên tên class và đường dẫn file.
- **LoadFileAbstract**: Lớp trừu tượng cho các job xử lý file, định nghĩa thuộc tính `filePath` và phương thức `Run()`.

## 4. Ghi log

- Sử dụng NLog, các hàm tiện ích trong `Utilities.cs` để ghi log hoạt động (`WriteOperationLog`), log lỗi (`WriteErrorLog`), log debug (`WriteDebugLog`).

## 5. Cấu hình

- Các tham số cấu hình được đọc từ file app.config thông qua `AppSettings.cs`, bao gồm:
  - Thông tin FTP: host, port, username, password
  - Danh sách các job lấy file, upload file, xử lý file nội bộ
  - Đường dẫn các thư mục dữ liệu, ảnh, ...

## 6. Mở rộng

- Để thêm job mới, chỉ cần tạo class kế thừa `LoadFileAbstract` và khai báo trong cấu hình.

---

*File này được tạo tự động dựa trên phân tích mã nguồn. Để chi tiết hơn về từng job, xem các file trong thư mục `SyncService/`.*