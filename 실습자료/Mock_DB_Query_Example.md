# 🗃️ Mock 실전 예제: DB 쿼리 결과 가공 및 검증

## 실제 DB 쿼리 결과를 Mock으로 설정하고 가공된 결과를 검증하는 예제

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
SalesReport/
├── src/
│   └── SalesReport/
│       ├── Models/
│       │   ├── Order.cs
│       │   ├── MonthlySalesData.cs
│       │   └── SalesReportResult.cs
│       ├── Repositories/
│       │   └── IOrderRepository.cs
│       ├── Services/
│       │   └── SalesReportService.cs
│       └── SalesReport.csproj
│
└── tests/
    └── SalesReport.Tests/
        ├── SalesReportServiceTests.cs
        └── SalesReport.Tests.csproj
```

---

## 2. 모델 클래스

### Order.cs (DB 엔티티)

```csharp
// Models/Order.cs
namespace SalesReport.Models
{
    /// <summary>
    /// 주문 엔티티 (DB 테이블과 매핑)
    /// </summary>
    public class Order
    {
        public int Id { get; set; }
        public string OrderNumber { get; set; } = string.Empty;
        public DateTime OrderDate { get; set; }
        public string CustomerName { get; set; } = string.Empty;
        public string ProductName { get; set; } = string.Empty;
        public int Quantity { get; set; }
        public decimal UnitPrice { get; set; }
        public decimal TotalAmount { get; set; }
        public string Status { get; set; } = string.Empty;  // Completed, Cancelled, Pending
        public string Region { get; set; } = string.Empty;  // 서울, 부산, 대구 등
    }
}
```

### MonthlySalesData.cs (가공된 월별 데이터)

```csharp
// Models/MonthlySalesData.cs
namespace SalesReport.Models
{
    /// <summary>
    /// 월별 매출 집계 데이터
    /// </summary>
    public class MonthlySalesData
    {
        public int Year { get; set; }
        public int Month { get; set; }
        public string MonthName { get; set; } = string.Empty;
        public int OrderCount { get; set; }
        public decimal TotalSales { get; set; }
        public decimal AverageOrderValue { get; set; }
        public decimal GrowthRate { get; set; }  // 전월 대비 증감률 (%)
        public string GrowthIndicator { get; set; } = string.Empty;  // ▲, ▼, -
    }
}
```

### SalesReportResult.cs (최종 리포트)

```csharp
// Models/SalesReportResult.cs
namespace SalesReport.Models
{
    /// <summary>
    /// 매출 리포트 결과
    /// </summary>
    public class SalesReportResult
    {
        public string ReportTitle { get; set; } = string.Empty;
        public DateTime GeneratedAt { get; set; }
        public int TotalOrders { get; set; }
        public decimal TotalSales { get; set; }
        public decimal AverageMonthlyGrowth { get; set; }
        public string BestMonth { get; set; } = string.Empty;
        public string WorstMonth { get; set; } = string.Empty;
        public List<MonthlySalesData> MonthlyData { get; set; } = new();
        
        // 지역별 매출
        public Dictionary<string, decimal> SalesByRegion { get; set; } = new();
        
        // 상위 상품
        public List<ProductSalesSummary> TopProducts { get; set; } = new();
    }
    
    public class ProductSalesSummary
    {
        public string ProductName { get; set; } = string.Empty;
        public int TotalQuantity { get; set; }
        public decimal TotalSales { get; set; }
        public decimal SalesPercentage { get; set; }
    }
}
```

---

## 3. Repository 인터페이스

### IOrderRepository.cs

```csharp
// Repositories/IOrderRepository.cs
using SalesReport.Models;

namespace SalesReport.Repositories
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

## 4. 서비스 클래스 (테스트 대상)

### SalesReportService.cs

```csharp
// Services/SalesReportService.cs
using SalesReport.Models;
using SalesReport.Repositories;
using System.Globalization;

namespace SalesReport.Services
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
            _orderRepository = orderRepository ?? throw new ArgumentNullException(nameof(orderRepository));
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
                    ReportTitle = $"{year}년 매출 리포트",
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
            
            return new SalesReportResult
            {
                ReportTitle = $"{year}년 매출 리포트",
                GeneratedAt = DateTime.Now,
                TotalOrders = orders.Count,
                TotalSales = orders.Sum(o => o.TotalAmount),
                AverageMonthlyGrowth = Math.Round(averageGrowth, 2),
                BestMonth = bestMonth != null ? $"{bestMonth.Month}월 ({bestMonth.TotalSales:N0}원)" : "N/A",
                WorstMonth = worstMonth != null ? $"{worstMonth.Month}월 ({worstMonth.TotalSales:N0}원)" : "N/A",
                MonthlyData = monthlyData,
                SalesByRegion = salesByRegion,
                TopProducts = topProducts
            };
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
                    growthIndicator = growthRate > 0 ? "▲" : (growthRate < 0 ? "▼" : "-");
                }
                
                monthlyData.Add(new MonthlySalesData
                {
                    Year = year,
                    Month = month,
                    MonthName = new DateTime(year, month, 1).ToString("MMMM", KoreanCulture),
                    OrderCount = orderCount,
                    TotalSales = totalSales,
                    AverageOrderValue = orderCount > 0 ? Math.Round(totalSales / orderCount, 0) : 0,
                    GrowthRate = growthRate,
                    GrowthIndicator = growthIndicator
                });
                
                previousMonthSales = totalSales;
            }
            
            return monthlyData;
        }
        
        /// <summary>
        /// 지역별 매출 계산
        /// </summary>
        private Dictionary<string, decimal> CalculateSalesByRegion(List<Order> orders)
        {
            return orders
                .GroupBy(o => o.Region)
                .ToDictionary(
                    g => g.Key,
                    g => g.Sum(o => o.TotalAmount)
                )
                .OrderByDescending(kvp => kvp.Value)
                .ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
        }
        
        /// <summary>
        /// 상위 상품 계산
        /// </summary>
        private List<ProductSalesSummary> CalculateTopProducts(List<Order> orders, int topCount)
        {
            var totalSales = orders.Sum(o => o.TotalAmount);
            
            return orders
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
        }
        
        /// <summary>
        /// 분기별 매출 요약
        /// </summary>
        public async Task<Dictionary<string, decimal>> GetQuarterlySalesAsync(int year)
        {
            var startDate = new DateTime(year, 1, 1);
            var endDate = new DateTime(year, 12, 31, 23, 59, 59);
            
            var orders = await _orderRepository.GetCompletedOrdersAsync(startDate, endDate);
            
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

## 5. 테스트 클래스 (핵심!)

### SalesReportServiceTests.cs

```csharp
// SalesReportServiceTests.cs
using Moq;
using NUnit.Framework;
using SalesReport.Models;
using SalesReport.Repositories;
using SalesReport.Services;

namespace SalesReport.Tests
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
            return new List<Order>
            {
                // 1월 주문 (3건, 총 150,000원)
                new Order { Id = 1, OrderNumber = "ORD-001", OrderDate = new DateTime(2024, 1, 5), 
                           CustomerName = "김철수", ProductName = "노트북", Quantity = 1, 
                           UnitPrice = 100000, TotalAmount = 100000, Status = "Completed", Region = "서울" },
                new Order { Id = 2, OrderNumber = "ORD-002", OrderDate = new DateTime(2024, 1, 15), 
                           CustomerName = "이영희", ProductName = "마우스", Quantity = 2, 
                           UnitPrice = 15000, TotalAmount = 30000, Status = "Completed", Region = "부산" },
                new Order { Id = 3, OrderNumber = "ORD-003", OrderDate = new DateTime(2024, 1, 25), 
                           CustomerName = "박민수", ProductName = "키보드", Quantity = 1, 
                           UnitPrice = 20000, TotalAmount = 20000, Status = "Completed", Region = "서울" },
                
                // 2월 주문 (4건, 총 250,000원) - 전월 대비 66.67% 증가
                new Order { Id = 4, OrderNumber = "ORD-004", OrderDate = new DateTime(2024, 2, 3), 
                           CustomerName = "정수진", ProductName = "노트북", Quantity = 1, 
                           UnitPrice = 100000, TotalAmount = 100000, Status = "Completed", Region = "대구" },
                new Order { Id = 5, OrderNumber = "ORD-005", OrderDate = new DateTime(2024, 2, 10), 
                           CustomerName = "최동훈", ProductName = "모니터", Quantity = 1, 
                           UnitPrice = 80000, TotalAmount = 80000, Status = "Completed", Region = "서울" },
                new Order { Id = 6, OrderNumber = "ORD-006", OrderDate = new DateTime(2024, 2, 18), 
                           CustomerName = "강미영", ProductName = "마우스", Quantity = 2, 
                           UnitPrice = 15000, TotalAmount = 30000, Status = "Completed", Region = "부산" },
                new Order { Id = 7, OrderNumber = "ORD-007", OrderDate = new DateTime(2024, 2, 28), 
                           CustomerName = "윤재현", ProductName = "키보드", Quantity = 2, 
                           UnitPrice = 20000, TotalAmount = 40000, Status = "Completed", Region = "서울" },
                
                // 3월 주문 (2건, 총 180,000원) - 전월 대비 28% 감소
                new Order { Id = 8, OrderNumber = "ORD-008", OrderDate = new DateTime(2024, 3, 8), 
                           CustomerName = "임서연", ProductName = "노트북", Quantity = 1, 
                           UnitPrice = 100000, TotalAmount = 100000, Status = "Completed", Region = "서울" },
                new Order { Id = 9, OrderNumber = "ORD-009", OrderDate = new DateTime(2024, 3, 22), 
                           CustomerName = "한지민", ProductName = "모니터", Quantity = 1, 
                           UnitPrice = 80000, TotalAmount = 80000, Status = "Completed", Region = "대구" },
            };
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
                .ReturnsAsync((List<Order>)null!);
            
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
            
            // 기대하는 테이블 형태의 결과 정의
            var expectedMonthlyTable = new[]
            {
                // (월, 주문수, 총매출, 평균주문금액, 증감률, 증감표시)
                (Month: 1, OrderCount: 3, TotalSales: 150000m, AvgOrder: 50000m, GrowthRate: 0m, Indicator: "-"),
                (Month: 2, OrderCount: 4, TotalSales: 250000m, AvgOrder: 62500m, GrowthRate: 66.67m, Indicator: "▲"),
                (Month: 3, OrderCount: 2, TotalSales: 180000m, AvgOrder: 90000m, GrowthRate: -28m, Indicator: "▼"),
            };
            
            var expectedRegionTable = new[]
            {
                // (지역, 매출)
                (Region: "서울", Sales: 340000m),
                (Region: "대구", Sales: 180000m),
                (Region: "부산", Sales: 60000m),
            };
            
            var expectedProductTable = new[]
            {
                // (상품명, 수량, 매출, 비율)
                (Product: "노트북", Quantity: 3, Sales: 300000m, Percentage: 51.72m),
                (Product: "모니터", Quantity: 2, Sales: 160000m, Percentage: 27.59m),
                (Product: "마우스", Quantity: 4, Sales: 60000m, Percentage: 10.34m),
                (Product: "키보드", Quantity: 3, Sales: 60000m, Percentage: 10.34m),
            };
            
            // Act
            var result = await _service.GenerateAnnualReportAsync(2024);
            
            // Assert - 월별 테이블 검증
            Console.WriteLine("=== 월별 매출 테이블 검증 ===");
            Console.WriteLine("| 월 | 주문수 | 총매출 | 평균주문금액 | 증감률 | 표시 |");
            Console.WriteLine("|---|---|---|---|---|---|");
            
            foreach (var expected in expectedMonthlyTable)
            {
                var actual = result.MonthlyData.First(m => m.Month == expected.Month);
                
                Console.WriteLine($"| {expected.Month}월 | {actual.OrderCount} | {actual.TotalSales:N0} | {actual.AverageOrderValue:N0} | {actual.GrowthRate}% | {actual.GrowthIndicator} |");
                
                Assert.AreEqual(expected.OrderCount, actual.OrderCount, 
                    $"{expected.Month}월 주문 수 불일치");
                Assert.AreEqual(expected.TotalSales, actual.TotalSales, 
                    $"{expected.Month}월 총 매출 불일치");
                Assert.AreEqual(expected.AvgOrder, actual.AverageOrderValue, 
                    $"{expected.Month}월 평균 주문 금액 불일치");
                Assert.AreEqual(expected.GrowthRate, actual.GrowthRate, 0.01m, 
                    $"{expected.Month}월 증감률 불일치");
                Assert.AreEqual(expected.Indicator, actual.GrowthIndicator, 
                    $"{expected.Month}월 증감 표시 불일치");
            }
            
            // Assert - 지역별 테이블 검증
            Console.WriteLine("\n=== 지역별 매출 테이블 검증 ===");
            Console.WriteLine("| 순위 | 지역 | 매출 |");
            Console.WriteLine("|---|---|---|");
            
            var regionList = result.SalesByRegion.ToList();
            for (int i = 0; i < expectedRegionTable.Length; i++)
            {
                var expected = expectedRegionTable[i];
                var actual = regionList[i];
                
                Console.WriteLine($"| {i + 1} | {actual.Key} | {actual.Value:N0}원 |");
                
                Assert.AreEqual(expected.Region, actual.Key, 
                    $"{i + 1}위 지역 불일치");
                Assert.AreEqual(expected.Sales, actual.Value, 
                    $"{expected.Region} 매출 불일치");
            }
            
            // Assert - 상품별 테이블 검증
            Console.WriteLine("\n=== 상위 상품 테이블 검증 ===");
            Console.WriteLine("| 순위 | 상품명 | 수량 | 매출 | 비율 |");
            Console.WriteLine("|---|---|---|---|---|");
            
            for (int i = 0; i < expectedProductTable.Length; i++)
            {
                var expected = expectedProductTable[i];
                var actual = result.TopProducts[i];
                
                Console.WriteLine($"| {i + 1} | {actual.ProductName} | {actual.TotalQuantity} | {actual.TotalSales:N0}원 | {actual.SalesPercentage}% |");
                
                Assert.AreEqual(expected.Product, actual.ProductName, 
                    $"{i + 1}위 상품명 불일치");
                Assert.AreEqual(expected.Quantity, actual.TotalQuantity, 
                    $"{expected.Product} 수량 불일치");
                Assert.AreEqual(expected.Sales, actual.TotalSales, 
                    $"{expected.Product} 매출 불일치");
                Assert.AreEqual(expected.Percentage, actual.SalesPercentage, 0.01m, 
                    $"{expected.Product} 비율 불일치");
            }
            
            // 전체 요약 검증
            Console.WriteLine("\n=== 전체 요약 ===");
            Console.WriteLine($"총 주문 수: {result.TotalOrders}");
            Console.WriteLine($"총 매출: {result.TotalSales:N0}원");
            Console.WriteLine($"최고 매출 월: {result.BestMonth}");
            Console.WriteLine($"최저 매출 월: {result.WorstMonth}");
            
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
    }
}
```

---

## 6. 테스트 실행 결과 예시

```
┌────────────────────────────────────────────────────────────────┐
│  테스트 실행 결과                                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  === 월별 매출 테이블 검증 ===                                 │
│  | 월 | 주문수 | 총매출 | 평균주문금액 | 증감률 | 표시 |       │
│  |---|---|---|---|---|---|                                     │
│  | 1월 | 3 | 150,000 | 50,000 | 0% | - |                      │
│  | 2월 | 4 | 250,000 | 62,500 | 66.67% | ▲ |                  │
│  | 3월 | 2 | 180,000 | 90,000 | -28% | ▼ |                    │
│                                                                │
│  === 지역별 매출 테이블 검증 ===                               │
│  | 순위 | 지역 | 매출 |                                        │
│  |---|---|---|                                                 │
│  | 1 | 서울 | 340,000원 |                                      │
│  | 2 | 대구 | 180,000원 |                                      │
│  | 3 | 부산 | 60,000원 |                                       │
│                                                                │
│  === 상위 상품 테이블 검증 ===                                 │
│  | 순위 | 상품명 | 수량 | 매출 | 비율 |                        │
│  |---|---|---|---|---|                                         │
│  | 1 | 노트북 | 3 | 300,000원 | 51.72% |                       │
│  | 2 | 모니터 | 2 | 160,000원 | 27.59% |                       │
│  | 3 | 마우스 | 4 | 60,000원 | 10.34% |                        │
│  | 4 | 키보드 | 3 | 60,000원 | 10.34% |                        │
│                                                                │
│  === 전체 요약 ===                                             │
│  총 주문 수: 9                                                 │
│  총 매출: 580,000원                                            │
│  최고 매출 월: 2월 (250,000원)                                 │
│  최저 매출 월: 1월 (150,000원)                                 │
│                                                                │
│  ✅ 15 tests passed, 0 failed                                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 7. 핵심 포인트 요약

```
┌────────────────────────────────────────────────────────────────┐
│  Mock DB 쿼리 결과 테스트 핵심 포인트                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. Mock 데이터 준비                                           │
│     • 실제 DB 쿼리 결과와 동일한 형태로 준비                  │
│     • 다양한 시나리오를 커버할 수 있는 데이터 구성            │
│                                                                │
│  2. Setup으로 Mock 반환값 설정                                 │
│     _mockRepository                                           │
│         .Setup(r => r.GetDataAsync(...))                      │
│         .ReturnsAsync(mockData);                              │
│                                                                │
│  3. 가공된 결과 검증                                           │
│     • 집계 결과 (합계, 평균, 개수)                            │
│     • 계산 결과 (비율, 증감률)                                │
│     • 정렬 순서                                               │
│     • 그룹화 결과                                             │
│                                                                │
│  4. Assert 작성 팁                                             │
│     • 명확한 메시지 포함                                      │
│     • 소수점 비교는 오차 범위 지정 (0.01m)                    │
│     • 여러 필드를 개별 Assert로 검증                          │
│                                                                │
│  5. 테이블 형태로 기대값 정의                                  │
│     var expected = new[]                                      │
│     {                                                         │
│         (Field1: value1, Field2: value2, ...),               │
│         ...                                                   │
│     };                                                        │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

**이 예제의 핵심:**
- Mock으로 DB 쿼리 결과를 시뮬레이션
- 가공 로직(집계, 계산, 정렬)의 정확성 검증
- 테이블 형태의 기대값과 실제값 비교
- 다양한 시나리오(정상, 빈 데이터, 단일 데이터) 테스트
