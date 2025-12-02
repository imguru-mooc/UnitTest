# 🧪 C# NUnit 단위 테스트 따라하기 실습

## Calculator Add 연산 테스트 - 처음부터 끝까지

> **실습 환경**: Visual Studio 2017
> **테스트 프레임워크**: NUnit 3.x
> **예상 소요 시간**: 약 60분

---

## 📋 목차

1. [실습 개요](#1-실습-개요)
2. [프로젝트 생성](#2-프로젝트-생성)
3. [NuGet 패키지 설치](#3-nuget-패키지-설치)
4. [Calculator 클래스 작성](#4-calculator-클래스-작성)
5. [첫 번째 테스트 작성](#5-첫-번째-테스트-작성)
6. [테스트 실행](#6-테스트-실행)
7. [다양한 테스트 케이스 추가](#7-다양한-테스트-케이스-추가)
8. [AAA 패턴 적용](#8-aaa-패턴-적용)
9. [파라미터화 테스트](#9-파라미터화-테스트)
10. [예외 테스트](#10-예외-테스트)
11. [SetUp과 TearDown](#11-setup과-teardown)
12. [전체 코드 정리](#12-전체-코드-정리)

---

## 1. 실습 개요

### 🎯 학습 목표

이 실습을 완료하면 다음을 할 수 있습니다:

- Visual Studio 2017에서 NUnit 테스트 프로젝트 생성
- 기본적인 단위 테스트 작성
- AAA 패턴 적용
- 파라미터화 테스트 작성
- 예외 테스트 작성

### 📁 프로젝트 구조 (최종)

```
CalculatorSolution/
├── Calculator/                    # 프로덕션 코드 프로젝트
│   ├── Calculator.cs
│   └── Calculator.csproj
└── Calculator.Tests/              # 테스트 프로젝트
    ├── CalculatorTests.cs
    └── Calculator.Tests.csproj
```

---

## 2. 프로젝트 생성

### Step 2.1: Visual Studio 2017 실행

1. **Visual Studio 2017**을 실행합니다.

### Step 2.2: 새 솔루션 생성

1. 메뉴에서 **파일(File)** → **새로 만들기(New)** → **프로젝트(Project)** 클릭

2. 다음과 같이 설정합니다:

```
┌─────────────────────────────────────────────────────────────────┐
│  새 프로젝트                                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  템플릿 선택:                                                   │
│  Visual C# → Windows 클래식 데스크톱 → 클래스 라이브러리       │
│  (.NET Framework)                                               │
│                                                                 │
│  이름(N): Calculator                                            │
│  위치(L): C:\Projects\                                          │
│  솔루션 이름(M): CalculatorSolution                             │
│                                                                 │
│  프레임워크: .NET Framework 4.6.1 (또는 4.5 이상)               │
│                                                                 │
│  ☑ 솔루션용 디렉터리 만들기                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

3. **확인** 버튼 클릭

> 💡 **팁**: .NET Framework 4.5 이상을 선택하세요. NUnit 3.x는 4.5 이상을 지원합니다.

### Step 2.3: 기본 파일 정리

1. 솔루션 탐색기에서 자동 생성된 `Class1.cs` 파일을 **삭제**합니다.
   - `Class1.cs` 우클릭 → **삭제(Delete)**

---

## 3. NuGet 패키지 설치

### Step 3.1: 테스트 프로젝트 추가

1. 솔루션 탐색기에서 **솔루션 'CalculatorSolution'** 우클릭
2. **추가(Add)** → **새 프로젝트(New Project)** 클릭
3. 다음과 같이 설정:

```
┌─────────────────────────────────────────────────────────────────┐
│  새 프로젝트                                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  템플릿 선택:                                                   │
│  Visual C# → 테스트 → 단위 테스트 프로젝트(.NET Framework)     │
│                                                                 │
│  이름(N): Calculator.Tests                                      │
│                                                                 │
│  프레임워크: .NET Framework 4.6.1 (Calculator와 동일)           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

4. **확인** 버튼 클릭

> ⚠️ **주의**: 테스트 프로젝트의 .NET Framework 버전은 Calculator 프로젝트와 동일해야 합니다.

### Step 3.2: NUnit 패키지 설치

1. 솔루션 탐색기에서 **Calculator.Tests** 프로젝트 우클릭
2. **NuGet 패키지 관리(Manage NuGet Packages)** 클릭
3. **찾아보기(Browse)** 탭 선택
4. 다음 패키지들을 검색하여 설치합니다:

#### 설치할 패키지 목록:

```
┌─────────────────────────────────────────────────────────────────┐
│  패키지 이름                    버전          설명              │
├─────────────────────────────────────────────────────────────────┤
│  NUnit                          3.12.0       테스트 프레임워크  │
│  NUnit3TestAdapter              3.15.1       VS 테스트 어댑터   │
│  Microsoft.NET.Test.Sdk         16.4.0       테스트 SDK         │
└─────────────────────────────────────────────────────────────────┘
```

**설치 방법:**

1. 검색창에 `NUnit` 입력
2. **NUnit** 선택 → **설치(Install)** 클릭
3. 라이선스 동의 → **확인**

4. 검색창에 `NUnit3TestAdapter` 입력
5. **NUnit3TestAdapter** 선택 → **설치(Install)** 클릭

6. 검색창에 `Microsoft.NET.Test.Sdk` 입력
7. **Microsoft.NET.Test.Sdk** 선택 → **설치(Install)** 클릭

> 💡 **팁**: VS 2017에서는 NUnit3TestAdapter가 있어야 테스트 탐색기에서 테스트를 인식합니다.

### Step 3.3: 프로젝트 참조 추가

Calculator.Tests 프로젝트에서 Calculator 프로젝트를 참조해야 합니다.

1. **Calculator.Tests** 프로젝트의 **참조(References)** 우클릭
2. **참조 추가(Add Reference)** 클릭
3. **프로젝트(Projects)** → **솔루션(Solution)** 선택
4. **Calculator** 체크 ☑
5. **확인** 클릭

```
┌─────────────────────────────────────────────────────────────────┐
│  참조 관리자                                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  프로젝트 > 솔루션                                              │
│                                                                 │
│  ☑ Calculator                                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Step 3.4: 불필요한 파일 정리

1. Calculator.Tests 프로젝트에서 자동 생성된 `UnitTest1.cs` 삭제
2. `packages.config` 파일이 있다면 그대로 둡니다 (NuGet 패키지 관리용)

---

## 4. Calculator 클래스 작성

### Step 4.1: Calculator 클래스 파일 생성

1. 솔루션 탐색기에서 **Calculator** 프로젝트 우클릭
2. **추가(Add)** → **클래스(Class)** 클릭
3. 이름: `Calculator.cs`
4. **추가** 클릭

### Step 4.2: 기본 코드 작성

`Calculator.cs` 파일을 다음과 같이 작성합니다:

```csharp
using System;

namespace Calculator
{
    /// <summary>
    /// 기본적인 사칙연산을 수행하는 계산기 클래스
    /// </summary>
    public class Calculator
    {
        /// <summary>
        /// 두 정수를 더합니다.
        /// </summary>
        /// <param name="a">첫 번째 숫자</param>
        /// <param name="b">두 번째 숫자</param>
        /// <returns>두 숫자의 합</returns>
        public int Add(int a, int b)
        {
            return a + b;
        }
    }
}
```

### Step 4.3: 빌드 확인

1. 메뉴에서 **빌드(Build)** → **솔루션 빌드(Build Solution)** 클릭
2. 또는 단축키 `Ctrl + Shift + B`
3. 출력 창에서 빌드 성공 확인

```
========== 빌드: 성공 2, 실패 0, 최신 0, 생략 0 ==========
```

---

## 5. 첫 번째 테스트 작성

### Step 5.1: 테스트 클래스 파일 생성

1. 솔루션 탐색기에서 **Calculator.Tests** 프로젝트 우클릭
2. **추가(Add)** → **클래스(Class)** 클릭
3. 이름: `CalculatorTests.cs`
4. **추가** 클릭

### Step 5.2: 첫 번째 테스트 코드 작성

`CalculatorTests.cs` 파일을 다음과 같이 작성합니다:

```csharp
using NUnit.Framework;

namespace Calculator.Tests
{
    [TestFixture]
    public class CalculatorTests
    {
        [Test]
        public void Add_TwoPositiveNumbers_ReturnsSum()
        {
            // Arrange (준비)
            var calculator = new Calculator();
            
            // Act (실행)
            int result = calculator.Add(2, 3);
            
            // Assert (검증)
            Assert.AreEqual(5, result);
        }
    }
}
```

### 코드 설명

```csharp
using NUnit.Framework;    // NUnit 프레임워크 사용
```

```csharp
[TestFixture]             // 이 클래스가 테스트를 포함함을 표시
public class CalculatorTests
```

```csharp
[Test]                    // 이 메서드가 테스트임을 표시
public void Add_TwoPositiveNumbers_ReturnsSum()
```

```csharp
// 테스트 메서드 명명 규칙: Method_Scenario_Expected
// Add                    - 테스트할 메서드
// TwoPositiveNumbers     - 시나리오 (두 양수)
// ReturnsSum             - 예상 결과 (합계 반환)
```

### Step 5.3: 빌드

1. `Ctrl + Shift + B`로 솔루션 빌드
2. 오류 없이 빌드되는지 확인

---

## 6. 테스트 실행

### Step 6.1: 테스트 탐색기 열기

1. 메뉴에서 **테스트(Test)** → **창(Windows)** → **테스트 탐색기(Test Explorer)** 클릭
2. 또는 단축키 `Ctrl + E, T`

```
┌─────────────────────────────────────────────────────────────────┐
│  테스트 탐색기                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🔍 검색                                                        │
│  ─────────────────────────────────────────────────────────────│
│                                                                 │
│  ▼ Calculator.Tests (1)                                        │
│    ▼ CalculatorTests (1)                                       │
│      ○ Add_TwoPositiveNumbers_ReturnsSum                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

> ⚠️ **테스트가 보이지 않는 경우:**
> 1. 솔루션을 다시 빌드해 보세요.
> 2. NUnit3TestAdapter가 설치되었는지 확인하세요.
> 3. Visual Studio를 재시작해 보세요.

### Step 6.2: 테스트 실행

**방법 1: 모든 테스트 실행**
- 테스트 탐색기에서 **모두 실행(Run All)** 클릭
- 또는 `Ctrl + R, A`

**방법 2: 특정 테스트 실행**
- 테스트 이름 우클릭 → **선택한 테스트 실행(Run Selected Tests)**
- 또는 테스트 메서드 내에서 `Ctrl + R, T`

**방법 3: 코드 편집기에서 실행**
- 테스트 메서드 위의 아이콘 클릭

### Step 6.3: 테스트 결과 확인

```
┌─────────────────────────────────────────────────────────────────┐
│  테스트 탐색기                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ 테스트 실행 완료: 1 통과, 0 실패                            │
│                                                                 │
│  ▼ 통과한 테스트 (1)                                           │
│    ✅ Add_TwoPositiveNumbers_ReturnsSum (15ms)                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

✅ **축하합니다!** 첫 번째 단위 테스트가 통과했습니다!

---

## 7. 다양한 테스트 케이스 추가

### Step 7.1: 음수 테스트

`CalculatorTests.cs`에 새 테스트를 추가합니다:

```csharp
[Test]
public void Add_TwoNegativeNumbers_ReturnsSum()
{
    // Arrange
    var calculator = new Calculator();
    
    // Act
    int result = calculator.Add(-2, -3);
    
    // Assert
    Assert.AreEqual(-5, result);
}
```

### Step 7.2: 양수와 음수 테스트

```csharp
[Test]
public void Add_PositiveAndNegativeNumber_ReturnsSum()
{
    // Arrange
    var calculator = new Calculator();
    
    // Act
    int result = calculator.Add(5, -3);
    
    // Assert
    Assert.AreEqual(2, result);
}
```

### Step 7.3: 0과의 덧셈 테스트

```csharp
[Test]
public void Add_NumberAndZero_ReturnsSameNumber()
{
    // Arrange
    var calculator = new Calculator();
    
    // Act
    int result = calculator.Add(5, 0);
    
    // Assert
    Assert.AreEqual(5, result);
}
```

### Step 7.4: 모든 테스트 실행

`Ctrl + R, A`로 모든 테스트를 실행합니다.

```
┌─────────────────────────────────────────────────────────────────┐
│  테스트 탐색기                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ 테스트 실행 완료: 4 통과, 0 실패                            │
│                                                                 │
│  ▼ 통과한 테스트 (4)                                           │
│    ✅ Add_TwoPositiveNumbers_ReturnsSum                         │
│    ✅ Add_TwoNegativeNumbers_ReturnsSum                         │
│    ✅ Add_PositiveAndNegativeNumber_ReturnsSum                  │
│    ✅ Add_NumberAndZero_ReturnsSameNumber                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. AAA 패턴 적용

### AAA 패턴이란?

```
┌─────────────────────────────────────────────────────────────────┐
│  AAA 패턴 (Arrange-Act-Assert)                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Arrange (준비)                                                 │
│  • 테스트에 필요한 객체와 데이터를 준비합니다.                 │
│  • 테스트 대상 객체를 생성합니다.                              │
│                                                                 │
│  Act (실행)                                                     │
│  • 테스트하려는 동작을 실행합니다.                             │
│  • 보통 한 줄의 코드입니다.                                    │
│                                                                 │
│  Assert (검증)                                                  │
│  • 실행 결과가 예상과 일치하는지 확인합니다.                   │
│  • Assert 메서드를 사용합니다.                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Step 8.1: AAA 패턴 적용 예시

```csharp
[Test]
public void Add_LargeNumbers_ReturnsCorrectSum()
{
    // ========================================
    // Arrange (준비)
    // ========================================
    // 테스트에 필요한 객체 생성
    var calculator = new Calculator();
    int number1 = 1000000;
    int number2 = 2000000;
    int expected = 3000000;
    
    // ========================================
    // Act (실행)
    // ========================================
    // 테스트 대상 메서드 호출
    int actual = calculator.Add(number1, number2);
    
    // ========================================
    // Assert (검증)
    // ========================================
    // 결과 확인
    Assert.AreEqual(expected, actual);
}
```

### Step 8.2: 다양한 Assert 메서드

```csharp
[Test]
public void Add_AssertExamples()
{
    var calculator = new Calculator();
    int result = calculator.Add(2, 3);
    
    // 같음 확인
    Assert.AreEqual(5, result);
    
    // 참 확인
    Assert.IsTrue(result == 5);
    
    // 거짓 확인
    Assert.IsFalse(result == 0);
    
    // null 아님 확인
    Assert.IsNotNull(calculator);
    
    // 범위 확인
    Assert.That(result, Is.GreaterThan(0));
    Assert.That(result, Is.LessThan(10));
    Assert.That(result, Is.InRange(1, 10));
}
```

---

## 9. 파라미터화 테스트

### 문제점: 중복 코드

현재 테스트들을 보면 많은 중복이 있습니다:

```csharp
// 비슷한 구조의 테스트가 반복됨
[Test]
public void Add_TwoPositiveNumbers_ReturnsSum() { ... }

[Test]
public void Add_TwoNegativeNumbers_ReturnsSum() { ... }

[Test]
public void Add_PositiveAndNegativeNumber_ReturnsSum() { ... }
```

### 해결책: TestCase 속성

### Step 9.1: TestCase로 파라미터화

```csharp
[TestCase(2, 3, 5)]           // 양수 + 양수
[TestCase(-2, -3, -5)]        // 음수 + 음수
[TestCase(5, -3, 2)]          // 양수 + 음수
[TestCase(-5, 3, -2)]         // 음수 + 양수
[TestCase(0, 0, 0)]           // 0 + 0
[TestCase(5, 0, 5)]           // 숫자 + 0
[TestCase(0, 5, 5)]           // 0 + 숫자
public void Add_VariousInputs_ReturnsExpectedSum(int a, int b, int expected)
{
    // Arrange
    var calculator = new Calculator();
    
    // Act
    int result = calculator.Add(a, b);
    
    // Assert
    Assert.AreEqual(expected, result);
}
```

### Step 9.2: 테스트 실행 결과

```
┌─────────────────────────────────────────────────────────────────┐
│  테스트 탐색기                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ▼ Add_VariousInputs_ReturnsExpectedSum (7)                     │
│    ✅ Add_VariousInputs_ReturnsExpectedSum(2, 3, 5)             │
│    ✅ Add_VariousInputs_ReturnsExpectedSum(-2, -3, -5)          │
│    ✅ Add_VariousInputs_ReturnsExpectedSum(5, -3, 2)            │
│    ✅ Add_VariousInputs_ReturnsExpectedSum(-5, 3, -2)           │
│    ✅ Add_VariousInputs_ReturnsExpectedSum(0, 0, 0)             │
│    ✅ Add_VariousInputs_ReturnsExpectedSum(5, 0, 5)             │
│    ✅ Add_VariousInputs_ReturnsExpectedSum(0, 5, 5)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Step 9.3: TestCaseSource 사용 (고급)

더 복잡한 테스트 데이터는 별도 소스로 분리할 수 있습니다:

```csharp
public class CalculatorTests
{
    // 테스트 데이터 소스
    private static object[] AddTestCases =
    {
        new object[] { 2, 3, 5 },
        new object[] { -2, -3, -5 },
        new object[] { 100, 200, 300 },
        new object[] { int.MaxValue - 1, 1, int.MaxValue }
    };
    
    [TestCaseSource(nameof(AddTestCases))]
    public void Add_WithTestCaseSource_ReturnsExpectedSum(int a, int b, int expected)
    {
        // Arrange
        var calculator = new Calculator();
        
        // Act
        int result = calculator.Add(a, b);
        
        // Assert
        Assert.AreEqual(expected, result);
    }
}
```

---

## 10. 예외 테스트

### Step 10.1: Calculator에 나눗셈 추가

`Calculator.cs`에 Divide 메서드를 추가합니다:

```csharp
using System;

namespace Calculator
{
    public class Calculator
    {
        public int Add(int a, int b)
        {
            return a + b;
        }
        
        /// <summary>
        /// 두 정수를 나눕니다.
        /// </summary>
        /// <param name="dividend">피제수 (나눠지는 수)</param>
        /// <param name="divisor">제수 (나누는 수)</param>
        /// <returns>나눗셈 결과</returns>
        /// <exception cref="DivideByZeroException">divisor가 0일 때</exception>
        public int Divide(int dividend, int divisor)
        {
            if (divisor == 0)
            {
                throw new DivideByZeroException("0으로 나눌 수 없습니다.");
            }
            
            return dividend / divisor;
        }
    }
}
```

### Step 10.2: 예외 테스트 작성

#### 방법 1: Assert.Throws 사용 (권장)

```csharp
[Test]
public void Divide_ByZero_ThrowsDivideByZeroException()
{
    // Arrange
    var calculator = new Calculator();
    
    // Act & Assert
    var exception = Assert.Throws<DivideByZeroException>(() => 
    {
        calculator.Divide(10, 0);
    });
    
    // 예외 메시지 검증 (선택사항)
    Assert.That(exception.Message, Does.Contain("0으로 나눌 수 없습니다"));
}
```

#### 방법 2: ExpectedException 속성 사용

```csharp
[Test]
public void Divide_ByZero_ThrowsException_AttributeStyle()
{
    // Arrange
    var calculator = new Calculator();
    
    // Act & Assert
    Assert.Throws<DivideByZeroException>(() => calculator.Divide(10, 0));
}
```

### Step 10.3: 정상 나눗셈 테스트

```csharp
[TestCase(10, 2, 5)]
[TestCase(9, 3, 3)]
[TestCase(-10, 2, -5)]
[TestCase(10, -2, -5)]
[TestCase(0, 5, 0)]
public void Divide_ValidInputs_ReturnsExpectedResult(int dividend, int divisor, int expected)
{
    // Arrange
    var calculator = new Calculator();
    
    // Act
    int result = calculator.Divide(dividend, divisor);
    
    // Assert
    Assert.AreEqual(expected, result);
}
```

---

## 11. SetUp과 TearDown

### 문제점: 매 테스트마다 Calculator 생성

```csharp
[Test]
public void Test1()
{
    var calculator = new Calculator();  // 반복!
    // ...
}

[Test]
public void Test2()
{
    var calculator = new Calculator();  // 반복!
    // ...
}
```

### 해결책: SetUp 사용

### Step 11.1: SetUp 메서드 추가

```csharp
using NUnit.Framework;

namespace Calculator.Tests
{
    [TestFixture]
    public class CalculatorTests
    {
        // 테스트 간 공유되는 필드
        private Calculator _calculator;
        
        [SetUp]
        public void SetUp()
        {
            // 각 테스트 실행 전에 호출됨
            _calculator = new Calculator();
        }
        
        [TearDown]
        public void TearDown()
        {
            // 각 테스트 실행 후에 호출됨
            // 정리 작업 (예: 파일 삭제, 연결 해제 등)
            _calculator = null;
        }
        
        [Test]
        public void Add_TwoPositiveNumbers_ReturnsSum()
        {
            // Arrange - SetUp에서 이미 준비됨
            
            // Act
            int result = _calculator.Add(2, 3);
            
            // Assert
            Assert.AreEqual(5, result);
        }
        
        [Test]
        public void Add_TwoNegativeNumbers_ReturnsSum()
        {
            // Act
            int result = _calculator.Add(-2, -3);
            
            // Assert
            Assert.AreEqual(-5, result);
        }
    }
}
```

### Step 11.2: 실행 순서

```
┌─────────────────────────────────────────────────────────────────┐
│  NUnit 실행 순서                                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [OneTimeSetUp]     ──→ 클래스 시작 시 1회 실행                │
│        │                                                        │
│        ▼                                                        │
│  ┌─────────────────────────────────────────────────────┐       │
│  │  [SetUp]         ──→ 각 테스트 전 실행              │       │
│  │        │                                            │       │
│  │        ▼                                            │       │
│  │  [Test] 메서드   ──→ 테스트 실행                   │ 반복  │
│  │        │                                            │       │
│  │        ▼                                            │       │
│  │  [TearDown]      ──→ 각 테스트 후 실행              │       │
│  └─────────────────────────────────────────────────────┘       │
│        │                                                        │
│        ▼                                                        │
│  [OneTimeTearDown]  ──→ 클래스 종료 시 1회 실행                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Step 11.3: OneTimeSetUp/OneTimeTearDown 예시

```csharp
[TestFixture]
public class CalculatorTests
{
    private Calculator _calculator;
    
    [OneTimeSetUp]
    public void OneTimeSetUp()
    {
        // 모든 테스트 전에 한 번만 실행
        // 예: 데이터베이스 연결, 설정 파일 로드
        Console.WriteLine("===== 테스트 시작 =====");
    }
    
    [OneTimeTearDown]
    public void OneTimeTearDown()
    {
        // 모든 테스트 후에 한 번만 실행
        // 예: 데이터베이스 연결 해제
        Console.WriteLine("===== 테스트 종료 =====");
    }
    
    [SetUp]
    public void SetUp()
    {
        // 각 테스트 전에 실행
        _calculator = new Calculator();
    }
    
    [TearDown]
    public void TearDown()
    {
        // 각 테스트 후에 실행
        _calculator = null;
    }
    
    // 테스트 메서드들...
}
```

---

## 12. 전체 코드 정리

### Calculator.cs (최종)

```csharp
using System;

namespace Calculator
{
    /// <summary>
    /// 기본적인 사칙연산을 수행하는 계산기 클래스
    /// </summary>
    public class Calculator
    {
        /// <summary>
        /// 두 정수를 더합니다.
        /// </summary>
        /// <param name="a">첫 번째 숫자</param>
        /// <param name="b">두 번째 숫자</param>
        /// <returns>두 숫자의 합</returns>
        public int Add(int a, int b)
        {
            return a + b;
        }
        
        /// <summary>
        /// 두 정수를 뺍니다.
        /// </summary>
        /// <param name="a">첫 번째 숫자</param>
        /// <param name="b">두 번째 숫자</param>
        /// <returns>두 숫자의 차</returns>
        public int Subtract(int a, int b)
        {
            return a - b;
        }
        
        /// <summary>
        /// 두 정수를 곱합니다.
        /// </summary>
        /// <param name="a">첫 번째 숫자</param>
        /// <param name="b">두 번째 숫자</param>
        /// <returns>두 숫자의 곱</returns>
        public int Multiply(int a, int b)
        {
            return a * b;
        }
        
        /// <summary>
        /// 두 정수를 나눕니다.
        /// </summary>
        /// <param name="dividend">피제수 (나눠지는 수)</param>
        /// <param name="divisor">제수 (나누는 수)</param>
        /// <returns>나눗셈 결과</returns>
        /// <exception cref="DivideByZeroException">divisor가 0일 때</exception>
        public int Divide(int dividend, int divisor)
        {
            if (divisor == 0)
            {
                throw new DivideByZeroException("0으로 나눌 수 없습니다.");
            }
            
            return dividend / divisor;
        }
    }
}
```

### CalculatorTests.cs (최종)

```csharp
using System;
using NUnit.Framework;

namespace Calculator.Tests
{
    [TestFixture]
    public class CalculatorTests
    {
        private Calculator _calculator;
        
        #region SetUp / TearDown
        
        [OneTimeSetUp]
        public void OneTimeSetUp()
        {
            // 전체 테스트 시작 전 1회 실행
            Console.WriteLine("===== Calculator 테스트 시작 =====");
        }
        
        [OneTimeTearDown]
        public void OneTimeTearDown()
        {
            // 전체 테스트 종료 후 1회 실행
            Console.WriteLine("===== Calculator 테스트 종료 =====");
        }
        
        [SetUp]
        public void SetUp()
        {
            // 각 테스트 전에 실행
            _calculator = new Calculator();
        }
        
        [TearDown]
        public void TearDown()
        {
            // 각 테스트 후에 실행
            _calculator = null;
        }
        
        #endregion
        
        #region Add 테스트
        
        [Test]
        public void Add_TwoPositiveNumbers_ReturnsSum()
        {
            // Arrange - SetUp에서 준비됨
            
            // Act
            int result = _calculator.Add(2, 3);
            
            // Assert
            Assert.AreEqual(5, result);
        }
        
        [TestCase(2, 3, 5, Description = "양수 + 양수")]
        [TestCase(-2, -3, -5, Description = "음수 + 음수")]
        [TestCase(5, -3, 2, Description = "양수 + 음수")]
        [TestCase(-5, 3, -2, Description = "음수 + 양수")]
        [TestCase(0, 0, 0, Description = "0 + 0")]
        [TestCase(5, 0, 5, Description = "숫자 + 0")]
        [TestCase(0, 5, 5, Description = "0 + 숫자")]
        [TestCase(1000000, 2000000, 3000000, Description = "큰 숫자")]
        public void Add_VariousInputs_ReturnsExpectedSum(int a, int b, int expected)
        {
            // Act
            int result = _calculator.Add(a, b);
            
            // Assert
            Assert.AreEqual(expected, result);
        }
        
        #endregion
        
        #region Subtract 테스트
        
        [TestCase(5, 3, 2)]
        [TestCase(3, 5, -2)]
        [TestCase(-5, -3, -2)]
        [TestCase(0, 5, -5)]
        public void Subtract_VariousInputs_ReturnsExpectedDifference(int a, int b, int expected)
        {
            // Act
            int result = _calculator.Subtract(a, b);
            
            // Assert
            Assert.AreEqual(expected, result);
        }
        
        #endregion
        
        #region Multiply 테스트
        
        [TestCase(2, 3, 6)]
        [TestCase(-2, 3, -6)]
        [TestCase(-2, -3, 6)]
        [TestCase(5, 0, 0)]
        [TestCase(0, 5, 0)]
        public void Multiply_VariousInputs_ReturnsExpectedProduct(int a, int b, int expected)
        {
            // Act
            int result = _calculator.Multiply(a, b);
            
            // Assert
            Assert.AreEqual(expected, result);
        }
        
        #endregion
        
        #region Divide 테스트
        
        [TestCase(10, 2, 5)]
        [TestCase(9, 3, 3)]
        [TestCase(-10, 2, -5)]
        [TestCase(10, -2, -5)]
        [TestCase(-10, -2, 5)]
        [TestCase(0, 5, 0)]
        public void Divide_ValidInputs_ReturnsExpectedResult(int dividend, int divisor, int expected)
        {
            // Act
            int result = _calculator.Divide(dividend, divisor);
            
            // Assert
            Assert.AreEqual(expected, result);
        }
        
        [Test]
        public void Divide_ByZero_ThrowsDivideByZeroException()
        {
            // Act & Assert
            var exception = Assert.Throws<DivideByZeroException>(() => 
            {
                _calculator.Divide(10, 0);
            });
            
            // 예외 메시지 검증
            Assert.That(exception.Message, Does.Contain("0으로 나눌 수 없습니다"));
        }
        
        #endregion
    }
}
```

---

## 📌 요약 체크리스트

### 이 실습에서 배운 것

```
✅ Visual Studio 2017에서 NUnit 테스트 프로젝트 생성
✅ NuGet으로 NUnit, NUnit3TestAdapter 설치
✅ [TestFixture], [Test] 속성 사용
✅ Assert.AreEqual() 등 검증 메서드
✅ AAA 패턴 (Arrange-Act-Assert)
✅ [TestCase]로 파라미터화 테스트
✅ Assert.Throws<T>()로 예외 테스트
✅ [SetUp], [TearDown] 설정/정리
✅ [OneTimeSetUp], [OneTimeTearDown]
```

### NUnit 주요 속성 정리

| 속성 | 설명 |
|------|------|
| `[TestFixture]` | 테스트 클래스 표시 |
| `[Test]` | 테스트 메서드 표시 |
| `[TestCase(값...)]` | 파라미터화 테스트 |
| `[TestCaseSource]` | 외부 데이터 소스 |
| `[SetUp]` | 각 테스트 전 실행 |
| `[TearDown]` | 각 테스트 후 실행 |
| `[OneTimeSetUp]` | 전체 테스트 전 1회 |
| `[OneTimeTearDown]` | 전체 테스트 후 1회 |
| `[Ignore("이유")]` | 테스트 건너뛰기 |
| `[Category("카테고리")]` | 테스트 분류 |

### Assert 주요 메서드 정리

| 메서드 | 설명 |
|--------|------|
| `Assert.AreEqual(expected, actual)` | 같음 확인 |
| `Assert.AreNotEqual(expected, actual)` | 다름 확인 |
| `Assert.IsTrue(condition)` | 참 확인 |
| `Assert.IsFalse(condition)` | 거짓 확인 |
| `Assert.IsNull(object)` | null 확인 |
| `Assert.IsNotNull(object)` | not null 확인 |
| `Assert.Throws<T>(code)` | 예외 발생 확인 |
| `Assert.That(actual, constraint)` | 제약 조건 확인 |

---

## 🎯 다음 단계

이 실습을 완료했다면, 다음을 시도해 보세요:

1. **Moq 라이브러리 학습** - 의존성 격리를 위한 목킹
2. **TDD 실습** - 테스트를 먼저 작성하는 개발 방법
3. **코드 커버리지 측정** - Coverlet + ReportGenerator
4. **CI/CD 통합** - GitHub Actions로 자동 테스트

---

**수고하셨습니다! 🎉**

이제 여러분은 NUnit으로 단위 테스트를 작성할 수 있습니다.
실제 프로젝트에 적용하며 지속적으로 연습하세요!
