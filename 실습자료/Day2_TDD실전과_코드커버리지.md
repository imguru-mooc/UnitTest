# Day 2: TDD 실전 적용 및 코드 커버리지

## 📋 학습 목표
- TDD의 Red-Green-Refactor 사이클을 체득한다
- 실패하는 테스트 작성부터 리팩토링까지 전 과정을 실습한다
- Visual Studio 코드 커버리지 분석 도구를 활용한다
- 예외 처리 및 경계값 테스트 작성법을 익힌다

---

## 1. TDD (Test-Driven Development) 개요

### 1.1 TDD란?
테스트 주도 개발은 **테스트를 먼저 작성**하고, 그 테스트를 통과하는 코드를 구현하는 개발 방법론입니다.

### 1.2 Red-Green-Refactor 사이클

```
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│     🔴 RED          →        🟢 GREEN        →    🔵 REFACTOR │
│   (실패하는              (테스트 통과하는         (코드 개선)    │
│    테스트 작성)           최소 코드 작성)                       │
│         ↑                                             │       │
│         └─────────────────────────────────────────────┘       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

| 단계 | 설명 | 핵심 원칙 |
|------|------|----------|
| 🔴 Red | 실패하는 테스트 작성 | 아직 구현되지 않은 기능 테스트 |
| 🟢 Green | 테스트 통과하는 최소 코드 | 가장 간단한 방법으로 통과 |
| 🔵 Refactor | 코드 품질 개선 | 테스트 통과 유지하며 리팩토링 |

### 1.3 TDD의 장점
- 설계 품질 향상
- 버그 조기 발견
- 안전한 리팩토링
- 실행 가능한 문서

---

## 2. 실습 1: TDD로 PasswordValidator 구현

### 2.1 요구사항 정의

비밀번호 유효성 검증기 구현:
1. 최소 8자 이상
2. 대문자 최소 1개 포함
3. 소문자 최소 1개 포함
4. 숫자 최소 1개 포함
5. 특수문자 최소 1개 포함 (!@#$%^&*)

### 2.2 Red 단계: 실패하는 테스트 작성

**Tests/PasswordValidatorTests.cs**
```csharp
using NUnit.Framework;

namespace PasswordValidator.Tests
{
    [TestFixture]
    public class PasswordValidatorTests
    {
        private Core.PasswordValidator _validator;

        [SetUp]
        public void Setup()
        {
            _validator = new Core.PasswordValidator();
        }

        #region 🔴 RED: 길이 검증 테스트

        [Test]
        public void Validate_PasswordTooShort_ReturnsFalse()
        {
            // Arrange
            string shortPassword = "Abc1!";  // 5자

            // Act
            var result = _validator.Validate(shortPassword);

            // Assert
            Assert.That(result.IsValid, Is.False);
            Assert.That(result.Errors, Does.Contain("비밀번호는 최소 8자 이상이어야 합니다."));
        }

        [Test]
        public void Validate_PasswordExactly8Chars_LengthOk()
        {
            // Arrange
            string password = "Abcd12!@";  // 정확히 8자

            // Act
            var result = _validator.Validate(password);

            // Assert
            Assert.That(result.Errors, Does.Not.Contain("비밀번호는 최소 8자 이상이어야 합니다."));
        }

        #endregion

        #region 🔴 RED: 대문자 검증 테스트

        [Test]
        public void Validate_NoUpperCase_ReturnsFalse()
        {
            // Arrange
            string password = "abcd1234!@";  // 대문자 없음

            // Act
            var result = _validator.Validate(password);

            // Assert
            Assert.That(result.IsValid, Is.False);
            Assert.That(result.Errors, Does.Contain("대문자가 최소 1개 포함되어야 합니다."));
        }

        [Test]
        public void Validate_WithUpperCase_NoUpperCaseError()
        {
            // Arrange
            string password = "Abcd1234!@";  // 대문자 포함

            // Act
            var result = _validator.Validate(password);

            // Assert
            Assert.That(result.Errors, Does.Not.Contain("대문자가 최소 1개 포함되어야 합니다."));
        }

        #endregion

        #region 🔴 RED: 소문자 검증 테스트

        [Test]
        public void Validate_NoLowerCase_ReturnsFalse()
        {
            // Arrange
            string password = "ABCD1234!@";  // 소문자 없음

            // Act
            var result = _validator.Validate(password);

            // Assert
            Assert.That(result.IsValid, Is.False);
            Assert.That(result.Errors, Does.Contain("소문자가 최소 1개 포함되어야 합니다."));
        }

        #endregion

        #region 🔴 RED: 숫자 검증 테스트

        [Test]
        public void Validate_NoDigit_ReturnsFalse()
        {
            // Arrange
            string password = "Abcdefgh!@";  // 숫자 없음

            // Act
            var result = _validator.Validate(password);

            // Assert
            Assert.That(result.IsValid, Is.False);
            Assert.That(result.Errors, Does.Contain("숫자가 최소 1개 포함되어야 합니다."));
        }

        #endregion

        #region 🔴 RED: 특수문자 검증 테스트

        [Test]
        public void Validate_NoSpecialChar_ReturnsFalse()
        {
            // Arrange
            string password = "Abcd1234ab";  // 특수문자 없음

            // Act
            var result = _validator.Validate(password);

            // Assert
            Assert.That(result.IsValid, Is.False);
            Assert.That(result.Errors, Does.Contain("특수문자(!@#$%^&*)가 최소 1개 포함되어야 합니다."));
        }

        #endregion

        #region 🔴 RED: 유효한 비밀번호 테스트

        [Test]
        public void Validate_ValidPassword_ReturnsTrue()
        {
            // Arrange
            string password = "Abcd1234!@";  // 모든 조건 충족

            // Act
            var result = _validator.Validate(password);

            // Assert
            Assert.That(result.IsValid, Is.True);
            Assert.That(result.Errors, Is.Empty);
        }

        [TestCase("Password1!")]
        [TestCase("MyP@ssw0rd")]
        [TestCase("Secure#123")]
        [TestCase("Test1234$%")]
        public void Validate_VariousValidPasswords_ReturnsTrue(string password)
        {
            var result = _validator.Validate(password);
            Assert.That(result.IsValid, Is.True);
        }

        #endregion

        #region 🔴 RED: Null/Empty 처리 테스트

        [Test]
        public void Validate_NullPassword_ReturnsFalse()
        {
            // Act
            var result = _validator.Validate(null);

            // Assert
            Assert.That(result.IsValid, Is.False);
            Assert.That(result.Errors, Does.Contain("비밀번호를 입력해주세요."));
        }

        [Test]
        public void Validate_EmptyPassword_ReturnsFalse()
        {
            // Act
            var result = _validator.Validate("");

            // Assert
            Assert.That(result.IsValid, Is.False);
            Assert.That(result.Errors, Does.Contain("비밀번호를 입력해주세요."));
        }

        #endregion

        #region 🔴 RED: 복합 오류 테스트

        [Test]
        public void Validate_MultipleErrors_ReturnsAllErrors()
        {
            // Arrange
            string password = "abc";  // 짧고, 대문자/숫자/특수문자 없음

            // Act
            var result = _validator.Validate(password);

            // Assert
            Assert.That(result.IsValid, Is.False);
            Assert.That(result.Errors.Count, Is.GreaterThanOrEqualTo(4));
        }

        #endregion
    }
}
```

### 2.3 Green 단계: 테스트 통과하는 코드 구현

**Core/PasswordValidator.cs**
```csharp
namespace PasswordValidator.Core
{
    public class ValidationResult
    {
        public bool IsValid => Errors.Count == 0;
        public List<string> Errors { get; } = new();

        public void AddError(string error)
        {
            Errors.Add(error);
        }
    }

    public class PasswordValidator
    {
        private const int MinLength = 8;
        private const string SpecialChars = "!@#$%^&*";

        public ValidationResult Validate(string password)
        {
            var result = new ValidationResult();

            // Null/Empty 검사
            if (string.IsNullOrEmpty(password))
            {
                result.AddError("비밀번호를 입력해주세요.");
                return result;
            }

            // 길이 검사
            if (password.Length < MinLength)
            {
                result.AddError("비밀번호는 최소 8자 이상이어야 합니다.");
            }

            // 대문자 검사
            if (!password.Any(char.IsUpper))
            {
                result.AddError("대문자가 최소 1개 포함되어야 합니다.");
            }

            // 소문자 검사
            if (!password.Any(char.IsLower))
            {
                result.AddError("소문자가 최소 1개 포함되어야 합니다.");
            }

            // 숫자 검사
            if (!password.Any(char.IsDigit))
            {
                result.AddError("숫자가 최소 1개 포함되어야 합니다.");
            }

            // 특수문자 검사
            if (!password.Any(c => SpecialChars.Contains(c)))
            {
                result.AddError("특수문자(!@#$%^&*)가 최소 1개 포함되어야 합니다.");
            }

            return result;
        }
    }
}
```

### 2.4 Refactor 단계: 코드 개선

**Core/PasswordValidator.cs (리팩토링 후)**
```csharp
namespace PasswordValidator.Core
{
    public class ValidationResult
    {
        public bool IsValid => Errors.Count == 0;
        public List<string> Errors { get; } = new();

        public void AddError(string error) => Errors.Add(error);
    }

    public class PasswordValidator
    {
        private readonly List<IPasswordRule> _rules;

        public PasswordValidator()
        {
            _rules = new List<IPasswordRule>
            {
                new NotEmptyRule(),
                new MinLengthRule(8),
                new UpperCaseRule(),
                new LowerCaseRule(),
                new DigitRule(),
                new SpecialCharRule("!@#$%^&*")
            };
        }

        public ValidationResult Validate(string password)
        {
            var result = new ValidationResult();

            foreach (var rule in _rules)
            {
                var error = rule.Validate(password);
                if (error != null)
                {
                    result.AddError(error);
                    
                    // NotEmpty 실패시 다른 규칙 검사 불필요
                    if (rule is NotEmptyRule)
                        break;
                }
            }

            return result;
        }
    }

    #region 규칙 인터페이스 및 구현

    public interface IPasswordRule
    {
        string? Validate(string password);
    }

    public class NotEmptyRule : IPasswordRule
    {
        public string? Validate(string password)
        {
            return string.IsNullOrEmpty(password) 
                ? "비밀번호를 입력해주세요." 
                : null;
        }
    }

    public class MinLengthRule : IPasswordRule
    {
        private readonly int _minLength;

        public MinLengthRule(int minLength)
        {
            _minLength = minLength;
        }

        public string? Validate(string password)
        {
            return password.Length < _minLength 
                ? $"비밀번호는 최소 {_minLength}자 이상이어야 합니다." 
                : null;
        }
    }

    public class UpperCaseRule : IPasswordRule
    {
        public string? Validate(string password)
        {
            return !password.Any(char.IsUpper) 
                ? "대문자가 최소 1개 포함되어야 합니다." 
                : null;
        }
    }

    public class LowerCaseRule : IPasswordRule
    {
        public string? Validate(string password)
        {
            return !password.Any(char.IsLower) 
                ? "소문자가 최소 1개 포함되어야 합니다." 
                : null;
        }
    }

    public class DigitRule : IPasswordRule
    {
        public string? Validate(string password)
        {
            return !password.Any(char.IsDigit) 
                ? "숫자가 최소 1개 포함되어야 합니다." 
                : null;
        }
    }

    public class SpecialCharRule : IPasswordRule
    {
        private readonly string _specialChars;

        public SpecialCharRule(string specialChars)
        {
            _specialChars = specialChars;
        }

        public string? Validate(string password)
        {
            return !password.Any(c => _specialChars.Contains(c)) 
                ? $"특수문자({_specialChars})가 최소 1개 포함되어야 합니다." 
                : null;
        }
    }

    #endregion
}
```

---

## 3. 실습 2: TDD로 ShoppingCart 구현

### 3.1 요구사항
- 상품 추가/제거
- 총액 계산
- 할인 적용
- 재고 확인

### 3.2 TDD 단계별 구현

**Tests/ShoppingCartTests.cs**
```csharp
using NUnit.Framework;
using Moq;

namespace ShoppingCart.Tests
{
    [TestFixture]
    public class ShoppingCartTests
    {
        private Mock<IInventoryService> _mockInventory;
        private Mock<IDiscountService> _mockDiscount;
        private Core.ShoppingCart _cart;

        [SetUp]
        public void Setup()
        {
            _mockInventory = new Mock<IInventoryService>();
            _mockDiscount = new Mock<IDiscountService>();
            _cart = new Core.ShoppingCart(_mockInventory.Object, _mockDiscount.Object);
        }

        #region 🔴🟢🔵 상품 추가 테스트

        [Test]
        public void AddItem_ValidItem_ItemAddedToCart()
        {
            // Arrange
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000 };
            _mockInventory.Setup(x => x.IsInStock(1)).Returns(true);

            // Act
            _cart.AddItem(product, 1);

            // Assert
            Assert.That(_cart.Items.Count, Is.EqualTo(1));
            Assert.That(_cart.Items[0].Product.Name, Is.EqualTo("노트북"));
        }

        [Test]
        public void AddItem_OutOfStock_ThrowsException()
        {
            // Arrange
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000 };
            _mockInventory.Setup(x => x.IsInStock(1)).Returns(false);

            // Act & Assert
            var ex = Assert.Throws<InvalidOperationException>(
                () => _cart.AddItem(product, 1));
            Assert.That(ex.Message, Does.Contain("재고"));
        }

        [Test]
        public void AddItem_SameProductTwice_QuantityIncreased()
        {
            // Arrange
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000 };
            _mockInventory.Setup(x => x.IsInStock(1)).Returns(true);

            // Act
            _cart.AddItem(product, 1);
            _cart.AddItem(product, 2);

            // Assert
            Assert.That(_cart.Items.Count, Is.EqualTo(1));
            Assert.That(_cart.Items[0].Quantity, Is.EqualTo(3));
        }

        [Test]
        public void AddItem_NegativeQuantity_ThrowsException()
        {
            // Arrange
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000 };

            // Act & Assert
            Assert.Throws<ArgumentException>(
                () => _cart.AddItem(product, -1));
        }

        #endregion

        #region 🔴🟢🔵 상품 제거 테스트

        [Test]
        public void RemoveItem_ExistingItem_ItemRemoved()
        {
            // Arrange
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000 };
            _mockInventory.Setup(x => x.IsInStock(1)).Returns(true);
            _cart.AddItem(product, 2);

            // Act
            _cart.RemoveItem(1, 1);

            // Assert
            Assert.That(_cart.Items[0].Quantity, Is.EqualTo(1));
        }

        [Test]
        public void RemoveItem_AllQuantity_ItemRemovedCompletely()
        {
            // Arrange
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000 };
            _mockInventory.Setup(x => x.IsInStock(1)).Returns(true);
            _cart.AddItem(product, 2);

            // Act
            _cart.RemoveItem(1, 2);

            // Assert
            Assert.That(_cart.Items.Count, Is.EqualTo(0));
        }

        [Test]
        public void RemoveItem_NonExistingItem_ThrowsException()
        {
            // Act & Assert
            Assert.Throws<KeyNotFoundException>(
                () => _cart.RemoveItem(999, 1));
        }

        #endregion

        #region 🔴🟢🔵 총액 계산 테스트

        [Test]
        public void GetTotal_EmptyCart_ReturnsZero()
        {
            // Act
            decimal total = _cart.GetTotal();

            // Assert
            Assert.That(total, Is.EqualTo(0));
        }

        [Test]
        public void GetTotal_SingleItem_ReturnsCorrectTotal()
        {
            // Arrange
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000 };
            _mockInventory.Setup(x => x.IsInStock(1)).Returns(true);
            _mockDiscount.Setup(x => x.GetDiscount(It.IsAny<decimal>())).Returns(0);
            _cart.AddItem(product, 2);

            // Act
            decimal total = _cart.GetTotal();

            // Assert
            Assert.That(total, Is.EqualTo(2000000));
        }

        [Test]
        public void GetTotal_MultipleItems_ReturnsCorrectTotal()
        {
            // Arrange
            var laptop = new Product { Id = 1, Name = "노트북", Price = 1000000 };
            var mouse = new Product { Id = 2, Name = "마우스", Price = 50000 };
            
            _mockInventory.Setup(x => x.IsInStock(It.IsAny<int>())).Returns(true);
            _mockDiscount.Setup(x => x.GetDiscount(It.IsAny<decimal>())).Returns(0);
            
            _cart.AddItem(laptop, 1);
            _cart.AddItem(mouse, 2);

            // Act
            decimal total = _cart.GetTotal();

            // Assert
            Assert.That(total, Is.EqualTo(1100000)); // 1000000 + 100000
        }

        #endregion

        #region 🔴🟢🔵 할인 적용 테스트

        [Test]
        public void GetTotal_WithDiscount_ReturnsDiscountedTotal()
        {
            // Arrange
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000 };
            _mockInventory.Setup(x => x.IsInStock(1)).Returns(true);
            _mockDiscount.Setup(x => x.GetDiscount(1000000m)).Returns(100000m); // 10% 할인

            _cart.AddItem(product, 1);

            // Act
            decimal total = _cart.GetTotal();

            // Assert
            Assert.That(total, Is.EqualTo(900000));
        }

        [Test]
        public void ApplyCoupon_ValidCoupon_DiscountApplied()
        {
            // Arrange
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000 };
            _mockInventory.Setup(x => x.IsInStock(1)).Returns(true);
            _mockDiscount.Setup(x => x.GetCouponDiscount("SAVE20")).Returns(200000m);
            _mockDiscount.Setup(x => x.GetDiscount(It.IsAny<decimal>())).Returns(0);

            _cart.AddItem(product, 1);

            // Act
            _cart.ApplyCoupon("SAVE20");
            decimal total = _cart.GetTotal();

            // Assert
            Assert.That(total, Is.EqualTo(800000));
        }

        [Test]
        public void ApplyCoupon_InvalidCoupon_ThrowsException()
        {
            // Arrange
            _mockDiscount.Setup(x => x.GetCouponDiscount("INVALID"))
                .Throws(new ArgumentException("유효하지 않은 쿠폰입니다."));

            // Act & Assert
            Assert.Throws<ArgumentException>(() => _cart.ApplyCoupon("INVALID"));
        }

        #endregion

        #region 🔴🟢🔵 카트 비우기 테스트

        [Test]
        public void Clear_NonEmptyCart_CartBecomesEmpty()
        {
            // Arrange
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000 };
            _mockInventory.Setup(x => x.IsInStock(1)).Returns(true);
            _cart.AddItem(product, 1);

            // Act
            _cart.Clear();

            // Assert
            Assert.That(_cart.Items, Is.Empty);
            Assert.That(_cart.GetTotal(), Is.EqualTo(0));
        }

        #endregion
    }
}
```

### 3.3 구현 코드

**Core/ShoppingCart.cs**
```csharp
namespace ShoppingCart.Core
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }

    public class CartItem
    {
        public Product Product { get; set; }
        public int Quantity { get; set; }

        public decimal Subtotal => Product.Price * Quantity;
    }

    public interface IInventoryService
    {
        bool IsInStock(int productId);
        int GetStockQuantity(int productId);
    }

    public interface IDiscountService
    {
        decimal GetDiscount(decimal subtotal);
        decimal GetCouponDiscount(string couponCode);
    }

    public class ShoppingCart
    {
        private readonly IInventoryService _inventoryService;
        private readonly IDiscountService _discountService;
        private readonly List<CartItem> _items = new();
        private string _appliedCoupon;

        public IReadOnlyList<CartItem> Items => _items.AsReadOnly();

        public ShoppingCart(IInventoryService inventoryService, IDiscountService discountService)
        {
            _inventoryService = inventoryService;
            _discountService = discountService;
        }

        public void AddItem(Product product, int quantity)
        {
            if (quantity <= 0)
                throw new ArgumentException("수량은 0보다 커야 합니다.");

            if (!_inventoryService.IsInStock(product.Id))
                throw new InvalidOperationException($"{product.Name}의 재고가 부족합니다.");

            var existingItem = _items.FirstOrDefault(i => i.Product.Id == product.Id);
            
            if (existingItem != null)
            {
                existingItem.Quantity += quantity;
            }
            else
            {
                _items.Add(new CartItem { Product = product, Quantity = quantity });
            }
        }

        public void RemoveItem(int productId, int quantity)
        {
            var item = _items.FirstOrDefault(i => i.Product.Id == productId);
            
            if (item == null)
                throw new KeyNotFoundException($"상품 ID {productId}를 찾을 수 없습니다.");

            item.Quantity -= quantity;
            
            if (item.Quantity <= 0)
                _items.Remove(item);
        }

        public decimal GetTotal()
        {
            decimal subtotal = _items.Sum(i => i.Subtotal);
            decimal discount = _discountService.GetDiscount(subtotal);
            
            decimal couponDiscount = 0;
            if (!string.IsNullOrEmpty(_appliedCoupon))
            {
                couponDiscount = _discountService.GetCouponDiscount(_appliedCoupon);
            }

            return subtotal - discount - couponDiscount;
        }

        public void ApplyCoupon(string couponCode)
        {
            // 유효성 검증 (예외 발생시 전파)
            _discountService.GetCouponDiscount(couponCode);
            _appliedCoupon = couponCode;
        }

        public void Clear()
        {
            _items.Clear();
            _appliedCoupon = null;
        }
    }
}
```

---

## 4. 코드 커버리지 분석

### 4.1 커버리지 도구 설치

```bash
# Coverlet 설치 (테스트 프로젝트에)
dotnet add package coverlet.collector
dotnet add package coverlet.msbuild

# ReportGenerator 설치 (전역)
dotnet tool install -g dotnet-reportgenerator-globaltool
```

### 4.2 커버리지 실행

```bash
# 커버리지 수집하며 테스트 실행
dotnet test --collect:"XPlat Code Coverage"

# 또는 Coverlet MSBuild 사용
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura

# HTML 리포트 생성
reportgenerator \
    -reports:"./TestResults/**/coverage.cobertura.xml" \
    -targetdir:"./CoverageReport" \
    -reporttypes:"Html"
```

### 4.3 Visual Studio에서 커버리지 분석

1. **Test Explorer** 열기: Test > Test Explorer
2. **커버리지 실행**: Test > Analyze Code Coverage > All Tests
3. **결과 분석**: Code Coverage Results 창에서 확인

### 4.4 커버리지 목표 설정

```xml
<!-- .csproj에 커버리지 임계값 설정 -->
<PropertyGroup>
  <CollectCoverage>true</CollectCoverage>
  <CoverletOutputFormat>cobertura</CoverletOutputFormat>
  <Threshold>80</Threshold>
  <ThresholdType>line,branch,method</ThresholdType>
  <ThresholdStat>total</ThresholdStat>
</PropertyGroup>
```

### 4.5 커버리지 유형

| 유형 | 설명 | 목표 |
|------|------|------|
| Line Coverage | 실행된 코드 라인 비율 | 80% 이상 |
| Branch Coverage | 실행된 분기(if/else) 비율 | 75% 이상 |
| Method Coverage | 호출된 메서드 비율 | 90% 이상 |

---

## 5. 예외 처리 및 경계값 테스트

### 5.1 경계값 테스트 (Boundary Value Testing)

**Tests/BoundaryValueTests.cs**
```csharp
using NUnit.Framework;

namespace BoundaryValue.Tests
{
    [TestFixture]
    public class AgeValidatorTests
    {
        private AgeValidator _validator;

        [SetUp]
        public void Setup()
        {
            _validator = new AgeValidator();
        }

        #region 경계값 테스트 - 성인 나이 검증 (18세 이상)

        [Test]
        public void IsAdult_Age17_ReturnsFalse()
        {
            // 경계값 바로 아래
            Assert.That(_validator.IsAdult(17), Is.False);
        }

        [Test]
        public void IsAdult_Age18_ReturnsTrue()
        {
            // 경계값 정확히
            Assert.That(_validator.IsAdult(18), Is.True);
        }

        [Test]
        public void IsAdult_Age19_ReturnsTrue()
        {
            // 경계값 바로 위
            Assert.That(_validator.IsAdult(19), Is.True);
        }

        #endregion

        #region 경계값 테스트 - 유효한 나이 범위 (0-150)

        [Test]
        public void IsValidAge_NegativeOne_ReturnsFalse()
        {
            // 하한 경계 아래
            Assert.That(_validator.IsValidAge(-1), Is.False);
        }

        [Test]
        public void IsValidAge_Zero_ReturnsTrue()
        {
            // 하한 경계 정확히
            Assert.That(_validator.IsValidAge(0), Is.True);
        }

        [Test]
        public void IsValidAge_One_ReturnsTrue()
        {
            // 하한 경계 바로 위
            Assert.That(_validator.IsValidAge(1), Is.True);
        }

        [Test]
        public void IsValidAge_149_ReturnsTrue()
        {
            // 상한 경계 바로 아래
            Assert.That(_validator.IsValidAge(149), Is.True);
        }

        [Test]
        public void IsValidAge_150_ReturnsTrue()
        {
            // 상한 경계 정확히
            Assert.That(_validator.IsValidAge(150), Is.True);
        }

        [Test]
        public void IsValidAge_151_ReturnsFalse()
        {
            // 상한 경계 위
            Assert.That(_validator.IsValidAge(151), Is.False);
        }

        #endregion

        #region 등가 분할 테스트 (Equivalence Partitioning)

        // 유효 등가 클래스: 0-17 (미성년), 18-64 (성인), 65-150 (노인)
        // 무효 등가 클래스: < 0, > 150

        [TestCase(-100, "Invalid")]  // 무효 등가 클래스
        [TestCase(10, "Minor")]       // 유효 등가 클래스 1
        [TestCase(30, "Adult")]       // 유효 등가 클래스 2
        [TestCase(70, "Senior")]      // 유효 등가 클래스 3
        [TestCase(200, "Invalid")]    // 무효 등가 클래스
        public void GetAgeCategory_VariousAges_ReturnsCorrectCategory(
            int age, string expectedCategory)
        {
            Assert.That(_validator.GetAgeCategory(age), Is.EqualTo(expectedCategory));
        }

        #endregion
    }

    public class AgeValidator
    {
        public bool IsAdult(int age) => age >= 18;

        public bool IsValidAge(int age) => age >= 0 && age <= 150;

        public string GetAgeCategory(int age)
        {
            if (!IsValidAge(age)) return "Invalid";
            if (age < 18) return "Minor";
            if (age < 65) return "Adult";
            return "Senior";
        }
    }
}
```

### 5.2 예외 처리 테스트

**Tests/ExceptionTests.cs**
```csharp
using NUnit.Framework;

namespace Exception.Tests
{
    [TestFixture]
    public class BankAccountTests
    {
        private BankAccount _account;

        [SetUp]
        public void Setup()
        {
            _account = new BankAccount("홍길동", 10000);
        }

        #region 예외 발생 테스트

        [Test]
        public void Withdraw_InsufficientFunds_ThrowsInsufficientFundsException()
        {
            // Act & Assert
            var ex = Assert.Throws<InsufficientFundsException>(
                () => _account.Withdraw(15000));
            
            Assert.That(ex.RequestedAmount, Is.EqualTo(15000));
            Assert.That(ex.AvailableBalance, Is.EqualTo(10000));
        }

        [Test]
        public void Withdraw_NegativeAmount_ThrowsArgumentException()
        {
            Assert.Throws<ArgumentException>(
                () => _account.Withdraw(-100));
        }

        [Test]
        public void Withdraw_ZeroAmount_ThrowsArgumentException()
        {
            Assert.Throws<ArgumentException>(
                () => _account.Withdraw(0));
        }

        [Test]
        public void Deposit_NegativeAmount_ThrowsArgumentException()
        {
            var ex = Assert.Throws<ArgumentException>(
                () => _account.Deposit(-100));
            
            Assert.That(ex.Message, Does.Contain("양수"));
        }

        #endregion

        #region 예외 미발생 테스트

        [Test]
        public void Withdraw_ValidAmount_DoesNotThrow()
        {
            Assert.DoesNotThrow(() => _account.Withdraw(5000));
        }

        [Test]
        public void Withdraw_ExactBalance_DoesNotThrow()
        {
            Assert.DoesNotThrow(() => _account.Withdraw(10000));
            Assert.That(_account.Balance, Is.EqualTo(0));
        }

        #endregion

        #region 비동기 예외 테스트

        [Test]
        public async Task TransferAsync_InsufficientFunds_ThrowsException()
        {
            var targetAccount = new BankAccount("김철수", 0);

            Assert.ThrowsAsync<InsufficientFundsException>(
                async () => await _account.TransferAsync(targetAccount, 15000));
        }

        [Test]
        public async Task TransferAsync_ValidAmount_TransfersSuccessfully()
        {
            var targetAccount = new BankAccount("김철수", 0);

            await _account.TransferAsync(targetAccount, 5000);

            Assert.That(_account.Balance, Is.EqualTo(5000));
            Assert.That(targetAccount.Balance, Is.EqualTo(5000));
        }

        #endregion
    }

    #region 구현 클래스

    public class InsufficientFundsException : Exception
    {
        public decimal RequestedAmount { get; }
        public decimal AvailableBalance { get; }

        public InsufficientFundsException(decimal requested, decimal available)
            : base($"잔액이 부족합니다. 요청: {requested:C}, 가용: {available:C}")
        {
            RequestedAmount = requested;
            AvailableBalance = available;
        }
    }

    public class BankAccount
    {
        public string Owner { get; }
        public decimal Balance { get; private set; }

        public BankAccount(string owner, decimal initialBalance)
        {
            Owner = owner;
            Balance = initialBalance;
        }

        public void Deposit(decimal amount)
        {
            if (amount <= 0)
                throw new ArgumentException("입금액은 양수여야 합니다.");

            Balance += amount;
        }

        public void Withdraw(decimal amount)
        {
            if (amount <= 0)
                throw new ArgumentException("출금액은 양수여야 합니다.");

            if (amount > Balance)
                throw new InsufficientFundsException(amount, Balance);

            Balance -= amount;
        }

        public async Task TransferAsync(BankAccount target, decimal amount)
        {
            // 시뮬레이션을 위한 지연
            await Task.Delay(10);

            Withdraw(amount);
            target.Deposit(amount);
        }
    }

    #endregion
}
```

---

## 6. 실습 3: 팀별 미니 프로젝트

### 6.1 프로젝트 주제: 도서관 대출 시스템

**요구사항:**
1. 도서 대출/반납
2. 대출 기한 관리 (14일)
3. 연체료 계산 (하루 100원)
4. 대출 이력 조회

### 6.2 인터페이스 정의

```csharp
public interface IBookRepository
{
    Book GetById(string isbn);
    bool IsAvailable(string isbn);
    void UpdateStatus(string isbn, BookStatus status);
}

public interface IMemberRepository
{
    Member GetById(int memberId);
    int GetCurrentLoanCount(int memberId);
}

public interface ILoanRepository
{
    void SaveLoan(Loan loan);
    Loan GetActiveLoan(string isbn);
    IEnumerable<Loan> GetLoanHistory(int memberId);
}

public interface IDateTimeProvider
{
    DateTime Now { get; }
}
```

### 6.3 TDD로 구현할 테스트 시나리오

```csharp
[TestFixture]
public class LibraryServiceTests
{
    // 1. 도서 대출 성공
    // 2. 이미 대출 중인 도서 대출 실패
    // 3. 대출 한도 초과시 대출 실패
    // 4. 도서 반납 성공
    // 5. 연체 도서 반납시 연체료 계산
    // 6. 대출 기한 확인
    
    [Test]
    public void BorrowBook_AvailableBook_LoanCreated()
    {
        // TDD로 구현하세요
    }

    [Test]
    public void BorrowBook_AlreadyBorrowed_ThrowsException()
    {
        // TDD로 구현하세요
    }

    [Test]
    public void ReturnBook_OverdueBook_CalculatesLateFee()
    {
        // TDD로 구현하세요
    }
}
```

---

## 7. 팀별 과제

### 과제 1: PasswordValidator 확장
다음 규칙을 추가하고 TDD로 구현하세요:
- 연속된 같은 문자 3개 이상 금지
- 사용자 아이디 포함 금지
- 이전 비밀번호와 3글자 이상 달라야 함

### 과제 2: 커버리지 80% 달성
ShoppingCart 클래스의 코드 커버리지를 80% 이상 달성하세요.

### 과제 3: 도서관 시스템 구현
팀별로 도서관 대출 시스템을 TDD로 구현하세요.

---

## 8. 핵심 정리

### TDD 사이클
```
🔴 Red   → 실패하는 테스트 먼저 작성
🟢 Green → 최소한의 코드로 테스트 통과
🔵 Refactor → 코드 품질 개선 (테스트 유지)
```

### 경계값 테스트 체크리스트
- [ ] 최소값
- [ ] 최소값 - 1
- [ ] 최소값 + 1
- [ ] 최대값
- [ ] 최대값 - 1
- [ ] 최대값 + 1
- [ ] 0 / null / empty

### 커버리지 목표
- Line Coverage: 80% 이상
- Branch Coverage: 75% 이상
- Method Coverage: 90% 이상

---

## 📝 다음 시간 예고
- **3일차**: 테스트 자동화 및 CI/CD
- GitHub Actions 파이프라인 구축
- 테스트 리포트 자동화
- 종합 프로젝트 발표
