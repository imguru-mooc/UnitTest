# C# 완벽 가이드: 다른 언어 경험자를 위한 빠른 입문

## Part 1: 기본 문법과 타입 시스템

> 💡 이 문서는 Python, Java, JavaScript 등 다른 언어를 알고 있는 개발자가 C#을 빠르게 습득할 수 있도록 작성되었습니다.

---

## 1. C# 언어 개요

### 1.1 C#의 특징

```
┌─────────────────────────────────────────────────────────────────┐
│                        C# 언어 특성                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✓ 강타입 (Strongly Typed)     - Java와 유사                    │
│  ✓ 객체지향 (OOP)              - 클래스 기반                     │
│  ✓ 가비지 컬렉션 (GC)          - 자동 메모리 관리                │
│  ✓ 크로스 플랫폼               - .NET Core/6+ 이후               │
│  ✓ 현대적 기능                 - async/await, LINQ, 패턴매칭     │
│  ✓ Microsoft 생태계            - Visual Studio, Azure 통합       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 다른 언어와의 비교

| 특성 | C# | Java | Python | JavaScript |
|------|-----|------|--------|------------|
| 타입 시스템 | 정적/강타입 | 정적/강타입 | 동적/강타입 | 동적/약타입 |
| 컴파일 | AOT/JIT | JIT | 인터프리터 | JIT |
| 메모리 관리 | GC | GC | GC | GC |
| Null 안전성 | Nullable 참조 타입 | Optional | None | undefined/null |
| 람다/클로저 | ✓ | ✓ | ✓ | ✓ |
| 제네릭 | ✓ (reified) | ✓ (erased) | ✓ (duck typing) | ✗ |

### 1.3 Hello World

```csharp
// C# 최신 문법 (Top-level statements, .NET 6+)
Console.WriteLine("Hello, World!");

// 전통적인 방식 (명시적 진입점)
namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

**다른 언어와 비교:**

```python
# Python
print("Hello, World!")
```

```java
// Java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

```javascript
// JavaScript
console.log("Hello, World!");
```

---

## 2. 변수와 데이터 타입

### 2.1 기본 타입 (Primitive Types)

```csharp
// 정수 타입
byte    b = 255;              // 0 ~ 255 (8비트, 부호 없음)
sbyte   sb = -128;            // -128 ~ 127 (8비트, 부호 있음)
short   s = 32767;            // -32,768 ~ 32,767 (16비트)
ushort  us = 65535;           // 0 ~ 65,535 (16비트, 부호 없음)
int     i = 2147483647;       // -2^31 ~ 2^31-1 (32비트) ⭐ 기본 정수
uint    ui = 4294967295;      // 0 ~ 2^32-1 (32비트, 부호 없음)
long    l = 9223372036854775807L;  // 64비트
ulong   ul = 18446744073709551615UL; // 64비트, 부호 없음

// 실수 타입
float   f = 3.14f;            // 32비트, 접미사 f 필수
double  d = 3.141592653589793; // 64비트 ⭐ 기본 실수
decimal m = 3.14159265358979323846m; // 128비트, 금융 계산용, 접미사 m 필수

// 문자와 문자열
char    c = 'A';              // 16비트 유니코드 문자 (작은따옴표)
string  str = "Hello";        // 문자열 (큰따옴표) - 참조 타입!

// 불리언
bool    flag = true;          // true 또는 false (1이나 0 불가!)

// 객체 (모든 타입의 기본)
object  obj = 42;             // 모든 타입의 부모
```

### 2.2 타입 비교표

```
┌─────────────────────────────────────────────────────────────────┐
│                    C# vs 다른 언어 타입 비교                      │
├──────────────┬──────────────┬──────────────┬───────────────────┤
│     C#       │    Java      │   Python     │    JavaScript     │
├──────────────┼──────────────┼──────────────┼───────────────────┤
│ int          │ int          │ int          │ number            │
│ long         │ long         │ int          │ number/BigInt     │
│ float        │ float        │ float        │ number            │
│ double       │ double       │ float        │ number            │
│ decimal      │ BigDecimal   │ Decimal      │ -                 │
│ bool         │ boolean      │ bool         │ boolean           │
│ char         │ char         │ str[0]       │ string[0]         │
│ string       │ String       │ str          │ string            │
│ object       │ Object       │ object       │ Object            │
│ var          │ var (10+)    │ -            │ let/const         │
│ dynamic      │ -            │ 기본 동작    │ 기본 동작          │
└──────────────┴──────────────┴──────────────┴───────────────────┘
```

### 2.3 변수 선언 방식

```csharp
// 1. 명시적 타입 선언
int number = 42;
string name = "Alice";
List<int> numbers = new List<int>();

// 2. 타입 추론 (var) - 컴파일 타임에 타입 결정
var count = 10;              // int로 추론
var message = "Hello";       // string으로 추론
var list = new List<int>();  // List<int>로 추론

// 3. 상수 선언
const int MaxValue = 100;    // 컴파일 타임 상수 (변경 불가)
readonly int _id;            // 런타임 상수 (생성자에서만 할당 가능)

// 4. Nullable 값 타입
int? nullableInt = null;     // int는 원래 null 불가, ?로 허용
double? nullableDouble = 3.14;

// 5. 기본값 할당
int defaultInt = default;    // 0
bool defaultBool = default;  // false
string defaultStr = default; // null (참조 타입)
```

### 2.4 값 타입 vs 참조 타입

```csharp
// ⭐ C#의 핵심 개념!

// 값 타입 (Value Type) - 스택에 저장
int a = 10;
int b = a;      // 값 복사
b = 20;         // a는 여전히 10

// 참조 타입 (Reference Type) - 힙에 저장, 스택에 참조
int[] arr1 = { 1, 2, 3 };
int[] arr2 = arr1;  // 참조 복사 (같은 배열을 가리킴)
arr2[0] = 100;      // arr1[0]도 100으로 변경됨!
```

```
┌─────────────────────────────────────────────────────────────────┐
│                   값 타입 vs 참조 타입                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   값 타입 (struct)              참조 타입 (class)                │
│   ─────────────────            ─────────────────               │
│   • 스택에 직접 저장            • 힙에 저장, 스택에 참조          │
│   • 복사 시 값 전체 복사        • 복사 시 참조만 복사             │
│   • 기본 타입: int, bool,      • string, array, class,         │
│     double, struct, enum        interface, delegate            │
│                                                                 │
│   Stack                        Stack         Heap              │
│   ┌─────┐                      ┌─────┐      ┌─────────┐        │
│   │  10 │ ← int a              │  ●──┼──────│ "Hello" │        │
│   └─────┘                      └─────┘      └─────────┘        │
│                                    ↑ string str                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.5 문자열 처리

```csharp
// 문자열 생성
string s1 = "Hello";
string s2 = new string('*', 10);  // "**********"

// 문자열 연결
string concat = s1 + " World";           // 전통적 방식
string concat2 = string.Concat(s1, " World");

// ⭐ 문자열 보간 (String Interpolation) - 가장 권장!
string name = "Alice";
int age = 25;
string message = $"Name: {name}, Age: {age}";
string formatted = $"Price: {price:C}";   // 통화 형식
string padded = $"ID: {id:D5}";           // 5자리 숫자

// Verbatim 문자열 (@) - 이스케이프 문자 무시
string path = @"C:\Users\Documents\file.txt";  // 백슬래시 그대로
string multiLine = @"
    첫 번째 줄
    두 번째 줄
    세 번째 줄";

// Raw String Literal (C# 11+) - 따옴표 3개
string json = """
    {
        "name": "Alice",
        "age": 25
    }
    """;

// 문자열 메서드
string text = "  Hello World  ";
text.Length              // 15
text.ToUpper()           // "  HELLO WORLD  "
text.ToLower()           // "  hello world  "
text.Trim()              // "Hello World"
text.Split(' ')          // ["", "", "Hello", "World", "", ""]
text.Replace("World", "C#")  // "  Hello C#  "
text.Contains("Hello")   // true
text.StartsWith("  H")   // true
text.Substring(2, 5)     // "Hello"
text.IndexOf("World")    // 8

// 문자열 비교
string.Equals(s1, s2, StringComparison.OrdinalIgnoreCase);  // 대소문자 무시
s1 == s2;  // 값 비교 (C#에서 string은 == 오버로드됨)

// StringBuilder - 많은 연결 작업 시 성능 향상
var sb = new StringBuilder();
sb.Append("Hello");
sb.Append(" ");
sb.Append("World");
sb.AppendLine("!");      // 줄바꿈 포함
string result = sb.ToString();
```

**다른 언어와 비교:**

```python
# Python
f"Name: {name}, Age: {age}"  # f-string
```

```javascript
// JavaScript
`Name: ${name}, Age: ${age}`  // Template literal
```

---

## 3. 연산자

### 3.1 산술 연산자

```csharp
int a = 10, b = 3;

a + b    // 13 (덧셈)
a - b    // 7  (뺄셈)
a * b    // 30 (곱셈)
a / b    // 3  (정수 나눗셈) ⚠️ 주의!
a % b    // 1  (나머지)

// 실수 나눗셈을 원한다면
double result = (double)a / b;  // 3.333...
double result2 = a / (double)b; // 3.333...

// 증감 연산자
int x = 5;
x++;     // 후위 증가: x는 6
++x;     // 전위 증가: x는 7
x--;     // 후위 감소: x는 6

// 복합 대입 연산자
x += 5;  // x = x + 5
x -= 3;  // x = x - 3
x *= 2;  // x = x * 2
x /= 4;  // x = x / 4
x %= 3;  // x = x % 3
```

### 3.2 비교 연산자

```csharp
int a = 5, b = 10;

a == b   // false (같음)
a != b   // true  (다름)
a < b    // true  (작음)
a > b    // false (큼)
a <= b   // true  (작거나 같음)
a >= b   // false (크거나 같음)

// 참조 타입 비교
string s1 = "hello";
string s2 = "hello";
string s3 = new string("hello".ToCharArray());

s1 == s2         // true  (값 비교 - string은 특별!)
s1 == s3         // true  (값 비교)
ReferenceEquals(s1, s2)  // true  (string interning)
ReferenceEquals(s1, s3)  // false (다른 인스턴스)

// object 비교
object.Equals(s1, s2)    // true
s1.Equals(s2)            // true
```

### 3.3 논리 연산자

```csharp
bool x = true, y = false;

x && y   // false (AND - 단락 평가)
x || y   // true  (OR - 단락 평가)
!x       // false (NOT)

x & y    // false (AND - 단락 평가 안 함)
x | y    // true  (OR - 단락 평가 안 함)
x ^ y    // true  (XOR)

// 단락 평가 (Short-circuit Evaluation)
// && : 첫 번째가 false면 두 번째 평가 안 함
// || : 첫 번째가 true면 두 번째 평가 안 함

bool result = false && SomeMethod();  // SomeMethod() 호출 안 됨
bool result2 = true || SomeMethod();  // SomeMethod() 호출 안 됨
```

### 3.4 Null 관련 연산자 ⭐

```csharp
// ⚠️ C#에서 매우 중요한 연산자들!

string name = null;

// 1. Null 조건 연산자 (?.)
int? length = name?.Length;  // name이 null이면 null, 아니면 Length
// Python의 name and len(name) 또는 JavaScript의 name?.length와 유사

// 2. Null 병합 연산자 (??)
string displayName = name ?? "Unknown";  // name이 null이면 "Unknown"
// Python의 name or "Unknown", JavaScript의 name ?? "Unknown"과 유사

// 3. Null 병합 대입 연산자 (??=)
name ??= "Default";  // name이 null일 때만 할당
// 동일: name = name ?? "Default";

// 4. Null 허용 참조 타입 (C# 8.0+)
string? nullableString = null;   // null 허용 명시
string nonNullString = "hello";  // null 불허 (경고 발생 가능)

// 5. Null 용서 연산자 (!)
string definitelyNotNull = nullableString!;  // "나는 이게 null 아닌 걸 안다"

// 실전 예시
Person? person = GetPerson();
string city = person?.Address?.City ?? "Unknown";

// 체이닝 예시
int? count = customers?.Orders?.Count;
var firstOrder = customers?.Orders?.FirstOrDefault();
```

### 3.5 패턴 매칭 연산자

```csharp
object obj = "Hello";

// is 연산자 (타입 체크)
if (obj is string)
{
    Console.WriteLine("It's a string!");
}

// is + 패턴 매칭 (타입 체크 + 캐스팅 동시에)
if (obj is string s)
{
    Console.WriteLine($"String length: {s.Length}");
}

// is not
if (obj is not null)
{
    Console.WriteLine("Not null!");
}

// switch 표현식과 패턴 매칭
string GetTypeDescription(object obj) => obj switch
{
    null => "null",
    int i => $"Integer: {i}",
    string s => $"String: {s}",
    double d when d > 0 => $"Positive double: {d}",
    double d => $"Non-positive double: {d}",
    _ => "Unknown type"
};

// as 연산자 (안전한 캐스팅)
string str = obj as string;  // 캐스팅 실패 시 null
if (str != null)
{
    Console.WriteLine(str.Length);
}
```

---

## 4. 제어 흐름

### 4.1 조건문

```csharp
// if-else
int score = 85;

if (score >= 90)
{
    Console.WriteLine("A");
}
else if (score >= 80)
{
    Console.WriteLine("B");
}
else if (score >= 70)
{
    Console.WriteLine("C");
}
else
{
    Console.WriteLine("F");
}

// 삼항 연산자
string result = score >= 60 ? "Pass" : "Fail";

// switch 문 (전통적)
int day = 3;
switch (day)
{
    case 1:
        Console.WriteLine("Monday");
        break;
    case 2:
        Console.WriteLine("Tuesday");
        break;
    case 3:
    case 4:
    case 5:
        Console.WriteLine("Midweek");
        break;
    default:
        Console.WriteLine("Weekend");
        break;
}

// switch 표현식 (C# 8.0+) ⭐ 권장
string dayName = day switch
{
    1 => "Monday",
    2 => "Tuesday",
    3 or 4 or 5 => "Midweek",
    6 or 7 => "Weekend",
    _ => "Invalid"
};

// 패턴 매칭 switch
object value = 42;
string description = value switch
{
    int n when n < 0 => "Negative integer",
    int n when n == 0 => "Zero",
    int n when n > 0 => "Positive integer",
    string s => $"String: {s}",
    null => "Null value",
    _ => "Unknown type"
};
```

### 4.2 반복문

```csharp
// for 문
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);  // 0, 1, 2, 3, 4
}

// while 문
int count = 0;
while (count < 5)
{
    Console.WriteLine(count);
    count++;
}

// do-while 문 (최소 1회 실행)
int num = 0;
do
{
    Console.WriteLine(num);
    num++;
} while (num < 5);

// foreach 문 ⭐ 컬렉션 순회에 권장
int[] numbers = { 1, 2, 3, 4, 5 };
foreach (int n in numbers)
{
    Console.WriteLine(n);
}

// foreach with index (LINQ 활용)
foreach (var (item, index) in numbers.Select((n, i) => (n, i)))
{
    Console.WriteLine($"Index {index}: {item}");
}

// break와 continue
for (int i = 0; i < 10; i++)
{
    if (i == 3) continue;  // 3 건너뛰기
    if (i == 7) break;     // 7에서 종료
    Console.WriteLine(i);  // 0, 1, 2, 4, 5, 6
}
```

**다른 언어와 비교:**

```python
# Python foreach
for n in numbers:
    print(n)

# Python enumerate
for i, n in enumerate(numbers):
    print(f"Index {i}: {n}")
```

```javascript
// JavaScript forEach
numbers.forEach((n, i) => console.log(`Index ${i}: ${n}`));

// JavaScript for...of
for (const n of numbers) {
    console.log(n);
}
```

---

## 5. 배열과 컬렉션

### 5.1 배열 (Array)

```csharp
// 배열 선언 및 초기화
int[] numbers = new int[5];           // 크기 5, 기본값 0
int[] nums = new int[] { 1, 2, 3 };   // 초기화와 함께
int[] nums2 = { 1, 2, 3, 4, 5 };      // 간단한 초기화
int[] nums3 = [1, 2, 3, 4, 5];        // C# 12+ 컬렉션 표현식

// 배열 접근
int first = nums[0];        // 첫 번째 요소
int last = nums[^1];        // 마지막 요소 (C# 8.0+)
int secondLast = nums[^2];  // 뒤에서 두 번째

// 배열 슬라이싱 (C# 8.0+)
int[] slice = nums[1..3];   // index 1, 2 (3 미포함)
int[] fromStart = nums[..3]; // index 0, 1, 2
int[] toEnd = nums[2..];    // index 2부터 끝까지
int[] copy = nums[..];      // 전체 복사

// 다차원 배열
int[,] matrix = new int[3, 4];        // 3x4 2차원 배열
int[,] matrix2 = { { 1, 2 }, { 3, 4 }, { 5, 6 } };
int value = matrix2[1, 0];            // 3

// 가변 배열 (Jagged Array)
int[][] jagged = new int[3][];
jagged[0] = new int[] { 1, 2 };
jagged[1] = new int[] { 3, 4, 5 };
jagged[2] = new int[] { 6 };

// 배열 메서드
int[] arr = { 5, 2, 8, 1, 9 };
Array.Sort(arr);              // 정렬: { 1, 2, 5, 8, 9 }
Array.Reverse(arr);           // 역순: { 9, 8, 5, 2, 1 }
int index = Array.IndexOf(arr, 5);  // 5의 인덱스
bool exists = Array.Exists(arr, x => x > 5);  // 조건 존재 여부
int[] copy2 = new int[5];
Array.Copy(arr, copy2, 5);    // 복사
```

### 5.2 List<T>

```csharp
// List 생성
List<int> list = new List<int>();
List<int> list2 = new List<int> { 1, 2, 3 };
List<int> list3 = [1, 2, 3, 4, 5];  // C# 12+ 컬렉션 표현식
var list4 = new List<string> { "a", "b", "c" };

// 요소 추가
list.Add(1);                  // 끝에 추가
list.AddRange(new[] { 2, 3, 4 });  // 여러 개 추가
list.Insert(0, 0);            // 특정 위치에 삽입

// 요소 접근
int first = list[0];          // 인덱서로 접근
int last = list[^1];          // 마지막 요소

// 요소 제거
list.Remove(3);               // 값으로 제거 (첫 번째 일치)
list.RemoveAt(0);             // 인덱스로 제거
list.RemoveAll(x => x < 0);   // 조건으로 모두 제거
list.Clear();                 // 전체 제거

// 검색
bool contains = list.Contains(5);
int index = list.IndexOf(5);
int lastIndex = list.LastIndexOf(5);
int found = list.Find(x => x > 10);       // 첫 번째 일치
List<int> all = list.FindAll(x => x > 10); // 모든 일치

// 정렬
list.Sort();                  // 오름차순
list.Sort((a, b) => b - a);   // 내림차순
list.Reverse();               // 역순

// 변환
int[] array = list.ToArray();
List<string> strings = list.ConvertAll(x => x.ToString());

// 속성
int count = list.Count;       // 요소 개수
int capacity = list.Capacity; // 내부 배열 크기
```

### 5.3 Dictionary<K, V>

```csharp
// Dictionary 생성
var dict = new Dictionary<string, int>();
var dict2 = new Dictionary<string, int>
{
    { "apple", 100 },
    { "banana", 200 },
    { "cherry", 300 }
};
// C# 6+ 인덱서 초기화 구문
var dict3 = new Dictionary<string, int>
{
    ["apple"] = 100,
    ["banana"] = 200
};

// 요소 추가/수정
dict["key"] = 100;            // 추가 또는 수정
dict.Add("newKey", 200);      // 추가만 (키 존재 시 예외)
dict.TryAdd("key", 300);      // 추가 시도 (실패해도 예외 없음)

// 요소 접근
int value = dict["apple"];    // 키 없으면 예외!
bool success = dict.TryGetValue("apple", out int val);  // 안전한 접근 ⭐

// 요소 제거
dict.Remove("apple");
dict.Clear();

// 검색
bool hasKey = dict.ContainsKey("apple");
bool hasValue = dict.ContainsValue(100);

// 순회
foreach (var kvp in dict)
{
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
}
foreach (var key in dict.Keys)
{
    Console.WriteLine(key);
}
foreach (var value in dict.Values)
{
    Console.WriteLine(value);
}

// Deconstruct를 활용한 순회 (권장)
foreach (var (key, value) in dict)
{
    Console.WriteLine($"{key}: {value}");
}
```

### 5.4 기타 컬렉션

```csharp
// HashSet<T> - 중복 없는 집합
var set = new HashSet<int> { 1, 2, 3, 3 };  // { 1, 2, 3 }
set.Add(4);
set.Remove(1);
bool contains = set.Contains(2);

// 집합 연산
var set2 = new HashSet<int> { 2, 3, 4, 5 };
set.UnionWith(set2);        // 합집합
set.IntersectWith(set2);    // 교집합
set.ExceptWith(set2);       // 차집합

// Queue<T> - FIFO
var queue = new Queue<string>();
queue.Enqueue("first");
queue.Enqueue("second");
string item = queue.Dequeue();  // "first"
string peek = queue.Peek();     // 제거하지 않고 확인

// Stack<T> - LIFO
var stack = new Stack<int>();
stack.Push(1);
stack.Push(2);
int top = stack.Pop();    // 2
int peek2 = stack.Peek(); // 1 (제거 안 함)

// LinkedList<T> - 연결 리스트
var linked = new LinkedList<int>();
linked.AddFirst(1);
linked.AddLast(3);
linked.AddAfter(linked.First, 2);

// SortedList<K, V> / SortedDictionary<K, V> - 정렬된 딕셔너리
var sorted = new SortedDictionary<string, int>
{
    ["banana"] = 2,
    ["apple"] = 1,
    ["cherry"] = 3
};
// 키 순서로 정렬됨: apple, banana, cherry
```

### 5.5 컬렉션 비교표

```
┌──────────────────────────────────────────────────────────────────────┐
│                         C# 컬렉션 선택 가이드                          │
├──────────────────┬───────────────────────────────────────────────────┤
│ 컬렉션            │ 사용 시점                                          │
├──────────────────┼───────────────────────────────────────────────────┤
│ Array            │ 고정 크기, 최고 성능 필요 시                         │
│ List<T>          │ 가변 크기 배열, 가장 일반적 ⭐                       │
│ Dictionary<K,V>  │ 키-값 쌍, 빠른 검색 ⭐                              │
│ HashSet<T>       │ 중복 없는 집합, 빠른 존재 확인                       │
│ Queue<T>         │ FIFO (선입선출)                                    │
│ Stack<T>         │ LIFO (후입선출)                                    │
│ LinkedList<T>    │ 빈번한 삽입/삭제                                    │
│ SortedList<K,V>  │ 정렬 유지 필요, 메모리 효율적                        │
│ SortedSet<T>     │ 정렬된 고유 요소 집합                               │
└──────────────────┴───────────────────────────────────────────────────┘
```

---

## 6. 예외 처리

### 6.1 기본 예외 처리

```csharp
try
{
    int result = 10 / 0;  // DivideByZeroException 발생
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"0으로 나눌 수 없습니다: {ex.Message}");
}
catch (Exception ex)  // 모든 예외 포착 (가장 마지막에)
{
    Console.WriteLine($"오류 발생: {ex.Message}");
}
finally
{
    // 예외 발생 여부와 관계없이 항상 실행
    Console.WriteLine("정리 작업 수행");
}
```

### 6.2 예외 필터와 재발생

```csharp
// 예외 필터 (when)
try
{
    ProcessData();
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
{
    Console.WriteLine("리소스를 찾을 수 없습니다.");
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.Unauthorized)
{
    Console.WriteLine("인증이 필요합니다.");
}

// 예외 재발생
try
{
    RiskyOperation();
}
catch (Exception ex)
{
    Log(ex);
    throw;  // 원래 예외 그대로 재발생 (스택 트레이스 유지)
}

// 새 예외로 래핑
try
{
    RiskyOperation();
}
catch (Exception ex)
{
    throw new ApplicationException("작업 실패", ex);  // InnerException으로 포함
}
```

### 6.3 사용자 정의 예외

```csharp
// 사용자 정의 예외 클래스
public class InsufficientFundsException : Exception
{
    public decimal RequestedAmount { get; }
    public decimal AvailableBalance { get; }

    public InsufficientFundsException(decimal requested, decimal available)
        : base($"잔액 부족. 요청: {requested:C}, 가용: {available:C}")
    {
        RequestedAmount = requested;
        AvailableBalance = available;
    }
}

// 사용
public void Withdraw(decimal amount)
{
    if (amount > Balance)
        throw new InsufficientFundsException(amount, Balance);
    
    Balance -= amount;
}

// 처리
try
{
    account.Withdraw(1000);
}
catch (InsufficientFundsException ex)
{
    Console.WriteLine($"부족 금액: {ex.RequestedAmount - ex.AvailableBalance:C}");
}
```

### 6.4 일반적인 예외 타입

```csharp
// 자주 사용되는 내장 예외
ArgumentNullException       // null 인자
ArgumentException           // 잘못된 인자
ArgumentOutOfRangeException // 범위 밖 인자
InvalidOperationException   // 잘못된 상태에서 작업
NotImplementedException     // 미구현
NotSupportedException       // 지원하지 않는 작업
NullReferenceException      // null 참조 접근
IndexOutOfRangeException    // 배열 인덱스 범위 초과
KeyNotFoundException        // 딕셔너리 키 없음
FileNotFoundException       // 파일 없음
IOException                 // I/O 오류

// 예외 발생 예시
public void SetName(string name)
{
    if (name == null)
        throw new ArgumentNullException(nameof(name));
    
    if (string.IsNullOrWhiteSpace(name))
        throw new ArgumentException("이름은 비어있을 수 없습니다.", nameof(name));
    
    _name = name;
}
```

---

## 7. 입출력

### 7.1 콘솔 입출력

```csharp
// 출력
Console.WriteLine("줄바꿈 포함 출력");
Console.Write("줄바꿈 없는 출력");
Console.WriteLine($"포맷팅: {value:F2}");  // 소수점 2자리

// 입력
Console.Write("이름을 입력하세요: ");
string name = Console.ReadLine();  // 문자열 입력

// 숫자 입력
Console.Write("나이를 입력하세요: ");
if (int.TryParse(Console.ReadLine(), out int age))
{
    Console.WriteLine($"나이: {age}");
}
else
{
    Console.WriteLine("잘못된 입력입니다.");
}

// 키 입력
ConsoleKeyInfo key = Console.ReadKey();  // 한 글자 입력
Console.ReadKey(true);  // 입력 문자 숨김
```

### 7.2 파일 입출력

```csharp
// 파일 쓰기 (간단)
File.WriteAllText("file.txt", "Hello, World!");
File.WriteAllLines("file.txt", new[] { "Line1", "Line2" });

// 파일 읽기 (간단)
string content = File.ReadAllText("file.txt");
string[] lines = File.ReadAllLines("file.txt");

// StreamWriter 사용 (큰 파일)
using (StreamWriter writer = new StreamWriter("file.txt"))
{
    writer.WriteLine("Line 1");
    writer.WriteLine("Line 2");
}

// StreamReader 사용
using (StreamReader reader = new StreamReader("file.txt"))
{
    string line;
    while ((line = reader.ReadLine()) != null)
    {
        Console.WriteLine(line);
    }
}

// 파일 존재 확인
if (File.Exists("file.txt"))
{
    File.Delete("file.txt");
}

// 디렉토리 작업
Directory.CreateDirectory("newFolder");
string[] files = Directory.GetFiles(".", "*.txt");
string[] dirs = Directory.GetDirectories(".");
```

---

## 📝 Part 1 핵심 정리

### C# 문법 핵심 요약

```csharp
// 1. 변수 선언
int number = 10;           // 명시적 타입
var inferred = 10;         // 타입 추론
int? nullable = null;      // Nullable

// 2. 문자열
string str = $"Value: {number}";  // 문자열 보간

// 3. Null 처리
string name = person?.Name ?? "Unknown";

// 4. 컬렉션
var list = new List<int> { 1, 2, 3 };
var dict = new Dictionary<string, int> { ["key"] = 1 };

// 5. 제어 흐름
string result = score switch
{
    >= 90 => "A",
    >= 80 => "B",
    _ => "C"
};

// 6. 예외 처리
try { } 
catch (Exception ex) when (ex is not null) { }
finally { }
```

### 다음 Part 예고

**Part 2: 객체지향 프로그래밍 (OOP)**
- 클래스와 구조체
- 상속과 다형성
- 인터페이스
- 접근 제한자
- 프로퍼티와 인덱서
