# C# 완벽 가이드: 다른 언어 경험자를 위한 빠른 입문

## Part 2: 객체지향 프로그래밍 (OOP)

---

## 1. 클래스와 객체

### 1.1 클래스 기본 구조

```csharp
// 클래스 정의
public class Person
{
    // 필드 (Fields) - 데이터 저장
    private string _name;
    private int _age;
    
    // 상수
    public const int MaxAge = 150;
    
    // 읽기 전용 필드
    public readonly DateTime CreatedAt;
    
    // 정적 필드
    private static int _instanceCount;
    
    // 생성자 (Constructor)
    public Person()
    {
        CreatedAt = DateTime.Now;
        _instanceCount++;
    }
    
    public Person(string name, int age) : this()  // 다른 생성자 호출
    {
        _name = name;
        _age = age;
    }
    
    // 프로퍼티 (Property) ⭐
    public string Name
    {
        get { return _name; }
        set { _name = value; }
    }
    
    // 자동 구현 프로퍼티 (가장 일반적)
    public string Email { get; set; }
    
    // 읽기 전용 프로퍼티
    public int Age
    {
        get { return _age; }
        private set { _age = value; }  // 외부에서 설정 불가
    }
    
    // 메서드 (Method)
    public void Introduce()
    {
        Console.WriteLine($"안녕하세요, {_name}입니다.");
    }
    
    public int GetBirthYear()
    {
        return DateTime.Now.Year - _age;
    }
    
    // 정적 메서드
    public static int GetInstanceCount()
    {
        return _instanceCount;
    }
    
    // 소멸자 (Finalizer) - 거의 사용하지 않음
    ~Person()
    {
        // 정리 작업 (GC가 호출)
    }
}

// 객체 생성 및 사용
Person person = new Person("Alice", 25);
Person person2 = new("Bob", 30);  // C# 9+ 대상 타입 추론

person.Name = "Alice Kim";
person.Email = "alice@example.com";
person.Introduce();

int count = Person.GetInstanceCount();  // 정적 메서드 호출
```

### 1.2 다른 언어와 비교

```java
// Java
public class Person {
    private String name;
    
    // Getter/Setter 직접 작성
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

```python
# Python
class Person:
    def __init__(self, name, age):
        self._name = name
        self._age = age
    
    @property
    def name(self):
        return self._name
    
    @name.setter
    def name(self, value):
        self._name = value
```

```csharp
// C# - 훨씬 간결!
public class Person
{
    public string Name { get; set; }  // 자동 프로퍼티
    public int Age { get; set; }
    
    public Person(string name, int age) => (Name, Age) = (name, age);
}
```

### 1.3 프로퍼티 심화

```csharp
public class Product
{
    // 1. 자동 구현 프로퍼티
    public string Name { get; set; }
    
    // 2. 기본값이 있는 프로퍼티
    public int Quantity { get; set; } = 0;
    
    // 3. 읽기 전용 프로퍼티 (C# 6+)
    public DateTime CreatedAt { get; } = DateTime.Now;
    
    // 4. init 전용 설정자 (C# 9+) - 초기화 시에만 설정 가능
    public string SKU { get; init; }
    
    // 5. 유효성 검증이 있는 프로퍼티
    private decimal _price;
    public decimal Price
    {
        get => _price;
        set
        {
            if (value < 0)
                throw new ArgumentException("가격은 0 이상이어야 합니다.");
            _price = value;
        }
    }
    
    // 6. 계산된 프로퍼티 (Expression-bodied)
    public decimal TotalValue => Price * Quantity;
    
    // 7. required 프로퍼티 (C# 11+) - 초기화 필수
    public required string Category { get; set; }
}

// 객체 초기화자 (Object Initializer)
var product = new Product
{
    Name = "Laptop",
    Price = 999.99m,
    Quantity = 10,
    SKU = "LAP-001",
    Category = "Electronics"
};

// C# 9+ with 표현식 (레코드/클래스)
var updated = product with { Price = 899.99m };
```

### 1.4 생성자 패턴

```csharp
public class DatabaseConnection
{
    // 필드
    private readonly string _connectionString;
    private readonly int _timeout;
    private readonly bool _usePooling;
    
    // 주 생성자 (Primary Constructor) - C# 12+
    public class Service(ILogger logger, IRepository repo)
    {
        public void DoWork() => logger.Log("Working...");
    }
    
    // 생성자 체이닝
    public DatabaseConnection() : this("localhost", 30, true)
    {
    }
    
    public DatabaseConnection(string server) : this(server, 30, true)
    {
    }
    
    public DatabaseConnection(string server, int timeout, bool usePooling)
    {
        _connectionString = $"Server={server}";
        _timeout = timeout;
        _usePooling = usePooling;
    }
    
    // 정적 팩토리 메서드 패턴
    public static DatabaseConnection CreateDefault()
        => new DatabaseConnection();
    
    public static DatabaseConnection CreateForTesting()
        => new DatabaseConnection("test-server", 5, false);
}
```

---

## 2. 접근 제한자

### 2.1 접근 제한자 종류

```csharp
public class AccessModifierExample
{
    public int PublicField;         // 어디서든 접근 가능
    private int _privateField;      // 해당 클래스 내에서만
    protected int ProtectedField;   // 해당 클래스 + 파생 클래스
    internal int InternalField;     // 같은 어셈블리(프로젝트) 내에서만
    protected internal int ProtectedInternalField;  // protected OR internal
    private protected int PrivateProtectedField;    // protected AND internal
}
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                        접근 제한자 범위                               │
├─────────────────┬───────┬───────┬──────────┬──────────┬────────────┤
│ 접근 제한자      │ 클래스 │ 파생   │ 같은     │ 다른     │ 다른 어셈블리│
│                 │ 내부   │ 클래스 │ 어셈블리  │ 어셈블리  │ 파생 클래스  │
├─────────────────┼───────┼───────┼──────────┼──────────┼────────────┤
│ public          │  ✓    │  ✓    │    ✓     │    ✓     │     ✓      │
│ private         │  ✓    │  ✗    │    ✗     │    ✗     │     ✗      │
│ protected       │  ✓    │  ✓    │    ✗     │    ✗     │     ✓      │
│ internal        │  ✓    │  ✓    │    ✓     │    ✗     │     ✗      │
│ protected       │  ✓    │  ✓    │    ✓     │    ✗     │     ✓      │
│   internal      │       │       │          │          │            │
│ private         │  ✓    │  ✓    │    ✗     │    ✗     │     ✗      │
│   protected     │       │(같은) │          │          │            │
└─────────────────┴───────┴───────┴──────────┴──────────┴────────────┘
```

### 2.2 기본 접근 제한자

```csharp
// 클래스 - 기본 internal
class DefaultClass { }              // internal
public class PublicClass { }        // public

// 클래스 멤버 - 기본 private
class MyClass
{
    int _field;                     // private
    void Method() { }               // private
    public int PublicField;         // public (명시)
}

// 인터페이스 멤버 - 기본 public
interface IMyInterface
{
    void Method();                  // public
}

// 구조체 멤버 - 기본 private
struct MyStruct
{
    int _field;                     // private
}
```

---

## 3. 상속

### 3.1 기본 상속

```csharp
// 기본 클래스 (부모)
public class Animal
{
    public string Name { get; set; }
    protected int Age { get; set; }  // 파생 클래스에서 접근 가능
    
    public Animal(string name)
    {
        Name = name;
    }
    
    public void Eat()
    {
        Console.WriteLine($"{Name}이(가) 먹습니다.");
    }
    
    // virtual: 파생 클래스에서 재정의 가능
    public virtual void MakeSound()
    {
        Console.WriteLine("동물이 소리를 냅니다.");
    }
}

// 파생 클래스 (자식)
public class Dog : Animal  // 콜론(:)으로 상속
{
    public string Breed { get; set; }
    
    // 부모 생성자 호출
    public Dog(string name, string breed) : base(name)
    {
        Breed = breed;
    }
    
    // override: 부모 메서드 재정의
    public override void MakeSound()
    {
        Console.WriteLine($"{Name}이(가) 멍멍 짖습니다.");
    }
    
    // 새 메서드
    public void Fetch()
    {
        Console.WriteLine($"{Name}이(가) 물건을 가져옵니다.");
    }
}

public class Cat : Animal
{
    public Cat(string name) : base(name) { }
    
    public override void MakeSound()
    {
        Console.WriteLine($"{Name}이(가) 야옹합니다.");
    }
    
    // sealed: 더 이상 재정의 불가
    public sealed override void MakeSound()
    {
        Console.WriteLine("야옹!");
    }
}

// 사용
Animal animal = new Dog("바둑이", "진돗개");
animal.MakeSound();  // "바둑이이(가) 멍멍 짖습니다." (다형성)

Dog dog = new Dog("초코", "푸들");
dog.Fetch();         // Dog 고유 메서드
```

### 3.2 다른 언어와 비교

```java
// Java
public class Dog extends Animal {  // extends 사용
    @Override
    public void makeSound() { }
}
```

```python
# Python
class Dog(Animal):  # 괄호에 부모 클래스
    def make_sound(self):
        pass
```

```csharp
// C#
public class Dog : Animal  // 콜론(:) 사용
{
    public override void MakeSound() { }  // override 키워드 필수
}
```

### 3.3 추상 클래스

```csharp
// 추상 클래스 - 직접 인스턴스화 불가
public abstract class Shape
{
    public string Color { get; set; }
    
    // 추상 메서드 - 구현 없음, 파생 클래스에서 반드시 구현
    public abstract double GetArea();
    public abstract double GetPerimeter();
    
    // 일반 메서드 - 구현 있음
    public void Display()
    {
        Console.WriteLine($"도형 색상: {Color}, 면적: {GetArea()}");
    }
    
    // 가상 메서드 - 기본 구현 있음, 재정의 가능
    public virtual void Draw()
    {
        Console.WriteLine("도형을 그립니다.");
    }
}

public class Circle : Shape
{
    public double Radius { get; set; }
    
    // 추상 메서드 반드시 구현
    public override double GetArea() => Math.PI * Radius * Radius;
    public override double GetPerimeter() => 2 * Math.PI * Radius;
}

public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }
    
    public override double GetArea() => Width * Height;
    public override double GetPerimeter() => 2 * (Width + Height);
    
    // 가상 메서드 재정의 (선택)
    public override void Draw()
    {
        Console.WriteLine($"사각형을 그립니다: {Width} x {Height}");
    }
}

// Shape shape = new Shape();  // 컴파일 에러! 추상 클래스는 인스턴스화 불가
Shape circle = new Circle { Radius = 5, Color = "Red" };
circle.Display();  // "도형 색상: Red, 면적: 78.54..."
```

### 3.4 sealed 클래스

```csharp
// sealed 클래스 - 상속 불가
public sealed class FinalClass
{
    public void Method() { }
}

// public class DerivedClass : FinalClass { }  // 컴파일 에러!

// sealed 메서드 - 더 이상 재정의 불가
public class Parent
{
    public virtual void Method() { }
}

public class Child : Parent
{
    public sealed override void Method() { }  // 여기서 재정의 종료
}

public class GrandChild : Child
{
    // public override void Method() { }  // 컴파일 에러!
}
```

---

## 4. 인터페이스

### 4.1 인터페이스 기본

```csharp
// 인터페이스 정의
public interface IAnimal
{
    // 프로퍼티
    string Name { get; set; }
    
    // 메서드 (구현 없음)
    void MakeSound();
    void Move();
    
    // C# 8.0+ 기본 구현 (Default Implementation)
    void Sleep()
    {
        Console.WriteLine($"{Name}이(가) 잠을 잡니다.");
    }
}

public interface IFlyable
{
    void Fly();
    int MaxAltitude { get; }
}

public interface ISwimmable
{
    void Swim();
}

// 다중 인터페이스 구현
public class Duck : IAnimal, IFlyable, ISwimmable
{
    public string Name { get; set; }
    public int MaxAltitude => 1000;
    
    public void MakeSound() => Console.WriteLine("꽥꽥!");
    public void Move() => Console.WriteLine("뒤뚱뒤뚱 걷습니다.");
    public void Fly() => Console.WriteLine("날아갑니다.");
    public void Swim() => Console.WriteLine("수영합니다.");
}

// 인터페이스 타입으로 사용
IAnimal animal = new Duck { Name = "도널드" };
animal.MakeSound();
animal.Sleep();  // 기본 구현 사용

IFlyable flyable = new Duck { Name = "도널드" };
flyable.Fly();

// 다형성 활용
void MakeSoundAll(IAnimal[] animals)
{
    foreach (var animal in animals)
    {
        animal.MakeSound();
    }
}
```

### 4.2 인터페이스 vs 추상 클래스

```
┌─────────────────────────────────────────────────────────────────────┐
│                   인터페이스 vs 추상 클래스                           │
├─────────────────────┬───────────────────────────────────────────────┤
│ 특성                 │ 인터페이스         │ 추상 클래스               │
├─────────────────────┼───────────────────┼─────────────────────────┤
│ 다중 상속            │ ✓ (여러 개 가능)  │ ✗ (하나만 가능)          │
│ 필드                 │ ✗                 │ ✓                       │
│ 생성자               │ ✗                 │ ✓                       │
│ 접근 제한자          │ 기본 public       │ 다양하게 지정 가능       │
│ 기본 구현            │ ✓ (C# 8.0+)      │ ✓                       │
│ 사용 목적            │ "할 수 있는 것"    │ "~이다" 관계            │
│                     │ (능력/계약)        │ (is-a 관계)             │
├─────────────────────┼───────────────────┼─────────────────────────┤
│ 예시                 │ IComparable       │ Stream                  │
│                     │ IDisposable       │ DbConnection            │
│                     │ IEnumerable       │ ControllerBase          │
└─────────────────────┴───────────────────┴─────────────────────────┘

선택 기준:
• "is-a" 관계 → 추상 클래스 (Dog is an Animal)
• "can-do" 관계 → 인터페이스 (Dog can ISwim)
• 다중 상속 필요 → 인터페이스
• 공통 코드 공유 → 추상 클래스
```

### 4.3 명시적 인터페이스 구현

```csharp
public interface IFileReader
{
    void Read();
}

public interface INetworkReader
{
    void Read();  // 같은 이름의 메서드
}

public class DataReader : IFileReader, INetworkReader
{
    // 암시적 구현 - 둘 다 적용
    public void Read()
    {
        Console.WriteLine("데이터 읽기");
    }
    
    // 명시적 구현 - 특정 인터페이스로 접근할 때만
    void IFileReader.Read()
    {
        Console.WriteLine("파일에서 읽기");
    }
    
    void INetworkReader.Read()
    {
        Console.WriteLine("네트워크에서 읽기");
    }
}

// 사용
var reader = new DataReader();
reader.Read();  // "데이터 읽기" (암시적 구현이 있는 경우)

IFileReader fileReader = reader;
fileReader.Read();  // "파일에서 읽기"

INetworkReader networkReader = reader;
networkReader.Read();  // "네트워크에서 읽기"
```

### 4.4 제네릭 인터페이스

```csharp
// 제네릭 인터페이스
public interface IRepository<T> where T : class
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
}

// 구현
public class UserRepository : IRepository<User>
{
    private readonly List<User> _users = new();
    
    public User GetById(int id) => _users.FirstOrDefault(u => u.Id == id);
    public IEnumerable<User> GetAll() => _users;
    public void Add(User entity) => _users.Add(entity);
    public void Update(User entity) { /* 구현 */ }
    public void Delete(int id) => _users.RemoveAll(u => u.Id == id);
}

// 공변성과 반공변성
public interface IProducer<out T>  // out = 공변성 (리턴 타입에만 사용)
{
    T Produce();
}

public interface IConsumer<in T>   // in = 반공변성 (파라미터에만 사용)
{
    void Consume(T item);
}
```

---

## 5. 구조체 (Struct)

### 5.1 구조체 기본

```csharp
// 구조체 정의 (값 타입)
public struct Point
{
    public double X { get; set; }
    public double Y { get; set; }
    
    // 생성자
    public Point(double x, double y)
    {
        X = x;
        Y = y;
    }
    
    // 메서드
    public double DistanceFromOrigin()
    {
        return Math.Sqrt(X * X + Y * Y);
    }
    
    // ToString 재정의
    public override string ToString() => $"({X}, {Y})";
}

// 사용
Point p1 = new Point(3, 4);
Point p2 = p1;  // 값 복사! (참조 복사 아님)
p2.X = 10;      // p1.X는 여전히 3

// 기본값
Point defaultPoint = default;  // (0, 0)
```

### 5.2 클래스 vs 구조체

```
┌─────────────────────────────────────────────────────────────────────┐
│                      클래스 vs 구조체                                │
├─────────────────────┬───────────────────┬─────────────────────────┤
│ 특성                 │ class (참조 타입) │ struct (값 타입)         │
├─────────────────────┼───────────────────┼─────────────────────────┤
│ 메모리 위치          │ 힙                │ 스택 (또는 인라인)       │
│ 복사 시              │ 참조 복사         │ 전체 값 복사             │
│ 기본값               │ null              │ 모든 필드 기본값         │
│ 상속                 │ ✓                 │ ✗ (인터페이스만 가능)    │
│ 파라미터 없는 생성자  │ ✓                 │ ✓ (C# 10+)             │
│ 필드 초기화          │ ✓                 │ ✓ (C# 10+)             │
│ GC 대상              │ ✓                 │ ✗ (스택에 있을 때)      │
├─────────────────────┼───────────────────┼─────────────────────────┤
│ 사용 시점            │ - 큰 데이터       │ - 작은 데이터 (16바이트↓)│
│                     │ - 복잡한 동작     │ - 불변 데이터            │
│                     │ - 상속 필요       │ - 많은 인스턴스          │
│                     │ - null 필요       │ - 값 의미론 필요         │
└─────────────────────┴───────────────────┴─────────────────────────┘
```

### 5.3 읽기 전용 구조체 (C# 7.2+)

```csharp
// 불변 구조체 - 모든 멤버가 읽기 전용
public readonly struct ImmutablePoint
{
    public double X { get; }  // init 아닌 get만
    public double Y { get; }
    
    public ImmutablePoint(double x, double y) => (X, Y) = (x, y);
    
    // readonly 메서드 - 상태를 변경하지 않음을 보장
    public readonly double DistanceFromOrigin() => Math.Sqrt(X * X + Y * Y);
}

// ref struct - 스택에만 할당 가능 (힙 불가)
public ref struct StackOnlyStruct
{
    public Span<int> Data;
    
    public StackOnlyStruct(Span<int> data) => Data = data;
}
```

---

## 6. 레코드 (Record)

### 6.1 레코드 기본 (C# 9+)

```csharp
// 레코드 정의 - 불변 참조 타입
public record Person(string Name, int Age);

// 위 한 줄은 아래와 동일
public record Person
{
    public string Name { get; init; }
    public int Age { get; init; }
    
    public Person(string name, int age) => (Name, Age) = (name, age);
    
    // 자동 생성되는 것들:
    // - Equals(), GetHashCode() (값 기반)
    // - ToString() ("Person { Name = Alice, Age = 25 }")
    // - Deconstruct()
    // - with 표현식 지원
}

// 사용
var person1 = new Person("Alice", 25);
var person2 = new Person("Alice", 25);

Console.WriteLine(person1 == person2);  // true (값 비교!)
Console.WriteLine(person1);             // "Person { Name = Alice, Age = 25 }"

// with 표현식 - 일부 속성만 변경한 복사본
var person3 = person1 with { Age = 26 };
Console.WriteLine(person3);  // "Person { Name = Alice, Age = 26 }"

// Deconstruct
var (name, age) = person1;
Console.WriteLine($"{name}은 {age}살입니다.");
```

### 6.2 레코드 상속

```csharp
// 레코드 상속
public record Person(string Name, int Age);
public record Employee(string Name, int Age, string Department) : Person(Name, Age);

var emp = new Employee("Bob", 30, "Engineering");
Console.WriteLine(emp);  // "Employee { Name = Bob, Age = 30, Department = Engineering }"

// 동등성 비교는 런타임 타입도 고려
Person p = new Person("Bob", 30);
Employee e = new Employee("Bob", 30, "Engineering");
Console.WriteLine(p == e);  // false (타입이 다름)
```

### 6.3 레코드 구조체 (C# 10+)

```csharp
// record struct - 값 타입 레코드
public record struct Point(double X, double Y);

// readonly record struct - 불변 값 타입 레코드
public readonly record struct ImmutablePoint(double X, double Y);

// 사용
var p1 = new Point(3, 4);
var p2 = p1;  // 값 복사
p2.X = 10;    // p1.X는 여전히 3

var ip1 = new ImmutablePoint(3, 4);
// ip1.X = 10;  // 컴파일 에러! readonly
```

### 6.4 record vs class vs struct

```
┌─────────────────────────────────────────────────────────────────────┐
│                   record vs class vs struct                         │
├─────────────────────┬──────────┬──────────┬──────────┬─────────────┤
│ 특성                 │ class    │ struct   │ record   │record struct│
├─────────────────────┼──────────┼──────────┼──────────┼─────────────┤
│ 타입                 │ 참조     │ 값       │ 참조     │ 값          │
│ 동등성 비교          │ 참조     │ 값       │ 값       │ 값          │
│ 상속                 │ ✓        │ ✗        │ ✓        │ ✗           │
│ with 표현식          │ ✗        │ ✗        │ ✓        │ ✓           │
│ 기본 불변성          │ ✗        │ ✗        │ ✓        │ 선택        │
│ ToString() 자동생성  │ ✗        │ ✗        │ ✓        │ ✓           │
├─────────────────────┴──────────┴──────────┴──────────┴─────────────┤
│ 사용 시점:                                                          │
│ • class: 복잡한 동작, 변경 가능한 상태                               │
│ • struct: 작은 값 타입, 성능 중요                                   │
│ • record: 불변 데이터 모델, DTO, 값 의미론 필요                      │
│ • record struct: 작은 불변 값 타입                                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. 다형성

### 7.1 메서드 오버로딩 (Overloading)

```csharp
public class Calculator
{
    // 같은 이름, 다른 매개변수
    public int Add(int a, int b) => a + b;
    public double Add(double a, double b) => a + b;
    public int Add(int a, int b, int c) => a + b + c;
    public string Add(string a, string b) => a + b;
    
    // 선택적 매개변수
    public int Add(int a, int b = 0, int c = 0) => a + b + c;
    
    // params 키워드 - 가변 인자
    public int AddAll(params int[] numbers) => numbers.Sum();
}

var calc = new Calculator();
calc.Add(1, 2);           // int 버전
calc.Add(1.5, 2.5);       // double 버전
calc.Add(1, 2, 3);        // 3개 인자 버전
calc.Add("Hello", " World");  // string 버전
calc.AddAll(1, 2, 3, 4, 5);   // params 사용
```

### 7.2 메서드 오버라이딩 (Overriding)

```csharp
public class Animal
{
    public virtual void Speak()  // virtual 필수
    {
        Console.WriteLine("...");
    }
}

public class Dog : Animal
{
    public override void Speak()  // override 필수
    {
        Console.WriteLine("멍멍!");
    }
}

public class Cat : Animal
{
    public override void Speak()
    {
        base.Speak();  // 부모 메서드 호출
        Console.WriteLine("야옹!");
    }
}

// 다형성
Animal[] animals = { new Dog(), new Cat(), new Animal() };
foreach (var animal in animals)
{
    animal.Speak();  // 각각 다른 출력
}
```

### 7.3 new 키워드 (Hiding)

```csharp
public class Parent
{
    public void Method() => Console.WriteLine("Parent");
}

public class Child : Parent
{
    // new: 부모 메서드 숨김 (재정의 아님)
    public new void Method() => Console.WriteLine("Child");
}

Child child = new Child();
child.Method();  // "Child"

Parent parent = child;
parent.Method(); // "Parent" (!!! override와 다름)
```

---

## 8. 기타 OOP 기능

### 8.1 정적 클래스와 멤버

```csharp
// 정적 클래스 - 인스턴스화 불가, 모든 멤버가 static
public static class MathHelper
{
    public static double Pi => 3.14159265358979;
    
    public static double Square(double x) => x * x;
    public static double Cube(double x) => x * x * x;
}

// 사용
double result = MathHelper.Square(5);

// 정적 생성자 - 클래스 최초 접근 시 한 번만 실행
public class Config
{
    public static string ConnectionString { get; private set; }
    
    static Config()  // 정적 생성자
    {
        ConnectionString = LoadFromFile();
    }
    
    private static string LoadFromFile() => "...";
}
```

### 8.2 확장 메서드 (Extension Method)

```csharp
// 기존 타입에 메서드 추가 (원본 수정 없이)
public static class StringExtensions
{
    // this 키워드로 확장 대상 지정
    public static bool IsNullOrEmpty(this string str)
    {
        return string.IsNullOrEmpty(str);
    }
    
    public static string Reverse(this string str)
    {
        return new string(str.ToCharArray().Reverse().ToArray());
    }
    
    public static int WordCount(this string str)
    {
        return str.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
    }
}

// 사용 - 마치 string의 메서드처럼!
string text = "Hello World";
bool isEmpty = text.IsNullOrEmpty();      // false
string reversed = text.Reverse();          // "dlroW olleH"
int words = text.WordCount();              // 2

// 컬렉션 확장
public static class EnumerableExtensions
{
    public static IEnumerable<T> WhereNotNull<T>(this IEnumerable<T?> source)
        where T : class
    {
        return source.Where(x => x != null)!;
    }
}
```

### 8.3 부분 클래스 (Partial Class)

```csharp
// 파일1: Person.Properties.cs
public partial class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

// 파일2: Person.Methods.cs
public partial class Person
{
    public void Introduce()
    {
        Console.WriteLine($"안녕하세요, {Name}입니다.");
    }
}

// 파일3: Person.Generated.cs (코드 생성기가 만든 코드)
public partial class Person
{
    public string GeneratedProperty { get; set; }
}

// 사용 - 하나의 클래스처럼 동작
var person = new Person { Name = "Alice", Age = 25 };
person.Introduce();
```

### 8.4 중첩 클래스 (Nested Class)

```csharp
public class Outer
{
    private int _outerField = 10;
    
    // 중첩 클래스
    public class Inner
    {
        private Outer _outer;
        
        public Inner(Outer outer)
        {
            _outer = outer;
        }
        
        public void AccessOuter()
        {
            // 외부 클래스의 private 멤버에 접근 가능!
            Console.WriteLine(_outer._outerField);
        }
    }
    
    // private 중첩 클래스 - 내부 구현용
    private class PrivateHelper
    {
        public void Help() { }
    }
}

// 사용
var outer = new Outer();
var inner = new Outer.Inner(outer);
inner.AccessOuter();
```

---

## 📝 Part 2 핵심 정리

### OOP 핵심 요약

```csharp
// 1. 클래스와 프로퍼티
public class Person
{
    public string Name { get; set; }        // 자동 프로퍼티
    public int Age { get; init; }           // init 전용
    public required string Email { get; set; }  // 필수 속성
}

// 2. 상속
public class Employee : Person
{
    public override void Introduce() => base.Introduce();
}

// 3. 인터페이스
public interface IWorker { void Work(); }
public class Employee : Person, IWorker { }

// 4. 레코드
public record UserDto(string Name, string Email);
var updated = original with { Name = "New" };

// 5. 확장 메서드
public static string Reverse(this string s) => ...;
"hello".Reverse();
```

### 다음 Part 예고

**Part 3: 고급 기능**
- 제네릭 (Generics)
- 델리게이트와 이벤트
- LINQ
- 비동기 프로그래밍 (async/await)
- 패턴 매칭 심화
