# Phân tích import dữ liệu nhân sự (LoadFileEmployee)
Phân tích cách thức LoadFileEmployee đọc và import dữ liệu
1. Quy trình tổng thể
Theo dõi thư mục: Sử dụng FileSystemWatcher để theo dõi thư mục chứa file nhân sự. Khi có file mới, gọi ImportCsv(newPath).
Đọc file CSV:
Hàm ImportCsv gọi LoadFileToCSVObject_Retry, thực chất là retry nhiều lần việc đọc file (tránh lỗi file đang bị ghi dở).
Hàm LoadFileToCSVObject sử dụng thư viện FileHelperEngine<CsvEmployee> để đọc file CSV thành mảng các đối tượng CsvEmployee.
2. Cấu trúc dữ liệu import
Class trung gian:
CsvEmployee là class đại diện cho từng dòng dữ liệu trong file CSV (không thấy định nghĩa trong đoạn này, nhưng thường sẽ có các trường như: FirstName, LastName, Phone, Address, PartnerID, TollId...).
Chuyển đổi sang DB object:
Hàm ConvertToDBObject(CsvEmployee input, out USER user, out STAFF staff) chuyển từng dòng CsvEmployee thành hai đối tượng:
USER: Chứa thông tin tài khoản đăng nhập (USER_ID, USER_NAME, PASSWORD, STATUS, TYPE, ...)
STAFF: Chứa thông tin nhân sự (STAFF_ID, ORG_ID, STAFF_TYPE, USER_ID, LOGIN_TOKEN, STATUS, MOBILE, ADDRESS, STAFF_FULL_NAME, PARTNER_STAFF_ID, ...)
Một số trường được chuyển đổi/chuẩn hóa qua các hàm tiện ích như Func.ConvertVniToUnicode.
3. Luồng xử lý import
Đọc file CSV thành mảng CsvEmployee[].
Với mỗi record:
Chuyển thành USER và STAFF.
Tùy vào tên file (_INSERT_, _UPDATE_, _DELETE_), gọi các hàm InsertDB, UpdateDB, DeleteDB để thao tác với DB.
Các hàm Insert/Update/Delete sẽ thao tác với bảng USERS, STAFFs, USER_ROLE trong DB.
4. Đặc điểm kỹ thuật
Đọc file:
Sử dụng UTF-8, đọc toàn bộ file một lần thành mảng đối tượng.
Có retry khi đọc file (tránh lỗi file lock).
Chuyển đổi dữ liệu:
Tách biệt rõ giữa dữ liệu đọc từ file (CsvEmployee) và dữ liệu ghi vào DB (USER, STAFF).
Có kiểm tra, chuẩn hóa dữ liệu đầu vào.
Ghi log:
Ghi log chi tiết các bước đọc file, chuyển đổi, lỗi phát sinh.
Tóm tắt cấu trúc data import:
File CSV → Mảng CsvEmployee (mỗi dòng là một nhân sự)
Mỗi CsvEmployee → 1 USER + 1 STAFF
Ghi vào DB: USERS, STAFFs, USER_ROLE

## 1. Quy trình import nhân sự

- Ứng dụng sử dụng `FileSystemWatcher` để theo dõi thư mục chứa file nhân sự.
- Khi có file CSV mới, hệ thống sẽ:
  1. Đọc toàn bộ file thành mảng các đối tượng `CsvEmployee`.
  2. Với mỗi dòng, chuyển thành hai đối tượng: `USER` (tài khoản) và `STAFF` (nhân sự).
  3. Tùy vào tên file (`_INSERT_`, `_UPDATE_`, `_DELETE_`), thực hiện thêm mới, cập nhật hoặc xóa dữ liệu trong DB.
  4. Ghi nhận file đã xử lý vào bảng `SYNCMTC_FILE` để tránh xử lý lại.

## 2. Định nghĩa class CsvEmployee

```csharp
[DelimitedRecord(",")]
public class CsvEmployee
{
    [FieldConverter(ConverterKind.Decimal)]
    public decimal PartnerID;
    [FieldConverter(ConverterKind.Decimal)]
    public decimal GroupID;
    public string FirstName;
    public string LastName;
    public string LastName_EN;
    public string Address;
    public string Phone;
    public string Note;
    public string TollId;
}
```

**Ý nghĩa các trường:**
- `PartnerID`: Mã nhân sự đối tác (dạng số)
- `GroupID`: Mã nhóm (dạng số)
- `FirstName`: Họ nhân sự
- `LastName`: Tên nhân sự
- `LastName_EN`: Tên tiếng Anh (nếu có)
- `Address`: Địa chỉ
- `Phone`: Số điện thoại
- `Note`: Ghi chú
- `TollId`: Mã trạm thu phí

## 3. Ví dụ file CSV mẫu

```
PartnerID,GroupID,FirstName,LastName,LastName_EN,Address,Phone,Note,TollId
1001,1,Nguyen,Van A,Van A,123 Đường ABC,0912345678,Ghi chú 1,T01
1002,1,Tran,Thi B,Thi B,456 Đường DEF,0987654321,Ghi chú 2,T02
1003,2,Le,Van C,Van C,789 Đường XYZ,0909123456,,T03
```

**Lưu ý:**
- Dòng đầu tiên là header, các dòng sau là dữ liệu.
- Các trường có thể để trống nếu không bắt buộc.
- Dữ liệu số phải đúng định dạng (không có ký tự lạ).

## 4. Tổng kết
- File CSV sẽ được đọc thành mảng `CsvEmployee`, mỗi dòng ánh xạ vào một nhân sự.
- Hệ thống sẽ tự động chuyển đổi, chuẩn hóa và ghi vào các bảng USERS, STAFFs, USER_ROLE trong DB.
- Ghi log chi tiết quá trình xử lý và lỗi (nếu có).

---
*File này tổng hợp chi tiết quy trình import nhân sự, cấu trúc dữ liệu và ví dụ thực tế cho việc tích hợp hoặc kiểm thử.*

File CSV mẫu cho import nhân sự không chứa trực tiếp các trường như USER_ID, USER_NAME, PASSWORD, STATUS, TYPE...
Các trường này sẽ được sinh ra hoặc gán mặc định trong quá trình chuyển đổi từ CsvEmployee sang đối tượng USER trong code.
Cụ thể:
USER_ID: Sinh tự động (sequence trong DB).
USER_NAME: Được tạo từ các trường của CsvEmployee (thường là từ FirstName, LastName, hoặc PartnerID, tuỳ logic trong hàm ConvertToDBObject_USERNAME).
PASSWORD: Gán mặc định (ví dụ: mã hash cố định).
STATUS, TYPE: Gán mặc định trong code (thường là "1" cho active, "1" cho loại user bình thường).

Ví dụ mapping từ file CSV sang USER:
Trường trong CSV	Trường USER (DB)	Ghi chú
PartnerID	USER_NAME	Có thể dùng PartnerID làm USER_NAME
FirstName, LastName	USER_NAME	Có thể ghép thành USER_NAME
(Không có trong CSV)	PASSWORD	Gán mặc định trong code
(Không có trong CSV)	STATUS	Gán mặc định trong code
(Không có trong CSV)	TYPE	Gán mặc định trong code
Ví dụ file CSV mẫu (chuẩn, chỉ chứa các trường của CsvEmployee):
Sau khi import, hệ thống sẽ tự động sinh ra các trường tài khoản như sau (ví dụ):

USER_ID	USER_NAME	PASSWORD	STATUS	TYPE
1	1001	(hash)	1	1
2	1002	(hash)	1	1
3	1003	(hash)	1	1
