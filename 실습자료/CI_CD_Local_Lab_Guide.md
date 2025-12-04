# 로컬 환경 CI/CD 실습 가이드 (TDD 방식)

*폐쇄망 환경에서의 CI/CD 파이프라인 구축 + TDD 실습*

> 📌 **코드 작성**: Visual Studio 2022 GUI  
> 📌 **Git 명령**: CMD (명령 프롬프트)  
> 📌 **개발 방식**: TDD (Test-Driven Development)

---

## 실습 시나리오: 주문 관리 시스템

### 비즈니스 요구사항

```
┌─────────────────────────────────────────────────────────────┐
│                    주문 관리 시스템                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 주문 검증 (OrderValidator)                              │
│     - 수량은 1개 이상이어야 함                               │
│     - 상품코드는 필수                                        │
│     - 단가는 0원 초과                                        │
│                                                             │
│  2. 가격 계산 (PriceCalculator)                             │
│     - 기본 가격 = 수량 × 단가                                │
│     - 수량 할인: 10개 이상 5%, 50개 이상 10%, 100개 이상 15% │
│     - 회원 등급 할인: VIP 10%, Gold 5%, Silver 3%           │
│                                                             │
│  3. 주문 처리 (OrderService)                                │
│     - 주문 검증 → 가격 계산 → 주문 생성                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### TDD + CI/CD 워크플로우

```
🔴 RED ──────► 🟢 GREEN ──────► 🔵 REFACTOR ──────► 🚀 DEPLOY
테스트 작성      코드 구현         코드 개선          git push
(실패)          (통과)           (통과 유지)        (자동 배포)
```

---

## 목차

- [Part 1. 환경 구성 및 프로젝트 생성 ](#part-1-환경-구성-및-프로젝트-생성)
- [Part 2. CI/CD 인프라 구축 ](#part-2-cicd-인프라-구축)
- [Part 3. TDD 실습 - 주문 검증 ](#part-3-tdd-실습---주문-검증)
- [Part 4. TDD 실습 - 가격 계산 ](#part-4-tdd-실습---가격-계산)
- [Part 5. TDD 실습 - 주문 서비스 ](#part-5-tdd-실습---주문-서비스)
- [Part 6. 종합 정리 및 마무리 ](#part-6-종합-정리-및-마무리)

---

## Part 1. 환경 구성 및 프로젝트 생성

### 1.1 Visual Studio에서 프로젝트 생성

#### 🔬 Lab 1-1: Visual Studio 실행

1. **시작 메뉴** → `Visual Studio 2022` 실행

#### 🔬 Lab 1-2: 콘솔 프로젝트 생성

1. **새 프로젝트 만들기** 클릭
2. 검색창에 `콘솔` 입력
3. **콘솔 앱** (C#, .NET) 선택 → **다음**
4. 설정:
   - **프로젝트 이름**: `OrderSystem.App`
   - **위치**: `C:\CICDLab`
   - **솔루션 이름**: `OrderSystem`
   - ☐ **솔루션 및 프로젝트를 같은 디렉터리에 배치** 체크 해제
5. **다음** → **프레임워크**: `.NET 8.0` → **만들기**

#### 🔬 Lab 1-3: 테스트 프로젝트 추가

1. **솔루션 탐색기**에서 솔루션 `OrderSystem` 우클릭
2. **추가** → **새 프로젝트**
3. 검색창에 `nunit` 입력
4. **NUnit 테스트 프로젝트** 선택 → **다음**
5. 설정:
   - **프로젝트 이름**: `OrderSystem.Tests`
   - **위치**: `C:\CICDLab`
6. **다음** → **만들기**

#### 🔬 Lab 1-4: 프로젝트 참조 추가

1. `OrderSystem.Tests` → **종속성** 우클릭
2. **프로젝트 참조 추가** → ☑️ `OrderSystem.App` 체크 → **확인**

#### 🔬 Lab 1-5: 빌드 확인

`Ctrl+Shift+B`로 빌드 → `빌드: 성공 2` 확인

### 1.2 도메인 모델 생성

#### 🔬 Lab 1-6: Models 폴더 생성

1. `OrderSystem.App` 프로젝트 우클릭
2. **추가** → **새 폴더** → 이름: `Models`

#### 🔬 Lab 1-7: CustomerGrade 열거형 생성

1. `Models` 폴더 우클릭 → **추가** → **클래스**
2. 이름: `CustomerGrade.cs` → **추가**
3. 내용:

```csharp
namespace OrderSystem.App.Models;

/// <summary>
/// 고객 등급
/// </summary>
public enum CustomerGrade
{
    Normal,     // 일반 회원
    Silver,     // 실버 회원 (3% 할인)
    Gold,       // 골드 회원 (5% 할인)
    VIP         // VIP 회원 (10% 할인)
}
```

`Ctrl+S`로 저장

#### 🔬 Lab 1-8: Order 클래스 생성

1. `Models` 폴더 우클릭 → **추가** → **클래스**
2. 이름: `Order.cs` → **추가**
3. 내용:

```csharp
namespace OrderSystem.App.Models;

/// <summary>
/// 주문 정보
/// </summary>
public class Order
{
    public string ProductCode { get; set; } = string.Empty;
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
    public CustomerGrade CustomerGrade { get; set; }
    public decimal TotalPrice { get; set; }
    public bool IsValid { get; set; }
    public string? ErrorMessage { get; set; }
}
```

`Ctrl+S`로 저장

### 1.3 서비스 클래스 준비 (빈 클래스)

#### 🔬 Lab 1-9: Services 폴더 생성

1. `OrderSystem.App` 프로젝트 우클릭
2. **추가** → **새 폴더** → 이름: `Services`

#### 🔬 Lab 1-10: OrderValidator 클래스 생성

1. `Services` 폴더 우클릭 → **추가** → **클래스**
2. 이름: `OrderValidator.cs` → **추가**
3. 내용:

```csharp
using OrderSystem.App.Models;

namespace OrderSystem.App.Services;

/// <summary>
/// 주문 유효성 검증
/// </summary>
public class OrderValidator
{
    // TDD: 테스트를 먼저 작성한 후 메서드를 추가합니다
}
```

`Ctrl+S`로 저장

#### 🔬 Lab 1-11: PriceCalculator 클래스 생성

1. `Services` 폴더 우클릭 → **추가** → **클래스**
2. 이름: `PriceCalculator.cs` → **추가**
3. 내용:

```csharp
using OrderSystem.App.Models;

namespace OrderSystem.App.Services;

/// <summary>
/// 가격 계산 서비스
/// </summary>
public class PriceCalculator
{
    // TDD: 테스트를 먼저 작성한 후 메서드를 추가합니다
}
```

`Ctrl+S`로 저장

#### 🔬 Lab 1-12: OrderService 클래스 생성

1. `Services` 폴더 우클릭 → **추가** → **클래스**
2. 이름: `OrderService.cs` → **추가**
3. 내용:

```csharp
using OrderSystem.App.Models;

namespace OrderSystem.App.Services;

/// <summary>
/// 주문 처리 서비스
/// </summary>
public class OrderService
{
    // TDD: 테스트를 먼저 작성한 후 메서드를 추가합니다
}
```

`Ctrl+S`로 저장

### 1.4 테스트 파일 준비

#### 🔬 Lab 1-13: 테스트 파일 초기화

1. `OrderSystem.Tests` → `UnitTest1.cs` 삭제 (우클릭 → 삭제)
2. `OrderSystem.Tests` 우클릭 → **추가** → **클래스**
3. 이름: `OrderValidatorTests.cs` → **추가**
4. 내용:

```csharp
using NUnit.Framework;
using OrderSystem.App.Models;
using OrderSystem.App.Services;

namespace OrderSystem.Tests;

[TestFixture]
public class OrderValidatorTests
{
    private OrderValidator _validator;

    [SetUp]
    public void Setup()
    {
        _validator = new OrderValidator();
    }

    // TDD: 여기에 테스트를 추가합니다
}
```

`Ctrl+S`로 저장

#### 🔬 Lab 1-14: 빌드 확인

`Ctrl+Shift+B`로 빌드 성공 확인

### 1.5 프로젝트 구조 확인

```
솔루션 'OrderSystem'
├── OrderSystem.App
│   ├── Models
│   │   ├── CustomerGrade.cs
│   │   └── Order.cs
│   ├── Services
│   │   ├── OrderValidator.cs
│   │   ├── PriceCalculator.cs
│   │   └── OrderService.cs
│   └── Program.cs
└── OrderSystem.Tests
    └── OrderValidatorTests.cs
```

---

## Part 2. CI/CD 인프라 구축

> ⚠️ 이 파트의 모든 명령은 **CMD**에서 실행합니다.

### 2.1 Git 저장소 초기화

#### 🔬 Lab 2-1: CMD 창 열기

`Win+R` → `cmd` → Enter

#### 🔬 Lab 2-2: Git 사용자 설정

```cmd
git config --global user.name "Hong Gildong"
```
```cmd
git config --global user.email "hong@company.com"
```
```cmd
git config --global init.defaultBranch main
```

#### 🔬 Lab 2-3: 프로젝트 폴더로 이동

```cmd
cd C:\CICDLab
```

#### 🔬 Lab 2-4: Git 저장소 초기화

```cmd
git init
```

#### 🔬 Lab 2-5: .gitignore 파일 생성

```cmd
notepad .gitignore
```

**새 파일 만들기** → 다음 내용 입력 후 저장:

```
bin/
obj/
.vs/
*.user
TestResults/
artifacts/
```

### 2.2 로컬 Git 서버 생성

#### 🔬 Lab 2-6: Bare Repository 생성

```cmd
mkdir C:\GitServer\OrderSystem.git
```
```cmd
cd C:\GitServer\OrderSystem.git
```
```cmd
git init --bare
```

### 2.3 Post-receive Hook 생성 (자동 배포)

#### 🔬 Lab 2-7: post-receive 파일 생성

```cmd
notepad hooks\post-receive
```

**새 파일 만들기** → 다음 내용 입력:

```bash
#!/bin/sh

echo ""
echo "========================================"
echo "  ORDER SYSTEM - CI/CD PIPELINE"
echo "========================================"
echo ""

# 경로 설정
WORK_DIR="/c/GitServer/OrderSystem-work"
DEPLOY_DIR="/c/Deploy/OrderSystem"
GIT_DIR="/c/GitServer/OrderSystem.git"

# Step 1: 소스 체크아웃
echo "[1/5] Checking out source..."
rm -rf "$WORK_DIR"
mkdir -p "$WORK_DIR"
git --work-tree="$WORK_DIR" --git-dir="$GIT_DIR" checkout -f main
echo "      Done."
echo ""

# Step 2: 패키지 복원
echo "[2/5] Restoring packages..."
cd "$WORK_DIR/OrderSystem"
dotnet restore --nologo -v q
if [ $? -ne 0 ]; then
    echo "      RESTORE FAILED!"
    exit 1
fi
echo "      Done."
echo ""

# Step 3: 빌드
echo "[3/5] Building..."
dotnet build -c Release --nologo -v q
if [ $? -ne 0 ]; then
    echo "      BUILD FAILED!"
    exit 1
fi
echo "      Done."
echo ""

# Step 4: 테스트 (TDD 검증)
echo "[4/5] Running Tests..."
dotnet test -c Release --nologo
if [ $? -ne 0 ]; then
    echo ""
    echo "========================================"
    echo "  TESTS FAILED! DEPLOYMENT CANCELLED!"
    echo "========================================"
    exit 1
fi
echo "      Done."
echo ""

# Step 5: 배포
echo "[5/5] Deploying..."
dotnet publish OrderSystem.App -c Release -o publish --nologo -v q

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
VERSION_DIR="$DEPLOY_DIR/$TIMESTAMP"
mkdir -p "$VERSION_DIR"
cp -r publish/* "$VERSION_DIR/"

echo "      Deployed to: $VERSION_DIR"
echo ""
echo "========================================"
echo "  DEPLOYMENT SUCCESS!"
echo "========================================"
echo ""
```

저장 후 닫기

### 2.4 배포 폴더 생성 및 원격 저장소 연결

#### 🔬 Lab 2-8: 배포 폴더 생성

```cmd
mkdir C:\Deploy\OrderSystem
```

#### 🔬 Lab 2-9: 프로젝트 폴더로 이동

```cmd
cd C:\CICDLab
```

#### 🔬 Lab 2-10: 원격 저장소 추가

```cmd
git remote add origin C:\GitServer\OrderSystem.git
```

### 2.5 첫 번째 커밋 및 Push

#### 🔬 Lab 2-11: 첫 번째 커밋

```cmd
git add .
```
```cmd
git commit -m "feat: 주문 시스템 초기 프로젝트 구조 생성"
```

#### 🔬 Lab 2-12: 첫 번째 Push

```cmd
git push -u origin main
```

#### 🔬 Lab 2-13: 배포 결과 확인

```cmd
dir C:\Deploy\OrderSystem
```
```cmd
C:\Deploy\OrderSystem\current\OrderSystem.App.exe
```

✅ CI/CD 파이프라인 정상 작동 확인!

---

## Part 3. TDD 실습 - 주문 검증


> 🎯 **목표**: OrderValidator 클래스를 TDD로 구현합니다.

### 3.1 TDD 사이클 #1: 수량 검증

#### 🔴 RED: 실패하는 테스트 작성

#### 🔬 Lab 3-1: 수량이 0일 때 검증 실패 테스트 (Visual Studio)

`OrderValidatorTests.cs`에 추가:

```csharp
    [Test]
    public void Validate_QuantityIsZero_ReturnsFalse()
    {
        // Arrange
        var order = new Order
        {
            ProductCode = "PROD-001",
            Quantity = 0,
            UnitPrice = 1000m
        };

        // Act
        var result = _validator.Validate(order);

        // Assert
        Assert.That(result.IsValid, Is.False);
        Assert.That(result.ErrorMessage, Does.Contain("수량"));
    }
```

`Ctrl+S` → `Ctrl+Shift+B`

❌ **빌드 실패**: `'OrderValidator'에 'Validate'에 대한 정의가 없습니다`

---

#### 🟢 GREEN: 최소한의 구현

#### 🔬 Lab 3-2: Validate 메서드 구현 (Visual Studio)

`OrderValidator.cs`를 열고 클래스 안에 추가:

```csharp
    public Order Validate(Order order)
    {
        if (order.Quantity <= 0)
        {
            order.IsValid = false;
            order.ErrorMessage = "수량은 1개 이상이어야 합니다.";
            return order;
        }

        order.IsValid = true;
        return order;
    }
```

`Ctrl+S` → `Ctrl+Shift+B` → `Ctrl+R, A`

✅ 테스트 통과!

---

#### 🔵 REFACTOR: 추가 테스트

#### 🔬 Lab 3-3: 음수 수량 테스트 추가 (Visual Studio)

```csharp
    [Test]
    public void Validate_QuantityIsNegative_ReturnsFalse()
    {
        var order = new Order
        {
            ProductCode = "PROD-001",
            Quantity = -5,
            UnitPrice = 1000m
        };

        var result = _validator.Validate(order);

        Assert.That(result.IsValid, Is.False);
    }

    [Test]
    public void Validate_QuantityIsPositive_ReturnsTrue()
    {
        var order = new Order
        {
            ProductCode = "PROD-001",
            Quantity = 10,
            UnitPrice = 1000m
        };

        var result = _validator.Validate(order);

        Assert.That(result.IsValid, Is.True);
    }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 3개 테스트 통과

---

#### 🚀 커밋 및 자동 배포

#### 🔬 Lab 3-4: 커밋 및 Push (CMD)

```cmd
cd C:\CICDLab
```
```cmd
git add .
```
```cmd
git commit -m "feat: 주문 수량 검증 기능 구현"
```
```cmd
git push origin main
```

✅ 테스트 3개 통과 → 자동 배포 성공!

---

### 3.2 TDD 사이클 #2: 상품코드 검증

#### 🔴 RED: 실패하는 테스트

#### 🔬 Lab 3-5: 상품코드 누락 테스트 (Visual Studio)

```csharp
    [Test]
    public void Validate_ProductCodeIsEmpty_ReturnsFalse()
    {
        var order = new Order
        {
            ProductCode = "",
            Quantity = 10,
            UnitPrice = 1000m
        };

        var result = _validator.Validate(order);

        Assert.That(result.IsValid, Is.False);
        Assert.That(result.ErrorMessage, Does.Contain("상품코드"));
    }
```

`Ctrl+S` → `Ctrl+R, A`

❌ **테스트 실패**: 상품코드 검증 로직이 없음

---

#### 🟢 GREEN: 상품코드 검증 추가

#### 🔬 Lab 3-6: Validate 메서드 수정 (Visual Studio)

`OrderValidator.cs`의 `Validate` 메서드를 다음으로 **교체**:

```csharp
    public Order Validate(Order order)
    {
        // 수량 검증
        if (order.Quantity <= 0)
        {
            order.IsValid = false;
            order.ErrorMessage = "수량은 1개 이상이어야 합니다.";
            return order;
        }

        // 상품코드 검증
        if (string.IsNullOrWhiteSpace(order.ProductCode))
        {
            order.IsValid = false;
            order.ErrorMessage = "상품코드는 필수입니다.";
            return order;
        }

        order.IsValid = true;
        return order;
    }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 4개 테스트 통과

---

#### 🚀 커밋 및 자동 배포

#### 🔬 Lab 3-7: 커밋 및 Push (CMD)

```cmd
git add .
```
```cmd
git commit -m "feat: 상품코드 검증 기능 추가"
```
```cmd
git push origin main
```

---

### 3.3 TDD 사이클 #3: 단가 검증

#### 🔴 RED → 🟢 GREEN → 🚀 DEPLOY

#### 🔬 Lab 3-8: 단가 검증 테스트 (Visual Studio)

```csharp
    [Test]
    public void Validate_UnitPriceIsZero_ReturnsFalse()
    {
        var order = new Order
        {
            ProductCode = "PROD-001",
            Quantity = 10,
            UnitPrice = 0m
        };

        var result = _validator.Validate(order);

        Assert.That(result.IsValid, Is.False);
        Assert.That(result.ErrorMessage, Does.Contain("단가"));
    }

    [Test]
    public void Validate_UnitPriceIsNegative_ReturnsFalse()
    {
        var order = new Order
        {
            ProductCode = "PROD-001",
            Quantity = 10,
            UnitPrice = -100m
        };

        var result = _validator.Validate(order);

        Assert.That(result.IsValid, Is.False);
    }
```

`Ctrl+S` → `Ctrl+R, A` → ❌ 테스트 실패

#### 🔬 Lab 3-9: 단가 검증 추가 (Visual Studio)

`Validate` 메서드에서 상품코드 검증 후 다음 코드 추가:

```csharp
        // 단가 검증
        if (order.UnitPrice <= 0)
        {
            order.IsValid = false;
            order.ErrorMessage = "단가는 0원보다 커야 합니다.";
            return order;
        }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 6개 테스트 통과

#### 🔬 Lab 3-10: 커밋 및 Push (CMD)

```cmd
git add .
```
```cmd
git commit -m "feat: 단가 검증 기능 추가"
```
```cmd
git push origin main
```

---

### 3.4 OrderValidator 최종 코드

```csharp
using OrderSystem.App.Models;

namespace OrderSystem.App.Services;

public class OrderValidator
{
    public Order Validate(Order order)
    {
        // 수량 검증
        if (order.Quantity <= 0)
        {
            order.IsValid = false;
            order.ErrorMessage = "수량은 1개 이상이어야 합니다.";
            return order;
        }

        // 상품코드 검증
        if (string.IsNullOrWhiteSpace(order.ProductCode))
        {
            order.IsValid = false;
            order.ErrorMessage = "상품코드는 필수입니다.";
            return order;
        }

        // 단가 검증
        if (order.UnitPrice <= 0)
        {
            order.IsValid = false;
            order.ErrorMessage = "단가는 0원보다 커야 합니다.";
            return order;
        }

        order.IsValid = true;
        return order;
    }
}
```

---

## Part 4. TDD 실습 - 가격 계산


> 🎯 **목표**: PriceCalculator 클래스를 TDD로 구현합니다.

### 4.1 테스트 파일 생성

#### 🔬 Lab 4-1: PriceCalculatorTests.cs 생성 (Visual Studio)

1. `OrderSystem.Tests` 우클릭 → **추가** → **클래스**
2. 이름: `PriceCalculatorTests.cs`
3. 내용:

```csharp
using NUnit.Framework;
using OrderSystem.App.Models;
using OrderSystem.App.Services;

namespace OrderSystem.Tests;

[TestFixture]
public class PriceCalculatorTests
{
    private PriceCalculator _calculator;

    [SetUp]
    public void Setup()
    {
        _calculator = new PriceCalculator();
    }

    // TDD: 여기에 테스트를 추가합니다
}
```

`Ctrl+S`로 저장

---

### 4.2 TDD 사이클 #1: 기본 가격 계산

#### 🔴 RED: 기본 가격 테스트

#### 🔬 Lab 4-2: 기본 가격 계산 테스트 (Visual Studio)

`PriceCalculatorTests.cs`에 추가:

```csharp
    [Test]
    public void CalculateBasePrice_QuantityAndUnitPrice_ReturnsProduct()
    {
        // Arrange
        var order = new Order
        {
            Quantity = 5,
            UnitPrice = 10000m
        };

        // Act
        var result = _calculator.CalculateBasePrice(order);

        // Assert
        Assert.That(result, Is.EqualTo(50000m));
    }
```

`Ctrl+S` → `Ctrl+Shift+B` → ❌ 빌드 실패

---

#### 🟢 GREEN: 구현

#### 🔬 Lab 4-3: CalculateBasePrice 구현 (Visual Studio)

`PriceCalculator.cs`에 추가:

```csharp
    public decimal CalculateBasePrice(Order order)
    {
        return order.Quantity * order.UnitPrice;
    }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 통과

---

#### 🔵 REFACTOR: 추가 케이스

#### 🔬 Lab 4-4: 다양한 케이스 테스트 (Visual Studio)

```csharp
    [TestCase(1, 5000, 5000)]
    [TestCase(10, 1000, 10000)]
    [TestCase(100, 500, 50000)]
    public void CalculateBasePrice_VariousInputs_ReturnsExpected(
        int quantity, decimal unitPrice, decimal expected)
    {
        var order = new Order { Quantity = quantity, UnitPrice = unitPrice };
        
        var result = _calculator.CalculateBasePrice(order);
        
        Assert.That(result, Is.EqualTo(expected));
    }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 통과

#### 🔬 Lab 4-5: 커밋 및 Push (CMD)

```cmd
git add .
```
```cmd
git commit -m "feat: 기본 가격 계산 기능 구현"
```
```cmd
git push origin main
```

---

### 4.3 TDD 사이클 #2: 수량 할인

#### 비즈니스 규칙
- 10개 이상: 5% 할인
- 50개 이상: 10% 할인
- 100개 이상: 15% 할인

#### 🔴 RED: 수량 할인 테스트

#### 🔬 Lab 4-6: 10개 미만 할인 없음 테스트 (Visual Studio)

```csharp
    [Test]
    public void GetQuantityDiscountRate_LessThan10_ReturnsZero()
    {
        var order = new Order { Quantity = 5 };

        var result = _calculator.GetQuantityDiscountRate(order);

        Assert.That(result, Is.EqualTo(0m));
    }
```

`Ctrl+S` → `Ctrl+Shift+B` → ❌ 빌드 실패

---

#### 🟢 GREEN: 수량 할인 구현

#### 🔬 Lab 4-7: GetQuantityDiscountRate 구현 (Visual Studio)

`PriceCalculator.cs`에 추가:

```csharp
    public decimal GetQuantityDiscountRate(Order order)
    {
        if (order.Quantity >= 100) return 0.15m;
        if (order.Quantity >= 50) return 0.10m;
        if (order.Quantity >= 10) return 0.05m;
        return 0m;
    }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 통과

---

#### 🔵 REFACTOR: 모든 구간 테스트

#### 🔬 Lab 4-8: 수량 할인 구간 테스트 (Visual Studio)

```csharp
    [TestCase(5, 0)]        // 10개 미만: 할인 없음
    [TestCase(9, 0)]
    [TestCase(10, 0.05)]    // 10개 이상: 5% 할인
    [TestCase(49, 0.05)]
    [TestCase(50, 0.10)]    // 50개 이상: 10% 할인
    [TestCase(99, 0.10)]
    [TestCase(100, 0.15)]   // 100개 이상: 15% 할인
    [TestCase(500, 0.15)]
    public void GetQuantityDiscountRate_VariousQuantities_ReturnsExpected(
        int quantity, decimal expectedRate)
    {
        var order = new Order { Quantity = quantity };

        var result = _calculator.GetQuantityDiscountRate(order);

        Assert.That(result, Is.EqualTo(expectedRate));
    }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 모든 테스트 통과

#### 🔬 Lab 4-9: 커밋 및 Push (CMD)

```cmd
git add .
```
```cmd
git commit -m "feat: 수량 할인 기능 구현"
```
```cmd
git push origin main
```

---

### 4.4 TDD 사이클 #3: 회원 등급 할인

#### 비즈니스 규칙
- Normal: 할인 없음
- Silver: 3% 할인
- Gold: 5% 할인
- VIP: 10% 할인

#### 🔴 RED → 🟢 GREEN

#### 🔬 Lab 4-10: 회원 등급 할인 테스트 및 구현 (Visual Studio)

**테스트 추가:**

```csharp
    [TestCase(CustomerGrade.Normal, 0)]
    [TestCase(CustomerGrade.Silver, 0.03)]
    [TestCase(CustomerGrade.Gold, 0.05)]
    [TestCase(CustomerGrade.VIP, 0.10)]
    public void GetGradeDiscountRate_VariousGrades_ReturnsExpected(
        CustomerGrade grade, decimal expectedRate)
    {
        var order = new Order { CustomerGrade = grade };

        var result = _calculator.GetGradeDiscountRate(order);

        Assert.That(result, Is.EqualTo(expectedRate));
    }
```

`Ctrl+S` → `Ctrl+Shift+B` → ❌ 빌드 실패

**구현 추가 (PriceCalculator.cs):**

```csharp
    public decimal GetGradeDiscountRate(Order order)
    {
        return order.CustomerGrade switch
        {
            CustomerGrade.VIP => 0.10m,
            CustomerGrade.Gold => 0.05m,
            CustomerGrade.Silver => 0.03m,
            _ => 0m
        };
    }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 통과

#### 🔬 Lab 4-11: 커밋 및 Push (CMD)

```cmd
git add .
```
```cmd
git commit -m "feat: 회원 등급 할인 기능 구현"
```
```cmd
git push origin main
```

---

### 4.5 TDD 사이클 #4: 최종 가격 계산

#### 🔴 RED: 최종 가격 테스트

#### 🔬 Lab 4-12: 최종 가격 계산 테스트 (Visual Studio)

```csharp
    [Test]
    public void CalculateTotalPrice_WithAllDiscounts_ReturnsCorrectPrice()
    {
        // Arrange: 100개 × 1000원, VIP 회원
        // 기본 가격: 100,000원
        // 수량 할인(15%): 15,000원
        // 등급 할인(10%): 10,000원
        // 최종 가격: 100,000 - 15,000 - 10,000 = 75,000원
        var order = new Order
        {
            Quantity = 100,
            UnitPrice = 1000m,
            CustomerGrade = CustomerGrade.VIP
        };

        // Act
        var result = _calculator.CalculateTotalPrice(order);

        // Assert
        Assert.That(result, Is.EqualTo(75000m));
    }
```

`Ctrl+S` → `Ctrl+Shift+B` → ❌ 빌드 실패

---

#### 🟢 GREEN: 최종 가격 계산 구현

#### 🔬 Lab 4-13: CalculateTotalPrice 구현 (Visual Studio)

`PriceCalculator.cs`에 추가:

```csharp
    public decimal CalculateTotalPrice(Order order)
    {
        var basePrice = CalculateBasePrice(order);
        var quantityDiscount = basePrice * GetQuantityDiscountRate(order);
        var gradeDiscount = basePrice * GetGradeDiscountRate(order);
        
        return basePrice - quantityDiscount - gradeDiscount;
    }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 통과

---

#### 🔵 REFACTOR: 다양한 시나리오 테스트

#### 🔬 Lab 4-14: 다양한 시나리오 테스트 (Visual Studio)

```csharp
    [Test]
    public void CalculateTotalPrice_NormalCustomer_NoGradeDiscount()
    {
        // 10개 × 1000원, 일반 회원
        // 기본 가격: 10,000원
        // 수량 할인(5%): 500원
        // 등급 할인: 0원
        // 최종 가격: 9,500원
        var order = new Order
        {
            Quantity = 10,
            UnitPrice = 1000m,
            CustomerGrade = CustomerGrade.Normal
        };

        var result = _calculator.CalculateTotalPrice(order);

        Assert.That(result, Is.EqualTo(9500m));
    }

    [Test]
    public void CalculateTotalPrice_SmallOrder_NoQuantityDiscount()
    {
        // 5개 × 2000원, Gold 회원
        // 기본 가격: 10,000원
        // 수량 할인: 0원
        // 등급 할인(5%): 500원
        // 최종 가격: 9,500원
        var order = new Order
        {
            Quantity = 5,
            UnitPrice = 2000m,
            CustomerGrade = CustomerGrade.Gold
        };

        var result = _calculator.CalculateTotalPrice(order);

        Assert.That(result, Is.EqualTo(9500m));
    }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 모든 테스트 통과

#### 🔬 Lab 4-15: 커밋 및 Push (CMD)

```cmd
git add .
```
```cmd
git commit -m "feat: 최종 가격 계산 기능 구현"
```
```cmd
git push origin main
```

---

### 4.6 PriceCalculator 최종 코드

```csharp
using OrderSystem.App.Models;

namespace OrderSystem.App.Services;

public class PriceCalculator
{
    public decimal CalculateBasePrice(Order order)
    {
        return order.Quantity * order.UnitPrice;
    }

    public decimal GetQuantityDiscountRate(Order order)
    {
        if (order.Quantity >= 100) return 0.15m;
        if (order.Quantity >= 50) return 0.10m;
        if (order.Quantity >= 10) return 0.05m;
        return 0m;
    }

    public decimal GetGradeDiscountRate(Order order)
    {
        return order.CustomerGrade switch
        {
            CustomerGrade.VIP => 0.10m,
            CustomerGrade.Gold => 0.05m,
            CustomerGrade.Silver => 0.03m,
            _ => 0m
        };
    }

    public decimal CalculateTotalPrice(Order order)
    {
        var basePrice = CalculateBasePrice(order);
        var quantityDiscount = basePrice * GetQuantityDiscountRate(order);
        var gradeDiscount = basePrice * GetGradeDiscountRate(order);
        
        return basePrice - quantityDiscount - gradeDiscount;
    }
}
```

---

## Part 5. TDD 실습 - 주문 서비스


> 🎯 **목표**: OrderService 클래스를 TDD로 구현합니다.

### 5.1 테스트 파일 생성

#### 🔬 Lab 5-1: OrderServiceTests.cs 생성 (Visual Studio)

1. `OrderSystem.Tests` 우클릭 → **추가** → **클래스**
2. 이름: `OrderServiceTests.cs`
3. 내용:

```csharp
using NUnit.Framework;
using OrderSystem.App.Models;
using OrderSystem.App.Services;

namespace OrderSystem.Tests;

[TestFixture]
public class OrderServiceTests
{
    private OrderService _orderService;

    [SetUp]
    public void Setup()
    {
        _orderService = new OrderService();
    }
}
```

`Ctrl+S`로 저장

---

### 5.2 TDD 사이클 #1: 주문 생성

#### 🔴 RED: 유효한 주문 생성 테스트

#### 🔬 Lab 5-2: 유효한 주문 테스트 (Visual Studio)

```csharp
    [Test]
    public void CreateOrder_ValidOrder_ReturnsOrderWithTotalPrice()
    {
        // Arrange: 10개 × 1000원, Gold 회원
        var order = new Order
        {
            ProductCode = "PROD-001",
            Quantity = 10,
            UnitPrice = 1000m,
            CustomerGrade = CustomerGrade.Gold
        };

        // Act
        var result = _orderService.CreateOrder(order);

        // Assert
        Assert.That(result.IsValid, Is.True);
        Assert.That(result.TotalPrice, Is.GreaterThan(0));
    }
```

`Ctrl+S` → `Ctrl+Shift+B` → ❌ 빌드 실패

---

#### 🟢 GREEN: CreateOrder 구현

#### 🔬 Lab 5-3: OrderService 구현 (Visual Studio)

`OrderService.cs`를 다음으로 교체:

```csharp
using OrderSystem.App.Models;

namespace OrderSystem.App.Services;

public class OrderService
{
    private readonly OrderValidator _validator;
    private readonly PriceCalculator _calculator;

    public OrderService()
    {
        _validator = new OrderValidator();
        _calculator = new PriceCalculator();
    }

    public Order CreateOrder(Order order)
    {
        // 1. 주문 검증
        var validatedOrder = _validator.Validate(order);
        if (!validatedOrder.IsValid)
        {
            return validatedOrder;
        }

        // 2. 가격 계산
        order.TotalPrice = _calculator.CalculateTotalPrice(order);
        order.IsValid = true;

        return order;
    }
}
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 통과

---

#### 🔵 REFACTOR: 다양한 시나리오 테스트

#### 🔬 Lab 5-4: 무효한 주문 테스트 (Visual Studio)

```csharp
    [Test]
    public void CreateOrder_InvalidQuantity_ReturnsInvalidOrder()
    {
        var order = new Order
        {
            ProductCode = "PROD-001",
            Quantity = 0,
            UnitPrice = 1000m
        };

        var result = _orderService.CreateOrder(order);

        Assert.That(result.IsValid, Is.False);
        Assert.That(result.TotalPrice, Is.EqualTo(0));
    }

    [Test]
    public void CreateOrder_ValidVIPOrder_AppliesAllDiscounts()
    {
        // 100개 × 1000원, VIP
        // 기본: 100,000 - 수량할인(15%): 15,000 - 등급할인(10%): 10,000
        // = 75,000원
        var order = new Order
        {
            ProductCode = "PROD-001",
            Quantity = 100,
            UnitPrice = 1000m,
            CustomerGrade = CustomerGrade.VIP
        };

        var result = _orderService.CreateOrder(order);

        Assert.That(result.IsValid, Is.True);
        Assert.That(result.TotalPrice, Is.EqualTo(75000m));
    }
```

`Ctrl+S` → `Ctrl+R, A` → ✅ 모든 테스트 통과

---

#### 🚀 커밋 및 자동 배포

#### 🔬 Lab 5-5: 커밋 및 Push (CMD)

```cmd
git add .
```
```cmd
git commit -m "feat: 주문 서비스 구현"
```
```cmd
git push origin main
```

---

### 5.3 데모 프로그램 작성

#### 🔬 Lab 5-6: Program.cs 수정 (Visual Studio)

```csharp
using OrderSystem.App.Models;
using OrderSystem.App.Services;

Console.WriteLine("==========================================");
Console.WriteLine("       주문 관리 시스템 v1.0");
Console.WriteLine("       Built with TDD + CI/CD");
Console.WriteLine("==========================================");
Console.WriteLine();

var orderService = new OrderService();

// 주문 시나리오 1: VIP 고객 대량 주문
Console.WriteLine("[주문 1] VIP 고객 대량 주문");
Console.WriteLine("─────────────────────────────");
var order1 = new Order
{
    ProductCode = "LAPTOP-001",
    Quantity = 100,
    UnitPrice = 1500000m,
    CustomerGrade = CustomerGrade.VIP
};
var result1 = orderService.CreateOrder(order1);
PrintOrderResult(result1);

// 주문 시나리오 2: Gold 고객 중간 주문
Console.WriteLine("[주문 2] Gold 고객 중간 주문");
Console.WriteLine("─────────────────────────────");
var order2 = new Order
{
    ProductCode = "MOUSE-002",
    Quantity = 50,
    UnitPrice = 35000m,
    CustomerGrade = CustomerGrade.Gold
};
var result2 = orderService.CreateOrder(order2);
PrintOrderResult(result2);

// 주문 시나리오 3: 무효한 주문
Console.WriteLine("[주문 3] 무효한 주문 (수량 0)");
Console.WriteLine("─────────────────────────────");
var order3 = new Order
{
    ProductCode = "KEYBOARD-003",
    Quantity = 0,
    UnitPrice = 50000m,
    CustomerGrade = CustomerGrade.Normal
};
var result3 = orderService.CreateOrder(order3);
PrintOrderResult(result3);

Console.WriteLine();
Console.WriteLine("Press any key to exit...");
Console.ReadKey();

static void PrintOrderResult(Order order)
{
    if (order.IsValid)
    {
        Console.WriteLine($"  상품코드: {order.ProductCode}");
        Console.WriteLine($"  수량: {order.Quantity:N0}개");
        Console.WriteLine($"  단가: {order.UnitPrice:N0}원");
        Console.WriteLine($"  고객등급: {order.CustomerGrade}");
        Console.WriteLine($"  ▶ 최종가격: {order.TotalPrice:N0}원");
    }
    else
    {
        Console.WriteLine($"  ❌ 주문 실패: {order.ErrorMessage}");
    }
    Console.WriteLine();
}
```

`Ctrl+S`로 저장

#### 🔬 Lab 5-7: 커밋 및 Push (CMD)

```cmd
git add .
```
```cmd
git commit -m "feat: 주문 시스템 데모 프로그램 작성"
```
```cmd
git push origin main
```

#### 🔬 Lab 5-8: 최종 결과 확인 (CMD)

```cmd
C:\Deploy\OrderSystem\current\OrderSystem.App.exe
```

**예상 출력:**
```
==========================================
       주문 관리 시스템 v1.0
       Built with TDD + CI/CD
==========================================

[주문 1] VIP 고객 대량 주문
─────────────────────────────
  상품코드: LAPTOP-001
  수량: 100개
  단가: 1,500,000원
  고객등급: VIP
  ▶ 최종가격: 112,500,000원

[주문 2] Gold 고객 중간 주문
─────────────────────────────
  상품코드: MOUSE-002
  수량: 50개
  단가: 35,000원
  고객등급: Gold
  ▶ 최종가격: 1,487,500원

[주문 3] 무효한 주문 (수량 0)
─────────────────────────────
  ❌ 주문 실패: 수량은 1개 이상이어야 합니다.

Press any key to exit...
```

---

## Part 6. 종합 정리 및 마무리


### 6.1 태그 생성

#### 🔬 Lab 6-1: 릴리스 태그 (CMD)

```cmd
cd C:\CICDLab
```
```cmd
git tag -a v1.0.0 -m "Release 1.0.0 - 주문 관리 시스템"
```
```cmd
git tag
```

### 6.2 커밋 히스토리 확인

#### 🔬 Lab 6-2: Git 로그 확인 (CMD)

```cmd
git log --oneline
```

**예상 출력:**
```
abc1234 feat: 주문 시스템 데모 프로그램 작성
def5678 feat: 주문 서비스 구현
ghi9012 feat: 최종 가격 계산 기능 구현
jkl3456 feat: 회원 등급 할인 기능 구현
mno7890 feat: 수량 할인 기능 구현
pqr1234 feat: 기본 가격 계산 기능 구현
stu5678 feat: 단가 검증 기능 추가
vwx9012 feat: 상품코드 검증 기능 추가
yza3456 feat: 주문 수량 검증 기능 구현
bcd7890 feat: 주문 시스템 초기 프로젝트 구조 생성
```

### 6.3 배포 이력 확인

#### 🔬 Lab 6-3: 배포 버전 목록 (CMD)

```cmd
dir C:\Deploy\OrderSystem
```

10개 이상의 버전 폴더가 생성되어 있습니다 (각 Push마다 1개)

### 6.4 테스트 현황 확인

#### 🔬 Lab 6-4: 전체 테스트 확인 (Visual Studio)

`Ctrl+R, A`로 모든 테스트 실행

**테스트 목록:**
```
OrderValidatorTests (6개)
├── Validate_QuantityIsZero_ReturnsFalse
├── Validate_QuantityIsNegative_ReturnsFalse
├── Validate_QuantityIsPositive_ReturnsTrue
├── Validate_ProductCodeIsEmpty_ReturnsFalse
├── Validate_UnitPriceIsZero_ReturnsFalse
└── Validate_UnitPriceIsNegative_ReturnsFalse

PriceCalculatorTests (17개)
├── CalculateBasePrice_QuantityAndUnitPrice_ReturnsProduct
├── CalculateBasePrice_VariousInputs_ReturnsExpected (3 케이스)
├── GetQuantityDiscountRate_LessThan10_ReturnsZero
├── GetQuantityDiscountRate_VariousQuantities_ReturnsExpected (8 케이스)
├── GetGradeDiscountRate_VariousGrades_ReturnsExpected (4 케이스)
├── CalculateTotalPrice_WithAllDiscounts_ReturnsCorrectPrice
├── CalculateTotalPrice_NormalCustomer_NoGradeDiscount
└── CalculateTotalPrice_SmallOrder_NoQuantityDiscount

OrderServiceTests (3개)
├── CreateOrder_ValidOrder_ReturnsOrderWithTotalPrice
├── CreateOrder_InvalidQuantity_ReturnsInvalidOrder
└── CreateOrder_ValidVIPOrder_AppliesAllDiscounts

총 26개 테스트 통과!
```

### 6.5 학습 내용 정리

| 파트 | TDD 대상 | 구현 기능 | 테스트 수 |
|------|----------|-----------|-----------|
| Part 1 | - | 프로젝트 구조, 도메인 모델 | - |
| Part 2 | - | CI/CD 인프라 (Git 서버, Hook) | - |
| Part 3 | OrderValidator | 수량/상품코드/단가 검증 | 6개 |
| Part 4 | PriceCalculator | 기본가격/수량할인/등급할인/최종가격 | 17개 |
| Part 5 | OrderService | 주문 생성 및 처리 | 3개 |
| **합계** | | | **26개** |

### 6.6 비즈니스 로직 요약

```
┌────────────────────────────────────────────────────────┐
│                    주문 처리 흐름                       │
├────────────────────────────────────────────────────────┤
│                                                        │
│  1. OrderValidator.Validate()                          │
│     ├── 수량 > 0 체크                                  │
│     ├── 상품코드 필수 체크                              │
│     └── 단가 > 0 체크                                  │
│                    │                                   │
│                    ▼                                   │
│  2. PriceCalculator.CalculateTotalPrice()              │
│     ├── 기본가격 = 수량 × 단가                          │
│     ├── 수량할인 = 기본가격 × 할인율(5%/10%/15%)        │
│     ├── 등급할인 = 기본가격 × 할인율(3%/5%/10%)         │
│     └── 최종가격 = 기본가격 - 수량할인 - 등급할인        │
│                    │                                   │
│                    ▼                                   │
│  3. Order 반환 (IsValid, TotalPrice)                   │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 6.7 주요 명령어 정리

#### Git 명령어 (CMD)

```cmd
git init                              # 저장소 초기화
git add .                             # 모든 파일 스테이징
git commit -m "메시지"                 # 커밋
git push origin main                  # Push (자동 배포)
git log --oneline                     # 커밋 히스토리
git tag -a v1.0.0 -m "메시지"          # 태그 생성
```

#### Visual Studio 단축키

| 단축키 | 기능 | TDD 활용 |
|--------|------|----------|
| `Ctrl+Shift+B` | 빌드 | RED 단계 확인 |
| `Ctrl+R, A` | 모든 테스트 실행 | GREEN 단계 확인 |
| `Ctrl+R, T` | 현재 테스트 실행 | 개별 테스트 |
| `Ctrl+E, T` | 테스트 탐색기 | 결과 확인 |

### 6.8 다음 단계 학습 제안

- **의존성 주입**: 생성자 주입으로 테스트 용이성 향상
- **Mock 객체**: Moq 라이브러리로 외부 의존성 격리
- **통합 테스트**: 데이터베이스 연동 테스트
- **코드 커버리지**: coverlet으로 테스트 커버리지 측정

---

## 부록: 트러블슈팅

**Q: git push 시 post-receive hook이 실행되지 않습니다.**
```
A: hooks 폴더에 post-receive 파일 확인
   - 파일 이름에 확장자(.txt)가 없어야 함
   - 첫 줄이 #!/bin/sh 인지 확인
```

**Q: 테스트가 실패해서 배포가 안 됩니다.**
```
A: 이것이 TDD + CI/CD의 핵심입니다!
   모든 테스트가 통과해야만 배포됩니다.
   Visual Studio에서 실패한 테스트를 수정하세요.
```

**Q: decimal 타입 비교가 실패합니다.**
```
A: 소수점 정밀도 문제일 수 있습니다.
   Assert.That(result, Is.EqualTo(expected).Within(0.01m));
```

---

## 부록: TDD 베스트 프랙티스

### 테스트 이름 규칙

```
[메서드명]_[시나리오]_[예상결과]

예시:
- Validate_QuantityIsZero_ReturnsFalse
- CalculateTotalPrice_WithAllDiscounts_ReturnsCorrectPrice
- CreateOrder_InvalidQuantity_ReturnsInvalidOrder
```

### AAA 패턴

```csharp
[Test]
public void CreateOrder_ValidOrder_ReturnsOrderWithTotalPrice()
{
    // Arrange (준비)
    var order = new Order { ... };

    // Act (실행)
    var result = _orderService.CreateOrder(order);

    // Assert (검증)
    Assert.That(result.IsValid, Is.True);
}
```

### 비즈니스 규칙 = 테스트 케이스

```
비즈니스 규칙: "100개 이상 주문 시 15% 할인"
     ↓
테스트 케이스: GetQuantityDiscountRate_100OrMore_Returns15Percent
```

---

*— TDD CI/CD 실습 자료 끝 —*
