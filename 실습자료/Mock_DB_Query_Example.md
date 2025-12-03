# 🗃️ Mock 실전 예제: DB 쿼리 결과 가공 및 검증

## Visual Studio 2017 호환 버전 (.NET Framework 4.7.2 / C# 7.3)

---

## 시나리오 설명

```
┌────────────────────────────────────────────────────────────────┐
│  시나리오: 월별 매출 리포트 생성                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. DB에서 주문 데이터를 조회                                  │
│  2. 월별로 그룹화하여 매출 집계                                │
│  3. 전월 대비 증감률 계산                                      │
│  4. 리포트 형태로 가공하여 반환                                │
│                                                                │
│  테스트 목표:                                                  │
│  • 실제 DB 없이 Mock 데이터로 테스트                          │
│  • 가공 로직이 올바르게 동작하는지 검증                        │
│  • 다양한 데이터 시나리오 테스트                               │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 1. 프로젝트 구조

```
SalesReportDemo/
├── SalesReportDemo/                    ← 프로덕션 프로젝트
│   ├── Models/
│   │   ├── Order.cs
│   │   ├── MonthlySalesData.cs
│   │   └── SalesReportResult.cs
│   ├── Repositories/
│   │   └── IOrderRepository.cs
│   ├── Services/
│   │   └── SalesReportService.cs
│   └── SalesReportDemo.csproj
│
├── SalesReportDemo.Tests/              ← 테스트 프로젝트
│   ├── SalesReportServiceTests.cs
│   └── SalesReportDemo.Tests.csproj
│
└── SalesReportDemo.sln
```

---

## 2. 프로젝트 생성 (Visual Studio 2017)

### Step 2.1: 솔루션 생성

```
1. Visual Studio 2017 실행
2. 파일 → 새로 만들기 → 프로젝트
3. Visual C# → 클래스 라이브러리(.NET Framework)
4. 이름: SalesReportDemo
5. 프레임워크: .NET Framework 4.7.2
6. 확인
```

### Step 2.2: 테스트 프로젝트 추가

```
1. 솔루션 우클릭 → 추가 → 새 프로젝트
2. Visual C# → 테스트 → 단위 테스트 프로젝트(.NET Framework)
3. 이름: SalesReportDemo.Tests
4. 프레임워크: .NET Framework 4.7.2
5. 확인
```

### Step 2.3: NuGet 패키지 설치

```
테스트 프로젝트(SalesReportDemo.Tests)에서:

도구 → NuGet 패키지 관리자 → 패키지 관리자 콘솔

PM> Install-Package NUnit -Version 3.12.0
PM> Install-Package NUnit3TestAdapter -Version 3.17.0
PM> Install-Package Moq -Version 4.16.1
PM> Install-Package Microsoft.NET.Test.Sdk -Version 16.9.4
```

### Step 2.4: 프로젝트 참조 추가

```
SalesReportDemo.Tests 프로젝트 우클릭
→ 추가 → 참조
→ 프로젝트 → SalesReportDemo 체크
→ 확인
```

---

## 3. 프로덕션 프로젝트 파일

### SalesReportDemo.csproj

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="15.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props" 
          Condition="Exists('$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props')" />
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProjectGuid>{YOUR-GUID-HERE}</ProjectGuid>
    <OutputType>Library</OutputType>
    <RootNamespace>SalesReportDemo</RootNamespace>
    <AssemblyName>SalesReportDemo</AssemblyName>
    <TargetFrameworkVersion>v4.7.2</TargetFrameworkVersion>
    <LangVersion>7.3</LangVersion>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
    <DebugSymbols>true</DebugSymbols>
    <DebugType>full</DebugType>
    <Optimize>false</Optimize>
    <OutputPath>bin\Debug\</OutputPath>
    <DefineConstants>DEBUG;TRACE</DefineConstants>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|AnyCPU' ">
    <DebugType>pdbonly</DebugType>
    <Optimize>true</Optimize>
    <OutputPath>bin\Release\</OutputPath>
    <DefineConstants>TRACE</DefineConstants>
  </PropertyGroup>
  <ItemGroup>
    <Reference Include="System" />
    <Reference Include="System.Core" />
  </ItemGroup>
  <ItemGroup>
    <Compile Include="Models\MonthlySalesData.cs" />
    <Compile Include="Models\Order.cs" />
    <Compile Include="Models\SalesReportResult.cs" />
    <Compile Include="Repositories\IOrderRepository.cs" />
    <Compile Include="Services\SalesReportService.cs" />
    <Compile Include="Properties\AssemblyInfo.cs" />
  </ItemGroup>
  <Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />
</Project>
```

---

## 4. 모델 클래스

### Models/Order.cs

```csharp
using System;

namespace SalesReportDemo.Models
{
    /// <summary>
    /// 주문 엔티티 (DB 테이블과 매핑)
    /// </summary>
    public class Order
    {
        public int Id { get; set; }
        public string OrderNumber { get; set; }
        public DateTime OrderDate { get; set; }
        public string CustomerName { get; set; }
        public string ProductName { get; set; }
        public int Quantity { get; set; }
        public decimal UnitPrice { get; set; }
        public decimal TotalAmount { get; set; }
        public string Status { get; set; }  // Completed, Cancelled, Pending
        public string Region { get; set; }  // 서울, 부산, 대구 등
        
        public Order()
        {
            OrderNumber = string.Empty;
            CustomerName = string.Empty;
            ProductName = string.Empty;
            Status = string.Empty;
            Region = string.Empty;
        }
    }
}
```

### Models/MonthlySalesData.cs

```csharp
namespace SalesReportDemo.Models
{
    /// <summary>
    /// 월별 매출 집계 데이터
    /// </summary>
    public class MonthlySalesData
    {
        public int Year { get; set; }
        public int Month { get; set; }
        public string MonthName { get; set; }
        public int OrderCount { get; set; }
        public decimal TotalSales { get; set; }
        public decimal AverageOrderValue { get; set; }
        public decimal GrowthRate { get; set; }  // 전월 대비 증감률 (%)
        public string GrowthIndicator { get; set; }  // ▲, ▼, -
        
        public MonthlySalesData()
        {
            MonthName = string.Empty;
            GrowthIndicator = string.Empty;
        }
    }
}
```

### Models/SalesReportResult.cs

```csharp
using System;
using System.Collections.Generic;

namespace SalesReportDemo.Models
{
    /// <summary>
    /// 매출 리포트 결과
    /// </summary>
    public class SalesReportResult
    {
        public string ReportTitle { get; set; }
        public DateTime GeneratedAt { get; set; }
        public int TotalOrders { get; set; }
        public decimal TotalSales { get; set; }
        public decimal AverageMonthlyGrowth { get; set; }
        public string BestMonth { get; set; }
        public string WorstMonth { get; set; }
        public List<MonthlySalesData> MonthlyData { get; set; }
        
        // 지역별 매출
        public Dictionary<string, decimal> SalesByRegion { get; set; }
        
        // 상위 상품
        public List<ProductSalesSummary> TopProducts { get; set; }
        
        public SalesReportResult()
        {
            ReportTitle = string.Empty;
            BestMonth = string.Empty;
            WorstMonth = string.Empty;
            MonthlyData = new List<MonthlySalesData>();
            SalesByRegion = new Dictionary<string, decimal>();
            TopProducts = new List<ProductSalesSummary>();
        }
    }
    
    /// <summary>
    /// 상품별 매출 요약
    /// </summary>
    public class ProductSalesSummary
    {
        public string ProductName { get; set; }
        public int TotalQuantity { get; set; }
        public decimal TotalSales { get; set; }
        public decimal SalesPercentage { get; set; }
        
        public ProductSalesSummary()
        {
            ProductName = string.Empty;
        }
    }
}
```

---

## 5. Repository 인터페이스

### Repositories/IOrderRepository.cs

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using SalesReportDemo.Models;

namespace SalesReportDemo.Repositories
{
    /// <summary>
    /// 주문 데이터 접근 인터페이스
    /// </summary>
    public interface IOrderRepository
    {
        /// <summary>
        /// 기간별 주문 조회
        /// </summary>
        Task<List<Order>> GetOrdersByDateRangeAsync(DateTime startDate, DateTime endDate);
        
        /// <summary>
        /// 완료된 주문만 조회
        /// </summary>
        Task<List<Order>> GetCompletedOrdersAsync(DateTime startDate, DateTime endDate);
        
        /// <summary>
        /// 지역별 주문 조회
        /// </summary>
        Task<List<Order>> GetOrdersByRegionAsync(string region, DateTime startDate, DateTime endDate);
        
        /// <summary>
        /// 상품별 주문 조회
        /// </summary>
        Task<List<Order>> GetOrdersByProductAsync(string productName, DateTime startDate, DateTime endDate);
    }
}
```

---

## 6. 서비스 클래스 (테스트 대상)

### Services/SalesReportService.cs

```csharp
using System;
using System.Collections.Generic;
using System.Globalization;
using System.Linq;
using System.Threading.Tasks;
using SalesReportDemo.Models;
using SalesReportDemo.Repositories;

namespace SalesReportDemo.Services
{
    /// <summary>
    /// 매출 리포트 생성 서비스
    /// </summary>
    public class SalesReportService
    {
        private readonly IOrderRepository _orderRepository;
        private static readonly CultureInfo KoreanCulture = new CultureInfo("ko-KR");
        
        public SalesReportService(IOrderRepository orderRepository)
        {
            if (orderRepository == null)
            {
                throw new ArgumentNullException("orderRepository");
            }
            _orderRepository = orderRepository;
        }
        
        /// <summary>
        /// 연간 매출 리포트 생성
        /// </summary>
        public async Task<SalesReportResult> GenerateAnnualReportAsync(int year)
        {
            var startDate = new DateTime(year, 1, 1);
            var endDate = new DateTime(year, 12, 31, 23, 59, 59);
            
            // DB에서 완료된 주문 조회
            var orders = await _orderRepository.GetCompletedOrdersAsync(startDate, endDate);
            
            if (orders == null || !orders.Any())
            {
                return new SalesReportResult
                {
                    ReportTitle = string.Format("{0}년 매출 리포트", year),
                    GeneratedAt = DateTime.Now,
                    TotalOrders = 0,
                    TotalSales = 0
                };
            }
            
            // 월별 데이터 집계
            var monthlyData = CalculateMonthlyData(orders, year);
            
            // 지역별 매출 계산
            var salesByRegion = CalculateSalesByRegion(orders);
            
            // 상위 상품 계산
            var topProducts = CalculateTopProducts(orders, 5);
            
            // 최고/최저 매출 월 찾기
            var bestMonth = monthlyData.OrderByDescending(m => m.TotalSales).FirstOrDefault();
            var worstMonth = monthlyData.Where(m => m.TotalSales > 0)
                                        .OrderBy(m => m.TotalSales).FirstOrDefault();
            
            // 평균 월별 성장률 계산
            var growthRates = monthlyData.Where(m => m.GrowthRate != 0).Select(m => m.GrowthRate);
            var averageGrowth = growthRates.Any() ? growthRates.Average() : 0;
            
            var result = new SalesReportResult
            {
                ReportTitle = string.Format("{0}년 매출 리포트", year),
                GeneratedAt = DateTime.Now,
                TotalOrders = orders.Count,
                TotalSales = orders.Sum(o => o.TotalAmount),
                AverageMonthlyGrowth = Math.Round(averageGrowth, 2),
                MonthlyData = monthlyData,
                SalesByRegion = salesByRegion,
                TopProducts = topProducts
            };
            
            // 최고/최저 매출 월 설정
            if (bestMonth != null)
            {
                result.BestMonth = string.Format("{0}월 ({1:N0}원)", bestMonth.Month, bestMonth.TotalSales);
            }
            else
            {
                result.BestMonth = "N/A";
            }
            
            if (worstMonth != null)
            {
                result.WorstMonth = string.Format("{0}월 ({1:N0}원)", worstMonth.Month, worstMonth.TotalSales);
            }
            else
            {
                result.WorstMonth = "N/A";
            }
            
            return result;
        }
        
        /// <summary>
        /// 월별 데이터 집계 및 증감률 계산
        /// </summary>
        private List<MonthlySalesData> CalculateMonthlyData(List<Order> orders, int year)
        {
            var monthlyData = new List<MonthlySalesData>();
            decimal previousMonthSales = 0;
            
            for (int month = 1; month <= 12; month++)
            {
                var monthOrders = orders.Where(o => o.OrderDate.Month == month).ToList();
                var totalSales = monthOrders.Sum(o => o.TotalAmount);
                var orderCount = monthOrders.Count;
                
                // 증감률 계산
                decimal growthRate = 0;
                string growthIndicator = "-";
                
                if (month > 1 && previousMonthSales > 0)
                {
                    growthRate = Math.Round(((totalSales - previousMonthSales) / previousMonthSales) * 100, 2);
                    if (growthRate > 0)
                    {
                        growthIndicator = "▲";
                    }
                    else if (growthRate < 0)
                    {
                        growthIndicator = "▼";
                    }
                    else
                    {
                        growthIndicator = "-";
                    }
                }
                
                var data = new MonthlySalesData
                {
                    Year = year,
                    Month = month,
                    MonthName = new DateTime(year, month, 1).ToString("MMMM", KoreanCulture),
                    OrderCount = orderCount,
                    TotalSales = totalSales,
                    AverageOrderValue = orderCount > 0 ? Math.Round(totalSales / orderCount, 0) : 0,
                    GrowthRate = growthRate,
                    GrowthIndicator = growthIndicator
                };
                
                monthlyData.Add(data);
                previousMonthSales = totalSales;
            }
            
            return monthlyData;
        }
        
        /// <summary>
        /// 지역별 매출 계산
        /// </summary>
        private Dictionary<string, decimal> CalculateSalesByRegion(List<Order> orders)
        {
            var grouped = orders
                .GroupBy(o => o.Region)
                .ToDictionary(
                    g => g.Key,
                    g => g.Sum(o => o.TotalAmount)
                );
            
            // 내림차순 정렬된 Dictionary 반환
            var sorted = grouped
                .OrderByDescending(kvp => kvp.Value)
                .ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
            
            return sorted;
        }
        
        /// <summary>
        /// 상위 상품 계산
        /// </summary>
        private List<ProductSalesSummary> CalculateTopProducts(List<Order> orders, int topCount)
        {
            var totalSales = orders.Sum(o => o.TotalAmount);
            
            var products = orders
                .GroupBy(o => o.ProductName)
                .Select(g => new ProductSalesSummary
                {
                    ProductName = g.Key,
                    TotalQuantity = g.Sum(o => o.Quantity),
                    TotalSales = g.Sum(o => o.TotalAmount),
                    SalesPercentage = totalSales > 0 
                        ? Math.Round((g.Sum(o => o.TotalAmount) / totalSales) * 100, 2) 
                        : 0
                })
                .OrderByDescending(p => p.TotalSales)
                .Take(topCount)
                .ToList();
            
            return products;
        }
        
        /// <summary>
        /// 분기별 매출 요약
        /// </summary>
        public async Task<Dictionary<string, decimal>> GetQuarterlySalesAsync(int year)
        {
            var startDate = new DateTime(year, 1, 1);
            var endDate = new DateTime(year, 12, 31, 23, 59, 59);
            
            var orders = await _orderRepository.GetCompletedOrdersAsync(startDate, endDate);
            
            if (orders == null)
            {
                orders = new List<Order>();
            }
            
            var quarterly = new Dictionary<string, decimal>
            {
                { "Q1", orders.Where(o => o.OrderDate.Month >= 1 && o.OrderDate.Month <= 3).Sum(o => o.TotalAmount) },
                { "Q2", orders.Where(o => o.OrderDate.Month >= 4 && o.OrderDate.Month <= 6).Sum(o => o.TotalAmount) },
                { "Q3", orders.Where(o => o.OrderDate.Month >= 7 && o.OrderDate.Month <= 9).Sum(o => o.TotalAmount) },
                { "Q4", orders.Where(o => o.OrderDate.Month >= 10 && o.OrderDate.Month <= 12).Sum(o => o.TotalAmount) }
            };
            
            return quarterly;
        }
    }
}
```

---

## 7. 테스트 프로젝트 파일

### SalesReportDemo.Tests.csproj

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="15.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props" 
          Condition="Exists('$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props')" />
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProjectGuid>{YOUR-TEST-GUID-HERE}</ProjectGuid>
    <OutputType>Library</OutputType>
    <RootNamespace>SalesReportDemo.Tests</RootNamespace>
    <AssemblyName>SalesReportDemo.Tests</AssemblyName>
    <TargetFrameworkVersion>v4.7.2</TargetFrameworkVersion>
    <LangVersion>7.3</LangVersion>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
    <DebugSymbols>true</DebugSymbols>
    <DebugType>full</DebugType>
    <Optimize>false</Optimize>
    <OutputPath>bin\Debug\</OutputPath>
    <DefineConstants>DEBUG;TRACE</DefineConstants>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|AnyCPU' ">
    <DebugType>pdbonly</DebugType>
    <Optimize>true</Optimize>
    <OutputPath>bin\Release\</OutputPath>
    <DefineConstants>TRACE</DefineConstants>
  </PropertyGroup>
  <ItemGroup>
    <Reference Include="System" />
    <Reference Include="System.Core" />
  </ItemGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="16.9.4" />
    <PackageReference Include="Moq" Version="4.16.1" />
    <PackageReference Include="NUnit" Version="3.12.0" />
    <PackageReference Include="NUnit3TestAdapter" Version="3.17.0" />
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="..\SalesReportDemo\SalesReportDemo.csproj">
      <Project>{YOUR-MAIN-PROJECT-GUID}</Project>
      <Name>SalesReportDemo</Name>
    </ProjectReference>
  </ItemGroup>
  <ItemGroup>
    <Compile Include="SalesReportServiceTests.cs" />
    <Compile Include="Properties\AssemblyInfo.cs" />
  </ItemGroup>
  <Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />
</Project>
```

---

## 8. 테스트 클래스 (VS 2017 호환)

### SalesReportServiceTests.cs

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Moq;
using NUnit.Framework;
using SalesReportDemo.Models;
using SalesReportDemo.Repositories;
using SalesReportDemo.Services;

namespace SalesReportDemo.Tests
{
    [TestFixture]
    public class SalesReportServiceTests
    {
        private Mock<IOrderRepository> _mockOrderRepository;
        private SalesReportService _service;
        
        [SetUp]
        public void SetUp()
        {
            _mockOrderRepository = new Mock<IOrderRepository>();
            _service = new SalesReportService(_mockOrderRepository.Object);
        }
        
        #region 테스트 데이터 생성 헬퍼
        
        /// <summary>
        /// 테스트용 주문 데이터 생성
        /// 실제 DB 쿼리 결과를 시뮬레이션
        /// </summary>
        private List<Order> CreateMockOrders()
        {
            var orders = new List<Order>();
            
            // 1월 주문 (3건, 총 150,000원)
            orders.Add(new Order
            {
                Id = 1,
                OrderNumber = "ORD-001",
                OrderDate = new DateTime(2024, 1, 5),
                CustomerName = "김철수",
                ProductName = "노트북",
                Quantity = 1,
                UnitPrice = 100000,
                TotalAmount = 100000,
                Status = "Completed",
                Region = "서울"
            });
            
            orders.Add(new Order
            {
                Id = 2,
                OrderNumber = "ORD-002",
                OrderDate = new DateTime(2024, 1, 15),
                CustomerName = "이영희",
                ProductName = "마우스",
                Quantity = 2,
                UnitPrice = 15000,
                TotalAmount = 30000,
                Status = "Completed",
                Region = "부산"
            });
            
            orders.Add(new Order
            {
                Id = 3,
                OrderNumber = "ORD-003",
                OrderDate = new DateTime(2024, 1, 25),
                CustomerName = "박민수",
                ProductName = "키보드",
                Quantity = 1,
                UnitPrice = 20000,
                TotalAmount = 20000,
                Status = "Completed",
                Region = "서울"
            });
            
            // 2월 주문 (4건, 총 250,000원) - 전월 대비 66.67% 증가
            orders.Add(new Order
            {
                Id = 4,
                OrderNumber = "ORD-004",
                OrderDate = new DateTime(2024, 2, 3),
                CustomerName = "정수진",
                ProductName = "노트북",
                Quantity = 1,
                UnitPrice = 100000,
                TotalAmount = 100000,
                Status = "Completed",
                Region = "대구"
            });
            
            orders.Add(new Order
            {
                Id = 5,
                OrderNumber = "ORD-005",
                OrderDate = new DateTime(2024, 2, 10),
                CustomerName = "최동훈",
                ProductName = "모니터",
                Quantity = 1,
                UnitPrice = 80000,
                TotalAmount = 80000,
                Status = "Completed",
                Region = "서울"
            });
            
            orders.Add(new Order
            {
                Id = 6,
                OrderNumber = "ORD-006",
                OrderDate = new DateTime(2024, 2, 18),
                CustomerName = "강미영",
                ProductName = "마우스",
                Quantity = 2,
                UnitPrice = 15000,
                TotalAmount = 30000,
                Status = "Completed",
                Region = "부산"
            });
            
            orders.Add(new Order
            {
                Id = 7,
                OrderNumber = "ORD-007",
                OrderDate = new DateTime(2024, 2, 28),
                CustomerName = "윤재현",
                ProductName = "키보드",
                Quantity = 2,
                UnitPrice = 20000,
                TotalAmount = 40000,
                Status = "Completed",
                Region = "서울"
            });
            
            // 3월 주문 (2건, 총 180,000원) - 전월 대비 28% 감소
            orders.Add(new Order
            {
                Id = 8,
                OrderNumber = "ORD-008",
                OrderDate = new DateTime(2024, 3, 8),
                CustomerName = "임서연",
                ProductName = "노트북",
                Quantity = 1,
                UnitPrice = 100000,
                TotalAmount = 100000,
                Status = "Completed",
                Region = "서울"
            });
            
            orders.Add(new Order
            {
                Id = 9,
                OrderNumber = "ORD-009",
                OrderDate = new DateTime(2024, 3, 22),
                CustomerName = "한지민",
                ProductName = "모니터",
                Quantity = 1,
                UnitPrice = 80000,
                TotalAmount = 80000,
                Status = "Completed",
                Region = "대구"
            });
            
            return orders;
        }
        
        #endregion
        
        #region 월별 매출 집계 테스트
        
        [Test]
        public async Task GenerateAnnualReport_WithMockData_ReturnsCorrectTotalSales()
        {
            // Arrange - Mock DB 쿼리 결과 설정
            var mockOrders = CreateMockOrders();
            
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(mockOrders);
            
            // Act - 서비스 메서드 호출
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert - 가공된 결과 검증
            // 총 매출 검증: 150,000 + 250,000 + 180,000 = 580,000
            Assert.AreEqual(580000, result.TotalSales, 
                "총 매출이 올바르게 계산되어야 합니다.");
            
            // 총 주문 수 검증
            Assert.AreEqual(9, result.TotalOrders, 
                "총 주문 수가 올바르게 계산되어야 합니다.");
        }
        
        [Test]
        public async Task GenerateAnnualReport_WithMockData_ReturnsCorrectMonthlyData()
        {
            // Arrange
            var mockOrders = CreateMockOrders();
            
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(mockOrders);
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert - 월별 데이터 검증
            var monthlyData = result.MonthlyData;
            
            // 1월 데이터 검증
            var january = monthlyData.First(m => m.Month == 1);
            Assert.AreEqual(3, january.OrderCount, "1월 주문 수");
            Assert.AreEqual(150000, january.TotalSales, "1월 총 매출");
            Assert.AreEqual(50000, january.AverageOrderValue, "1월 평균 주문 금액");
            
            // 2월 데이터 검증
            var february = monthlyData.First(m => m.Month == 2);
            Assert.AreEqual(4, february.OrderCount, "2월 주문 수");
            Assert.AreEqual(250000, february.TotalSales, "2월 총 매출");
            Assert.AreEqual(62500, february.AverageOrderValue, "2월 평균 주문 금액");
            
            // 3월 데이터 검증
            var march = monthlyData.First(m => m.Month == 3);
            Assert.AreEqual(2, march.OrderCount, "3월 주문 수");
            Assert.AreEqual(180000, march.TotalSales, "3월 총 매출");
            Assert.AreEqual(90000, march.AverageOrderValue, "3월 평균 주문 금액");
        }
        
        [Test]
        public async Task GenerateAnnualReport_WithMockData_CalculatesGrowthRateCorrectly()
        {
            // Arrange
            var mockOrders = CreateMockOrders();
            
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(mockOrders);
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            var monthlyData = result.MonthlyData;
            
            // Assert - 증감률 검증
            
            // 1월은 첫 달이므로 증감률 0
            var january = monthlyData.First(m => m.Month == 1);
            Assert.AreEqual(0, january.GrowthRate, "1월 증감률 (첫 달)");
            Assert.AreEqual("-", january.GrowthIndicator, "1월 증감 표시");
            
            // 2월: (250,000 - 150,000) / 150,000 * 100 = 66.67%
            var february = monthlyData.First(m => m.Month == 2);
            Assert.AreEqual(66.67m, february.GrowthRate, 0.01m, "2월 증감률");
            Assert.AreEqual("▲", february.GrowthIndicator, "2월 증감 표시 (상승)");
            
            // 3월: (180,000 - 250,000) / 250,000 * 100 = -28%
            var march = monthlyData.First(m => m.Month == 3);
            Assert.AreEqual(-28m, march.GrowthRate, 0.01m, "3월 증감률");
            Assert.AreEqual("▼", march.GrowthIndicator, "3월 증감 표시 (하락)");
        }
        
        #endregion
        
        #region 지역별 매출 테스트
        
        [Test]
        public async Task GenerateAnnualReport_WithMockData_CalculatesSalesByRegionCorrectly()
        {
            // Arrange
            var mockOrders = CreateMockOrders();
            
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(mockOrders);
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert - 지역별 매출 검증
            var salesByRegion = result.SalesByRegion;
            
            // 서울: 100,000 + 20,000 + 80,000 + 40,000 + 100,000 = 340,000
            Assert.IsTrue(salesByRegion.ContainsKey("서울"), "서울 지역이 존재해야 함");
            Assert.AreEqual(340000, salesByRegion["서울"], "서울 매출");
            
            // 부산: 30,000 + 30,000 = 60,000
            Assert.IsTrue(salesByRegion.ContainsKey("부산"), "부산 지역이 존재해야 함");
            Assert.AreEqual(60000, salesByRegion["부산"], "부산 매출");
            
            // 대구: 100,000 + 80,000 = 180,000
            Assert.IsTrue(salesByRegion.ContainsKey("대구"), "대구 지역이 존재해야 함");
            Assert.AreEqual(180000, salesByRegion["대구"], "대구 매출");
            
            // 지역 수 검증
            Assert.AreEqual(3, salesByRegion.Count, "총 지역 수");
        }
        
        [Test]
        public async Task GenerateAnnualReport_WithMockData_SalesByRegionIsSortedDescending()
        {
            // Arrange
            var mockOrders = CreateMockOrders();
            
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(mockOrders);
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert - 내림차순 정렬 검증
            var regions = result.SalesByRegion.Keys.ToList();
            var sales = result.SalesByRegion.Values.ToList();
            
            // 첫 번째가 서울 (최고 매출)
            Assert.AreEqual("서울", regions[0], "1위는 서울");
            Assert.AreEqual(340000, sales[0], "서울 매출");
            
            // 두 번째가 대구
            Assert.AreEqual("대구", regions[1], "2위는 대구");
            Assert.AreEqual(180000, sales[1], "대구 매출");
            
            // 세 번째가 부산
            Assert.AreEqual("부산", regions[2], "3위는 부산");
            Assert.AreEqual(60000, sales[2], "부산 매출");
        }
        
        #endregion
        
        #region 상위 상품 테스트
        
        [Test]
        public async Task GenerateAnnualReport_WithMockData_ReturnsTopProductsCorrectly()
        {
            // Arrange
            var mockOrders = CreateMockOrders();
            
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(mockOrders);
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert - 상위 상품 검증
            var topProducts = result.TopProducts;
            
            // 노트북: 3개, 300,000원, 51.72%
            var laptop = topProducts.First(p => p.ProductName == "노트북");
            Assert.AreEqual(3, laptop.TotalQuantity, "노트북 판매 수량");
            Assert.AreEqual(300000, laptop.TotalSales, "노트북 총 매출");
            Assert.AreEqual(51.72m, laptop.SalesPercentage, 0.01m, "노트북 매출 비율");
            
            // 모니터: 2개, 160,000원, 27.59%
            var monitor = topProducts.First(p => p.ProductName == "모니터");
            Assert.AreEqual(2, monitor.TotalQuantity, "모니터 판매 수량");
            Assert.AreEqual(160000, monitor.TotalSales, "모니터 총 매출");
            Assert.AreEqual(27.59m, monitor.SalesPercentage, 0.01m, "모니터 매출 비율");
            
            // 마우스: 4개, 60,000원, 10.34%
            var mouse = topProducts.First(p => p.ProductName == "마우스");
            Assert.AreEqual(4, mouse.TotalQuantity, "마우스 판매 수량");
            Assert.AreEqual(60000, mouse.TotalSales, "마우스 총 매출");
            Assert.AreEqual(10.34m, mouse.SalesPercentage, 0.01m, "마우스 매출 비율");
            
            // 키보드: 3개, 60,000원, 10.34%
            var keyboard = topProducts.First(p => p.ProductName == "키보드");
            Assert.AreEqual(3, keyboard.TotalQuantity, "키보드 판매 수량");
            Assert.AreEqual(60000, keyboard.TotalSales, "키보드 총 매출");
            Assert.AreEqual(10.34m, keyboard.SalesPercentage, 0.01m, "키보드 매출 비율");
        }
        
        [Test]
        public async Task GenerateAnnualReport_WithMockData_TopProductsAreSortedBySales()
        {
            // Arrange
            var mockOrders = CreateMockOrders();
            
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(mockOrders);
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert - 매출 내림차순 정렬 검증
            var topProducts = result.TopProducts;
            
            Assert.AreEqual("노트북", topProducts[0].ProductName, "1위 상품");
            Assert.AreEqual("모니터", topProducts[1].ProductName, "2위 상품");
            // 3, 4위는 마우스/키보드 (같은 매출이므로 순서 무관)
            
            // 매출 순서 검증
            for (int i = 0; i < topProducts.Count - 1; i++)
            {
                Assert.GreaterOrEqual(
                    topProducts[i].TotalSales, 
                    topProducts[i + 1].TotalSales,
                    "상품이 매출 내림차순으로 정렬되어야 함");
            }
        }
        
        #endregion
        
        #region 분기별 매출 테스트
        
        [Test]
        public async Task GetQuarterlySales_WithMockData_ReturnsCorrectQuarterlyData()
        {
            // Arrange
            var mockOrders = CreateMockOrders();
            
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(mockOrders);
            
            // Act
            var result = await _service.GetQuarterlySalesAsync(2024);
            
            // Assert - 분기별 매출 검증
            // Q1: 1월(150,000) + 2월(250,000) + 3월(180,000) = 580,000
            Assert.AreEqual(580000, result["Q1"], "1분기 매출");
            
            // Q2, Q3, Q4는 데이터 없음 (0원)
            Assert.AreEqual(0, result["Q2"], "2분기 매출");
            Assert.AreEqual(0, result["Q3"], "3분기 매출");
            Assert.AreEqual(0, result["Q4"], "4분기 매출");
        }
        
        #endregion
        
        #region 엣지 케이스 테스트
        
        [Test]
        public async Task GenerateAnnualReport_WithEmptyData_ReturnsEmptyReport()
        {
            // Arrange - 빈 결과 반환
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(new List<Order>());
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert
            Assert.AreEqual(0, result.TotalOrders, "주문 수는 0");
            Assert.AreEqual(0, result.TotalSales, "매출은 0");
            Assert.IsEmpty(result.MonthlyData, "월별 데이터 없음");
        }
        
        [Test]
        public async Task GenerateAnnualReport_WithNullData_ReturnsEmptyReport()
        {
            // Arrange - null 반환
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync((List<Order>)null);
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert
            Assert.AreEqual(0, result.TotalOrders);
            Assert.AreEqual(0, result.TotalSales);
        }
        
        [Test]
        public async Task GenerateAnnualReport_WithSingleOrder_CalculatesCorrectly()
        {
            // Arrange - 단일 주문
            var singleOrder = new List<Order>
            {
                new Order
                {
                    Id = 1,
                    OrderDate = new DateTime(2024, 6, 15),
                    ProductName = "테스트 상품",
                    TotalAmount = 50000,
                    Status = "Completed",
                    Region = "서울",
                    Quantity = 1
                }
            };
            
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(singleOrder);
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert
            Assert.AreEqual(1, result.TotalOrders);
            Assert.AreEqual(50000, result.TotalSales);
            
            var juneData = result.MonthlyData.First(m => m.Month == 6);
            Assert.AreEqual(1, juneData.OrderCount);
            Assert.AreEqual(50000, juneData.TotalSales);
            Assert.AreEqual(50000, juneData.AverageOrderValue);
        }
        
        #endregion
        
        #region 종합 테이블 형태 검증 테스트
        
        [Test]
        public async Task GenerateAnnualReport_WithMockData_ProducesExpectedTableData()
        {
            // Arrange - Mock DB 결과 설정
            var mockOrders = CreateMockOrders();
            
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(mockOrders);
            
            // 기대하는 월별 결과 정의
            var expectedMonthlyResults = new Dictionary<int, ExpectedMonthlyData>
            {
                { 1, new ExpectedMonthlyData { OrderCount = 3, TotalSales = 150000m, AvgOrder = 50000m, GrowthRate = 0m, Indicator = "-" } },
                { 2, new ExpectedMonthlyData { OrderCount = 4, TotalSales = 250000m, AvgOrder = 62500m, GrowthRate = 66.67m, Indicator = "▲" } },
                { 3, new ExpectedMonthlyData { OrderCount = 2, TotalSales = 180000m, AvgOrder = 90000m, GrowthRate = -28m, Indicator = "▼" } }
            };
            
            // 기대하는 지역별 결과 정의
            var expectedRegionResults = new Dictionary<string, decimal>
            {
                { "서울", 340000m },
                { "대구", 180000m },
                { "부산", 60000m }
            };
            
            // 기대하는 상품별 결과 정의
            var expectedProductResults = new List<ExpectedProductData>
            {
                new ExpectedProductData { ProductName = "노트북", Quantity = 3, Sales = 300000m, Percentage = 51.72m },
                new ExpectedProductData { ProductName = "모니터", Quantity = 2, Sales = 160000m, Percentage = 27.59m },
                new ExpectedProductData { ProductName = "마우스", Quantity = 4, Sales = 60000m, Percentage = 10.34m },
                new ExpectedProductData { ProductName = "키보드", Quantity = 3, Sales = 60000m, Percentage = 10.34m }
            };
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert - 월별 테이블 검증
            Console.WriteLine("=== 월별 매출 테이블 검증 ===");
            Console.WriteLine("| 월 | 주문수 | 총매출 | 평균주문금액 | 증감률 | 표시 |");
            Console.WriteLine("|---|---|---|---|---|---|");
            
            foreach (var expected in expectedMonthlyResults)
            {
                var actual = result.MonthlyData.First(m => m.Month == expected.Key);
                var exp = expected.Value;
                
                Console.WriteLine(string.Format("| {0}월 | {1} | {2:N0} | {3:N0} | {4}% | {5} |",
                    expected.Key, actual.OrderCount, actual.TotalSales, 
                    actual.AverageOrderValue, actual.GrowthRate, actual.GrowthIndicator));
                
                Assert.AreEqual(exp.OrderCount, actual.OrderCount, 
                    string.Format("{0}월 주문 수 불일치", expected.Key));
                Assert.AreEqual(exp.TotalSales, actual.TotalSales, 
                    string.Format("{0}월 총 매출 불일치", expected.Key));
                Assert.AreEqual(exp.AvgOrder, actual.AverageOrderValue, 
                    string.Format("{0}월 평균 주문 금액 불일치", expected.Key));
                Assert.AreEqual(exp.GrowthRate, actual.GrowthRate, 0.01m, 
                    string.Format("{0}월 증감률 불일치", expected.Key));
                Assert.AreEqual(exp.Indicator, actual.GrowthIndicator, 
                    string.Format("{0}월 증감 표시 불일치", expected.Key));
            }
            
            // Assert - 지역별 테이블 검증
            Console.WriteLine("\n=== 지역별 매출 테이블 검증 ===");
            Console.WriteLine("| 순위 | 지역 | 매출 |");
            Console.WriteLine("|---|---|---|");
            
            var regionList = result.SalesByRegion.ToList();
            int rank = 1;
            foreach (var actual in regionList)
            {
                Console.WriteLine(string.Format("| {0} | {1} | {2:N0}원 |", 
                    rank, actual.Key, actual.Value));
                
                Assert.IsTrue(expectedRegionResults.ContainsKey(actual.Key),
                    string.Format("{0} 지역이 예상 결과에 없음", actual.Key));
                Assert.AreEqual(expectedRegionResults[actual.Key], actual.Value, 
                    string.Format("{0} 매출 불일치", actual.Key));
                    
                rank++;
            }
            
            // Assert - 상품별 테이블 검증
            Console.WriteLine("\n=== 상위 상품 테이블 검증 ===");
            Console.WriteLine("| 순위 | 상품명 | 수량 | 매출 | 비율 |");
            Console.WriteLine("|---|---|---|---|---|");
            
            for (int i = 0; i < expectedProductResults.Count; i++)
            {
                var expected = expectedProductResults[i];
                var actual = result.TopProducts[i];
                
                Console.WriteLine(string.Format("| {0} | {1} | {2} | {3:N0}원 | {4}% |",
                    i + 1, actual.ProductName, actual.TotalQuantity, 
                    actual.TotalSales, actual.SalesPercentage));
                
                Assert.AreEqual(expected.ProductName, actual.ProductName, 
                    string.Format("{0}위 상품명 불일치", i + 1));
                Assert.AreEqual(expected.Quantity, actual.TotalQuantity, 
                    string.Format("{0} 수량 불일치", expected.ProductName));
                Assert.AreEqual(expected.Sales, actual.TotalSales, 
                    string.Format("{0} 매출 불일치", expected.ProductName));
                Assert.AreEqual(expected.Percentage, actual.SalesPercentage, 0.01m, 
                    string.Format("{0} 비율 불일치", expected.ProductName));
            }
            
            // 전체 요약 검증
            Console.WriteLine("\n=== 전체 요약 ===");
            Console.WriteLine(string.Format("총 주문 수: {0}", result.TotalOrders));
            Console.WriteLine(string.Format("총 매출: {0:N0}원", result.TotalSales));
            Console.WriteLine(string.Format("최고 매출 월: {0}", result.BestMonth));
            Console.WriteLine(string.Format("최저 매출 월: {0}", result.WorstMonth));
            
            Assert.AreEqual(9, result.TotalOrders);
            Assert.AreEqual(580000, result.TotalSales);
        }
        
        #endregion
        
        #region Verify 호출 검증
        
        [Test]
        public async Task GenerateAnnualReport_CallsRepositoryWithCorrectDateRange()
        {
            // Arrange
            _mockOrderRepository
                .Setup(repo => repo.GetCompletedOrdersAsync(
                    It.IsAny<DateTime>(), 
                    It.IsAny<DateTime>()))
                .ReturnsAsync(new List<Order>());
            
            // Act
            await _service.GenerateAnnualReportAsync(2024);
            
            // Assert - Repository가 올바른 날짜 범위로 호출되었는지 검증
            _mockOrderRepository.Verify(
                repo => repo.GetCompletedOrdersAsync(
                    It.Is<DateTime>(d => d == new DateTime(2024, 1, 1)),
                    It.Is<DateTime>(d => d.Year == 2024 && d.Month == 12 && d.Day == 31)),
                Times.Once,
                "Repository가 2024년 전체 기간으로 호출되어야 함");
        }
        
        #endregion
        
        #region 기대값 헬퍼 클래스
        
        /// <summary>
        /// 월별 기대값 헬퍼 클래스
        /// </summary>
        private class ExpectedMonthlyData
        {
            public int OrderCount { get; set; }
            public decimal TotalSales { get; set; }
            public decimal AvgOrder { get; set; }
            public decimal GrowthRate { get; set; }
            public string Indicator { get; set; }
        }
        
        /// <summary>
        /// 상품별 기대값 헬퍼 클래스
        /// </summary>
        private class ExpectedProductData
        {
            public string ProductName { get; set; }
            public int Quantity { get; set; }
            public decimal Sales { get; set; }
            public decimal Percentage { get; set; }
        }
        
        #endregion
    }
}
```

---

## 9. Visual Studio 2017에서 실행

### Step 9.1: 테스트 실행

```
1. 빌드 → 솔루션 빌드 (Ctrl + Shift + B)
2. 테스트 → 테스트 탐색기 (Ctrl + E, T)
3. "모두 실행" 클릭
```

### Step 9.2: 예상 결과

```
┌────────────────────────────────────────────────────────────────┐
│  테스트 탐색기                                                 │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ✅ SalesReportServiceTests (15 테스트)                        │
│  │                                                             │
│  ├── ✅ GenerateAnnualReport_WithMockData_ReturnsCorrect...   │
│  ├── ✅ GenerateAnnualReport_WithMockData_ReturnsCorrect...   │
│  ├── ✅ GenerateAnnualReport_WithMockData_CalculatesGrowth... │
│  ├── ✅ GenerateAnnualReport_WithMockData_CalculatesSales...  │
│  ├── ✅ GenerateAnnualReport_WithMockData_SalesByRegionIs...  │
│  ├── ✅ GenerateAnnualReport_WithMockData_ReturnsTopProd...   │
│  ├── ✅ GenerateAnnualReport_WithMockData_TopProductsAre...   │
│  ├── ✅ GetQuarterlySales_WithMockData_ReturnsCorrectQua...   │
│  ├── ✅ GenerateAnnualReport_WithEmptyData_ReturnsEmptyR...   │
│  ├── ✅ GenerateAnnualReport_WithNullData_ReturnsEmptyRe...   │
│  ├── ✅ GenerateAnnualReport_WithSingleOrder_CalculatesCo...  │
│  ├── ✅ GenerateAnnualReport_WithMockData_ProducesExpect...   │
│  └── ✅ GenerateAnnualReport_CallsRepositoryWithCorrectD...   │
│                                                                │
│  통과: 15  실패: 0  건너뜀: 0                                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 10. VS 2017 호환을 위한 주요 변경 사항

```
┌────────────────────────────────────────────────────────────────┐
│  VS 2017 (.NET Framework 4.7.2) 호환 변경 사항                 │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. 문법 변경                                                  │
│  ─────────────────────────────────────────────────────────────│
│  ❌ string Property { get; set; } = string.Empty;             │
│  ✅ 생성자에서 초기화                                         │
│                                                                │
│  ❌ nameof(parameter)를 ArgumentNullException에서 사용        │
│  ✅ 문자열 리터럴 사용 "parameterName"                        │
│                                                                │
│  ❌ 문자열 보간 $"text {value}"                               │
│  ✅ string.Format("text {0}", value) 사용                     │
│                                                                │
│  ❌ 튜플 (Month: 1, Sales: 100)                               │
│  ✅ 헬퍼 클래스 또는 Dictionary 사용                          │
│                                                                │
│  2. 프로젝트 설정                                              │
│  ─────────────────────────────────────────────────────────────│
│  • TargetFrameworkVersion: v4.7.2                             │
│  • LangVersion: 7.3                                           │
│                                                                │
│  3. NuGet 패키지 버전                                          │
│  ─────────────────────────────────────────────────────────────│
│  • NUnit: 3.12.0                                              │
│  • NUnit3TestAdapter: 3.17.0                                  │
│  • Moq: 4.16.1                                                │
│  • Microsoft.NET.Test.Sdk: 16.9.4                             │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

**수고하셨습니다! 🎉**
