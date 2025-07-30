# Phân tích chi tiết các Job và các hàm nghiệp vụ chính

## 1. Phân tích các Job

### 1.1. LoadFileCommuterTicket
- Theo dõi thư mục vé tháng, tự động import file CSV vào DB khi có file mới.
- Sử dụng FileSystemWatcher để phát hiện file mới.
- Phân loại thao tác dựa vào tên file:
  - `_INSERT_`: Thêm mới vé tháng (InsertDB)
  - `_UPDATE_`: Cập nhật vé tháng (UpdateDB)
  - `_DELETE_`/`_DEL_`: Xóa vé tháng (DeleteDB)
- Ghi nhận file đã xử lý vào bảng SYNCMTC_FILE.

### 1.2. LoadFileTollTicket
- Theo dõi thư mục vé thu phí, tự động import file CSV vào DB khi có file mới.
- Sử dụng FileSystemWatcher để phát hiện file mới.
- Phân loại thao tác dựa vào tên file:
  - `_INSERT_`: Import vé thu phí (InsertDB_ImportTicket)
  - `_DEL_`/`_DELETE_`: Đánh dấu xóa vé (ActiveDeleteDB)
- Ghi nhận file đã xử lý vào bảng SYNCMTC_FILE.

### 1.3. LoadFileEmployee
- Theo dõi thư mục nhân sự, tự động import/update/delete thông tin nhân sự.
- Đọc từng dòng file CSV, chuyển thành USER và STAFF.
- Phân loại thao tác dựa vào tên file:
  - `_INSERT_`: Thêm mới nhân sự (InsertDB)
  - `_UPDATE_`: Cập nhật nhân sự (UpdateDB)
  - `_DELETE_`/`_DEL_`: Xóa nhân sự (DeleteDB)
- Ghi nhận file đã xử lý vào bảng SYNCMTC_FILE.

### 1.4. LoadFileVehiclePlate
- Theo dõi thư mục biển số xe, tự động import/update/delete thông tin phương tiện.
- Đọc file CSV thành danh sách phương tiện.
- Phân loại thao tác dựa vào tên file:
  - `_INSERT_`: Thêm mới phương tiện (InsertDB_Delay)
  - `_UPDATE_`: Cập nhật phương tiện (UpdateDB_Delay)
  - `_DELETE_`/`_DEL_`: Xóa phương tiện (DeleteDB)
- Ghi nhận file đã xử lý vào bảng SYNCMTC_FILE.

### 1.5. LoadFileImageHinhChuan
- Theo dõi thư mục ảnh chuẩn, tự động copy ảnh mới vào vị trí chuẩn hóa.
- Khi có file mới, copy sang thư mục đích dựa trên cấu hình.

### 1.6. LoadFileNull
- Job rỗng, dùng cho cấu hình không hợp lệ hoặc không cần xử lý.

---

## 2. Phân tích các hàm nghiệp vụ chính

### 2.1. InsertDB_ImportTicket (LoadFileTollTicket)
- Nhận mảng CsvTollTicket, chuyển đổi thành các đối tượng TICKET, AVAILABLE_TICKET, TICKET_TRANSACTION_DETAIL, TICKET_TRANSACTION.
- Sinh sequence cho từng loại đối tượng.
- Lặp qua từng record, tạo các bản ghi tương ứng và thêm vào danh sách.
- Gọi các hàm InsertDB_TICKET, InsertDB_AVTICKET để lưu batch vào DB.
- Ghi log chi tiết quá trình xử lý.

### 2.2. ActiveDeleteDB (LoadFileTollTicket)
- Nhận mảng CsvTollTicket, tìm các vé tương ứng trong DB.
- Đánh dấu trạng thái vé là đã xóa (hoặc trạng thái khác tùy nghiệp vụ).
- Ghi log quá trình xử lý.

### 2.3. DeleteDB (LoadFileCommuterTicket, LoadFileEmployee, LoadFileVehiclePlate)
- Nhận mảng dữ liệu (vé tháng, nhân sự, phương tiện).
- Tìm các bản ghi tương ứng trong DB.
- Đánh dấu trạng thái là đã xóa hoặc cập nhật trạng thái phù hợp.
- Ghi log quá trình xử lý.

### 2.4. InsertDB/UpdateDB (LoadFileCommuterTicket, LoadFileEmployee, LoadFileVehiclePlate)
- InsertDB: Thêm mới dữ liệu vào các bảng liên quan (TICKET, TICKET_MONTHLY, VEHICLE, USER, STAFF...)
- UpdateDB: Tìm bản ghi theo khóa chính hoặc trường định danh, cập nhật các trường thông tin mới.
- Sử dụng batch insert/update để tối ưu hiệu năng.
- Ghi log chi tiết quá trình xử lý.

---

*File này tổng hợp phân tích chi tiết các job và các hàm nghiệp vụ chính dựa trên mã nguồn thực tế. Để hiểu sâu hơn từng hàm, xem chi tiết trong các file code tương ứng.*