# 🎯 Moq 고급 기능 따라하기 실습

## Part 9: Sequential Returns, Protected, Properties, Events

---

## 📋 실습 개요

```
┌────────────────────────────────────────────────────────────────┐
│  학습 목표                                                     │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. SetupSequence: 호출할 때마다 다른 값 반환                  │
│  2. Protected Mock: 추상 클래스의 protected 메서드 Mock        │
│  3. Properties Mock: 속성 Get/Set Mock                         │
│  4. Events Mock: 이벤트 발생 시뮬레이션                        │
│                                                                │
│  실습 환경:                                                    │
│  • Visual Studio 2017                                          │
│  • .NET Framework 4.7.2                                        │
│  • NUnit 3.12.0                                                │
│  • Moq 4.16.1                                                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 1. 프로젝트 설정

### Step 1.1: 솔루션 생성

```
1. Visual Studio 2017 실행
2. 파일 → 새로 만들기 → 프로젝트
3. Visual C# → 클래스 라이브러리(.NET Framework)
4. 이름: MoqAdvancedDemo
5. 프레임워크: .NET Framework 4.7.2
6. 확인
```

### Step 1.2: 테스트 프로젝트 추가

```
1. 솔루션 우클릭 → 추가 → 새 프로젝트
2. Visual C# → 테스트 → 단위 테스트 프로젝트(.NET Framework)
3. 이름: MoqAdvancedDemo.Tests
4. 확인
```

### Step 1.3: NuGet 패키지 설치

```
도구 → NuGet 패키지 관리자 → 패키지 관리자 콘솔

PM> Install-Package NUnit -Version 3.12.0
PM> Install-Package NUnit3TestAdapter -Version 3.17.0
PM> Install-Package Moq -Version 4.16.1
PM> Install-Package Microsoft.NET.Test.Sdk -Version 16.9.4
```

### Step 1.4: 프로젝트 구조

```
MoqAdvancedDemo/
├── MoqAdvancedDemo/
│   ├── Models/
│   │   └── User.cs
│   ├── Interfaces/
│   │   ├── IUserRepository.cs
│   │   ├── IEmailService.cs
│   │   ├── IConfiguration.cs
│   │   └── INotificationService.cs
│   ├── Services/
│   │   ├── BaseService.cs
│   │   ├── UserService.cs
│   │   └── RetryService.cs
│   └── MoqAdvancedDemo.csproj
│
└── MoqAdvancedDemo.Tests/
    ├── SequenceTests.cs
    ├── ProtectedTests.cs
    ├── PropertyTests.cs
    ├── EventTests.cs
    └── MoqAdvancedDemo.Tests.csproj
```

---

## 2. 프로덕션 코드 작성

### Step 2.1: User 모델

```csharp
// Models/User.cs
namespace MoqAdvancedDemo.Models
{
    /// <summary>
    /// 사용자 모델
    /// </summary>
    public class User
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Email { get; set; }
        public bool IsActive { get; set; }
        
        public User()
        {
            Name = string.Empty;
            Email = string.Empty;
            IsActive = true;
        }
    }
}
```

### Step 2.2: IUserRepository 인터페이스

```csharp
// Interfaces/IUserRepository.cs
using MoqAdvancedDemo.Models;

namespace MoqAdvancedDemo.Interfaces
{
    /// <summary>
    /// 사용자 저장소 인터페이스
    /// </summary>
    public interface IUserRepository
    {
        User GetById(int id);
        bool Save(User user);
        bool Delete(int id);
    }
}
```

### Step 2.3: IEmailService 인터페이스

```csharp
// Interfaces/IEmailService.cs
namespace MoqAdvancedDemo.Interfaces
{
    /// <summary>
    /// 이메일 서비스 인터페이스
    /// </summary>
    public interface IEmailService
    {
        bool SendEmail(string to, string subject, string body);
        bool SendBulkEmail(string[] recipients, string subject, string body);
    }
}
```

### Step 2.4: IConfiguration 인터페이스

```csharp
// Interfaces/IConfiguration.cs
namespace MoqAdvancedDemo.Interfaces
{
    /// <summary>
    /// 설정 인터페이스
    /// </summary>
    public interface IConfiguration
    {
        // 읽기/쓰기 속성
        string ConnectionString { get; set; }
        
        // 읽기 전용 속성
        int Timeout { get; }
        
        // 읽기/쓰기 속성
        bool IsEnabled { get; set; }
        
        // 읽기/쓰기 속성
        int MaxRetryCount { get; set; }
        
        // 읽기/쓰기 속성
        string ApplicationName { get; set; }
    }
}
```

### Step 2.5: INotificationService 인터페이스

```csharp
// Interfaces/INotificationService.cs
using System;

namespace MoqAdvancedDemo.Interfaces
{
    /// <summary>
    /// 알림 서비스 인터페이스 (이벤트 포함)
    /// </summary>
    public interface INotificationService
    {
        // 알림 수신 이벤트
        event EventHandler<string> NotificationReceived;
        
        // 연결 상태 변경 이벤트
        event EventHandler<bool> ConnectionStatusChanged;
        
        // 메서드
        void Subscribe();
        void Unsubscribe();
        void Connect();
        void Disconnect();
    }
}
```

### Step 2.6: BaseService 추상 클래스

```csharp
// Services/BaseService.cs
namespace MoqAdvancedDemo.Services
{
    /// <summary>
    /// 기본 서비스 추상 클래스 (Protected 메서드 포함)
    /// </summary>
    public abstract class BaseService
    {
        /// <summary>
        /// 입력값 검증 (protected - 서브클래스에서 구현)
        /// </summary>
        protected abstract bool ValidateInput(string input);
        
        /// <summary>
        /// 데이터 변환 (protected - 서브클래스에서 구현)
        /// </summary>
        protected abstract string TransformData(string data);
        
        /// <summary>
        /// 입력 처리 (public - 외부에서 호출)
        /// </summary>
        public bool ProcessInput(string input)
        {
            // protected 메서드 호출
            if (!ValidateInput(input))
            {
                return false;
            }
            
            // 처리 로직...
            var transformed = TransformData(input);
            
            return !string.IsNullOrEmpty(transformed);
        }
        
        /// <summary>
        /// 데이터 처리 (public)
        /// </summary>
        public string ProcessData(string data)
        {
            if (string.IsNullOrEmpty(data))
            {
                return null;
            }
            
            return TransformData(data);
        }
    }
}
```

### Step 2.7: RetryService 클래스

```csharp
// Services/RetryService.cs
using System;
using MoqAdvancedDemo.Interfaces;

namespace MoqAdvancedDemo.Services
{
    /// <summary>
    /// 재시도 로직을 포함한 서비스
    /// </summary>
    public class RetryService
    {
        private readonly IEmailService _emailService;
        private readonly IConfiguration _configuration;
        
        public RetryService(IEmailService emailService, IConfiguration configuration)
        {
            _emailService = emailService;
            _configuration = configuration;
        }
        
        /// <summary>
        /// 재시도 로직이 포함된 이메일 발송
        /// </summary>
        public bool SendEmailWithRetry(string to, string subject, string body)
        {
            int maxRetry = _configuration.MaxRetryCount;
            int attempt = 0;
            
            while (attempt < maxRetry)
            {
                attempt++;
                
                try
                {
                    if (_emailService.SendEmail(to, subject, body))
                    {
                        return true;
                    }
                }
                catch (Exception)
                {
                    // 마지막 시도가 아니면 계속
                    if (attempt >= maxRetry)
                    {
                        throw;
                    }
                }
            }
            
            return false;
        }
    }
}
```

### Step 2.8: UserService 클래스

```csharp
// Services/UserService.cs
using System;
using MoqAdvancedDemo.Interfaces;
using MoqAdvancedDemo.Models;

namespace MoqAdvancedDemo.Services
{
    /// <summary>
    /// 사용자 서비스
    /// </summary>
    public class UserService
    {
        private readonly IUserRepository _repository;
        private readonly IConfiguration _configuration;
        
        public UserService(IUserRepository repository, IConfiguration configuration)
        {
            _repository = repository;
            _configuration = configuration;
        }
        
        /// <summary>
        /// 사용자 조회 (재시도 포함)
        /// </summary>
        public User GetUserWithRetry(int id)
        {
            int maxRetry = _configuration.MaxRetryCount;
            
            for (int i = 0; i < maxRetry; i++)
            {
                var user = _repository.GetById(id);
                if (user != null)
                {
                    return user;
                }
            }
            
            return null;
        }
        
        /// <summary>
        /// 설정이 활성화된 경우에만 사용자 조회
        /// </summary>
        public User GetUserIfEnabled(int id)
        {
            if (!_configuration.IsEnabled)
            {
                return null;
            }
            
            return _repository.GetById(id);
        }
    }
}
```

---

## 3. SetupSequence 테스트 (순차 반환)

### Step 3.1: SequenceTests.cs 작성

```csharp
// SequenceTests.cs
using System;
using Moq;
using NUnit.Framework;
using MoqAdvancedDemo.Interfaces;
using MoqAdvancedDemo.Models;
using MoqAdvancedDemo.Services;

namespace MoqAdvancedDemo.Tests
{
    /// <summary>
    /// SetupSequence 테스트
    /// 같은 메서드 호출에 대해 순차적으로 다른 값을 반환
    /// </summary>
    [TestFixture]
    public class SequenceTests
    {
        private Mock<IUserRepository> _mockRepository;
        private Mock<IEmailService> _mockEmailService;
        private Mock<IConfiguration> _mockConfiguration;
        
        [SetUp]
        public void SetUp()
        {
            _mockRepository = new Mock<IUserRepository>();
            _mockEmailService = new Mock<IEmailService>();
            _mockConfiguration = new Mock<IConfiguration>();
        }
        
        #region 기본 SetupSequence 테스트
        
        /// <summary>
        /// 테스트 1: 첫 번째 호출은 null, 두 번째 호출은 User 반환
        /// </summary>
        [Test]
        public void GetById_FirstCallReturnsNull_SecondCallReturnsUser()
        {
            // Arrange - 첫 번째는 null, 두 번째는 User
            _mockRepository
                .SetupSequence(x => x.GetById(1))
                .Returns((User)null)                              // 1번째 호출: null
                .Returns(new User { Id = 1, Name = "홍길동" });   // 2번째 호출: User
            
            // Act
            var result1 = _mockRepository.Object.GetById(1);
            var result2 = _mockRepository.Object.GetById(1);
            
            // Assert
            Assert.IsNull(result1, "첫 번째 호출은 null이어야 합니다");
            Assert.IsNotNull(result2, "두 번째 호출은 User여야 합니다");
            Assert.AreEqual("홍길동", result2.Name);
            
            // 출력
            Console.WriteLine("=== SetupSequence 기본 테스트 ===");
            Console.WriteLine("1번째 호출: " + (result1 == null ? "null" : result1.Name));
            Console.WriteLine("2번째 호출: " + result2.Name);
        }
        
        /// <summary>
        /// 테스트 2: 세 번 연속 다른 값 반환
        /// </summary>
        [Test]
        public void GetById_ThreeSequentialCalls_ReturnsDifferentValues()
        {
            // Arrange - 세 가지 다른 결과
            _mockRepository
                .SetupSequence(x => x.GetById(It.IsAny<int>()))
                .Returns(new User { Id = 1, Name = "첫 번째" })
                .Returns(new User { Id = 2, Name = "두 번째" })
                .Returns(new User { Id = 3, Name = "세 번째" });
            
            // Act
            var result1 = _mockRepository.Object.GetById(1);
            var result2 = _mockRepository.Object.GetById(1);
            var result3 = _mockRepository.Object.GetById(1);
            
            // Assert
            Assert.AreEqual("첫 번째", result1.Name);
            Assert.AreEqual("두 번째", result2.Name);
            Assert.AreEqual("세 번째", result3.Name);
            
            // 출력
            Console.WriteLine("=== 세 번 연속 호출 테스트 ===");
            Console.WriteLine("1번째: " + result1.Name);
            Console.WriteLine("2번째: " + result2.Name);
            Console.WriteLine("3번째: " + result3.Name);
        }
        
        #endregion
        
        #region 예외와 성공 혼합 테스트
        
        /// <summary>
        /// 테스트 3: 첫 번째는 예외, 두 번째는 성공
        /// </summary>
        [Test]
        public void SendEmail_FirstThrowsException_SecondSucceeds()
        {
            // Arrange - 첫 번째는 예외, 두 번째는 성공
            _mockEmailService
                .SetupSequence(x => x.SendEmail(
                    It.IsAny<string>(), 
                    It.IsAny<string>(), 
                    It.IsAny<string>()))
                .Throws(new Exception("네트워크 오류"))  // 1번째: 예외
                .Returns(true);                          // 2번째: 성공
            
            // Act & Assert - 첫 번째 호출 (예외)
            var exception = Assert.Throws<Exception>(() => 
                _mockEmailService.Object.SendEmail("to@test.com", "제목", "내용"));
            
            Assert.AreEqual("네트워크 오류", exception.Message);
            Console.WriteLine("1번째 호출: 예외 발생 - " + exception.Message);
            
            // 두 번째 호출 (성공)
            var result = _mockEmailService.Object.SendEmail("to@test.com", "제목", "내용");
            
            Assert.IsTrue(result);
            Console.WriteLine("2번째 호출: 성공 - " + result);
        }
        
        /// <summary>
        /// 테스트 4: 두 번 실패 후 성공
        /// </summary>
        [Test]
        public void SendEmail_TwoFailuresThenSuccess()
        {
            // Arrange - 두 번 실패, 세 번째 성공
            _mockEmailService
                .SetupSequence(x => x.SendEmail(
                    It.IsAny<string>(), 
                    It.IsAny<string>(), 
                    It.IsAny<string>()))
                .Throws(new Exception("1차 실패"))
                .Throws(new Exception("2차 실패"))
                .Returns(true);
            
            // Act & Assert
            Console.WriteLine("=== 재시도 시뮬레이션 ===");
            
            // 1차 시도
            try
            {
                _mockEmailService.Object.SendEmail("to", "subject", "body");
            }
            catch (Exception ex)
            {
                Console.WriteLine("1차 시도: 실패 - " + ex.Message);
            }
            
            // 2차 시도
            try
            {
                _mockEmailService.Object.SendEmail("to", "subject", "body");
            }
            catch (Exception ex)
            {
                Console.WriteLine("2차 시도: 실패 - " + ex.Message);
            }
            
            // 3차 시도
            var result = _mockEmailService.Object.SendEmail("to", "subject", "body");
            Console.WriteLine("3차 시도: 성공 - " + result);
            
            Assert.IsTrue(result);
        }
        
        #endregion
        
        #region 실제 서비스와 통합 테스트
        
        /// <summary>
        /// 테스트 5: RetryService와 SetupSequence 통합
        /// </summary>
        [Test]
        public void RetryService_RetriesUntilSuccess()
        {
            // Arrange
            _mockConfiguration.Setup(x => x.MaxRetryCount).Returns(3);
            
            // 첫 번째와 두 번째는 false, 세 번째는 true
            _mockEmailService
                .SetupSequence(x => x.SendEmail(
                    It.IsAny<string>(), 
                    It.IsAny<string>(), 
                    It.IsAny<string>()))
                .Returns(false)
                .Returns(false)
                .Returns(true);
            
            var retryService = new RetryService(
                _mockEmailService.Object, 
                _mockConfiguration.Object);
            
            // Act
            var result = retryService.SendEmailWithRetry("to@test.com", "제목", "내용");
            
            // Assert
            Assert.IsTrue(result, "3번째 시도에서 성공해야 합니다");
            
            // Verify - 3번 호출되었는지 확인
            _mockEmailService.Verify(
                x => x.SendEmail(It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()),
                Times.Exactly(3));
            
            Console.WriteLine("=== RetryService 통합 테스트 ===");
            Console.WriteLine("총 시도 횟수: 3회");
            Console.WriteLine("결과: 성공");
        }
        
        /// <summary>
        /// 테스트 6: UserService 재시도 로직
        /// </summary>
        [Test]
        public void UserService_GetUserWithRetry_SucceedsOnThirdAttempt()
        {
            // Arrange
            _mockConfiguration.Setup(x => x.MaxRetryCount).Returns(3);
            
            // 처음 두 번은 null, 세 번째는 User
            _mockRepository
                .SetupSequence(x => x.GetById(1))
                .Returns((User)null)
                .Returns((User)null)
                .Returns(new User { Id = 1, Name = "홍길동" });
            
            var userService = new UserService(
                _mockRepository.Object, 
                _mockConfiguration.Object);
            
            // Act
            var result = userService.GetUserWithRetry(1);
            
            // Assert
            Assert.IsNotNull(result);
            Assert.AreEqual("홍길동", result.Name);
            
            Console.WriteLine("=== UserService 재시도 테스트 ===");
            Console.WriteLine("3번째 시도에서 사용자 조회 성공: " + result.Name);
        }
        
        #endregion
        
        #region 페이징 시뮬레이션
        
        /// <summary>
        /// 테스트 7: 페이징 시뮬레이션
        /// </summary>
        [Test]
        public void Paging_ReturnsDataThenEmptyList()
        {
            // Arrange - 페이지 1, 2는 데이터, 페이지 3은 빈 리스트
            var mockPagingRepo = new Mock<IUserRepository>();
            
            mockPagingRepo
                .SetupSequence(x => x.GetById(It.IsAny<int>()))
                .Returns(new User { Id = 1, Name = "페이지1 사용자" })
                .Returns(new User { Id = 2, Name = "페이지2 사용자" })
                .Returns((User)null);  // 더 이상 데이터 없음
            
            // Act - 페이징 시뮬레이션
            Console.WriteLine("=== 페이징 시뮬레이션 ===");
            
            int page = 1;
            User user;
            
            while ((user = mockPagingRepo.Object.GetById(page)) != null)
            {
                Console.WriteLine(string.Format("페이지 {0}: {1}", page, user.Name));
                page++;
            }
            
            Console.WriteLine(string.Format("페이지 {0}: 데이터 없음 (종료)", page));
            
            // Assert
            Assert.AreEqual(3, page, "3페이지에서 종료되어야 합니다");
        }
        
        #endregion
    }
}
```

### Step 3.2: SequenceTests 실행 결과

```
=== SetupSequence 기본 테스트 ===
1번째 호출: null
2번째 호출: 홍길동

=== 세 번 연속 호출 테스트 ===
1번째: 첫 번째
2번째: 두 번째
3번째: 세 번째

=== 재시도 시뮬레이션 ===
1차 시도: 실패 - 1차 실패
2차 시도: 실패 - 2차 실패
3차 시도: 성공 - True

=== RetryService 통합 테스트 ===
총 시도 횟수: 3회
결과: 성공

✅ 7 tests passed
```

---

## 4. Protected 멤버 Mock 테스트

### Step 4.1: ProtectedTests.cs 작성

```csharp
// ProtectedTests.cs
using Moq;
using Moq.Protected;  // Protected Mock을 위한 네임스페이스
using NUnit.Framework;
using MoqAdvancedDemo.Services;

namespace MoqAdvancedDemo.Tests
{
    /// <summary>
    /// Protected 멤버 Mock 테스트
    /// 추상 클래스의 protected 메서드를 Mock
    /// </summary>
    [TestFixture]
    public class ProtectedTests
    {
        #region 기본 Protected Mock 테스트
        
        /// <summary>
        /// 테스트 1: Protected 메서드가 true 반환하도록 설정
        /// </summary>
        [Test]
        public void ProcessInput_ValidateInputReturnsTrue_ReturnsTrue()
        {
            // Arrange
            var mockService = new Mock<BaseService>();
            
            // Protected 메서드 Setup
            // "ValidateInput" - 메서드 이름 (문자열)
            // ItExpr.IsAny<string>() - 파라미터 매칭 (It.IsAny가 아닌 ItExpr 사용!)
            mockService
                .Protected()
                .Setup<bool>("ValidateInput", ItExpr.IsAny<string>())
                .Returns(true);
            
            // TransformData도 설정
            mockService
                .Protected()
                .Setup<string>("TransformData", ItExpr.IsAny<string>())
                .Returns("변환된 데이터");
            
            // Act
            var result = mockService.Object.ProcessInput("테스트 입력");
            
            // Assert
            Assert.IsTrue(result, "ValidateInput이 true를 반환하면 ProcessInput도 true여야 합니다");
            
            Console.WriteLine("=== Protected Mock 기본 테스트 ===");
            Console.WriteLine("ValidateInput 반환값: true");
            Console.WriteLine("ProcessInput 결과: " + result);
        }
        
        /// <summary>
        /// 테스트 2: Protected 메서드가 false 반환하도록 설정
        /// </summary>
        [Test]
        public void ProcessInput_ValidateInputReturnsFalse_ReturnsFalse()
        {
            // Arrange
            var mockService = new Mock<BaseService>();
            
            // ValidateInput이 false 반환
            mockService
                .Protected()
                .Setup<bool>("ValidateInput", ItExpr.IsAny<string>())
                .Returns(false);
            
            // Act
            var result = mockService.Object.ProcessInput("테스트 입력");
            
            // Assert
            Assert.IsFalse(result, "ValidateInput이 false를 반환하면 ProcessInput도 false여야 합니다");
            
            Console.WriteLine("=== Protected Mock false 테스트 ===");
            Console.WriteLine("ValidateInput 반환값: false");
            Console.WriteLine("ProcessInput 결과: " + result);
        }
        
        #endregion
        
        #region 특정 파라미터에 대한 Protected Mock
        
        /// <summary>
        /// 테스트 3: 특정 파라미터 값에 따라 다른 결과
        /// </summary>
        [Test]
        public void ProcessInput_SpecificParameter_ReturnsDifferentResults()
        {
            // Arrange
            var mockService = new Mock<BaseService>();
            
            // "valid"일 때만 true
            mockService
                .Protected()
                .Setup<bool>("ValidateInput", ItExpr.Is<string>(s => s == "valid"))
                .Returns(true);
            
            // "invalid"일 때 false
            mockService
                .Protected()
                .Setup<bool>("ValidateInput", ItExpr.Is<string>(s => s == "invalid"))
                .Returns(false);
            
            // TransformData 설정
            mockService
                .Protected()
                .Setup<string>("TransformData", ItExpr.IsAny<string>())
                .Returns("변환됨");
            
            // Act
            var validResult = mockService.Object.ProcessInput("valid");
            var invalidResult = mockService.Object.ProcessInput("invalid");
            
            // Assert
            Assert.IsTrue(validResult, "valid 입력은 true여야 합니다");
            Assert.IsFalse(invalidResult, "invalid 입력은 false여야 합니다");
            
            Console.WriteLine("=== 파라미터별 다른 결과 ===");
            Console.WriteLine("입력 'valid': " + validResult);
            Console.WriteLine("입력 'invalid': " + invalidResult);
        }
        
        #endregion
        
        #region Protected 메서드 호출 검증
        
        /// <summary>
        /// 테스트 4: Protected 메서드 호출 검증 (Verify)
        /// </summary>
        [Test]
        public void ProcessInput_VerifyProtectedMethodCalled()
        {
            // Arrange
            var mockService = new Mock<BaseService>();
            
            mockService
                .Protected()
                .Setup<bool>("ValidateInput", ItExpr.IsAny<string>())
                .Returns(true);
            
            mockService
                .Protected()
                .Setup<string>("TransformData", ItExpr.IsAny<string>())
                .Returns("결과");
            
            // Act
            mockService.Object.ProcessInput("테스트");
            
            // Assert - Protected 메서드 호출 검증
            mockService
                .Protected()
                .Verify<bool>("ValidateInput", Times.Once(), ItExpr.IsAny<string>());
            
            mockService
                .Protected()
                .Verify<string>("TransformData", Times.Once(), ItExpr.IsAny<string>());
            
            Console.WriteLine("=== Protected 메서드 호출 검증 ===");
            Console.WriteLine("ValidateInput 호출됨: 1회");
            Console.WriteLine("TransformData 호출됨: 1회");
        }
        
        #endregion
        
        #region TransformData 테스트
        
        /// <summary>
        /// 테스트 5: TransformData Mock
        /// </summary>
        [Test]
        public void ProcessData_TransformDataMocked_ReturnsTransformedValue()
        {
            // Arrange
            var mockService = new Mock<BaseService>();
            
            mockService
                .Protected()
                .Setup<string>("TransformData", ItExpr.Is<string>(s => s == "input"))
                .Returns("TRANSFORMED_INPUT");
            
            // Act
            var result = mockService.Object.ProcessData("input");
            
            // Assert
            Assert.AreEqual("TRANSFORMED_INPUT", result);
            
            Console.WriteLine("=== TransformData Mock 테스트 ===");
            Console.WriteLine("입력: input");
            Console.WriteLine("변환 결과: " + result);
        }
        
        /// <summary>
        /// 테스트 6: TransformData가 null 반환
        /// </summary>
        [Test]
        public void ProcessData_TransformDataReturnsNull_ReturnsNull()
        {
            // Arrange
            var mockService = new Mock<BaseService>();
            
            mockService
                .Protected()
                .Setup<string>("TransformData", ItExpr.IsAny<string>())
                .Returns((string)null);
            
            // Act
            var result = mockService.Object.ProcessData("test");
            
            // Assert
            Assert.IsNull(result);
            
            Console.WriteLine("=== TransformData null 반환 테스트 ===");
            Console.WriteLine("결과: null");
        }
        
        #endregion
    }
}
```

### Step 4.2: Protected Mock 핵심 포인트

```
┌────────────────────────────────────────────────────────────────┐
│  Protected Mock 핵심 포인트                                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. 네임스페이스 추가                                          │
│     using Moq.Protected;                                       │
│                                                                │
│  2. Setup 문법                                                 │
│     mockService                                                │
│         .Protected()                                           │
│         .Setup<반환타입>("메서드이름", ItExpr.IsAny<T>())      │
│         .Returns(값);                                          │
│                                                                │
│  3. ItExpr 사용 (It 대신)                                      │
│     • ItExpr.IsAny<T>()                                       │
│     • ItExpr.Is<T>(x => 조건)                                 │
│                                                                │
│  4. Verify 문법                                                │
│     mockService                                                │
│         .Protected()                                           │
│         .Verify<반환타입>("메서드이름", Times.Once(),          │
│             ItExpr.IsAny<T>());                                │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 5. Properties Mock 테스트

### Step 5.1: PropertyTests.cs 작성

```csharp
// PropertyTests.cs
using Moq;
using NUnit.Framework;
using MoqAdvancedDemo.Interfaces;
using MoqAdvancedDemo.Services;
using MoqAdvancedDemo.Models;

namespace MoqAdvancedDemo.Tests
{
    /// <summary>
    /// 속성(Properties) Mock 테스트
    /// </summary>
    [TestFixture]
    public class PropertyTests
    {
        private Mock<IConfiguration> _mockConfig;
        
        [SetUp]
        public void SetUp()
        {
            _mockConfig = new Mock<IConfiguration>();
        }
        
        #region 읽기 전용 속성 테스트
        
        /// <summary>
        /// 테스트 1: 읽기 전용 속성 Setup
        /// </summary>
        [Test]
        public void ReadOnlyProperty_Setup_ReturnsValue()
        {
            // Arrange
            _mockConfig.Setup(x => x.Timeout).Returns(30);
            
            // Act
            var timeout = _mockConfig.Object.Timeout;
            
            // Assert
            Assert.AreEqual(30, timeout);
            
            Console.WriteLine("=== 읽기 전용 속성 테스트 ===");
            Console.WriteLine("Timeout: " + timeout);
        }
        
        #endregion
        
        #region SetupProperty 테스트 (Get/Set)
        
        /// <summary>
        /// 테스트 2: SetupProperty로 Get/Set 모두 지원
        /// </summary>
        [Test]
        public void SetupProperty_GetAndSet_Works()
        {
            // Arrange - 초기값과 함께 속성 설정
            _mockConfig.SetupProperty(x => x.ConnectionString, "Server=localhost");
            _mockConfig.SetupProperty(x => x.IsEnabled, true);
            
            // Act - Get
            var connectionString = _mockConfig.Object.ConnectionString;
            var isEnabled = _mockConfig.Object.IsEnabled;
            
            Console.WriteLine("=== SetupProperty Get/Set 테스트 ===");
            Console.WriteLine("초기값 - ConnectionString: " + connectionString);
            Console.WriteLine("초기값 - IsEnabled: " + isEnabled);
            
            // Assert - 초기값 확인
            Assert.AreEqual("Server=localhost", connectionString);
            Assert.IsTrue(isEnabled);
            
            // Act - Set (값 변경)
            _mockConfig.Object.ConnectionString = "Server=production";
            _mockConfig.Object.IsEnabled = false;
            
            // Assert - 변경된 값 확인
            Assert.AreEqual("Server=production", _mockConfig.Object.ConnectionString);
            Assert.IsFalse(_mockConfig.Object.IsEnabled);
            
            Console.WriteLine("변경 후 - ConnectionString: " + _mockConfig.Object.ConnectionString);
            Console.WriteLine("변경 후 - IsEnabled: " + _mockConfig.Object.IsEnabled);
        }
        
        /// <summary>
        /// 테스트 3: SetupProperty 초기값 없이
        /// </summary>
        [Test]
        public void SetupProperty_NoInitialValue_DefaultsToTypeDefault()
        {
            // Arrange - 초기값 없이 속성 설정
            _mockConfig.SetupProperty(x => x.MaxRetryCount);
            _mockConfig.SetupProperty(x => x.ApplicationName);
            
            // Act - 기본값 확인
            var maxRetry = _mockConfig.Object.MaxRetryCount;  // int 기본값: 0
            var appName = _mockConfig.Object.ApplicationName;  // string 기본값: null
            
            Console.WriteLine("=== SetupProperty 기본값 테스트 ===");
            Console.WriteLine("MaxRetryCount (int 기본값): " + maxRetry);
            Console.WriteLine("ApplicationName (string 기본값): " + (appName ?? "null"));
            
            // Assert
            Assert.AreEqual(0, maxRetry);
            Assert.IsNull(appName);
            
            // Act - 값 설정
            _mockConfig.Object.MaxRetryCount = 5;
            _mockConfig.Object.ApplicationName = "MyApp";
            
            // Assert
            Assert.AreEqual(5, _mockConfig.Object.MaxRetryCount);
            Assert.AreEqual("MyApp", _mockConfig.Object.ApplicationName);
            
            Console.WriteLine("설정 후 - MaxRetryCount: " + _mockConfig.Object.MaxRetryCount);
            Console.WriteLine("설정 후 - ApplicationName: " + _mockConfig.Object.ApplicationName);
        }
        
        #endregion
        
        #region SetupAllProperties 테스트
        
        /// <summary>
        /// 테스트 4: SetupAllProperties로 모든 속성 추적
        /// </summary>
        [Test]
        public void SetupAllProperties_AllPropertiesTrackable()
        {
            // Arrange - 모든 속성을 추적 가능하게 설정
            _mockConfig.SetupAllProperties();
            
            // Timeout은 읽기 전용이므로 별도 Setup 필요
            _mockConfig.Setup(x => x.Timeout).Returns(60);
            
            // Act - 값 설정
            _mockConfig.Object.ConnectionString = "Server=test";
            _mockConfig.Object.IsEnabled = true;
            _mockConfig.Object.MaxRetryCount = 3;
            _mockConfig.Object.ApplicationName = "TestApp";
            
            // Assert
            Assert.AreEqual("Server=test", _mockConfig.Object.ConnectionString);
            Assert.IsTrue(_mockConfig.Object.IsEnabled);
            Assert.AreEqual(3, _mockConfig.Object.MaxRetryCount);
            Assert.AreEqual("TestApp", _mockConfig.Object.ApplicationName);
            Assert.AreEqual(60, _mockConfig.Object.Timeout);
            
            Console.WriteLine("=== SetupAllProperties 테스트 ===");
            Console.WriteLine("ConnectionString: " + _mockConfig.Object.ConnectionString);
            Console.WriteLine("IsEnabled: " + _mockConfig.Object.IsEnabled);
            Console.WriteLine("MaxRetryCount: " + _mockConfig.Object.MaxRetryCount);
            Console.WriteLine("ApplicationName: " + _mockConfig.Object.ApplicationName);
            Console.WriteLine("Timeout (읽기 전용): " + _mockConfig.Object.Timeout);
        }
        
        #endregion
        
        #region 서비스와 통합 테스트
        
        /// <summary>
        /// 테스트 5: UserService와 Configuration 속성 통합
        /// </summary>
        [Test]
        public void UserService_UsesConfigurationProperties()
        {
            // Arrange
            var mockRepo = new Mock<IUserRepository>();
            
            _mockConfig.SetupAllProperties();
            _mockConfig.Object.IsEnabled = true;
            _mockConfig.Object.MaxRetryCount = 2;
            
            mockRepo.Setup(x => x.GetById(1)).Returns(new User { Id = 1, Name = "테스트" });
            
            var userService = new UserService(mockRepo.Object, _mockConfig.Object);
            
            // Act
            var user = userService.GetUserIfEnabled(1);
            
            // Assert
            Assert.IsNotNull(user);
            Assert.AreEqual("테스트", user.Name);
            
            Console.WriteLine("=== 서비스 통합 테스트 ===");
            Console.WriteLine("IsEnabled: true → 사용자 조회 성공");
            Console.WriteLine("조회된 사용자: " + user.Name);
        }
        
        /// <summary>
        /// 테스트 6: IsEnabled가 false일 때
        /// </summary>
        [Test]
        public void UserService_IsEnabledFalse_ReturnsNull()
        {
            // Arrange
            var mockRepo = new Mock<IUserRepository>();
            
            _mockConfig.SetupAllProperties();
            _mockConfig.Object.IsEnabled = false;  // 비활성화
            
            var userService = new UserService(mockRepo.Object, _mockConfig.Object);
            
            // Act
            var user = userService.GetUserIfEnabled(1);
            
            // Assert
            Assert.IsNull(user);
            
            // Repository가 호출되지 않았는지 확인
            mockRepo.Verify(x => x.GetById(It.IsAny<int>()), Times.Never);
            
            Console.WriteLine("=== IsEnabled false 테스트 ===");
            Console.WriteLine("IsEnabled: false → 사용자 조회 안함");
            Console.WriteLine("Repository 호출 횟수: 0");
        }
        
        #endregion
        
        #region 속성 값 변경 추적
        
        /// <summary>
        /// 테스트 7: 속성 값 변경 여러 번
        /// </summary>
        [Test]
        public void Property_MultipleChanges_TracksAllChanges()
        {
            // Arrange
            _mockConfig.SetupProperty(x => x.MaxRetryCount, 1);
            
            Console.WriteLine("=== 속성 값 변경 추적 ===");
            Console.WriteLine("초기값: " + _mockConfig.Object.MaxRetryCount);
            
            // Act & Assert - 여러 번 변경
            _mockConfig.Object.MaxRetryCount = 2;
            Assert.AreEqual(2, _mockConfig.Object.MaxRetryCount);
            Console.WriteLine("1차 변경: " + _mockConfig.Object.MaxRetryCount);
            
            _mockConfig.Object.MaxRetryCount = 5;
            Assert.AreEqual(5, _mockConfig.Object.MaxRetryCount);
            Console.WriteLine("2차 변경: " + _mockConfig.Object.MaxRetryCount);
            
            _mockConfig.Object.MaxRetryCount = 10;
            Assert.AreEqual(10, _mockConfig.Object.MaxRetryCount);
            Console.WriteLine("3차 변경: " + _mockConfig.Object.MaxRetryCount);
        }
        
        #endregion
    }
}
```

### Step 5.2: Properties Mock 핵심 포인트

```
┌────────────────────────────────────────────────────────────────┐
│  Properties Mock 핵심 포인트                                   │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. 읽기 전용 속성                                             │
│     mockConfig.Setup(x => x.Timeout).Returns(30);              │
│     → Get만 가능                                               │
│                                                                │
│  2. SetupProperty (개별 속성)                                  │
│     mockConfig.SetupProperty(x => x.Name, "초기값");           │
│     → Get/Set 모두 가능                                        │
│     → 값이 추적됨 (변경하면 변경된 값 유지)                   │
│                                                                │
│  3. SetupAllProperties (모든 속성)                             │
│     mockConfig.SetupAllProperties();                           │
│     → 모든 속성을 Get/Set 가능하게 설정                       │
│     → 읽기 전용 속성은 별도 Setup 필요                        │
│                                                                │
│  비교:                                                         │
│  ┌─────────────────┬─────────────────┬─────────────────┐      │
│  │     방법        │      Get        │      Set        │      │
│  ├─────────────────┼─────────────────┼─────────────────┤      │
│  │ Setup           │       ✅        │       ❌        │      │
│  │ SetupProperty   │       ✅        │       ✅        │      │
│  │ SetupAllProps   │       ✅        │       ✅        │      │
│  └─────────────────┴─────────────────┴─────────────────┘      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 6. Events Mock 테스트

### Step 6.1: EventTests.cs 작성

```csharp
// EventTests.cs
using System;
using Moq;
using NUnit.Framework;
using MoqAdvancedDemo.Interfaces;

namespace MoqAdvancedDemo.Tests
{
    /// <summary>
    /// 이벤트(Events) Mock 테스트
    /// </summary>
    [TestFixture]
    public class EventTests
    {
        private Mock<INotificationService> _mockNotification;
        
        [SetUp]
        public void SetUp()
        {
            _mockNotification = new Mock<INotificationService>();
        }
        
        #region 기본 이벤트 발생 테스트
        
        /// <summary>
        /// 테스트 1: 이벤트 발생 시뮬레이션
        /// </summary>
        [Test]
        public void RaiseEvent_NotificationReceived_HandlerCalled()
        {
            // Arrange
            string receivedMessage = null;
            
            // 이벤트 핸들러 등록
            _mockNotification.Object.NotificationReceived += (sender, message) =>
            {
                receivedMessage = message;
                Console.WriteLine("이벤트 수신: " + message);
            };
            
            // Act - 이벤트 발생 시뮬레이션
            _mockNotification.Raise(
                x => x.NotificationReceived += null,  // 이벤트 지정
                _mockNotification.Object,              // sender
                "새 알림입니다"                        // EventArgs (string)
            );
            
            // Assert
            Assert.AreEqual("새 알림입니다", receivedMessage);
            
            Console.WriteLine("=== 이벤트 발생 테스트 ===");
            Console.WriteLine("발생된 메시지: " + receivedMessage);
        }
        
        /// <summary>
        /// 테스트 2: 여러 번 이벤트 발생
        /// </summary>
        [Test]
        public void RaiseEvent_MultipleTimes_AllHandled()
        {
            // Arrange
            var receivedMessages = new System.Collections.Generic.List<string>();
            
            _mockNotification.Object.NotificationReceived += (sender, message) =>
            {
                receivedMessages.Add(message);
            };
            
            // Act - 여러 번 이벤트 발생
            _mockNotification.Raise(
                x => x.NotificationReceived += null,
                _mockNotification.Object,
                "첫 번째 알림");
            
            _mockNotification.Raise(
                x => x.NotificationReceived += null,
                _mockNotification.Object,
                "두 번째 알림");
            
            _mockNotification.Raise(
                x => x.NotificationReceived += null,
                _mockNotification.Object,
                "세 번째 알림");
            
            // Assert
            Assert.AreEqual(3, receivedMessages.Count);
            Assert.AreEqual("첫 번째 알림", receivedMessages[0]);
            Assert.AreEqual("두 번째 알림", receivedMessages[1]);
            Assert.AreEqual("세 번째 알림", receivedMessages[2]);
            
            Console.WriteLine("=== 다중 이벤트 테스트 ===");
            for (int i = 0; i < receivedMessages.Count; i++)
            {
                Console.WriteLine(string.Format("알림 {0}: {1}", i + 1, receivedMessages[i]));
            }
        }
        
        #endregion
        
        #region 연결 상태 이벤트 테스트
        
        /// <summary>
        /// 테스트 3: 연결 상태 변경 이벤트
        /// </summary>
        [Test]
        public void RaiseEvent_ConnectionStatusChanged_TracksStatus()
        {
            // Arrange
            bool? lastStatus = null;
            int statusChangeCount = 0;
            
            _mockNotification.Object.ConnectionStatusChanged += (sender, isConnected) =>
            {
                lastStatus = isConnected;
                statusChangeCount++;
                Console.WriteLine(string.Format("상태 변경 {0}: {1}", 
                    statusChangeCount, 
                    isConnected ? "연결됨" : "연결 해제됨"));
            };
            
            // Act - 연결 → 연결 해제 → 다시 연결
            _mockNotification.Raise(
                x => x.ConnectionStatusChanged += null,
                _mockNotification.Object,
                true);  // 연결됨
            
            _mockNotification.Raise(
                x => x.ConnectionStatusChanged += null,
                _mockNotification.Object,
                false);  // 연결 해제됨
            
            _mockNotification.Raise(
                x => x.ConnectionStatusChanged += null,
                _mockNotification.Object,
                true);  // 다시 연결됨
            
            // Assert
            Assert.AreEqual(3, statusChangeCount);
            Assert.IsTrue(lastStatus.Value);
            
            Console.WriteLine("=== 연결 상태 이벤트 테스트 ===");
            Console.WriteLine("총 상태 변경 횟수: " + statusChangeCount);
            Console.WriteLine("최종 상태: " + (lastStatus.Value ? "연결됨" : "연결 해제됨"));
        }
        
        #endregion
        
        #region 메서드 호출 시 이벤트 발생
        
        /// <summary>
        /// 테스트 4: 메서드 호출 시 자동으로 이벤트 발생
        /// </summary>
        [Test]
        public void Connect_RaisesConnectionEvent()
        {
            // Arrange
            bool connectionEventRaised = false;
            bool isConnected = false;
            
            _mockNotification.Object.ConnectionStatusChanged += (sender, status) =>
            {
                connectionEventRaised = true;
                isConnected = status;
            };
            
            // Connect 호출 시 이벤트 발생하도록 설정
            _mockNotification
                .Setup(x => x.Connect())
                .Raises(
                    x => x.ConnectionStatusChanged += null,
                    _mockNotification.Object,
                    true);
            
            // Act
            _mockNotification.Object.Connect();
            
            // Assert
            Assert.IsTrue(connectionEventRaised, "이벤트가 발생해야 합니다");
            Assert.IsTrue(isConnected, "연결 상태가 true여야 합니다");
            
            Console.WriteLine("=== 메서드 호출 시 이벤트 발생 ===");
            Console.WriteLine("Connect() 호출 → ConnectionStatusChanged 이벤트 발생");
            Console.WriteLine("연결 상태: " + isConnected);
        }
        
        /// <summary>
        /// 테스트 5: Disconnect 호출 시 이벤트 발생
        /// </summary>
        [Test]
        public void Disconnect_RaisesConnectionEvent()
        {
            // Arrange
            bool? connectionStatus = null;
            
            _mockNotification.Object.ConnectionStatusChanged += (sender, status) =>
            {
                connectionStatus = status;
            };
            
            // Disconnect 호출 시 이벤트 발생
            _mockNotification
                .Setup(x => x.Disconnect())
                .Raises(
                    x => x.ConnectionStatusChanged += null,
                    _mockNotification.Object,
                    false);
            
            // Act
            _mockNotification.Object.Disconnect();
            
            // Assert
            Assert.IsFalse(connectionStatus.Value);
            
            Console.WriteLine("=== Disconnect 이벤트 테스트 ===");
            Console.WriteLine("Disconnect() 호출 → 연결 해제됨");
        }
        
        #endregion
        
        #region 여러 핸들러 테스트
        
        /// <summary>
        /// 테스트 6: 여러 이벤트 핸들러
        /// </summary>
        [Test]
        public void RaiseEvent_MultipleHandlers_AllCalled()
        {
            // Arrange
            bool handler1Called = false;
            bool handler2Called = false;
            bool handler3Called = false;
            
            // 여러 핸들러 등록
            _mockNotification.Object.NotificationReceived += (s, m) => 
            {
                handler1Called = true;
                Console.WriteLine("핸들러 1 호출됨");
            };
            
            _mockNotification.Object.NotificationReceived += (s, m) => 
            {
                handler2Called = true;
                Console.WriteLine("핸들러 2 호출됨");
            };
            
            _mockNotification.Object.NotificationReceived += (s, m) => 
            {
                handler3Called = true;
                Console.WriteLine("핸들러 3 호출됨");
            };
            
            // Act
            _mockNotification.Raise(
                x => x.NotificationReceived += null,
                _mockNotification.Object,
                "테스트 메시지");
            
            // Assert
            Assert.IsTrue(handler1Called, "핸들러 1이 호출되어야 합니다");
            Assert.IsTrue(handler2Called, "핸들러 2가 호출되어야 합니다");
            Assert.IsTrue(handler3Called, "핸들러 3이 호출되어야 합니다");
            
            Console.WriteLine("=== 다중 핸들러 테스트 ===");
            Console.WriteLine("모든 핸들러가 호출됨!");
        }
        
        #endregion
        
        #region 이벤트 핸들러 제거 테스트
        
        /// <summary>
        /// 테스트 7: 이벤트 핸들러 등록/제거
        /// </summary>
        [Test]
        public void EventHandler_AddRemove_Works()
        {
            // Arrange
            int callCount = 0;
            EventHandler<string> handler = (s, m) => 
            {
                callCount++;
                Console.WriteLine("핸들러 호출됨 (카운트: " + callCount + ")");
            };
            
            // 핸들러 등록
            _mockNotification.Object.NotificationReceived += handler;
            
            // Act - 첫 번째 이벤트 발생
            _mockNotification.Raise(
                x => x.NotificationReceived += null,
                _mockNotification.Object,
                "메시지 1");
            
            Assert.AreEqual(1, callCount);
            
            // 핸들러 제거
            _mockNotification.Object.NotificationReceived -= handler;
            
            // 두 번째 이벤트 발생 (핸들러 제거됨)
            _mockNotification.Raise(
                x => x.NotificationReceived += null,
                _mockNotification.Object,
                "메시지 2");
            
            // Assert - 카운트가 증가하지 않아야 함
            Assert.AreEqual(1, callCount, "핸들러 제거 후에는 호출되지 않아야 합니다");
            
            Console.WriteLine("=== 핸들러 제거 테스트 ===");
            Console.WriteLine("핸들러 제거 후 이벤트 발생: 호출 안됨");
            Console.WriteLine("최종 호출 횟수: " + callCount);
        }
        
        #endregion
    }
}
```

### Step 6.2: Events Mock 핵심 포인트

```
┌────────────────────────────────────────────────────────────────┐
│  Events Mock 핵심 포인트                                       │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. 이벤트 발생 (Raise)                                        │
│     mockService.Raise(                                         │
│         x => x.EventName += null,  // 이벤트 지정             │
│         sender,                     // 보낸 객체              │
│         eventArgs                   // 이벤트 인자            │
│     );                                                         │
│                                                                │
│  2. 메서드 호출 시 이벤트 발생 (Raises)                        │
│     mockService                                                │
│         .Setup(x => x.Method())                                │
│         .Raises(x => x.EventName += null, sender, args);       │
│                                                                │
│  3. 핸들러 등록/제거                                           │
│     mockService.Object.EventName += handler;  // 등록         │
│     mockService.Object.EventName -= handler;  // 제거         │
│                                                                │
│  주의사항:                                                     │
│  • += null 형태로 이벤트를 지정                               │
│  • sender는 보통 mockService.Object 사용                      │
│  • EventArgs 타입은 이벤트 정의와 일치해야 함                 │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 7. 전체 테스트 실행

### Step 7.1: 테스트 실행

```
1. 빌드 → 솔루션 빌드 (Ctrl + Shift + B)
2. 테스트 → 테스트 탐색기 (Ctrl + E, T)
3. "모두 실행" 클릭
```

### Step 7.2: 예상 결과

```
┌────────────────────────────────────────────────────────────────┐
│  테스트 탐색기                                                 │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ✅ SequenceTests (7개)                                        │
│  ├── ✅ GetById_FirstCallReturnsNull_SecondCallReturnsUser    │
│  ├── ✅ GetById_ThreeSequentialCalls_ReturnsDifferentValues   │
│  ├── ✅ SendEmail_FirstThrowsException_SecondSucceeds         │
│  ├── ✅ SendEmail_TwoFailuresThenSuccess                      │
│  ├── ✅ RetryService_RetriesUntilSuccess                      │
│  ├── ✅ UserService_GetUserWithRetry_SucceedsOnThirdAttempt   │
│  └── ✅ Paging_ReturnsDataThenEmptyList                       │
│                                                                │
│  ✅ ProtectedTests (6개)                                       │
│  ├── ✅ ProcessInput_ValidateInputReturnsTrue_ReturnsTrue     │
│  ├── ✅ ProcessInput_ValidateInputReturnsFalse_ReturnsFalse   │
│  ├── ✅ ProcessInput_SpecificParameter_ReturnsDifferentResults│
│  ├── ✅ ProcessInput_VerifyProtectedMethodCalled              │
│  ├── ✅ ProcessData_TransformDataMocked_ReturnsTransformedVal │
│  └── ✅ ProcessData_TransformDataReturnsNull_ReturnsNull      │
│                                                                │
│  ✅ PropertyTests (7개)                                        │
│  ├── ✅ ReadOnlyProperty_Setup_ReturnsValue                   │
│  ├── ✅ SetupProperty_GetAndSet_Works                         │
│  ├── ✅ SetupProperty_NoInitialValue_DefaultsToTypeDefault    │
│  ├── ✅ SetupAllProperties_AllPropertiesTrackable             │
│  ├── ✅ UserService_UsesConfigurationProperties               │
│  ├── ✅ UserService_IsEnabledFalse_ReturnsNull                │
│  └── ✅ Property_MultipleChanges_TracksAllChanges             │
│                                                                │
│  ✅ EventTests (7개)                                           │
│  ├── ✅ RaiseEvent_NotificationReceived_HandlerCalled         │
│  ├── ✅ RaiseEvent_MultipleTimes_AllHandled                   │
│  ├── ✅ RaiseEvent_ConnectionStatusChanged_TracksStatus       │
│  ├── ✅ Connect_RaisesConnectionEvent                         │
│  ├── ✅ Disconnect_RaisesConnectionEvent                      │
│  ├── ✅ RaiseEvent_MultipleHandlers_AllCalled                 │
│  └── ✅ EventHandler_AddRemove_Works                          │
│                                                                │
│  통과: 27  실패: 0  건너뜀: 0                                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 8. 핵심 요약

### Step 8.1: 4가지 고급 기능 비교

```
┌────────────────────────────────────────────────────────────────┐
│  Moq 고급 기능 요약                                            │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. SetupSequence (순차 반환)                                  │
│  ─────────────────────────────────────────────────────────────│
│  용도: 호출할 때마다 다른 값 반환                             │
│  문법: .SetupSequence(x => x.Method())                        │
│        .Returns(값1)                                          │
│        .Returns(값2)                                          │
│        .Throws(예외)                                          │
│  활용: 재시도 로직, 페이징, 상태 변화 테스트                  │
│                                                                │
│  2. Protected Mock                                             │
│  ─────────────────────────────────────────────────────────────│
│  용도: 추상 클래스의 protected 메서드 Mock                    │
│  문법: .Protected()                                           │
│        .Setup<반환타입>("메서드명", ItExpr.IsAny<T>())        │
│        .Returns(값)                                           │
│  주의: ItExpr 사용 (It 대신), using Moq.Protected 필요       │
│                                                                │
│  3. Properties Mock                                            │
│  ─────────────────────────────────────────────────────────────│
│  용도: 속성 Get/Set Mock                                      │
│  문법:                                                        │
│  • Setup: 읽기 전용                                           │
│  • SetupProperty: 개별 속성 Get/Set                           │
│  • SetupAllProperties: 모든 속성 Get/Set                      │
│                                                                │
│  4. Events Mock                                                │
│  ─────────────────────────────────────────────────────────────│
│  용도: 이벤트 발생 시뮬레이션                                 │
│  문법: .Raise(x => x.Event += null, sender, args)             │
│  추가: .Raises() - 메서드 호출 시 이벤트 자동 발생           │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 9. 자주 하는 실수

```
┌────────────────────────────────────────────────────────────────┐
│  자주 하는 실수와 해결 방법                                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ❌ 실수 1: Protected에서 It 사용                              │
│     .Setup<bool>("Method", It.IsAny<string>())  // 에러!      │
│  ✅ 해결: ItExpr 사용                                          │
│     .Setup<bool>("Method", ItExpr.IsAny<string>())            │
│                                                                │
│  ❌ 실수 2: SetupSequence 후 추가 호출                         │
│     4번째 호출 시 기본값(null/0) 반환                         │
│  ✅ 해결: 필요한 만큼 Returns 추가 또는 마지막에 기본 Setup   │
│                                                                │
│  ❌ 실수 3: 이벤트 Raise 문법 오류                             │
│     .Raise(x => x.Event, sender, args)  // += null 누락       │
│  ✅ 해결: += null 포함                                         │
│     .Raise(x => x.Event += null, sender, args)                │
│                                                                │
│  ❌ 실수 4: SetupProperty 없이 Set 시도                        │
│     mock.Object.Property = value;  // 값이 저장 안됨          │
│  ✅ 해결: SetupProperty 또는 SetupAllProperties 사용          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

**수고하셨습니다! 🎉**

이제 Moq의 고급 기능을 마스터했습니다:
- **SetupSequence**: 재시도 로직 테스트
- **Protected**: 추상 클래스 테스트
- **Properties**: 설정 값 테스트
- **Events**: 이벤트 기반 코드 테스트
