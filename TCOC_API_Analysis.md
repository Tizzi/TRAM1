# Phân tích Logic Hoạt Động - TCOC.API

## Tổng quan
TCOC.API là một thư viện C# được thiết kế để giao tiếp với hệ thống thu phí đường bộ (Toll Collection System) thông qua giao thức TCP/IP với mã hóa AES. Đây là một client API cho phép các ứng dụng tích hợp với hệ thống thu phí.

🔄 Workflow chính:
1. Connect → Kết nối TCP và xác thực
2. Handshake → Duy trì session định kỳ
3. Transaction → Check-in → Commit/Rollback → Charge
   
## Kiến trúc Tổng thể

### 1. Cấu trúc Project
```
TCOC.API/
├── APIClient.cs                    # Client chính để giao tiếp với server
├── ConnectionErrorEventArgs.cs      # Event args cho lỗi kết nối
├── Encrypt/                        # Module mã hóa
│   ├── IEncrypter.cs              # Interface cho mã hóa
│   └── AesEncrypter.cs            # Implementation AES
├── Extensions/                     # Extension methods
│   ├── Helper.cs                   # Helper functions
│   └── StreamExtensions.cs         # Stream extensions
└── Objects/                        # Data models
    ├── Base.cs                     # Base class cho tất cả messages
    ├── Command.cs                  # Enum các loại command
    ├── IBase.cs                    # Interface cho base objects
    ├── ISerializable.cs            # Interface cho serialization
    └── [Các request/response models]
```

## 2. Core Components

### 2.1 APIClient - Trung tâm điều khiển
**File:** `APIClient.cs`

**Chức năng chính:**
- Quản lý kết nối TCP với server
- Xử lý gửi/nhận messages
- Quản lý session và handshake
- Mã hóa/giải mã dữ liệu
- Xử lý timeout và retry

**Các thuộc tính quan trọng:**
```csharp
public int Session { get; private set; }           // Session ID
public IEncrypter Encrypter { get; }               // Mã hóa
private TcpClient _tcpClient;                       // TCP connection
private Stream _stream;                             // Network stream
private readonly SemaphoreSlim _semaphore;         // Thread safety
private readonly Timer _timer;                      // Handshake timer
```

**Các phương thức chính:**
- `ConnectAsync()` - Kết nối đến server
- `ShakeAsync()` - Handshake định kỳ
- `TerminateAsync()` - Đóng kết nối
- `CheckinAsync()` - Gửi thông tin check-in
- `CommitAsync()` - Commit transaction
- `RollbackAsync()` - Rollback transaction
- `ChargeAsync()` - Gửi thông tin thu phí

### 2.2 Hệ thống Mã hóa
**File:** `Encrypt/AesEncrypter.cs`

**Chức năng:**
- Mã hóa/giải mã dữ liệu bằng AES-256-CBC
- Sử dụng PKCS7 padding
- Block size: 128 bits
- Key size: 256 bits

**Implementation:**
```csharp
public class AesEncrypter : IEncrypter
{
    private readonly byte[] _password;  // Encryption key
    private readonly byte[] _iv;        // Initialization vector
    
    public byte[] Encrypt(byte[] data)  // Mã hóa
    public byte[] Decrypt(byte[] data)  // Giải mã
}
```

### 2.3 Hệ thống Serialization
**File:** `Objects/Base.cs`

**Chức năng:**
- Serialize/deserialize messages thành byte arrays
- Định dạng message: `[Length][Command][RequestId][SessionId][Body]`

**Cấu trúc Message:**
```csharp
public class Base<T> : IBase, ISerializable
{
    public int Length { get; set; }        // 4 bytes
    public Command Command { get; set; }    // 4 bytes  
    public int RequestId { get; set; }      // 4 bytes
    public int SessionId { get; set; }      // 4 bytes
    public T Body { get; set; }             // Variable length
}
```

## 3. Giao thức Giao tiếp

### 3.1 Các Command Types
**File:** `Objects/Command.cs`

```csharp
public enum Command
{
    ConnectRequest,      // Kết nối
    ConnectResponse,     // Phản hồi kết nối
    ShakeRequest,        // Handshake
    ShakeResponse,       // Phản hồi handshake
    CheckinRequest,      // Check-in
    CheckinResponse,     // Phản hồi check-in
    CommitRequest,       // Commit transaction
    CommitResponse,      // Phản hồi commit
    RollbackRequest,     // Rollback transaction
    RollbackResponse,    // Phản hồi rollback
    TerminateRequest,    // Kết thúc session
    TerminateResponse,   // Phản hồi kết thúc
    ChargeRequest,       // Thu phí
    ChargeResponse       // Phản hồi thu phí
}
```

### 3.2 Workflow Giao tiếp

#### 3.2.1 Kết nối (Connection)
1. **ConnectAsync()** - Tạo TCP connection
2. **Send ConnectRequest** - Gửi thông tin đăng nhập
3. **Receive ConnectResponse** - Nhận session ID
4. **Start Handshake Timer** - Bắt đầu timer handshake

#### 3.2.2 Handshake (Định kỳ)
1. **Timer Elapsed** - Timer trigger
2. **Send ShakeRequest** - Gửi handshake
3. **Receive ShakeResponse** - Nhận phản hồi
4. **Update Session** - Cập nhật session

#### 3.2.3 Transaction Flow
1. **Checkin** - Gửi thông tin xe vào
2. **Commit/Rollback** - Xác nhận/hủy transaction
3. **Charge** - Thu phí (nếu cần)

## 4. Data Models

### 4.1 Request Models
- **ConnectRequestBody** - Thông tin đăng nhập
- **ChargeRequestBody** - Thông tin thu phí
- **TransactionRequestBody** - Thông tin transaction
- **RollbackRequestBody** - Thông tin rollback

### 4.2 Response Models  
- **ConnectResponseBody** - Phản hồi kết nối
- **SessionResponseBody** - Phản hồi session
- **CheckinResponseBody** - Phản hồi check-in
- **TransactionResponseBody** - Phản hồi transaction

### 4.3 Ví dụ ConnectRequestBody
```csharp
public class ConnectRequestBody : ISerializable
{
    public string Username { get; set; }    // 10 bytes
    public string Password { get; set; }    // 10 bytes
    public int Station { get; set; }        // 4 bytes
    public int Timeout { get; set; }        // 4 bytes
}
```

## 5. Thread Safety và Async Operations

### 5.1 SemaphoreSlim
```csharp
private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1);
```
- Đảm bảo chỉ một request được xử lý tại một thời điểm
- Tránh race condition khi gửi/nhận messages

### 5.2 Async/Await Pattern
- Tất cả network operations đều async
- Sử dụng `ConfigureAwait(false)` để tối ưu performance
- Proper exception handling cho network errors

## 6. Error Handling

### 6.1 Connection Errors
```csharp
public EventHandler<ConnectionErrorEventArgs> ConnectionError;
```
- Xử lý lỗi kết nối TCP
- Timeout handling
- Automatic retry logic

### 6.2 Exception Handling
- Network exceptions
- Serialization exceptions  
- Encryption/decryption exceptions

## 7. Configuration và Dependencies

### 7.1 Dependencies
- **Newtonsoft.Json** - JSON serialization
- **VETC.FE.Common** - Common utilities
- **System.Net.Sockets** - TCP networking
- **System.Security.Cryptography** - AES encryption

### 7.2 Configuration Parameters
```csharp
public APIClient(string clientIp, int clientPort, int handShakeInterval, 
                 int BEtimeout, string encryptKey, byte[] iv)
```
- `clientIp/clientPort` - Client network info
- `handShakeInterval` - Handshake frequency (ms)
- `BEtimeout` - Network timeout (ms)
- `encryptKey/iv` - Encryption parameters

## 8. Use Cases

### 8.1 Khởi tạo Client
```csharp
var client = new APIClient("192.168.1.100", 8080, 30000, 5000, 
                           "encryption_key", iv_bytes);
```

### 8.2 Kết nối và Check-in
```csharp
// Connect
var connectBody = new ConnectRequestBody("user", "pass", 1);
var response = await client.ConnectAsync("server_ip", 8080, connectBody);

// Check-in
var chargeBody = new ChargeRequestBody("etag123", 1, 1, "tid123");
var checkinResponse = await client.CheckinAsync(transactionId, chargeBody);
```

### 8.3 Commit Transaction
```csharp
var transactionBody = new TransactionRequestBody(amount, plate);
var commitResponse = await client.CommitAsync(transactionId, transactionBody);
```

## 9. Đặc điểm Kỹ thuật

### 9.1 Performance
- Async/await pattern cho non-blocking operations
- SemaphoreSlim cho thread safety
- Efficient binary serialization
- Configurable timeouts

### 9.2 Security
- AES-256-CBC encryption
- Configurable encryption keys
- Secure TCP communication
- Session management

### 9.3 Reliability
- Automatic handshake để duy trì connection
- Proper error handling và logging
- Timeout management
- Connection state monitoring

## 10. Kết luận

TCOC.API là một thư viện client API được thiết kế tốt cho hệ thống thu phí đường bộ với các đặc điểm:

**Ưu điểm:**
- Kiến trúc modular và extensible
- Thread-safe với async operations
- Mã hóa bảo mật AES-256
- Error handling comprehensive
- Session management tự động

**Ứng dụng:**
- Hệ thống thu phí đường bộ
- Giao tiếp với backend servers
- Transaction processing
- Real-time communication

**Công nghệ sử dụng:**
- C# .NET Framework 4.5.2
- TCP/IP networking
- AES encryption
- Async programming
- Binary serialization 
