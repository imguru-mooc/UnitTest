# Day 1: 단위테스트 개요 및 기본 실습

## 📋 학습 목표
- 단위테스트의 필요성과 목적을 이해한다
- TDD 개념과 FIRST 원칙을 학습한다
- NUnit/xUnit 프레임워크로 테스트 코드를 작성한다
- Mock, Stub, Fake를 활용한 외부 의존성 격리를 실습한다

---

## 1. 단위테스트 개요

### 1.1 단위테스트란?
단위테스트(Unit Test)는 소프트웨어의 가장 작은 단위(함수, 메서드, 클래스)가 의도한 대로 동작하는지 검증하는 테스트입니다.

### 1.2 단위테스트의 필요성
- **버그 조기 발견**: 개발 초기 단계에서 결함 발견
- **리팩토링 안전망**: 코드 변경 시 기존 기능 보장
- **문서화 역할**: 코드의 사용법과 의도를 명시
- **설계 개선**: 테스트하기 쉬운 코드는 좋은 설계

### 1.3 FIRST 원칙

| 원칙 | 설명 |
|------|------|
| **F**ast | 테스트는 빠르게 실행되어야 함 |
| **I**ndependent | 테스트 간 독립성 보장 |
| **R**epeatable | 어떤 환경에서든 동일한 결과 |
| **S**elf-Validating | 성공/실패를 명확하게 판단 |
| **T**imely | 적절한 시점에 작성 (TDD) |

---

## 2. 환경 설정

### 2.1 프로젝트 생성

```bash
# 솔루션 생성
dotnet new sln -n UnitTestLab

# 프로덕션 코드 프로젝트
dotnet new classlib -n Calculator.Core
dotnet sln add Calculator.Core

# NUnit 테스트 프로젝트
dotnet new nunit -n Calculator.Tests
dotnet sln add Calculator.Tests

# 프로젝트 참조 추가
dotnet add Calculator.Tests reference Calculator.Core

# Moq 라이브러리 설치
dotnet add Calculator.Tests package Moq
```

### 2.2 프로젝트 구조

```
UnitTestLab/
├── Calculator.Core/
│   ├── Calculator.cs
│   ├── IDataService.cs
│   └── OrderProcessor.cs
├── Calculator.Tests/
│   ├── CalculatorTests.cs
│   ├── CalculatorAdvancedTests.cs
│   └── OrderProcessorTests.cs
└── UnitTestLab.sln
```

---

## 3. 실습 1: 기본 단위테스트 작성

### 3.1 Calculator 클래스 구현

**Calculator.Core/Calculator.cs**
```csharp
namespace Calculator.Core
{
    public class Calculator
    {
        public int Add(int a, int b)
        {
            return a + b;
        }

        public int Subtract(int a, int b)
        {
            return a - b;
        }

        public int Multiply(int a, int b)
        {
            return a * b;
        }

        public double Divide(int a, int b)
        {
            if (b == 0)
                throw new DivideByZeroException("0으로 나눌 수 없습니다.");
            
            return (double)a / b;
        }

        public bool IsPositive(int number)
        {
            return number > 0;
        }

        public int Factorial(int n)
        {
            if (n < 0)
                throw new ArgumentException("음수의 팩토리얼은 정의되지 않습니다.");
            
            if (n <= 1)
                return 1;
            
            return n * Factorial(n - 1);
        }
    }
}
```

### 3.2 NUnit 테스트 코드 작성

**Calculator.Tests/CalculatorTests.cs**
```csharp
using NUnit.Framework;
using Calculator.Core;

namespace Calculator.Tests
{
    [TestFixture]
    public class CalculatorTests
    {
        private Calculator.Core.Calculator _calculator;

        [SetUp]
        public void Setup()
        {
            // 각 테스트 전에 실행 - 테스트 격리 보장
            _calculator = new Calculator.Core.Calculator();
        }

        #region Add 메서드 테스트

        [Test]
        public void Add_TwoPositiveNumbers_ReturnsCorrectSum()
        {
            // Arrange (준비)
            int a = 5;
            int b = 3;

            // Act (실행)
            int result = _calculator.Add(a, b);

            // Assert (검증)
            Assert.That(result, Is.EqualTo(8));
        }

        [Test]
        public void Add_PositiveAndNegative_ReturnsCorrectSum()
        {
            // Arrange
            int a = 10;
            int b = -3;

            // Act
            int result = _calculator.Add(a, b);

            // Assert
            Assert.That(result, Is.EqualTo(7));
        }

        [Test]
        public void Add_TwoNegativeNumbers_ReturnsCorrectSum()
        {
            // Arrange
            int a = -5;
            int b = -3;

            // Act
            int result = _calculator.Add(a, b);

            // Assert
            Assert.That(result, Is.EqualTo(-8));
        }

        // TestCase를 활용한 파라미터화 테스트
        [TestCase(1, 1, 2)]
        [TestCase(0, 0, 0)]
        [TestCase(-1, 1, 0)]
        [TestCase(100, 200, 300)]
        [TestCase(int.MaxValue - 1, 1, int.MaxValue)]
        public void Add_VariousInputs_ReturnsExpectedResult(int a, int b, int expected)
        {
            // Act
            int result = _calculator.Add(a, b);

            // Assert
            Assert.That(result, Is.EqualTo(expected));
        }

        #endregion

        #region Subtract 메서드 테스트

        [Test]
        public void Subtract_TwoPositiveNumbers_ReturnsCorrectDifference()
        {
            // Arrange & Act
            int result = _calculator.Subtract(10, 4);

            // Assert
            Assert.That(result, Is.EqualTo(6));
        }

        [TestCase(10, 5, 5)]
        [TestCase(5, 10, -5)]
        [TestCase(0, 0, 0)]
        [TestCase(-5, -3, -2)]
        public void Subtract_VariousInputs_ReturnsExpectedResult(int a, int b, int expected)
        {
            Assert.That(_calculator.Subtract(a, b), Is.EqualTo(expected));
        }

        #endregion

        #region Multiply 메서드 테스트

        [TestCase(2, 3, 6)]
        [TestCase(0, 100, 0)]
        [TestCase(-2, 3, -6)]
        [TestCase(-2, -3, 6)]
        public void Multiply_VariousInputs_ReturnsExpectedResult(int a, int b, int expected)
        {
            Assert.That(_calculator.Multiply(a, b), Is.EqualTo(expected));
        }

        #endregion

        #region Divide 메서드 테스트

        [Test]
        public void Divide_ValidNumbers_ReturnsCorrectQuotient()
        {
            // Arrange & Act
            double result = _calculator.Divide(10, 2);

            // Assert
            Assert.That(result, Is.EqualTo(5.0));
        }

        [Test]
        public void Divide_ResultWithDecimal_ReturnsCorrectQuotient()
        {
            // Act
            double result = _calculator.Divide(7, 2);

            // Assert - 소수점 결과 검증
            Assert.That(result, Is.EqualTo(3.5));
        }

        [Test]
        public void Divide_ByZero_ThrowsDivideByZeroException()
        {
            // Assert - 예외 발생 검증
            Assert.Throws<DivideByZeroException>(() => _calculator.Divide(10, 0));
        }

        [Test]
        public void Divide_ByZero_ThrowsExceptionWithCorrectMessage()
        {
            // Act & Assert
            var exception = Assert.Throws<DivideByZeroException>(
                () => _calculator.Divide(10, 0));
            
            Assert.That(exception.Message, Is.EqualTo("0으로 나눌 수 없습니다."));
        }

        #endregion

        #region IsPositive 메서드 테스트

        [Test]
        public void IsPositive_PositiveNumber_ReturnsTrue()
        {
            Assert.That(_calculator.IsPositive(5), Is.True);
        }

        [Test]
        public void IsPositive_NegativeNumber_ReturnsFalse()
        {
            Assert.That(_calculator.IsPositive(-5), Is.False);
        }

        [Test]
        public void IsPositive_Zero_ReturnsFalse()
        {
            Assert.That(_calculator.IsPositive(0), Is.False);
        }

        #endregion

        #region Factorial 메서드 테스트

        [TestCase(0, 1)]
        [TestCase(1, 1)]
        [TestCase(5, 120)]
        [TestCase(10, 3628800)]
        public void Factorial_ValidInput_ReturnsCorrectResult(int n, int expected)
        {
            Assert.That(_calculator.Factorial(n), Is.EqualTo(expected));
        }

        [Test]
        public void Factorial_NegativeNumber_ThrowsArgumentException()
        {
            Assert.Throws<ArgumentException>(() => _calculator.Factorial(-1));
        }

        #endregion

        [TearDown]
        public void TearDown()
        {
            // 각 테스트 후 정리 작업
            _calculator = null;
        }
    }
}
```

### 3.3 테스트 실행

```bash
# 모든 테스트 실행
dotnet test

# 상세 결과 출력
dotnet test --logger "console;verbosity=detailed"

# 특정 테스트만 실행
dotnet test --filter "FullyQualifiedName~Add"
```

---

## 4. 실습 2: Assert 메서드 심화

### 4.1 다양한 Assert 메서드

**Calculator.Tests/CalculatorAdvancedTests.cs**
```csharp
using NUnit.Framework;
using Calculator.Core;
using System.Collections.Generic;

namespace Calculator.Tests
{
    [TestFixture]
    public class CalculatorAdvancedTests
    {
        private Calculator.Core.Calculator _calculator;

        [SetUp]
        public void Setup()
        {
            _calculator = new Calculator.Core.Calculator();
        }

        #region 기본 Assert 메서드들

        [Test]
        public void Assert_Equality_Examples()
        {
            // 기본 동등성 검사
            Assert.That(5, Is.EqualTo(5));
            Assert.That("hello", Is.EqualTo("hello"));
            
            // 대소문자 무시
            Assert.That("HELLO", Is.EqualTo("hello").IgnoreCase);
        }

        [Test]
        public void Assert_Comparison_Examples()
        {
            int value = 10;

            Assert.That(value, Is.GreaterThan(5));
            Assert.That(value, Is.GreaterThanOrEqualTo(10));
            Assert.That(value, Is.LessThan(20));
            Assert.That(value, Is.LessThanOrEqualTo(10));
            Assert.That(value, Is.InRange(5, 15));
        }

        [Test]
        public void Assert_Null_Examples()
        {
            string nullString = null;
            string notNullString = "hello";

            Assert.That(nullString, Is.Null);
            Assert.That(notNullString, Is.Not.Null);
        }

        [Test]
        public void Assert_Boolean_Examples()
        {
            bool trueValue = true;
            bool falseValue = false;

            Assert.That(trueValue, Is.True);
            Assert.That(falseValue, Is.False);
        }

        #endregion

        #region 컬렉션 Assert

        [Test]
        public void Assert_Collection_Examples()
        {
            var numbers = new List<int> { 1, 2, 3, 4, 5 };
            var emptyList = new List<int>();

            // 컬렉션 포함 여부
            Assert.That(numbers, Does.Contain(3));
            Assert.That(numbers, Does.Not.Contain(10));

            // 컬렉션 크기
            Assert.That(numbers, Has.Count.EqualTo(5));
            Assert.That(emptyList, Is.Empty);
            Assert.That(numbers, Is.Not.Empty);

            // 모든 요소 검증
            Assert.That(numbers, Is.All.GreaterThan(0));
            Assert.That(numbers, Is.All.LessThan(10));

            // 정렬 검증
            Assert.That(numbers, Is.Ordered);

            // 고유성 검증
            Assert.That(numbers, Is.Unique);
        }

        [Test]
        public void Assert_CollectionEquality_Examples()
        {
            var list1 = new List<int> { 1, 2, 3 };
            var list2 = new List<int> { 1, 2, 3 };
            var list3 = new List<int> { 3, 2, 1 };

            // 동일 순서, 동일 요소
            Assert.That(list1, Is.EqualTo(list2));

            // 순서 무시하고 동일 요소
            Assert.That(list1, Is.EquivalentTo(list3));
        }

        #endregion

        #region 문자열 Assert

        [Test]
        public void Assert_String_Examples()
        {
            string text = "Hello, World!";

            // 포함 여부
            Assert.That(text, Does.Contain("World"));
            Assert.That(text, Does.Not.Contain("Goodbye"));

            // 시작/끝 검사
            Assert.That(text, Does.StartWith("Hello"));
            Assert.That(text, Does.EndWith("!"));

            // 정규식 매칭
            Assert.That(text, Does.Match("Hello.*World"));

            // 길이 검사
            Assert.That(text, Has.Length.GreaterThan(5));
        }

        #endregion

        #region 예외 Assert

        [Test]
        public void Assert_Exception_Examples()
        {
            // 특정 예외 타입 검증
            Assert.Throws<DivideByZeroException>(
                () => _calculator.Divide(10, 0));

            // 예외 메시지 검증
            var ex = Assert.Throws<DivideByZeroException>(
                () => _calculator.Divide(10, 0));
            Assert.That(ex.Message, Does.Contain("0으로"));

            // 예외가 발생하지 않음을 검증
            Assert.DoesNotThrow(() => _calculator.Divide(10, 2));
        }

        #endregion

        #region 부동소수점 Assert

        [Test]
        public void Assert_FloatingPoint_Examples()
        {
            double result = 0.1 + 0.2;

            // 부동소수점 비교 - 허용 오차 사용
            Assert.That(result, Is.EqualTo(0.3).Within(0.0001));

            // 퍼센트 허용 오차
            Assert.That(result, Is.EqualTo(0.3).Within(1).Percent);
        }

        #endregion

        #region 복합 조건

        [Test]
        public void Assert_Complex_Conditions()
        {
            int value = 15;

            // AND 조건
            Assert.That(value, Is.GreaterThan(10).And.LessThan(20));

            // OR 조건
            Assert.That(value, Is.LessThan(10).Or.GreaterThan(12));
        }

        #endregion
    }
}
```

---

## 5. 실습 3: Mock, Stub, Fake 활용

### 5.1 개념 이해

| 구분 | 설명 | 사용 시점 |
|------|------|----------|
| **Stub** | 미리 정의된 값을 반환 | 입력에 대한 고정 응답 필요 시 |
| **Mock** | 호출 여부와 방식을 검증 | 상호작용 검증이 필요 시 |
| **Fake** | 실제 동작하는 간단한 구현체 | 가벼운 대체 구현 필요 시 |

### 5.2 인터페이스 정의

**Calculator.Core/IDataService.cs**
```csharp
namespace Calculator.Core
{
    public interface IDataService
    {
        int GetValue(string key);
        void SaveValue(string key, int value);
        bool IsConnected();
        List<int> GetAllValues();
    }

    public interface IEmailService
    {
        bool SendEmail(string to, string subject, string body);
        int GetUnreadCount();
    }

    public interface ILogger
    {
        void Log(string message);
        void LogError(string message, Exception ex);
    }
}
```

### 5.3 OrderProcessor 클래스

**Calculator.Core/OrderProcessor.cs**
```csharp
namespace Calculator.Core
{
    public class Order
    {
        public int Id { get; set; }
        public string CustomerEmail { get; set; }
        public decimal Amount { get; set; }
        public string Status { get; set; }
    }

    public class OrderProcessor
    {
        private readonly IDataService _dataService;
        private readonly IEmailService _emailService;
        private readonly ILogger _logger;

        public OrderProcessor(
            IDataService dataService, 
            IEmailService emailService,
            ILogger logger)
        {
            _dataService = dataService;
            _emailService = emailService;
            _logger = logger;
        }

        public bool ProcessOrder(Order order)
        {
            try
            {
                // 연결 상태 확인
                if (!_dataService.IsConnected())
                {
                    _logger.LogError("데이터 서비스 연결 실패", null);
                    return false;
                }

                // 주문 처리 로직
                var orderValue = _dataService.GetValue($"order_{order.Id}");
                if (orderValue > 0)
                {
                    // 이미 처리된 주문
                    return false;
                }

                // 주문 저장
                _dataService.SaveValue($"order_{order.Id}", (int)order.Amount);
                order.Status = "Processed";

                // 확인 이메일 발송
                _emailService.SendEmail(
                    order.CustomerEmail,
                    "주문 확인",
                    $"주문 #{order.Id}이 처리되었습니다. 금액: {order.Amount:C}");

                _logger.Log($"주문 {order.Id} 처리 완료");
                return true;
            }
            catch (Exception ex)
            {
                _logger.LogError($"주문 처리 실패: {order.Id}", ex);
                return false;
            }
        }

        public int CalculateTotal()
        {
            var values = _dataService.GetAllValues();
            return values?.Sum() ?? 0;
        }
    }
}
```

### 5.4 Moq를 활용한 테스트

**Calculator.Tests/OrderProcessorTests.cs**
```csharp
using NUnit.Framework;
using Moq;
using Calculator.Core;

namespace Calculator.Tests
{
    [TestFixture]
    public class OrderProcessorTests
    {
        private Mock<IDataService> _mockDataService;
        private Mock<IEmailService> _mockEmailService;
        private Mock<ILogger> _mockLogger;
        private OrderProcessor _processor;

        [SetUp]
        public void Setup()
        {
            // Mock 객체 생성
            _mockDataService = new Mock<IDataService>();
            _mockEmailService = new Mock<IEmailService>();
            _mockLogger = new Mock<ILogger>();

            // 테스트 대상 생성 (의존성 주입)
            _processor = new OrderProcessor(
                _mockDataService.Object,
                _mockEmailService.Object,
                _mockLogger.Object);
        }

        #region Stub 예제 - 고정 반환값 설정

        [Test]
        public void ProcessOrder_WhenNotConnected_ReturnsFalse()
        {
            // Arrange - Stub 설정: IsConnected가 false 반환
            _mockDataService.Setup(x => x.IsConnected()).Returns(false);

            var order = new Order 
            { 
                Id = 1, 
                CustomerEmail = "test@test.com", 
                Amount = 100m 
            };

            // Act
            bool result = _processor.ProcessOrder(order);

            // Assert
            Assert.That(result, Is.False);
        }

        [Test]
        public void ProcessOrder_WhenConnected_ReturnsTrue()
        {
            // Arrange - 여러 Stub 설정
            _mockDataService.Setup(x => x.IsConnected()).Returns(true);
            _mockDataService.Setup(x => x.GetValue(It.IsAny<string>())).Returns(0);
            _mockEmailService.Setup(x => x.SendEmail(
                It.IsAny<string>(), 
                It.IsAny<string>(), 
                It.IsAny<string>())).Returns(true);

            var order = new Order 
            { 
                Id = 1, 
                CustomerEmail = "test@test.com", 
                Amount = 100m 
            };

            // Act
            bool result = _processor.ProcessOrder(order);

            // Assert
            Assert.That(result, Is.True);
            Assert.That(order.Status, Is.EqualTo("Processed"));
        }

        [Test]
        public void ProcessOrder_WhenAlreadyProcessed_ReturnsFalse()
        {
            // Arrange - 이미 처리된 주문 시나리오
            _mockDataService.Setup(x => x.IsConnected()).Returns(true);
            _mockDataService.Setup(x => x.GetValue("order_1")).Returns(100); // 이미 값 존재

            var order = new Order { Id = 1, CustomerEmail = "test@test.com", Amount = 100m };

            // Act
            bool result = _processor.ProcessOrder(order);

            // Assert
            Assert.That(result, Is.False);
        }

        #endregion

        #region Mock 예제 - 호출 검증

        [Test]
        public void ProcessOrder_WhenSuccess_SavesOrderValue()
        {
            // Arrange
            _mockDataService.Setup(x => x.IsConnected()).Returns(true);
            _mockDataService.Setup(x => x.GetValue(It.IsAny<string>())).Returns(0);

            var order = new Order { Id = 123, CustomerEmail = "test@test.com", Amount = 500m };

            // Act
            _processor.ProcessOrder(order);

            // Assert - SaveValue가 정확한 파라미터로 호출되었는지 검증
            _mockDataService.Verify(
                x => x.SaveValue("order_123", 500), 
                Times.Once());
        }

        [Test]
        public void ProcessOrder_WhenSuccess_SendsConfirmationEmail()
        {
            // Arrange
            _mockDataService.Setup(x => x.IsConnected()).Returns(true);
            _mockDataService.Setup(x => x.GetValue(It.IsAny<string>())).Returns(0);

            var order = new Order 
            { 
                Id = 1, 
                CustomerEmail = "customer@example.com", 
                Amount = 100m 
            };

            // Act
            _processor.ProcessOrder(order);

            // Assert - 이메일이 올바른 수신자에게 발송되었는지 검증
            _mockEmailService.Verify(
                x => x.SendEmail(
                    "customer@example.com",
                    "주문 확인",
                    It.Is<string>(body => body.Contains("주문 #1"))),
                Times.Once());
        }

        [Test]
        public void ProcessOrder_WhenNotConnected_LogsError()
        {
            // Arrange
            _mockDataService.Setup(x => x.IsConnected()).Returns(false);

            var order = new Order { Id = 1, CustomerEmail = "test@test.com", Amount = 100m };

            // Act
            _processor.ProcessOrder(order);

            // Assert - 에러 로그가 기록되었는지 검증
            _mockLogger.Verify(
                x => x.LogError("데이터 서비스 연결 실패", null),
                Times.Once());
        }

        [Test]
        public void ProcessOrder_WhenNotConnected_DoesNotSendEmail()
        {
            // Arrange
            _mockDataService.Setup(x => x.IsConnected()).Returns(false);

            var order = new Order { Id = 1, CustomerEmail = "test@test.com", Amount = 100m };

            // Act
            _processor.ProcessOrder(order);

            // Assert - 이메일이 발송되지 않았는지 검증
            _mockEmailService.Verify(
                x => x.SendEmail(It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()),
                Times.Never());
        }

        #endregion

        #region 고급 Mock 기능

        [Test]
        public void CalculateTotal_ReturnsSum()
        {
            // Arrange - 컬렉션 반환 Stub
            _mockDataService.Setup(x => x.GetAllValues())
                .Returns(new List<int> { 100, 200, 300 });

            // Act
            int total = _processor.CalculateTotal();

            // Assert
            Assert.That(total, Is.EqualTo(600));
        }

        [Test]
        public void CalculateTotal_WhenNull_ReturnsZero()
        {
            // Arrange
            _mockDataService.Setup(x => x.GetAllValues())
                .Returns((List<int>)null);

            // Act
            int total = _processor.CalculateTotal();

            // Assert
            Assert.That(total, Is.EqualTo(0));
        }

        [Test]
        public void ProcessOrder_SequentialCalls_ReturnsDifferentValues()
        {
            // Arrange - 순차적으로 다른 값 반환
            _mockDataService.Setup(x => x.IsConnected()).Returns(true);
            _mockDataService.SetupSequence(x => x.GetValue(It.IsAny<string>()))
                .Returns(0)   // 첫 번째 호출
                .Returns(100) // 두 번째 호출
                .Returns(200); // 세 번째 호출

            var order1 = new Order { Id = 1, CustomerEmail = "a@test.com", Amount = 100m };
            var order2 = new Order { Id = 2, CustomerEmail = "b@test.com", Amount = 200m };

            // Act & Assert
            Assert.That(_processor.ProcessOrder(order1), Is.True);  // 0 반환
            Assert.That(_processor.ProcessOrder(order2), Is.False); // 100 반환
        }

        [Test]
        public void ProcessOrder_ThrowsException_LogsAndReturnsFalse()
        {
            // Arrange - 예외 발생 시나리오
            _mockDataService.Setup(x => x.IsConnected()).Returns(true);
            _mockDataService.Setup(x => x.GetValue(It.IsAny<string>()))
                .Throws(new Exception("DB 연결 오류"));

            var order = new Order { Id = 1, CustomerEmail = "test@test.com", Amount = 100m };

            // Act
            bool result = _processor.ProcessOrder(order);

            // Assert
            Assert.That(result, Is.False);
            _mockLogger.Verify(
                x => x.LogError(It.IsAny<string>(), It.IsAny<Exception>()),
                Times.Once());
        }

        #endregion

        #region Callback을 활용한 고급 검증

        [Test]
        public void ProcessOrder_CapturesParameters()
        {
            // Arrange
            string capturedKey = null;
            int capturedValue = 0;

            _mockDataService.Setup(x => x.IsConnected()).Returns(true);
            _mockDataService.Setup(x => x.GetValue(It.IsAny<string>())).Returns(0);
            _mockDataService.Setup(x => x.SaveValue(It.IsAny<string>(), It.IsAny<int>()))
                .Callback<string, int>((key, value) => 
                {
                    capturedKey = key;
                    capturedValue = value;
                });

            var order = new Order { Id = 42, CustomerEmail = "test@test.com", Amount = 999m };

            // Act
            _processor.ProcessOrder(order);

            // Assert
            Assert.That(capturedKey, Is.EqualTo("order_42"));
            Assert.That(capturedValue, Is.EqualTo(999));
        }

        #endregion
    }
}
```

---

## 6. 실습 4: Fake 객체 구현

### 6.1 Fake 구현체

**Calculator.Tests/Fakes/FakeDataService.cs**
```csharp
using Calculator.Core;

namespace Calculator.Tests.Fakes
{
    /// <summary>
    /// 인메모리 딕셔너리를 사용하는 Fake 구현체
    /// 실제 데이터베이스 없이 동작 테스트 가능
    /// </summary>
    public class FakeDataService : IDataService
    {
        private readonly Dictionary<string, int> _data = new();
        private bool _isConnected = true;

        public void SetConnected(bool connected)
        {
            _isConnected = connected;
        }

        public bool IsConnected()
        {
            return _isConnected;
        }

        public int GetValue(string key)
        {
            return _data.TryGetValue(key, out int value) ? value : 0;
        }

        public void SaveValue(string key, int value)
        {
            _data[key] = value;
        }

        public List<int> GetAllValues()
        {
            return _data.Values.ToList();
        }

        // 테스트 헬퍼 메서드
        public void Clear()
        {
            _data.Clear();
        }

        public int Count => _data.Count;
    }

    public class FakeEmailService : IEmailService
    {
        public List<(string To, string Subject, string Body)> SentEmails { get; } = new();

        public bool SendEmail(string to, string subject, string body)
        {
            SentEmails.Add((to, subject, body));
            return true;
        }

        public int GetUnreadCount()
        {
            return 0;
        }
    }

    public class FakeLogger : ILogger
    {
        public List<string> Logs { get; } = new();
        public List<(string Message, Exception Ex)> Errors { get; } = new();

        public void Log(string message)
        {
            Logs.Add(message);
        }

        public void LogError(string message, Exception ex)
        {
            Errors.Add((message, ex));
        }
    }
}
```

### 6.2 Fake를 활용한 테스트

**Calculator.Tests/OrderProcessorFakeTests.cs**
```csharp
using NUnit.Framework;
using Calculator.Core;
using Calculator.Tests.Fakes;

namespace Calculator.Tests
{
    [TestFixture]
    public class OrderProcessorFakeTests
    {
        private FakeDataService _fakeDataService;
        private FakeEmailService _fakeEmailService;
        private FakeLogger _fakeLogger;
        private OrderProcessor _processor;

        [SetUp]
        public void Setup()
        {
            _fakeDataService = new FakeDataService();
            _fakeEmailService = new FakeEmailService();
            _fakeLogger = new FakeLogger();

            _processor = new OrderProcessor(
                _fakeDataService,
                _fakeEmailService,
                _fakeLogger);
        }

        [Test]
        public void ProcessOrder_StoresOrderInFakeDatabase()
        {
            // Arrange
            var order = new Order 
            { 
                Id = 1, 
                CustomerEmail = "test@test.com", 
                Amount = 500m 
            };

            // Act
            _processor.ProcessOrder(order);

            // Assert - Fake 데이터베이스에 저장 확인
            Assert.That(_fakeDataService.GetValue("order_1"), Is.EqualTo(500));
            Assert.That(_fakeDataService.Count, Is.EqualTo(1));
        }

        [Test]
        public void ProcessOrder_SendsEmailWithCorrectContent()
        {
            // Arrange
            var order = new Order 
            { 
                Id = 123, 
                CustomerEmail = "customer@example.com", 
                Amount = 1500m 
            };

            // Act
            _processor.ProcessOrder(order);

            // Assert - Fake 이메일 서비스에서 발송 내역 확인
            Assert.That(_fakeEmailService.SentEmails.Count, Is.EqualTo(1));
            
            var sentEmail = _fakeEmailService.SentEmails[0];
            Assert.That(sentEmail.To, Is.EqualTo("customer@example.com"));
            Assert.That(sentEmail.Subject, Is.EqualTo("주문 확인"));
            Assert.That(sentEmail.Body, Does.Contain("주문 #123"));
        }

        [Test]
        public void ProcessMultipleOrders_AllStoredCorrectly()
        {
            // Arrange
            var orders = new[]
            {
                new Order { Id = 1, CustomerEmail = "a@test.com", Amount = 100m },
                new Order { Id = 2, CustomerEmail = "b@test.com", Amount = 200m },
                new Order { Id = 3, CustomerEmail = "c@test.com", Amount = 300m }
            };

            // Act
            foreach (var order in orders)
            {
                _processor.ProcessOrder(order);
            }

            // Assert
            Assert.That(_fakeDataService.Count, Is.EqualTo(3));
            Assert.That(_fakeEmailService.SentEmails.Count, Is.EqualTo(3));
            Assert.That(_processor.CalculateTotal(), Is.EqualTo(600));
        }

        [Test]
        public void ProcessOrder_WhenDisconnected_LogsError()
        {
            // Arrange
            _fakeDataService.SetConnected(false);
            var order = new Order { Id = 1, CustomerEmail = "test@test.com", Amount = 100m };

            // Act
            _processor.ProcessOrder(order);

            // Assert
            Assert.That(_fakeLogger.Errors.Count, Is.EqualTo(1));
            Assert.That(_fakeLogger.Errors[0].Message, Does.Contain("연결 실패"));
        }
    }
}
```

---

## 7. 연습 문제

### 문제 1: StringCalculator 테스트
다음 클래스의 테스트를 작성하세요:

```csharp
public class StringCalculator
{
    public int Add(string numbers)
    {
        if (string.IsNullOrEmpty(numbers))
            return 0;
        
        var nums = numbers.Split(',');
        return nums.Sum(n => int.Parse(n.Trim()));
    }
}
```

테스트 케이스:
- 빈 문자열 → 0
- "1" → 1
- "1, 2" → 3
- "1, 2, 3, 4, 5" → 15

### 문제 2: UserService Mock 테스트
IUserRepository를 주입받는 UserService의 테스트를 Mock을 활용해 작성하세요.

```csharp
public interface IUserRepository
{
    User GetById(int id);
    bool Save(User user);
}

public class UserService
{
    private readonly IUserRepository _repository;
    
    public UserService(IUserRepository repository)
    {
        _repository = repository;
    }
    
    public string GetUserName(int id)
    {
        var user = _repository.GetById(id);
        return user?.Name ?? "Unknown";
    }
}
```

---

## 8. 핵심 정리

### AAA 패턴
```
Arrange → Act → Assert
(준비)    (실행)  (검증)
```

### Mock vs Stub vs Fake

| 구분 | 목적 | 특징 |
|------|------|------|
| Stub | 테스트 입력 제공 | 미리 정의된 값 반환 |
| Mock | 동작 검증 | 호출 여부/횟수/파라미터 검증 |
| Fake | 실제 동작 대체 | 간단한 인메모리 구현 |

### 테스트 명명 규칙
```
[테스트대상]_[시나리오]_[기대결과]
예: Add_TwoPositiveNumbers_ReturnsCorrectSum
```

---

## 📝 다음 시간 예고
- **2일차**: TDD 실전 적용 (Red-Green-Refactor)
- 코드 커버리지 분석
- 예외 처리 및 경계값 테스트
