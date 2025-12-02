# C# 완벽 가이드: 다른 언어 경험자를 위한 빠른 입문

## Part 3: 고급 기능

---

## 1. 제네릭 (Generics)

### 1.1 제네릭 기본

```csharp
// 제네릭 클래스
public class Box<T>
{
    private T _value;
    
    public T Value
    {
        get => _value;
        set => _value = value;
    }
    
    public Box(T value) => _value = value;
}

// 사용
Box<int> intBox = new Box<int>(42);
Box<string> strBox = new Box<string>("Hello");
Box<List<int>> listBox = new Box<List<int>>(new List<int> { 1, 2, 3 });

// 다중 타입 파라미터
public class Pair<TKey, TValue>
{
    public TKey Key { get; set; }
    public TValue Value { get; set; }
    
    public Pair(TKey key, TValue value) => (Key, Value) = (key, value);
    
    public void Deconstruct(out TKey key, out TValue value)
    {
        key = Key;
        value = Value;
    }
}

var pair = new Pair<string, int>("age", 25);
var (key, value) = pair;  // Deconstruct 활용
```

### 1.2 제네릭 메서드

```csharp
public class Utilities
{
    // 제네릭 메서드
    public static void Swap<T>(ref T a, ref T b)
    {
        T temp = a;
        a = b;
        b = temp;
    }
    
    public static T Max<T>(T a, T b) where T : IComparable<T>
    {
        return a.CompareTo(b) > 0 ? a : b;
    }
    
    public static List<T> CreateList<T>(params T[] items)
    {
        return new List<T>(items);
    }
}

// 사용
int x = 1, y = 2;
Utilities.Swap(ref x, ref y);  // 타입 추론
Utilities.Swap<int>(ref x, ref y);  // 명시적 타입

int max = Utilities.Max(10, 20);  // 20
var list = Utilities.CreateList(1, 2, 3, 4, 5);
```

### 1.3 제네릭 제약 조건 (Constraints)

```csharp
// 다양한 제약 조건
public class Repository<T> where T : class  // 참조 타입만
{
    public void Add(T item) { }
}

public class Calculator<T> where T : struct  // 값 타입만
{
    public T Add(T a, T b) => throw new NotImplementedException();
}

public class Factory<T> where T : new()  // 파라미터 없는 생성자 필수
{
    public T Create() => new T();
}

public class Service<T> where T : IDisposable  // 인터페이스 구현 필수
{
    public void Process(T resource)
    {
        using (resource)
        {
            // 작업
        }
    }
}

public class Derived<T> where T : BaseClass  // 특정 클래스 상속 필수
{
}

// 복합 제약 조건
public class ComplexService<T> 
    where T : class, IComparable<T>, IDisposable, new()
{
    public T CreateAndCompare(T other)
    {
        var instance = new T();
        if (instance.CompareTo(other) > 0)
            return instance;
        return other;
    }
}

// 다중 타입 파라미터 각각에 제약
public class Converter<TInput, TOutput>
    where TInput : class
    where TOutput : class, new()
{
    public TOutput Convert(TInput input) => new TOutput();
}
```

### 1.4 다른 언어와 비교

```java
// Java - Type Erasure (런타임에 타입 정보 손실)
List<String> list = new ArrayList<>();
// 런타임에는 List<Object>로 취급

// Java에서 불가능한 것:
// new T();  // 불가
// T.class;  // 불가
```

```csharp
// C# - Reified Generics (런타임에도 타입 정보 유지!)
List<string> list = new List<string>();
Type type = list.GetType().GetGenericArguments()[0];  // System.String

// C#에서 가능:
T instance = new T();  // where T : new() 필요
Type t = typeof(T);    // 타입 정보 접근 가능
```

---

## 2. 델리게이트 (Delegate)

### 2.1 델리게이트 기본

```csharp
// 델리게이트 정의 - 메서드 시그니처 정의
public delegate int MathOperation(int a, int b);
public delegate void Logger(string message);
public delegate bool Predicate<T>(T item);

// 메서드
public static int Add(int a, int b) => a + b;
public static int Multiply(int a, int b) => a * b;

// 델리게이트 사용
MathOperation operation = Add;
int result = operation(5, 3);  // 8

operation = Multiply;
result = operation(5, 3);  // 15

// 멀티캐스트 델리게이트
Logger logger = Console.WriteLine;
logger += message => Debug.WriteLine(message);
logger += message => File.AppendAllText("log.txt", message);

logger("Hello!");  // 세 곳에 모두 로그
```

### 2.2 내장 델리게이트 (가장 많이 사용) ⭐

```csharp
// Action - 반환값 없음
Action sayHello = () => Console.WriteLine("Hello");
Action<string> greet = name => Console.WriteLine($"Hello, {name}");
Action<int, int> printSum = (a, b) => Console.WriteLine(a + b);

sayHello();           // "Hello"
greet("Alice");       // "Hello, Alice"
printSum(3, 5);       // "8"

// Func - 반환값 있음 (마지막이 반환 타입)
Func<int> getNumber = () => 42;
Func<int, int> square = x => x * x;
Func<int, int, int> add = (a, b) => a + b;
Func<string, int> getLength = s => s.Length;

int num = getNumber();     // 42
int sq = square(5);        // 25
int sum = add(3, 4);       // 7
int len = getLength("Hi"); // 2

// Predicate - bool 반환 (조건 검사)
Predicate<int> isPositive = x => x > 0;
Predicate<string> isLong = s => s.Length > 10;

bool positive = isPositive(5);   // true
bool longStr = isLong("Hi");     // false

// 실전 사용 예시
List<int> numbers = new List<int> { 1, -2, 3, -4, 5 };
List<int> positives = numbers.FindAll(isPositive);  // [1, 3, 5]
```

### 2.3 다른 언어와 비교

```python
# Python - 함수는 일급 객체
def add(a, b):
    return a + b

operation = add  # 함수를 변수에 할당
result = operation(5, 3)

# 람다
square = lambda x: x * x
```

```javascript
// JavaScript - 함수는 일급 객체
const add = (a, b) => a + b;
const operation = add;
const result = operation(5, 3);
```

```csharp
// C# - 델리게이트 또는 Func/Action 사용
Func<int, int, int> add = (a, b) => a + b;
var operation = add;
int result = operation(5, 3);
```

---

## 3. 람다 표현식

### 3.1 람다 기본 문법

```csharp
// 람다 표현식 문법
Func<int, int> square1 = (int x) => { return x * x; };  // 전체 문법
Func<int, int> square2 = (x) => { return x * x; };      // 타입 추론
Func<int, int> square3 = x => { return x * x; };        // 괄호 생략 (인자 1개)
Func<int, int> square4 = x => x * x;                    // 식 본문 (한 줄)

// 여러 매개변수
Func<int, int, int> add = (a, b) => a + b;
Action<string, int> print = (name, age) => Console.WriteLine($"{name}: {age}");

// 매개변수 없음
Action sayHello = () => Console.WriteLine("Hello");
Func<DateTime> getNow = () => DateTime.Now;

// 여러 문장
Func<int, int> factorial = n =>
{
    int result = 1;
    for (int i = 1; i <= n; i++)
        result *= i;
    return result;
};
```

### 3.2 클로저 (Closure)

```csharp
// 클로저 - 외부 변수 캡처
int multiplier = 10;
Func<int, int> multiply = x => x * multiplier;  // multiplier 캡처

Console.WriteLine(multiply(5));  // 50

multiplier = 20;  // 변수 변경
Console.WriteLine(multiply(5));  // 100 (캡처된 변수 참조!)

// 반복문에서의 클로저 주의점
var actions = new List<Action>();
for (int i = 0; i < 5; i++)
{
    actions.Add(() => Console.WriteLine(i));  // 모두 같은 i를 캡처!
}
foreach (var action in actions)
{
    action();  // 5, 5, 5, 5, 5 출력 (예상: 0, 1, 2, 3, 4)
}

// 해결 방법 - 로컬 복사
for (int i = 0; i < 5; i++)
{
    int localI = i;  // 로컬 변수로 복사
    actions.Add(() => Console.WriteLine(localI));
}
```

### 3.3 정적 람다 (C# 9+)

```csharp
// 일반 람다 - 외부 변수 캡처 가능
int factor = 2;
Func<int, int> normalLambda = x => x * factor;

// 정적 람다 - 외부 변수 캡처 불가 (성능 향상)
Func<int, int> staticLambda = static x => x * 2;
// Func<int, int> error = static x => x * factor;  // 컴파일 에러!
```

---

## 4. 이벤트 (Event)

### 4.1 이벤트 기본

```csharp
// 이벤트 정의
public class Button
{
    // 이벤트 선언 (EventHandler 델리게이트 사용)
    public event EventHandler? Clicked;
    
    // 이벤트 발생 메서드
    protected virtual void OnClicked()
    {
        Clicked?.Invoke(this, EventArgs.Empty);
    }
    
    public void SimulateClick()
    {
        Console.WriteLine("버튼 클릭됨");
        OnClicked();
    }
}

// 사용
var button = new Button();

// 이벤트 구독 (+=)
button.Clicked += (sender, e) => Console.WriteLine("핸들러 1 실행");
button.Clicked += Button_Clicked;  // 메서드 참조

button.SimulateClick();

// 이벤트 구독 해제 (-=)
button.Clicked -= Button_Clicked;

void Button_Clicked(object? sender, EventArgs e)
{
    Console.WriteLine("핸들러 2 실행");
}
```

### 4.2 사용자 정의 이벤트 인자

```csharp
// 커스텀 이벤트 인자
public class OrderEventArgs : EventArgs
{
    public int OrderId { get; }
    public decimal Amount { get; }
    public DateTime OrderDate { get; }
    
    public OrderEventArgs(int orderId, decimal amount)
    {
        OrderId = orderId;
        Amount = amount;
        OrderDate = DateTime.Now;
    }
}

// 이벤트를 가진 클래스
public class OrderProcessor
{
    public event EventHandler<OrderEventArgs>? OrderPlaced;
    public event EventHandler<OrderEventArgs>? OrderShipped;
    public event EventHandler<OrderEventArgs>? OrderCancelled;
    
    public void PlaceOrder(int orderId, decimal amount)
    {
        // 주문 처리 로직
        Console.WriteLine($"주문 {orderId} 처리 중...");
        
        // 이벤트 발생
        OrderPlaced?.Invoke(this, new OrderEventArgs(orderId, amount));
    }
    
    public void ShipOrder(int orderId, decimal amount)
    {
        OrderShipped?.Invoke(this, new OrderEventArgs(orderId, amount));
    }
}

// 사용
var processor = new OrderProcessor();

processor.OrderPlaced += (sender, e) =>
{
    Console.WriteLine($"주문 완료: #{e.OrderId}, 금액: {e.Amount:C}");
};

processor.OrderPlaced += (sender, e) =>
{
    // 이메일 발송
    Console.WriteLine($"확인 이메일 발송: 주문 #{e.OrderId}");
};

processor.PlaceOrder(1001, 99.99m);
```

### 4.3 이벤트 vs 델리게이트

```
┌─────────────────────────────────────────────────────────────────────┐
│                     이벤트 vs 델리게이트                             │
├─────────────────────┬───────────────────────────────────────────────┤
│ 특성                 │ 델리게이트           │ 이벤트                  │
├─────────────────────┼─────────────────────┼────────────────────────┤
│ 외부에서 호출        │ ✓ (가능)            │ ✗ (불가 - 클래스만)     │
│ 외부에서 = 할당      │ ✓ (전체 교체 가능)  │ ✗ (불가 - +=/-=만)     │
│ null 할당           │ ✓                   │ ✗                      │
│ 캡슐화              │ 낮음                 │ 높음                   │
│ 사용 목적           │ 콜백, 함수 전달      │ 알림, 발행-구독        │
└─────────────────────┴─────────────────────┴────────────────────────┘
```

---

## 5. LINQ (Language Integrated Query)

### 5.1 LINQ 기본

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// 메서드 체이닝 문법 (권장) ⭐
var evenNumbers = numbers
    .Where(n => n % 2 == 0)
    .Select(n => n * 2)
    .OrderByDescending(n => n)
    .ToList();
// [20, 16, 12, 8, 4]

// 쿼리 문법 (SQL 스타일)
var evenNumbers2 = (from n in numbers
                    where n % 2 == 0
                    orderby n descending
                    select n * 2).ToList();

// 두 문법은 동일한 결과!
```

### 5.2 주요 LINQ 메서드

```csharp
var people = new List<Person>
{
    new("Alice", 25, "Engineering"),
    new("Bob", 30, "Sales"),
    new("Charlie", 35, "Engineering"),
    new("Diana", 28, "Marketing"),
    new("Eve", 32, "Engineering")
};

// ===== 필터링 =====
var adults = people.Where(p => p.Age >= 30);
var firstEngineer = people.First(p => p.Department == "Engineering");
var lastEngineer = people.Last(p => p.Department == "Engineering");
var singleMarketing = people.Single(p => p.Department == "Marketing");

// OrDefault - 없으면 null 반환 (예외 대신)
var notFound = people.FirstOrDefault(p => p.Name == "Zoe");  // null

// ===== 투영 (Projection) =====
var names = people.Select(p => p.Name);  // ["Alice", "Bob", ...]
var info = people.Select(p => new { p.Name, p.Age });  // 익명 타입
var tuples = people.Select(p => (p.Name, p.Age));  // 튜플

// ===== 정렬 =====
var byAge = people.OrderBy(p => p.Age);
var byAgeDesc = people.OrderByDescending(p => p.Age);
var byDeptThenAge = people.OrderBy(p => p.Department).ThenBy(p => p.Age);

// ===== 그룹화 =====
var byDepartment = people.GroupBy(p => p.Department);
foreach (var group in byDepartment)
{
    Console.WriteLine($"{group.Key}: {group.Count()}명");
}

// ===== 집계 =====
int count = people.Count();
int engineerCount = people.Count(p => p.Department == "Engineering");
int totalAge = people.Sum(p => p.Age);
double avgAge = people.Average(p => p.Age);
int maxAge = people.Max(p => p.Age);
int minAge = people.Min(p => p.Age);
Person oldest = people.MaxBy(p => p.Age);  // C# 10+

// ===== 요소 검사 =====
bool anyEngineers = people.Any(p => p.Department == "Engineering");  // true
bool allAdults = people.All(p => p.Age >= 18);  // true
bool hasAlice = people.Contains(people[0]);

// ===== 집합 연산 =====
var dept1 = new[] { "Alice", "Bob" };
var dept2 = new[] { "Bob", "Charlie" };
var union = dept1.Union(dept2);         // ["Alice", "Bob", "Charlie"]
var intersect = dept1.Intersect(dept2); // ["Bob"]
var except = dept1.Except(dept2);       // ["Alice"]
var distinct = new[] { 1, 1, 2, 2, 3 }.Distinct();  // [1, 2, 3]

// ===== 변환 =====
var array = people.ToArray();
var list = people.ToList();
var dict = people.ToDictionary(p => p.Name);  // Dictionary<string, Person>
var lookup = people.ToLookup(p => p.Department);  // ILookup (1:다)
var hashSet = people.ToHashSet();

// ===== 페이징 =====
var page2 = people.Skip(2).Take(2);  // 3번째, 4번째 요소

// ===== 결합 =====
var orders = new List<Order> { /* ... */ };
var joined = people.Join(
    orders,
    person => person.Name,
    order => order.CustomerName,
    (person, order) => new { person.Name, order.Amount }
);
```

### 5.3 지연 실행 (Deferred Execution)

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5 };

// 쿼리 정의 - 아직 실행되지 않음!
var query = numbers.Where(n => n > 2);

numbers.Add(6);  // 리스트에 추가

// 쿼리 실행 - 이 시점에 실행!
foreach (var n in query)
{
    Console.WriteLine(n);  // 3, 4, 5, 6 출력
}

// 즉시 실행하려면:
var immediate = numbers.Where(n => n > 2).ToList();  // 즉시 실행
```

### 5.4 다른 언어와 비교

```python
# Python
# 필터
evens = [n for n in numbers if n % 2 == 0]
evens = list(filter(lambda n: n % 2 == 0, numbers))

# 맵
doubled = [n * 2 for n in numbers]
doubled = list(map(lambda n: n * 2, numbers))
```

```javascript
// JavaScript
const evens = numbers.filter(n => n % 2 === 0);
const doubled = numbers.map(n => n * 2);
const sum = numbers.reduce((a, b) => a + b, 0);
```

```csharp
// C# LINQ
var evens = numbers.Where(n => n % 2 == 0);
var doubled = numbers.Select(n => n * 2);
var sum = numbers.Sum();
```

---

## 6. 비동기 프로그래밍 (Async/Await)

### 6.1 비동기 기본

```csharp
// 동기 메서드
public string DownloadSync(string url)
{
    using var client = new HttpClient();
    return client.GetStringAsync(url).Result;  // 블로킹!
}

// 비동기 메서드 ⭐
public async Task<string> DownloadAsync(string url)
{
    using var client = new HttpClient();
    return await client.GetStringAsync(url);  // 논블로킹!
}

// 비동기 메서드 호출
public async Task ProcessDataAsync()
{
    string html = await DownloadAsync("https://example.com");
    Console.WriteLine(html.Length);
}

// async void - 이벤트 핸들러에서만 사용!
private async void Button_Click(object sender, EventArgs e)
{
    await ProcessDataAsync();
}
```

### 6.2 Task와 Task<T>

```csharp
// Task - 반환값 없는 비동기 작업
public async Task SaveDataAsync(string data)
{
    await File.WriteAllTextAsync("data.txt", data);
}

// Task<T> - 반환값 있는 비동기 작업
public async Task<int> CalculateAsync(int x)
{
    await Task.Delay(1000);  // 1초 대기
    return x * x;
}

// 여러 Task 동시 실행
public async Task ProcessAllAsync()
{
    // 동시 시작
    Task<int> task1 = CalculateAsync(5);
    Task<int> task2 = CalculateAsync(10);
    Task<int> task3 = CalculateAsync(15);
    
    // 모두 완료 대기
    int[] results = await Task.WhenAll(task1, task2, task3);
    // [25, 100, 225]
    
    // 하나라도 완료되면
    Task<int> firstCompleted = await Task.WhenAny(task1, task2, task3);
}

// ValueTask<T> - 자주 동기적으로 완료되는 경우 성능 향상
public async ValueTask<int> GetCachedValueAsync(string key)
{
    if (_cache.TryGetValue(key, out int value))
        return value;  // 동기적 반환 (힙 할당 없음)
    
    return await FetchFromDatabaseAsync(key);
}
```

### 6.3 비동기 스트림 (C# 8.0+)

```csharp
// IAsyncEnumerable - 비동기 스트림
public async IAsyncEnumerable<int> GenerateNumbersAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);  // 비동기 작업
        yield return i;
    }
}

// 비동기 스트림 소비
public async Task ConsumeAsync()
{
    await foreach (var number in GenerateNumbersAsync())
    {
        Console.WriteLine(number);
    }
}

// 취소 지원
public async IAsyncEnumerable<int> GenerateWithCancellationAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    for (int i = 0; i < 100; i++)
    {
        ct.ThrowIfCancellationRequested();
        await Task.Delay(100, ct);
        yield return i;
    }
}
```

### 6.4 비동기 패턴

```csharp
// 1. 취소 토큰 (CancellationToken)
public async Task LongOperationAsync(CancellationToken ct = default)
{
    for (int i = 0; i < 100; i++)
    {
        ct.ThrowIfCancellationRequested();  // 취소 확인
        await Task.Delay(100, ct);
    }
}

// 사용
var cts = new CancellationTokenSource();
cts.CancelAfter(TimeSpan.FromSeconds(5));  // 5초 후 취소

try
{
    await LongOperationAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("작업이 취소되었습니다.");
}

// 2. 타임아웃
public async Task<string> FetchWithTimeoutAsync(string url)
{
    using var client = new HttpClient();
    client.Timeout = TimeSpan.FromSeconds(10);
    return await client.GetStringAsync(url);
}

// 3. 재시도 로직
public async Task<T> RetryAsync<T>(Func<Task<T>> operation, int maxRetries = 3)
{
    for (int i = 0; i < maxRetries; i++)
    {
        try
        {
            return await operation();
        }
        catch (Exception) when (i < maxRetries - 1)
        {
            await Task.Delay(TimeSpan.FromSeconds(Math.Pow(2, i)));  // 지수 백오프
        }
    }
    return await operation();  // 마지막 시도
}
```

### 6.5 다른 언어와 비교

```python
# Python
async def download_async(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

# 실행
asyncio.run(download_async("https://example.com"))
```

```javascript
// JavaScript
async function downloadAsync(url) {
    const response = await fetch(url);
    return await response.text();
}

// Promise
downloadAsync("https://example.com")
    .then(html => console.log(html))
    .catch(err => console.error(err));
```

```csharp
// C#
public async Task<string> DownloadAsync(string url)
{
    using var client = new HttpClient();
    return await client.GetStringAsync(url);
}

// 실행
string html = await DownloadAsync("https://example.com");
```

---

## 7. 패턴 매칭 심화

### 7.1 타입 패턴

```csharp
object obj = "Hello";

// is 패턴
if (obj is string s)
{
    Console.WriteLine(s.Length);
}

// switch 표현식
string result = obj switch
{
    string s => $"문자열: {s}",
    int i => $"정수: {i}",
    null => "null",
    _ => "기타"
};
```

### 7.2 프로퍼티 패턴

```csharp
public record Person(string Name, int Age, Address? Address);
public record Address(string City, string Country);

// 프로퍼티 패턴
string GetDiscount(Person person) => person switch
{
    { Age: < 18 } => "청소년 할인",
    { Age: >= 65 } => "경로 할인",
    { Address.Country: "Korea" } => "국내 할인",
    { Name: "VIP" or "VVIP" } => "VIP 할인",
    _ => "일반"
};

// 중첩 프로퍼티 패턴
bool IsSeoulResident(Person person) => person is 
{
    Address: { City: "Seoul", Country: "Korea" }
};
```

### 7.3 위치 패턴 (Positional Pattern)

```csharp
// 튜플 패턴
string GetQuadrant((int x, int y) point) => point switch
{
    (0, 0) => "원점",
    (> 0, > 0) => "1사분면",
    (< 0, > 0) => "2사분면",
    (< 0, < 0) => "3사분면",
    (> 0, < 0) => "4사분면",
    (0, _) => "Y축",
    (_, 0) => "X축"
};

// 레코드 분해
public record Point(int X, int Y);

string DescribePoint(Point point) => point switch
{
    (0, 0) => "원점",
    (var x, var y) when x == y => "대각선",
    (var x, 0) => $"X축 위 ({x})",
    (0, var y) => $"Y축 위 ({y})",
    var (x, y) => $"일반 점 ({x}, {y})"
};
```

### 7.4 리스트 패턴 (C# 11+)

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// 리스트 패턴
string result = numbers switch
{
    [] => "빈 배열",
    [var single] => $"요소 1개: {single}",
    [var first, var second] => $"요소 2개: {first}, {second}",
    [var first, .., var last] => $"첫: {first}, 끝: {last}",
    [1, 2, ..] => "1, 2로 시작",
    [.., 4, 5] => "4, 5로 끝남"
};

// 슬라이스 패턴
if (numbers is [var head, .. var tail])
{
    Console.WriteLine($"Head: {head}");
    Console.WriteLine($"Tail: [{string.Join(", ", tail)}]");
}
```

### 7.5 복합 패턴

```csharp
public record Order(int Id, decimal Amount, string Status, Customer Customer);
public record Customer(string Name, string Type, bool IsPremium);

// 복합 패턴 매칭
decimal CalculateDiscount(Order order) => order switch
{
    // and, or, not 패턴
    { Amount: > 100 and < 500 } => 5,
    { Amount: >= 500 or < 1000 } => 10,
    { Status: not "Cancelled" } => 0,
    
    // 중첩 프로퍼티 + 조건
    { Customer: { IsPremium: true }, Amount: > 100 } => 15,
    { Customer.Type: "VIP" or "VVIP", Status: "Confirmed" } => 20,
    
    // 변수 캡처 + when 절
    { Amount: var a } when a > 1000 => 25,
    
    _ => 0
};
```

---

## 8. 유용한 C# 기능들

### 8.1 튜플 (Tuple)

```csharp
// 튜플 생성
var tuple1 = (1, "hello", 3.14);
(int, string, double) tuple2 = (1, "hello", 3.14);
(int Id, string Name, double Score) tuple3 = (1, "Alice", 95.5);

// 접근
int item1 = tuple1.Item1;       // 1
string name = tuple3.Name;      // "Alice"

// 분해 (Deconstruction)
var (id, name2, score) = tuple3;
(int x, string y, _) = tuple3;  // _ 로 무시

// 메서드 반환
public (int Min, int Max) GetMinMax(int[] numbers)
{
    return (numbers.Min(), numbers.Max());
}

var (min, max) = GetMinMax(new[] { 1, 5, 3, 9, 2 });
```

### 8.2 레코드의 Deconstruct

```csharp
public record Person(string Name, int Age);

var person = new Person("Alice", 25);

// 자동 분해
var (name, age) = person;
Console.WriteLine($"{name} is {age} years old");
```

### 8.3 using 선언 (C# 8.0+)

```csharp
// 전통적인 using 문
using (var file = File.OpenRead("data.txt"))
using (var reader = new StreamReader(file))
{
    string content = reader.ReadToEnd();
}

// using 선언 (더 간결)
using var file = File.OpenRead("data.txt");
using var reader = new StreamReader(file);
string content = reader.ReadToEnd();
// 스코프 끝에서 자동 Dispose
```

### 8.4 범위와 인덱스 (C# 8.0+)

```csharp
int[] array = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// Index (^)
int last = array[^1];      // 9 (마지막)
int secondLast = array[^2]; // 8 (뒤에서 두 번째)

// Range (..)
int[] slice1 = array[2..5];    // [2, 3, 4]
int[] slice2 = array[..3];     // [0, 1, 2]
int[] slice3 = array[7..];     // [7, 8, 9]
int[] slice4 = array[^3..];    // [7, 8, 9]
int[] copy = array[..];        // 전체 복사

// Range 변수
Range range = 2..5;
int[] slice = array[range];
```

### 8.5 로컬 함수

```csharp
public int Factorial(int n)
{
    // 로컬 함수 정의
    int Calculate(int value)
    {
        if (value <= 1) return 1;
        return value * Calculate(value - 1);
    }
    
    if (n < 0)
        throw new ArgumentException("음수 불가");
    
    return Calculate(n);
}

// 정적 로컬 함수 (외부 변수 캡처 불가)
public int Process(int x)
{
    int factor = 10;
    
    // static 로컬 함수 - 캡처 없음 (성능 향상)
    static int Double(int n) => n * 2;
    
    return Double(x) * factor;
}
```

### 8.6 널 허용 참조 타입 (C# 8.0+)

```csharp
#nullable enable  // 프로젝트 또는 파일 단위로 활성화

// 널 허용 (?)
string? nullableName = null;  // OK
string nonNullName = null;    // 경고!

// 널 검사
public void Process(string? input)
{
    // 경고: input이 null일 수 있음
    // Console.WriteLine(input.Length);
    
    // 올바른 방법들
    if (input != null)
    {
        Console.WriteLine(input.Length);
    }
    
    Console.WriteLine(input?.Length ?? 0);
    
    // null이 아님을 확신할 때
    Console.WriteLine(input!.Length);  // ! 연산자
}

// 널 허용 주석
public class Person
{
    public string Name { get; set; } = "";  // 기본값 필수 (또는 생성자)
    public string? MiddleName { get; set; } // 널 허용
}
```

---

## 📝 Part 3 핵심 정리

### 고급 기능 요약

```csharp
// 1. 제네릭
public class Repository<T> where T : class, new() { }

// 2. 델리게이트 / 람다
Func<int, int> square = x => x * x;
Action<string> log = msg => Console.WriteLine(msg);

// 3. LINQ
var result = items.Where(x => x.Active).Select(x => x.Name).ToList();

// 4. 비동기
public async Task<string> FetchAsync() => await httpClient.GetStringAsync(url);

// 5. 패턴 매칭
string desc = obj switch
{
    int i when i > 0 => "양수",
    string { Length: > 10 } => "긴 문자열",
    null => "null",
    _ => "기타"
};

// 6. 널 안전성
string? name = GetName();
Console.WriteLine(name?.Length ?? 0);
```

### 다음 Part 예고

**Part 4: 실전 활용**
- NuGet 패키지 관리
- 파일 I/O와 직렬화
- HTTP 클라이언트
- 의존성 주입 (DI)
- 단위 테스트 기초
