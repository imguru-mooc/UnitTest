# C# 완벽 가이드: 다른 언어 경험자를 위한 빠른 입문

## Part 4: 실전 활용

---

## 1. 프로젝트 구조와 네임스페이스

### 1.1 .NET 프로젝트 구조

```
MyApplication/
├── MyApplication.sln              # 솔루션 파일
├── src/
│   ├── MyApplication.Core/        # 핵심 비즈니스 로직
│   │   ├── MyApplication.Core.csproj
│   │   ├── Models/
│   │   │   └── User.cs
│   │   ├── Services/
│   │   │   └── UserService.cs
│   │   └── Interfaces/
│   │       └── IUserRepository.cs
│   ├── MyApplication.Infrastructure/  # 외부 연동 (DB, API 등)
│   │   ├── MyApplication.Infrastructure.csproj
│   │   └── Repositories/
│   │       └── UserRepository.cs
│   └── MyApplication.Api/         # API 계층
│       ├── MyApplication.Api.csproj
│       ├── Controllers/
│       │   └── UserController.cs
│       └── Program.cs
└── tests/
    ├── MyApplication.Core.Tests/
    │   └── Services/
    │       └── UserServiceTests.cs
    └── MyApplication.Api.Tests/
```

### 1.2 네임스페이스

```csharp
// 전통적인 네임스페이스
namespace MyApplication.Core.Models
{
    public class User
    {
        public int Id { get; set; }
        public string Name { get; set; }
    }
}

// 파일 범위 네임스페이스 (C# 10+) - 권장 ⭐
namespace MyApplication.Core.Models;

public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
}

// using 지시문
using System;
using System.Collections.Generic;
using System.Linq;

// 전역 using (C# 10+) - 프로젝트 전체에 적용
// GlobalUsings.cs 파일에 작성
global using System;
global using System.Collections.Generic;
global using System.Linq;

// using 별칭
using Dict = System.Collections.Generic.Dictionary<string, int>;
using static System.Console;  // 정적 멤버 직접 사용
using static System.Math;

// 사용
Dict myDict = new Dict();
WriteLine("Hello");  // Console.WriteLine 대신
double result = Sqrt(16);  // Math.Sqrt 대신
```

### 1.3 프로젝트 파일 (.csproj)

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <RootNamespace>MyApplication.Core</RootNamespace>
    <AssemblyName>MyApplication.Core</AssemblyName>
    <Version>1.0.0</Version>
    <Authors>Your Name</Authors>
  </PropertyGroup>

  <!-- NuGet 패키지 참조 -->
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
    <PackageReference Include="Microsoft.Extensions.Logging" Version="8.0.0" />
  </ItemGroup>

  <!-- 프로젝트 참조 -->
  <ItemGroup>
    <ProjectReference Include="..\MyApplication.Core\MyApplication.Core.csproj" />
  </ItemGroup>

</Project>
```

---

## 2. NuGet 패키지 관리

### 2.1 CLI 명령어

```bash
# 패키지 추가
dotnet add package Newtonsoft.Json
dotnet add package Newtonsoft.Json --version 13.0.3

# 패키지 제거
dotnet remove package Newtonsoft.Json

# 패키지 목록 확인
dotnet list package

# 오래된 패키지 확인
dotnet list package --outdated

# 패키지 복원
dotnet restore
```

### 2.2 유용한 패키지들

```xml
<!-- JSON 처리 -->
<PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
<PackageReference Include="System.Text.Json" Version="8.0.0" />

<!-- HTTP 클라이언트 -->
<PackageReference Include="RestSharp" Version="110.2.0" />
<PackageReference Include="Refit" Version="7.0.0" />

<!-- 로깅 -->
<PackageReference Include="Serilog" Version="3.1.1" />
<PackageReference Include="NLog" Version="5.2.8" />

<!-- 테스트 -->
<PackageReference Include="NUnit" Version="3.14.0" />
<PackageReference Include="Moq" Version="4.20.70" />
<PackageReference Include="FluentAssertions" Version="6.12.0" />

<!-- ORM -->
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
<PackageReference Include="Dapper" Version="2.1.24" />

<!-- 유효성 검사 -->
<PackageReference Include="FluentValidation" Version="11.9.0" />

<!-- AutoMapper -->
<PackageReference Include="AutoMapper" Version="12.0.1" />
```

---

## 3. 파일 I/O

### 3.1 파일 읽기/쓰기

```csharp
// ===== 간단한 파일 작업 =====

// 전체 텍스트 읽기/쓰기
string content = File.ReadAllText("file.txt");
File.WriteAllText("output.txt", "Hello, World!");

// 줄 단위 읽기/쓰기
string[] lines = File.ReadAllLines("file.txt");
File.WriteAllLines("output.txt", new[] { "Line 1", "Line 2" });

// 바이트 읽기/쓰기
byte[] bytes = File.ReadAllBytes("image.png");
File.WriteAllBytes("copy.png", bytes);

// 파일 추가 (Append)
File.AppendAllText("log.txt", "New log entry\n");
File.AppendAllLines("log.txt", new[] { "Entry 1", "Entry 2" });

// ===== 스트림 사용 (대용량 파일) =====

// StreamReader로 읽기
using var reader = new StreamReader("large-file.txt");
string? line;
while ((line = reader.ReadLine()) != null)
{
    Console.WriteLine(line);
}

// StreamWriter로 쓰기
using var writer = new StreamWriter("output.txt");
writer.WriteLine("First line");
writer.WriteLine("Second line");

// FileStream (바이너리)
using var fs = new FileStream("data.bin", FileMode.Create);
byte[] data = { 0x00, 0x01, 0x02 };
fs.Write(data, 0, data.Length);

// ===== 비동기 파일 작업 =====
string asyncContent = await File.ReadAllTextAsync("file.txt");
await File.WriteAllTextAsync("output.txt", "Async content");
```

### 3.2 파일/디렉토리 관리

```csharp
// 파일 존재 확인
if (File.Exists("file.txt"))
{
    File.Delete("file.txt");
}

// 파일 복사/이동
File.Copy("source.txt", "destination.txt");
File.Copy("source.txt", "destination.txt", overwrite: true);
File.Move("old.txt", "new.txt");

// 파일 정보
var fileInfo = new FileInfo("file.txt");
Console.WriteLine($"크기: {fileInfo.Length} bytes");
Console.WriteLine($"생성일: {fileInfo.CreationTime}");
Console.WriteLine($"수정일: {fileInfo.LastWriteTime}");

// 디렉토리 작업
Directory.CreateDirectory("new-folder");
Directory.Delete("folder", recursive: true);
bool exists = Directory.Exists("folder");

// 파일 목록
string[] files = Directory.GetFiles(".", "*.txt");
string[] allFiles = Directory.GetFiles(".", "*", SearchOption.AllDirectories);
string[] dirs = Directory.GetDirectories(".");

// DirectoryInfo
var dirInfo = new DirectoryInfo(".");
foreach (var file in dirInfo.GetFiles("*.cs"))
{
    Console.WriteLine($"{file.Name}: {file.Length} bytes");
}

// Path 유틸리티
string fileName = Path.GetFileName("/path/to/file.txt");      // "file.txt"
string ext = Path.GetExtension("/path/to/file.txt");          // ".txt"
string dir = Path.GetDirectoryName("/path/to/file.txt");      // "/path/to"
string combined = Path.Combine("folder", "subfolder", "file.txt");
string temp = Path.GetTempPath();
string tempFile = Path.GetTempFileName();
```

---

## 4. JSON 처리

### 4.1 System.Text.Json (내장, 권장)

```csharp
using System.Text.Json;
using System.Text.Json.Serialization;

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    [JsonPropertyName("email_address")]  // JSON 키 이름 지정
    public string Email { get; set; }
    
    [JsonIgnore]  // 직렬화 제외
    public string Password { get; set; }
}

// 직렬화 (객체 → JSON)
var person = new Person { Name = "Alice", Age = 25, Email = "alice@example.com" };
string json = JsonSerializer.Serialize(person);
// {"Name":"Alice","Age":25,"email_address":"alice@example.com"}

// 옵션 지정
var options = new JsonSerializerOptions
{
    WriteIndented = true,  // 들여쓰기
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,  // camelCase
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull
};
string prettyJson = JsonSerializer.Serialize(person, options);

// 역직렬화 (JSON → 객체)
string jsonInput = """{"Name":"Bob","Age":30}""";
Person? parsed = JsonSerializer.Deserialize<Person>(jsonInput);

// 익명 타입으로 역직렬화
using var doc = JsonDocument.Parse(jsonInput);
string name = doc.RootElement.GetProperty("Name").GetString();
int age = doc.RootElement.GetProperty("Age").GetInt32();

// 비동기 처리
await using var stream = File.OpenRead("data.json");
Person? fromFile = await JsonSerializer.DeserializeAsync<Person>(stream);

// 컬렉션 처리
var people = new List<Person> { person };
string jsonArray = JsonSerializer.Serialize(people);
List<Person>? parsedList = JsonSerializer.Deserialize<List<Person>>(jsonArray);
```

### 4.2 Newtonsoft.Json (서드파티)

```csharp
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

// 직렬화/역직렬화
string json = JsonConvert.SerializeObject(person);
string prettyJson = JsonConvert.SerializeObject(person, Formatting.Indented);
Person? parsed = JsonConvert.DeserializeObject<Person>(json);

// 동적 JSON 처리 (JObject)
JObject obj = JObject.Parse(json);
string name = obj["Name"]?.ToString();
obj["Age"] = 26;  // 수정
obj["NewProperty"] = "value";  // 추가

// LINQ to JSON
var people = JArray.Parse(jsonArray);
var adults = people.Where(p => (int)p["Age"] >= 18);

// JSON 경로 쿼리
JToken token = obj.SelectToken("$.Address.City");
```

---

## 5. HTTP 클라이언트

### 5.1 HttpClient 사용

```csharp
// ⚠️ HttpClient는 재사용해야 함 (DI 또는 static)
public class ApiService
{
    private readonly HttpClient _client;
    
    public ApiService(HttpClient client)
    {
        _client = client;
        _client.BaseAddress = new Uri("https://api.example.com");
        _client.DefaultRequestHeaders.Add("Accept", "application/json");
    }
    
    // GET 요청
    public async Task<User?> GetUserAsync(int id)
    {
        var response = await _client.GetAsync($"/users/{id}");
        response.EnsureSuccessStatusCode();
        
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<User>(json);
    }
    
    // GET (간단한 방법)
    public async Task<User?> GetUserSimpleAsync(int id)
    {
        return await _client.GetFromJsonAsync<User>($"/users/{id}");
    }
    
    // POST 요청
    public async Task<User?> CreateUserAsync(User user)
    {
        var response = await _client.PostAsJsonAsync("/users", user);
        response.EnsureSuccessStatusCode();
        
        return await response.Content.ReadFromJsonAsync<User>();
    }
    
    // PUT 요청
    public async Task UpdateUserAsync(int id, User user)
    {
        var response = await _client.PutAsJsonAsync($"/users/{id}", user);
        response.EnsureSuccessStatusCode();
    }
    
    // DELETE 요청
    public async Task DeleteUserAsync(int id)
    {
        var response = await _client.DeleteAsync($"/users/{id}");
        response.EnsureSuccessStatusCode();
    }
    
    // 커스텀 요청
    public async Task<string> CustomRequestAsync()
    {
        var request = new HttpRequestMessage(HttpMethod.Get, "/data");
        request.Headers.Add("Authorization", "Bearer token123");
        request.Headers.Add("X-Custom-Header", "value");
        
        var response = await _client.SendAsync(request);
        return await response.Content.ReadAsStringAsync();
    }
}

// IHttpClientFactory 사용 (DI, 권장)
// Program.cs
builder.Services.AddHttpClient<ApiService>(client =>
{
    client.BaseAddress = new Uri("https://api.example.com");
});
```

### 5.2 파일 업로드

```csharp
public async Task UploadFileAsync(string filePath)
{
    using var content = new MultipartFormDataContent();
    
    // 파일 추가
    var fileContent = new ByteArrayContent(await File.ReadAllBytesAsync(filePath));
    fileContent.Headers.ContentType = MediaTypeHeaderValue.Parse("image/png");
    content.Add(fileContent, "file", Path.GetFileName(filePath));
    
    // 추가 데이터
    content.Add(new StringContent("value"), "key");
    
    var response = await _client.PostAsync("/upload", content);
    response.EnsureSuccessStatusCode();
}
```

---

## 6. 의존성 주입 (Dependency Injection)

### 6.1 DI 기본 개념

```csharp
// ❌ 강한 결합 (테스트하기 어려움)
public class OrderService
{
    private readonly SqlDatabase _database = new SqlDatabase();  // 직접 생성
    
    public void ProcessOrder(Order order)
    {
        _database.Save(order);
    }
}

// ✅ 느슨한 결합 (DI 사용)
public interface IDatabase
{
    void Save<T>(T entity);
}

public class OrderService
{
    private readonly IDatabase _database;
    
    // 생성자 주입
    public OrderService(IDatabase database)
    {
        _database = database;
    }
    
    public void ProcessOrder(Order order)
    {
        _database.Save(order);
    }
}
```

### 6.2 .NET 내장 DI 컨테이너

```csharp
using Microsoft.Extensions.DependencyInjection;

// 서비스 등록
var services = new ServiceCollection();

// 수명 주기 옵션
services.AddTransient<IEmailService, EmailService>();   // 매번 새 인스턴스
services.AddScoped<IOrderService, OrderService>();      // 요청당 하나
services.AddSingleton<ILogger, ConsoleLogger>();        // 앱 전체에서 하나

// 옵션 패턴
services.Configure<SmtpSettings>(config =>
{
    config.Server = "smtp.example.com";
    config.Port = 587;
});

// 팩토리 등록
services.AddTransient<IDbConnection>(sp =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    return new SqlConnection(config.GetConnectionString("Default"));
});

// 서비스 프로바이더 생성
var serviceProvider = services.BuildServiceProvider();

// 서비스 해결
var orderService = serviceProvider.GetRequiredService<IOrderService>();
var emailService = serviceProvider.GetService<IEmailService>();  // null 가능
```

### 6.3 ASP.NET Core에서의 DI

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// 서비스 등록
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddHttpClient<IExternalApi, ExternalApi>();

// 옵션 등록
builder.Services.Configure<AppSettings>(
    builder.Configuration.GetSection("AppSettings"));

var app = builder.Build();

// 컨트롤러에서 사용
public class UserController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly IOptions<AppSettings> _settings;
    
    public UserController(
        IUserService userService,
        IOptions<AppSettings> settings)
    {
        _userService = userService;
        _settings = settings;
    }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetUser(int id)
    {
        var user = await _userService.GetByIdAsync(id);
        return Ok(user);
    }
}
```

---

## 7. 단위 테스트 기초

### 7.1 NUnit 기본

```csharp
using NUnit.Framework;

[TestFixture]  // 테스트 클래스
public class CalculatorTests
{
    private Calculator _calculator;
    
    [SetUp]  // 각 테스트 전 실행
    public void Setup()
    {
        _calculator = new Calculator();
    }
    
    [Test]  // 테스트 메서드
    public void Add_TwoPositiveNumbers_ReturnsCorrectSum()
    {
        // Arrange (준비)
        int a = 5, b = 3;
        
        // Act (실행)
        int result = _calculator.Add(a, b);
        
        // Assert (검증)
        Assert.That(result, Is.EqualTo(8));
    }
    
    [Test]
    public void Divide_ByZero_ThrowsException()
    {
        Assert.Throws<DivideByZeroException>(
            () => _calculator.Divide(10, 0));
    }
    
    // 파라미터화 테스트
    [TestCase(1, 1, 2)]
    [TestCase(0, 0, 0)]
    [TestCase(-1, 1, 0)]
    public void Add_VariousInputs_ReturnsExpectedResult(int a, int b, int expected)
    {
        Assert.That(_calculator.Add(a, b), Is.EqualTo(expected));
    }
    
    [TearDown]  // 각 테스트 후 실행
    public void TearDown()
    {
        _calculator = null;
    }
}
```

### 7.2 Moq를 사용한 Mock

```csharp
using Moq;
using NUnit.Framework;

public interface IEmailService
{
    bool SendEmail(string to, string subject, string body);
}

public class OrderService
{
    private readonly IEmailService _emailService;
    
    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
    
    public bool ProcessOrder(Order order)
    {
        // 비즈니스 로직...
        return _emailService.SendEmail(
            order.CustomerEmail,
            "주문 확인",
            $"주문 #{order.Id}이 처리되었습니다.");
    }
}

[TestFixture]
public class OrderServiceTests
{
    private Mock<IEmailService> _mockEmailService;
    private OrderService _orderService;
    
    [SetUp]
    public void Setup()
    {
        _mockEmailService = new Mock<IEmailService>();
        _orderService = new OrderService(_mockEmailService.Object);
    }
    
    [Test]
    public void ProcessOrder_ValidOrder_SendsEmail()
    {
        // Arrange - Mock 설정
        _mockEmailService
            .Setup(x => x.SendEmail(
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>()))
            .Returns(true);
        
        var order = new Order { Id = 1, CustomerEmail = "test@test.com" };
        
        // Act
        var result = _orderService.ProcessOrder(order);
        
        // Assert
        Assert.That(result, Is.True);
        
        // 호출 검증
        _mockEmailService.Verify(
            x => x.SendEmail(
                "test@test.com",
                "주문 확인",
                It.Is<string>(s => s.Contains("주문 #1"))),
            Times.Once);
    }
}
```

### 7.3 FluentAssertions

```csharp
using FluentAssertions;

[Test]
public void FluentAssertions_Examples()
{
    // 기본 타입
    int value = 5;
    value.Should().Be(5);
    value.Should().BeGreaterThan(3);
    value.Should().BeInRange(1, 10);
    
    // 문자열
    string text = "Hello World";
    text.Should().StartWith("Hello");
    text.Should().Contain("World");
    text.Should().HaveLength(11);
    
    // 컬렉션
    var list = new List<int> { 1, 2, 3 };
    list.Should().HaveCount(3);
    list.Should().Contain(2);
    list.Should().BeInAscendingOrder();
    list.Should().OnlyContain(x => x > 0);
    
    // 예외
    Action act = () => throw new InvalidOperationException("error");
    act.Should().Throw<InvalidOperationException>()
       .WithMessage("error");
    
    // 객체
    var person = new Person { Name = "Alice", Age = 25 };
    person.Should().BeEquivalentTo(new { Name = "Alice", Age = 25 });
}
```

---

## 8. 로깅

### 8.1 Microsoft.Extensions.Logging

```csharp
using Microsoft.Extensions.Logging;

public class UserService
{
    private readonly ILogger<UserService> _logger;
    
    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
    }
    
    public async Task<User?> GetUserAsync(int id)
    {
        _logger.LogDebug("사용자 조회 시작: {UserId}", id);
        
        try
        {
            var user = await _repository.GetByIdAsync(id);
            
            if (user == null)
            {
                _logger.LogWarning("사용자를 찾을 수 없음: {UserId}", id);
                return null;
            }
            
            _logger.LogInformation("사용자 조회 성공: {UserId}, {UserName}", id, user.Name);
            return user;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "사용자 조회 실패: {UserId}", id);
            throw;
        }
    }
}

// 로그 레벨
_logger.LogTrace("가장 상세한 로그");
_logger.LogDebug("디버깅용 로그");
_logger.LogInformation("일반 정보");
_logger.LogWarning("경고");
_logger.LogError("오류");
_logger.LogCritical("치명적 오류");

// 구성 (appsettings.json)
{
    "Logging": {
        "LogLevel": {
            "Default": "Information",
            "Microsoft": "Warning",
            "MyApplication": "Debug"
        }
    }
}
```

### 8.2 Serilog

```csharp
using Serilog;

// 설정
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.Console()
    .WriteTo.File("logs/app-.log", rollingInterval: RollingInterval.Day)
    .WriteTo.Seq("http://localhost:5341")  // 로그 서버
    .CreateLogger();

// 사용
Log.Information("앱 시작됨");
Log.Warning("경고: {Message}", "주의가 필요합니다");
Log.Error(exception, "오류 발생: {ErrorCode}", 500);

// 구조화된 로깅
Log.Information("사용자 {UserId}가 {Action}을 수행함", userId, "로그인");

// 컨텍스트 추가
using (LogContext.PushProperty("CorrelationId", Guid.NewGuid()))
{
    Log.Information("요청 처리 시작");
    // 모든 로그에 CorrelationId가 포함됨
}

// ASP.NET Core 통합
builder.Host.UseSerilog((context, config) =>
{
    config.ReadFrom.Configuration(context.Configuration);
});
```

---

## 9. 구성 관리 (Configuration)

### 9.1 appsettings.json

```json
{
    "AppSettings": {
        "ApplicationName": "MyApp",
        "MaxRetries": 3,
        "EnableFeatureX": true
    },
    "ConnectionStrings": {
        "DefaultConnection": "Server=localhost;Database=mydb;..."
    },
    "SmtpSettings": {
        "Server": "smtp.example.com",
        "Port": 587,
        "Username": "user",
        "Password": "pass"
    }
}
```

### 9.2 구성 읽기

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Options;

// IConfiguration 직접 사용
public class MyService
{
    private readonly IConfiguration _config;
    
    public MyService(IConfiguration config)
    {
        _config = config;
    }
    
    public void DoWork()
    {
        string appName = _config["AppSettings:ApplicationName"];
        int maxRetries = _config.GetValue<int>("AppSettings:MaxRetries");
        string connStr = _config.GetConnectionString("DefaultConnection");
        
        // 섹션 바인딩
        var smtpSettings = new SmtpSettings();
        _config.GetSection("SmtpSettings").Bind(smtpSettings);
    }
}

// Options 패턴 (권장) ⭐
public class SmtpSettings
{
    public string Server { get; set; }
    public int Port { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
}

// Program.cs
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("SmtpSettings"));

// 서비스에서 사용
public class EmailService
{
    private readonly SmtpSettings _settings;
    
    public EmailService(IOptions<SmtpSettings> options)
    {
        _settings = options.Value;
    }
}

// 환경별 구성
// appsettings.json (기본)
// appsettings.Development.json (개발)
// appsettings.Production.json (운영)
```

---

## 10. 자주 사용하는 패턴

### 10.1 Builder 패턴

```csharp
public class Email
{
    public string To { get; private set; }
    public string From { get; private set; }
    public string Subject { get; private set; }
    public string Body { get; private set; }
    public List<string> Cc { get; private set; } = new();
    
    private Email() { }
    
    public class Builder
    {
        private readonly Email _email = new();
        
        public Builder To(string to)
        {
            _email.To = to;
            return this;
        }
        
        public Builder From(string from)
        {
            _email.From = from;
            return this;
        }
        
        public Builder Subject(string subject)
        {
            _email.Subject = subject;
            return this;
        }
        
        public Builder Body(string body)
        {
            _email.Body = body;
            return this;
        }
        
        public Builder AddCc(string cc)
        {
            _email.Cc.Add(cc);
            return this;
        }
        
        public Email Build()
        {
            if (string.IsNullOrEmpty(_email.To))
                throw new InvalidOperationException("수신자 필수");
            return _email;
        }
    }
}

// 사용
var email = new Email.Builder()
    .To("alice@example.com")
    .From("bob@example.com")
    .Subject("안녕하세요")
    .Body("본문 내용")
    .AddCc("charlie@example.com")
    .Build();
```

### 10.2 Repository 패턴

```csharp
// 제네릭 Repository 인터페이스
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

// 구현
public class UserRepository : IRepository<User>
{
    private readonly AppDbContext _context;
    
    public UserRepository(AppDbContext context)
    {
        _context = context;
    }
    
    public async Task<User?> GetByIdAsync(int id)
    {
        return await _context.Users.FindAsync(id);
    }
    
    public async Task<IEnumerable<User>> GetAllAsync()
    {
        return await _context.Users.ToListAsync();
    }
    
    public async Task<User> AddAsync(User entity)
    {
        _context.Users.Add(entity);
        await _context.SaveChangesAsync();
        return entity;
    }
    
    public async Task UpdateAsync(User entity)
    {
        _context.Users.Update(entity);
        await _context.SaveChangesAsync();
    }
    
    public async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        if (entity != null)
        {
            _context.Users.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}
```

### 10.3 Result 패턴

```csharp
// Result 클래스
public class Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public string? Error { get; }
    
    private Result(bool isSuccess, T? value, string? error)
    {
        IsSuccess = isSuccess;
        Value = value;
        Error = error;
    }
    
    public static Result<T> Success(T value) => new(true, value, null);
    public static Result<T> Failure(string error) => new(false, default, error);
}

// 사용
public class UserService
{
    public async Task<Result<User>> CreateUserAsync(CreateUserRequest request)
    {
        // 유효성 검사
        if (string.IsNullOrEmpty(request.Email))
            return Result<User>.Failure("이메일은 필수입니다.");
        
        // 중복 검사
        if (await _repository.ExistsByEmailAsync(request.Email))
            return Result<User>.Failure("이미 존재하는 이메일입니다.");
        
        // 생성
        var user = new User { Email = request.Email, Name = request.Name };
        await _repository.AddAsync(user);
        
        return Result<User>.Success(user);
    }
}

// 호출
var result = await userService.CreateUserAsync(request);
if (result.IsSuccess)
{
    Console.WriteLine($"생성됨: {result.Value.Id}");
}
else
{
    Console.WriteLine($"실패: {result.Error}");
}
```

---

## 📝 Part 4 핵심 정리

### 실전 활용 요약

```csharp
// 1. 네임스페이스 (파일 범위)
namespace MyApp.Services;

// 2. JSON 직렬화
var json = JsonSerializer.Serialize(obj, new JsonSerializerOptions { WriteIndented = true });
var obj = JsonSerializer.Deserialize<MyClass>(json);

// 3. HTTP 클라이언트
var user = await _httpClient.GetFromJsonAsync<User>($"/users/{id}");
await _httpClient.PostAsJsonAsync("/users", newUser);

// 4. 의존성 주입
services.AddScoped<IUserService, UserService>();
services.Configure<AppSettings>(config.GetSection("AppSettings"));

// 5. 테스트
[Test]
public void Method_Scenario_ExpectedResult()
{
    // Arrange, Act, Assert
    result.Should().Be(expected);
}

// 6. 로깅
_logger.LogInformation("사용자 {UserId} 생성됨", user.Id);
```

---

## 🎉 학습 완료!

이제 C#의 핵심 개념을 모두 학습하셨습니다.

### 다음 학습 추천

1. **ASP.NET Core** - 웹 API/MVC 개발
2. **Entity Framework Core** - ORM/데이터베이스
3. **Blazor** - C#으로 프론트엔드 개발
4. **Azure/AWS SDK** - 클라우드 서비스 연동
5. **gRPC** - 고성능 RPC 통신

### 참고 자료

- [Microsoft C# 문서](https://docs.microsoft.com/ko-kr/dotnet/csharp/)
- [.NET API 브라우저](https://docs.microsoft.com/ko-kr/dotnet/api/)
- [C# 코딩 컨벤션](https://docs.microsoft.com/ko-kr/dotnet/csharp/fundamentals/coding-style/coding-conventions)
