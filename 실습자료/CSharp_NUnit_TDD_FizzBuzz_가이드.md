# 🧪 C# NUnit TDD로 FizzBuzz 개발하기

> NUnit 프레임워크를 사용한 Test-Driven Development(TDD)로 FizzBuzz를 단계별로 구현합니다.

---

## 📋 목차

- [TDD 소개](#tdd-소개)
- [프로젝트 설정](#프로젝트-설정)
- [NUnit vs xUnit 비교](#nunit-vs-xunit-비교)
- [TDD 사이클 1: 숫자 반환](#tdd-사이클-1-숫자-반환)
- [TDD 사이클 2: Fizz 구현](#tdd-사이클-2-fizz-구현)
- [TDD 사이클 3: Buzz 구현](#tdd-사이클-3-buzz-구현)
- [TDD 사이클 4: FizzBuzz 구현](#tdd-사이클-4-fizzbuzz-구현)
- [리팩토링](#리팩토링)
- [전체 코드](#전체-코드)

---

## TDD 소개

### TDD란?

**Test-Driven Development(테스트 주도 개발)**는 테스트를 먼저 작성하고, 그 테스트를 통과하는 코드를 작성하는 개발 방법론입니다.

### TDD 3단계 사이클

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│    🔴 RED          🟢 GREEN         🔵 REFACTOR    │
│                                                     │
│  실패하는        테스트를           코드를          │
│  테스트 작성  →  통과시키는    →    개선하기        │
│                  최소 코드 작성                     │
│                                                     │
│         ←─────────────────────────────────┘         │
│                    반복                             │
└─────────────────────────────────────────────────────┘
```

### FizzBuzz 규칙

| 조건 | 출력 |
|------|------|
| 3으로 나누어 떨어짐 | "Fizz" |
| 5로 나누어 떨어짐 | "Buzz" |
| 3과 5 모두로 나누어 떨어짐 | "FizzBuzz" |
| 그 외 | 숫자 그대로 |

**예시**: 1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz, 11, Fizz, 13, 14, FizzBuzz, 16...

---

## 프로젝트 설정

### Step 1: 솔루션 생성

```bash
# 솔루션 폴더 생성
mkdir FizzBuzzTDD
cd FizzBuzzTDD

# 솔루션 생성
dotnet new sln -n FizzBuzzTDD

# 프로젝트 생성
dotnet new classlib -n FizzBuzz
dotnet new nunit -n FizzBuzz.Tests    # NUnit 테스트 프로젝트

# 솔루션에 프로젝트 추가
dotnet sln add FizzBuzz/FizzBuzz.csproj
dotnet sln add FizzBuzz.Tests/FizzBuzz.Tests.csproj

# 테스트 프로젝트에서 메인 프로젝트 참조
dotnet add FizzBuzz.Tests/FizzBuzz.Tests.csproj reference FizzBuzz/FizzBuzz.csproj
```

### Step 2: NUnit 패키지 확인

**FizzBuzz.Tests/FizzBuzz.Tests.csproj**:
```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="NUnit" Version="3.14.0" />
    <PackageReference Include="NUnit3TestAdapter" Version="4.5.0" />
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\FizzBuzz\FizzBuzz.csproj" />
  </ItemGroup>

</Project>
```

### Step 3: 프로젝트 구조

```
FizzBuzzTDD/
├── FizzBuzzTDD.sln
├── FizzBuzz/
│   ├── FizzBuzz.csproj
│   └── FizzBuzzGame.cs        ← 구현 코드
└── FizzBuzz.Tests/
    ├── FizzBuzz.Tests.csproj
    └── FizzBuzzTests.cs       ← 테스트 코드
```

### Step 4: 기본 클래스 생성

**FizzBuzz/FizzBuzzGame.cs** - 빈 클래스로 시작:
```csharp
namespace FizzBuzz;

public class FizzBuzzGame
{
    // 아직 아무 코드도 없음
}
```

**FizzBuzz.Tests/FizzBuzzTests.cs** - 빈 테스트 클래스:
```csharp
using NUnit.Framework;

namespace FizzBuzz.Tests;

[TestFixture]
public class FizzBuzzTests
{
    // 테스트를 여기에 작성할 예정
}
```

---

## NUnit vs xUnit 비교

| 기능 | NUnit | xUnit |
|------|-------|-------|
| 테스트 클래스 | `[TestFixture]` | 없음 |
| 단일 테스트 | `[Test]` | `[Fact]` |
| 파라미터 테스트 | `[TestCase]` | `[Theory]` + `[InlineData]` |
| 초기화 | `[SetUp]` | 생성자 |
| 정리 | `[TearDown]` | `IDisposable` |
| Assert 스타일 | `Assert.That()` / `Assert.AreEqual()` | `Assert.Equal()` |

### NUnit Assert 스타일

```csharp
// Classic 스타일
Assert.AreEqual(expected, actual);
Assert.IsTrue(condition);
Assert.IsNotNull(obj);

// Constraint 스타일 (권장)
Assert.That(actual, Is.EqualTo(expected));
Assert.That(condition, Is.True);
Assert.That(obj, Is.Not.Null);
```

---

## TDD 사이클 1: 숫자 반환

> **목표**: 3이나 5로 나누어 떨어지지 않으면 숫자를 그대로 반환

### 🔴 RED: 실패하는 테스트 작성

**FizzBuzz.Tests/FizzBuzzTests.cs**:
```csharp
using NUnit.Framework;

namespace FizzBuzz.Tests;

[TestFixture]
public class FizzBuzzTests
{
    private FizzBuzzGame _game;

    [SetUp]
    public void Setup()
    {
        _game = new FizzBuzzGame();
    }

    [Test]
    public void Given1_ShouldReturn1()
    {
        // Arrange - SetUp에서 처리됨
        
        // Act
        var result = _game.Convert(1);
        
        // Assert
        Assert.That(result, Is.EqualTo("1"));
    }
}
```

**테스트 실행**:
```bash
dotnet test
```

**결과**: ❌ 실패 (Convert 메서드가 없음)
```
Error CS1061: 'FizzBuzzGame' does not contain a definition for 'Convert'
```

### 🟢 GREEN: 테스트 통과시키기

**FizzBuzz/FizzBuzzGame.cs**:
```csharp
namespace FizzBuzz;

public class FizzBuzzGame
{
    public string Convert(int number)
    {
        return "1";  // 가장 단순한 구현
    }
}
```

**테스트 실행**:
```bash
dotnet test
```

**결과**: ✅ 통과

### 🔴 RED: 더 많은 테스트 추가

```csharp
[Test]
public void Given2_ShouldReturn2()
{
    var result = _game.Convert(2);
    Assert.That(result, Is.EqualTo("2"));
}
```

**결과**: ❌ 실패 (항상 "1"만 반환하므로)

### 🟢 GREEN: 일반화

**FizzBuzz/FizzBuzzGame.cs**:
```csharp
namespace FizzBuzz;

public class FizzBuzzGame
{
    public string Convert(int number)
    {
        return number.ToString();  // 숫자를 문자열로 변환
    }
}
```

**테스트 실행**:
```bash
dotnet test
```

**결과**: ✅ 모든 테스트 통과

### 📝 TestCase로 리팩토링

```csharp
using NUnit.Framework;

namespace FizzBuzz.Tests;

[TestFixture]
public class FizzBuzzTests
{
    private FizzBuzzGame _game;

    [SetUp]
    public void Setup()
    {
        _game = new FizzBuzzGame();
    }

    [TestCase(1, "1")]
    [TestCase(2, "2")]
    [TestCase(4, "4")]
    public void GivenNormalNumber_ShouldReturnNumberAsString(int input, string expected)
    {
        var result = _game.Convert(input);
        Assert.That(result, Is.EqualTo(expected));
    }
}
```

> 💡 **TIP**: `[TestCase]`를 사용하면 여러 케이스를 하나의 테스트 메서드로 작성할 수 있습니다.

---

## TDD 사이클 2: Fizz 구현

> **목표**: 3으로 나누어 떨어지면 "Fizz" 반환

### 🔴 RED: 실패하는 테스트 작성

```csharp
[Test]
public void Given3_ShouldReturnFizz()
{
    var result = _game.Convert(3);
    Assert.That(result, Is.EqualTo("Fizz"));
}
```

**테스트 실행**:
```bash
dotnet test
```

**결과**: ❌ 실패
```
Expected: "Fizz"
But was:  "3"
```

### 🟢 GREEN: 테스트 통과시키기

**FizzBuzz/FizzBuzzGame.cs**:
```csharp
namespace FizzBuzz;

public class FizzBuzzGame
{
    public string Convert(int number)
    {
        if (number == 3)
            return "Fizz";
            
        return number.ToString();
    }
}
```

**결과**: ✅ 통과

### 🔴 RED: 6에 대한 테스트 추가

```csharp
[Test]
public void Given6_ShouldReturnFizz()
{
    var result = _game.Convert(6);
    Assert.That(result, Is.EqualTo("Fizz"));
}
```

**결과**: ❌ 실패 (6은 3이 아니므로)

### 🟢 GREEN: 3의 배수로 일반화

**FizzBuzz/FizzBuzzGame.cs**:
```csharp
namespace FizzBuzz;

public class FizzBuzzGame
{
    public string Convert(int number)
    {
        if (number % 3 == 0)  // 3의 배수
            return "Fizz";
            
        return number.ToString();
    }
}
```

**결과**: ✅ 모든 테스트 통과

### 📝 TestCase로 정리

```csharp
[TestCase(3)]
[TestCase(6)]
[TestCase(9)]
[TestCase(12)]
public void GivenMultipleOf3_ShouldReturnFizz(int input)
{
    var result = _game.Convert(input);
    Assert.That(result, Is.EqualTo("Fizz"));
}
```

---

## TDD 사이클 3: Buzz 구현

> **목표**: 5로 나누어 떨어지면 "Buzz" 반환

### 🔴 RED: 실패하는 테스트 작성

```csharp
[TestCase(5)]
[TestCase(10)]
[TestCase(20)]
public void GivenMultipleOf5_ShouldReturnBuzz(int input)
{
    var result = _game.Convert(input);
    Assert.That(result, Is.EqualTo("Buzz"));
}
```

**테스트 실행**:
```bash
dotnet test
```

**결과**: ❌ 실패
```
Expected: "Buzz"
But was:  "5"
```

### 🟢 GREEN: 테스트 통과시키기

**FizzBuzz/FizzBuzzGame.cs**:
```csharp
namespace FizzBuzz;

public class FizzBuzzGame
{
    public string Convert(int number)
    {
        if (number % 3 == 0)
            return "Fizz";
            
        if (number % 5 == 0)  // 5의 배수 추가
            return "Buzz";
            
        return number.ToString();
    }
}
```

**테스트 실행**:
```bash
dotnet test
```

**결과**: ✅ 모든 테스트 통과

---

## TDD 사이클 4: FizzBuzz 구현

> **목표**: 3과 5 모두로 나누어 떨어지면 "FizzBuzz" 반환

### 🔴 RED: 실패하는 테스트 작성

```csharp
[TestCase(15)]
[TestCase(30)]
[TestCase(45)]
public void GivenMultipleOf3And5_ShouldReturnFizzBuzz(int input)
{
    var result = _game.Convert(input);
    Assert.That(result, Is.EqualTo("FizzBuzz"));
}
```

**테스트 실행**:
```bash
dotnet test
```

**결과**: ❌ 실패
```
Expected: "FizzBuzz"
But was:  "Fizz"
```

> 💡 15는 3으로도 나누어 떨어지므로 "Fizz"가 먼저 반환됨!

### 🟢 GREEN: 테스트 통과시키기

**FizzBuzz/FizzBuzzGame.cs**:
```csharp
namespace FizzBuzz;

public class FizzBuzzGame
{
    public string Convert(int number)
    {
        // FizzBuzz를 먼저 체크해야 함!
        if (number % 3 == 0 && number % 5 == 0)
            return "FizzBuzz";
            
        if (number % 3 == 0)
            return "Fizz";
            
        if (number % 5 == 0)
            return "Buzz";
            
        return number.ToString();
    }
}
```

**테스트 실행**:
```bash
dotnet test
```

**결과**: ✅ 모든 테스트 통과! 🎉

---

## 리팩토링

### 🔵 REFACTOR: 코드 개선

모든 테스트가 통과하므로 이제 코드를 개선할 수 있습니다.

#### 리팩토링 버전 1: 문자열 연결 방식

```csharp
namespace FizzBuzz;

public class FizzBuzzGame
{
    public string Convert(int number)
    {
        var result = string.Empty;
        
        if (number % 3 == 0)
            result += "Fizz";
            
        if (number % 5 == 0)
            result += "Buzz";
            
        return string.IsNullOrEmpty(result) ? number.ToString() : result;
    }
}
```

**테스트 실행**:
```bash
dotnet test
```

**결과**: ✅ 여전히 모든 테스트 통과

> 💡 **리팩토링 규칙**: 리팩토링 후에는 반드시 테스트를 실행하여 기존 기능이 깨지지 않았는지 확인!

#### 리팩토링 버전 2: 메서드 분리

```csharp
namespace FizzBuzz;

public class FizzBuzzGame
{
    public string Convert(int number)
    {
        var result = string.Empty;
        
        if (IsDivisibleBy(number, 3))
            result += "Fizz";
            
        if (IsDivisibleBy(number, 5))
            result += "Buzz";
            
        return string.IsNullOrEmpty(result) ? number.ToString() : result;
    }
    
    private static bool IsDivisibleBy(int number, int divisor)
    {
        return number % divisor == 0;
    }
}
```

#### 리팩토링 버전 3: Expression Body 활용

```csharp
namespace FizzBuzz;

public class FizzBuzzGame
{
    public string Convert(int number)
    {
        var fizz = IsDivisibleBy(number, 3) ? "Fizz" : "";
        var buzz = IsDivisibleBy(number, 5) ? "Buzz" : "";
        var result = fizz + buzz;
        
        return string.IsNullOrEmpty(result) ? number.ToString() : result;
    }
    
    private static bool IsDivisibleBy(int number, int divisor) 
        => number % divisor == 0;
}
```

---

## 전체 코드

### 최종 구현 코드

**FizzBuzz/FizzBuzzGame.cs**:
```csharp
namespace FizzBuzz;

/// <summary>
/// FizzBuzz 게임 클래스
/// - 3의 배수: "Fizz"
/// - 5의 배수: "Buzz"
/// - 3과 5의 공배수: "FizzBuzz"
/// - 그 외: 숫자 그대로
/// </summary>
public class FizzBuzzGame
{
    /// <summary>
    /// 숫자를 FizzBuzz 규칙에 따라 변환합니다.
    /// </summary>
    /// <param name="number">변환할 숫자</param>
    /// <returns>변환된 문자열</returns>
    public string Convert(int number)
    {
        var fizz = IsDivisibleBy(number, 3) ? "Fizz" : "";
        var buzz = IsDivisibleBy(number, 5) ? "Buzz" : "";
        var result = fizz + buzz;
        
        return string.IsNullOrEmpty(result) ? number.ToString() : result;
    }
    
    private static bool IsDivisibleBy(int number, int divisor) 
        => number % divisor == 0;
}
```

### 최종 테스트 코드 (NUnit)

**FizzBuzz.Tests/FizzBuzzTests.cs**:
```csharp
using NUnit.Framework;

namespace FizzBuzz.Tests;

[TestFixture]
public class FizzBuzzTests
{
    private FizzBuzzGame _game;

    [SetUp]
    public void Setup()
    {
        _game = new FizzBuzzGame();
    }

    #region 일반 숫자 테스트
    
    [TestCase(1, "1")]
    [TestCase(2, "2")]
    [TestCase(4, "4")]
    [TestCase(7, "7")]
    [TestCase(11, "11")]
    public void GivenNormalNumber_ShouldReturnNumberAsString(int input, string expected)
    {
        // Act
        var result = _game.Convert(input);
        
        // Assert
        Assert.That(result, Is.EqualTo(expected));
    }
    
    #endregion

    #region Fizz 테스트 (3의 배수)
    
    [TestCase(3)]
    [TestCase(6)]
    [TestCase(9)]
    [TestCase(12)]
    public void GivenMultipleOf3_ShouldReturnFizz(int input)
    {
        // Act
        var result = _game.Convert(input);
        
        // Assert
        Assert.That(result, Is.EqualTo("Fizz"));
    }
    
    #endregion

    #region Buzz 테스트 (5의 배수)
    
    [TestCase(5)]
    [TestCase(10)]
    [TestCase(20)]
    [TestCase(25)]
    public void GivenMultipleOf5_ShouldReturnBuzz(int input)
    {
        // Act
        var result = _game.Convert(input);
        
        // Assert
        Assert.That(result, Is.EqualTo("Buzz"));
    }
    
    #endregion

    #region FizzBuzz 테스트 (3과 5의 공배수)
    
    [TestCase(15)]
    [TestCase(30)]
    [TestCase(45)]
    [TestCase(60)]
    public void GivenMultipleOf3And5_ShouldReturnFizzBuzz(int input)
    {
        // Act
        var result = _game.Convert(input);
        
        // Assert
        Assert.That(result, Is.EqualTo("FizzBuzz"));
    }
    
    #endregion

    #region 연속 숫자 테스트
    
    [Test]
    public void GivenSequence1To15_ShouldReturnCorrectSequence()
    {
        // Arrange
        var expected = new[]
        {
            "1", "2", "Fizz", "4", "Buzz",
            "Fizz", "7", "8", "Fizz", "Buzz",
            "11", "Fizz", "13", "14", "FizzBuzz"
        };
        
        // Act & Assert
        for (int i = 1; i <= 15; i++)
        {
            var result = _game.Convert(i);
            Assert.That(result, Is.EqualTo(expected[i - 1]), 
                $"Failed for input {i}");
        }
    }
    
    #endregion

    #region 경계값 테스트
    
    [Test]
    public void GivenLargeMultipleOf15_ShouldReturnFizzBuzz()
    {
        // Arrange
        var input = 150;
        
        // Act
        var result = _game.Convert(input);
        
        // Assert
        Assert.That(result, Is.EqualTo("FizzBuzz"));
    }
    
    #endregion
}
```

### 테스트 실행

```bash
dotnet test --verbosity normal
```

**결과**:
```
Starting test execution, please wait...
A total of 1 test files matched the specified pattern.

Passed!  - Failed:     0, Passed:    18, Skipped:     0, Total:    18
```

---

## NUnit 고급 기능

### TestCaseSource 활용

대량의 테스트 데이터가 필요할 때 사용합니다.

```csharp
[TestFixture]
public class FizzBuzzAdvancedTests
{
    private FizzBuzzGame _game;

    [SetUp]
    public void Setup()
    {
        _game = new FizzBuzzGame();
    }

    // 테스트 데이터 소스
    private static IEnumerable<TestCaseData> FizzBuzzTestCases()
    {
        yield return new TestCaseData(1, "1").SetName("Input_1_Returns_1");
        yield return new TestCaseData(2, "2").SetName("Input_2_Returns_2");
        yield return new TestCaseData(3, "Fizz").SetName("Input_3_Returns_Fizz");
        yield return new TestCaseData(5, "Buzz").SetName("Input_5_Returns_Buzz");
        yield return new TestCaseData(15, "FizzBuzz").SetName("Input_15_Returns_FizzBuzz");
        yield return new TestCaseData(30, "FizzBuzz").SetName("Input_30_Returns_FizzBuzz");
    }

    [TestCaseSource(nameof(FizzBuzzTestCases))]
    public void Convert_WithVariousInputs_ReturnsExpectedResult(int input, string expected)
    {
        var result = _game.Convert(input);
        Assert.That(result, Is.EqualTo(expected));
    }
}
```

### Range 테스트

```csharp
[Test]
public void Convert_WithRange1To100_ShouldNotThrowException()
{
    // Assert - 예외가 발생하지 않아야 함
    Assert.DoesNotThrow(() =>
    {
        for (int i = 1; i <= 100; i++)
        {
            _game.Convert(i);
        }
    });
}
```

### Fluent Assertions 스타일

```csharp
[TestCase(3)]
[TestCase(6)]
[TestCase(9)]
public void GivenMultipleOf3_ResultShouldContainFizz(int input)
{
    var result = _game.Convert(input);
    
    Assert.That(result, Does.Contain("Fizz"));
}

[TestCase(5)]
[TestCase(10)]
public void GivenMultipleOf5Only_ResultShouldNotContainFizz(int input)
{
    var result = _game.Convert(input);
    
    Assert.That(result, Does.Not.Contain("Fizz"));
    Assert.That(result, Does.Contain("Buzz"));
}
```

### 예외 테스트

```csharp
[TestCase(0)]
[TestCase(-1)]
[TestCase(-15)]
public void GivenZeroOrNegative_ShouldThrowArgumentException(int input)
{
    Assert.Throws<ArgumentException>(() => _game.Convert(input));
}

// 또는 Constraint 스타일
[Test]
public void GivenNegativeNumber_ShouldThrowWithMessage()
{
    var ex = Assert.Throws<ArgumentException>(() => _game.Convert(-1));
    Assert.That(ex.Message, Does.Contain("positive"));
}
```

---

## 추가 실습: 확장 기능

### 실습 1: 입력 검증 추가

**테스트 먼저 작성**:
```csharp
[TestCase(0)]
[TestCase(-1)]
[TestCase(-15)]
public void GivenZeroOrNegative_ShouldThrowArgumentException(int input)
{
    // Assert
    var ex = Assert.Throws<ArgumentException>(() => _game.Convert(input));
    Assert.That(ex.Message, Does.Contain("positive"));
}
```

**구현**:
```csharp
public string Convert(int number)
{
    if (number <= 0)
        throw new ArgumentException("Number must be positive", nameof(number));
        
    var fizz = IsDivisibleBy(number, 3) ? "Fizz" : "";
    var buzz = IsDivisibleBy(number, 5) ? "Buzz" : "";
    var result = fizz + buzz;
    
    return string.IsNullOrEmpty(result) ? number.ToString() : result;
}
```

### 실습 2: 7의 배수 추가 ("Bazz")

**테스트 먼저 작성**:
```csharp
[TestCase(7)]
[TestCase(14)]
[TestCase(49)]
public void GivenMultipleOf7_ShouldReturnBazz(int input)
{
    var result = _game.Convert(input);
    Assert.That(result, Is.EqualTo("Bazz"));
}

[Test]
public void Given105_ShouldReturnFizzBuzzBazz()
{
    // 105 = 3 × 5 × 7
    var result = _game.Convert(105);
    Assert.That(result, Is.EqualTo("FizzBuzzBazz"));
}
```

---

## 실행 명령어 정리

```bash
# 테스트 실행
dotnet test

# 상세 출력으로 테스트 실행
dotnet test --verbosity normal

# 특정 테스트만 실행 (이름 필터)
dotnet test --filter "Name~Fizz"

# 특정 카테고리만 실행
dotnet test --filter "TestCategory=Unit"

# 코드 커버리지 확인
dotnet test --collect:"XPlat Code Coverage"

# 감시 모드 (코드 변경 시 자동 테스트)
dotnet watch test --project FizzBuzz.Tests

# 결과를 파일로 출력
dotnet test --logger "trx;LogFileName=test_results.trx"
```

---

## NUnit Assert 메서드 요약

### Constraint 모델 (권장)

```csharp
// 동등성
Assert.That(actual, Is.EqualTo(expected));
Assert.That(actual, Is.Not.EqualTo(other));

// Null 체크
Assert.That(obj, Is.Null);
Assert.That(obj, Is.Not.Null);

// Boolean
Assert.That(condition, Is.True);
Assert.That(condition, Is.False);

// 문자열
Assert.That(str, Is.Empty);
Assert.That(str, Is.Not.Empty);
Assert.That(str, Does.Contain("substring"));
Assert.That(str, Does.StartWith("prefix"));
Assert.That(str, Does.EndWith("suffix"));
Assert.That(str, Does.Match("regex"));

// 숫자 비교
Assert.That(value, Is.GreaterThan(5));
Assert.That(value, Is.LessThan(10));
Assert.That(value, Is.InRange(1, 100));

// 컬렉션
Assert.That(collection, Is.Empty);
Assert.That(collection, Has.Count.EqualTo(5));
Assert.That(collection, Does.Contain(item));
Assert.That(collection, Is.All.GreaterThan(0));

// 예외
Assert.Throws<ArgumentException>(() => Method());
Assert.DoesNotThrow(() => Method());
```

---

## TDD 요약

### 핵심 원칙

| 단계 | 설명 | 목표 |
|------|------|------|
| 🔴 RED | 실패하는 테스트 작성 | 기능 정의 |
| 🟢 GREEN | 테스트 통과하는 최소 코드 | 동작하는 코드 |
| 🔵 REFACTOR | 코드 개선 | 깨끗한 코드 |

### TDD 3법칙 (Uncle Bob)

1. 실패하는 단위 테스트를 작성하기 전에는 프로덕션 코드를 작성하지 않는다.
2. 컴파일은 되지만 실패하는 정도로만 테스트를 작성한다.
3. 현재 실패하는 테스트를 통과할 정도로만 프로덕션 코드를 작성한다.

---

## 🎉 완료!

NUnit과 TDD 방식으로 FizzBuzz를 성공적으로 구현했습니다!

**학습한 내용**:
- ✅ Red-Green-Refactor 사이클
- ✅ NUnit 테스트 프레임워크 사용법
- ✅ `[Test]`, `[TestCase]`, `[TestCaseSource]` 어트리뷰트
- ✅ `[SetUp]`을 이용한 테스트 초기화
- ✅ Assert.That() Constraint 모델
- ✅ 점진적인 기능 구현
- ✅ 안전한 리팩토링

---

*작성일: 2025년 12월*
