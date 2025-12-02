# Day 3: 테스트 자동화 및 CI/CD 파이프라인

## 📋 학습 목표
- CI/CD와 테스트 자동화의 개념을 이해한다
- GitHub Actions로 C# 테스트 자동화 파이프라인을 구축한다
- 테스트 코드의 품질 관리 방법을 학습한다
- 종합 프로젝트를 통해 실무 적용 역량을 강화한다

---

## 1. CI/CD 개요

### 1.1 CI/CD란?

```
┌─────────────────────────────────────────────────────────────────────┐
│                      CI/CD 파이프라인                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   개발자 → Commit → Build → Test → Deploy → 운영환경               │
│     ↑         │        │       │        │                          │
│     │    ┌────┴────┐   │       │        │                          │
│     │    │   CI    │   │       │        │                          │
│     │    │ (지속적 │   │       │        │                          │
│     │    │  통합)  │◄──┘       │        │                          │
│     │    └─────────┘           │        │                          │
│     │                    ┌─────┴────────┴────┐                     │
│     │                    │        CD         │                     │
│     │                    │ (지속적 배포/전달) │                     │
│     │                    └───────────────────┘                     │
│     │                                                              │
│     └──────────────── 피드백 ◄────────────────────────────         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

| 용어 | 설명 |
|------|------|
| **CI (Continuous Integration)** | 코드 변경사항을 자동으로 빌드하고 테스트 |
| **CD (Continuous Delivery)** | 테스트 통과한 코드를 배포 준비 상태로 유지 |
| **CD (Continuous Deployment)** | 자동으로 프로덕션까지 배포 |

### 1.2 CI/CD의 이점

- 버그 조기 발견
- 빠른 피드백 루프
- 배포 위험 감소
- 개발 생산성 향상
- 일관된 빌드 환경

---

## 2. GitHub Actions 소개

### 2.1 주요 개념

| 개념 | 설명 |
|------|------|
| **Workflow** | 자동화된 프로세스 (YAML 파일로 정의) |
| **Event** | 워크플로우를 트리거하는 이벤트 (push, PR 등) |
| **Job** | 같은 러너에서 실행되는 단계들의 집합 |
| **Step** | Job 내의 개별 작업 단위 |
| **Action** | 재사용 가능한 작업 단위 |
| **Runner** | 워크플로우를 실행하는 서버 |

### 2.2 워크플로우 파일 구조

```yaml
# .github/workflows/ci.yml
name: 워크플로우 이름

on:                          # 트리거 이벤트
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:                        # 실행할 작업들
  job-name:
    runs-on: ubuntu-latest   # 실행 환경
    steps:                   # 작업 단계들
      - name: 단계 이름
        uses: 액션이름@버전
        with:
          파라미터: 값
```

---

## 3. 실습 1: 기본 CI 파이프라인 구축

### 3.1 프로젝트 구조

```
MyProject/
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   └── MyProject.Core/
│       ├── Calculator.cs
│       └── MyProject.Core.csproj
├── tests/
│   └── MyProject.Tests/
│       ├── CalculatorTests.cs
│       └── MyProject.Tests.csproj
├── MyProject.sln
└── README.md
```

### 3.2 기본 CI 워크플로우

**.github/workflows/ci.yml**
```yaml
name: .NET CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  DOTNET_VERSION: '8.0.x'

jobs:
  build-and-test:
    name: Build and Test
    runs-on: ubuntu-latest

    steps:
    # 1. 코드 체크아웃
    - name: Checkout code
      uses: actions/checkout@v4

    # 2. .NET SDK 설치
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    # 3. 의존성 캐시
    - name: Cache NuGet packages
      uses: actions/cache@v4
      with:
        path: ~/.nuget/packages
        key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj') }}
        restore-keys: |
          ${{ runner.os }}-nuget-

    # 4. 의존성 복원
    - name: Restore dependencies
      run: dotnet restore

    # 5. 빌드
    - name: Build
      run: dotnet build --no-restore --configuration Release

    # 6. 테스트 실행
    - name: Test
      run: dotnet test --no-build --configuration Release --verbosity normal
```

### 3.3 워크플로우 실행 확인

1. 코드를 GitHub에 push
2. **Actions** 탭에서 워크플로우 확인
3. 빌드/테스트 결과 확인

---

## 4. 실습 2: 코드 커버리지 통합

### 4.1 커버리지 수집 워크플로우

**.github/workflows/ci-coverage.yml**
```yaml
name: .NET CI with Coverage

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  DOTNET_VERSION: '8.0.x'

jobs:
  build-test-coverage:
    name: Build, Test & Coverage
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore --configuration Release

    # 커버리지와 함께 테스트 실행
    - name: Test with Coverage
      run: |
        dotnet test --no-build --configuration Release \
          --collect:"XPlat Code Coverage" \
          --results-directory ./coverage \
          -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=cobertura

    # 커버리지 리포트 생성
    - name: Generate Coverage Report
      uses: danielpalme/ReportGenerator-GitHub-Action@5.2.0
      with:
        reports: 'coverage/**/coverage.cobertura.xml'
        targetdir: 'coveragereport'
        reporttypes: 'HtmlInline_AzurePipelines;Cobertura;Badges'
        
    # 커버리지 리포트를 아티팩트로 업로드
    - name: Upload Coverage Report
      uses: actions/upload-artifact@v4
      with:
        name: coverage-report
        path: coveragereport

    # PR에 커버리지 코멘트 추가
    - name: Add Coverage PR Comment
      uses: marocchino/sticky-pull-request-comment@v2
      if: github.event_name == 'pull_request'
      with:
        recreate: true
        path: coveragereport/Summary.md

    # 커버리지 배지 업데이트
    - name: Update Coverage Badge
      if: github.ref == 'refs/heads/main'
      uses: schneegans/dynamic-badges-action@v1.7.0
      with:
        auth: ${{ secrets.GIST_TOKEN }}
        gistID: YOUR_GIST_ID
        filename: coverage.json
        label: Coverage
        message: ${{ steps.coverage.outputs.coverage }}%
        valColorRange: ${{ steps.coverage.outputs.coverage }}
        maxColorRange: 100
        minColorRange: 0
```

### 4.2 커버리지 임계값 설정

**테스트 프로젝트 .csproj에 추가:**
```xml
<PropertyGroup>
  <CollectCoverage>true</CollectCoverage>
  <CoverletOutputFormat>cobertura</CoverletOutputFormat>
  <Threshold>80</Threshold>
  <ThresholdType>line,branch,method</ThresholdType>
  <ThresholdStat>total</ThresholdStat>
</PropertyGroup>
```

### 4.3 커버리지 실패시 빌드 실패

```yaml
    - name: Check Coverage Threshold
      run: |
        COVERAGE=$(grep -oP '(?<=line-rate=")[^"]*' coverage/**/coverage.cobertura.xml | head -1)
        COVERAGE_PERCENT=$(echo "$COVERAGE * 100" | bc)
        echo "Coverage: $COVERAGE_PERCENT%"
        if (( $(echo "$COVERAGE_PERCENT < 80" | bc -l) )); then
          echo "Coverage is below 80%!"
          exit 1
        fi
```

---

## 5. 실습 3: 고급 CI/CD 파이프라인

### 5.1 멀티 환경 테스트

**.github/workflows/ci-matrix.yml**
```yaml
name: .NET CI Matrix

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    name: Test on ${{ matrix.os }} with .NET ${{ matrix.dotnet }}
    runs-on: ${{ matrix.os }}
    
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        dotnet: ['6.0.x', '7.0.x', '8.0.x']
      fail-fast: false  # 하나가 실패해도 다른 작업 계속 실행

    steps:
    - uses: actions/checkout@v4

    - name: Setup .NET ${{ matrix.dotnet }}
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ matrix.dotnet }}

    - name: Restore
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore

    - name: Test
      run: dotnet test --no-build --verbosity normal
```

### 5.2 PR 검증 워크플로우

**.github/workflows/pr-check.yml**
```yaml
name: PR Validation

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  validate:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0  # 전체 히스토리 (PR 비교용)

    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'

    - name: Restore
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore --warnaserror

    # 테스트 실행 및 결과 리포트
    - name: Test
      run: |
        dotnet test --no-build \
          --logger "trx;LogFileName=test-results.trx" \
          --results-directory ./TestResults

    # 테스트 결과 PR에 표시
    - name: Publish Test Results
      uses: dorny/test-reporter@v1
      if: always()
      with:
        name: Test Results
        path: TestResults/*.trx
        reporter: dotnet-trx

    # 코드 스타일 검사
    - name: Format Check
      run: dotnet format --verify-no-changes

  # 코드 분석
  analyze:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4

    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'

    # Roslyn 분석기로 코드 품질 검사
    - name: Code Analysis
      run: dotnet build /p:TreatWarningsAsErrors=true
```

### 5.3 배포 파이프라인

**.github/workflows/deploy.yml**
```yaml
name: Deploy to Production

on:
  push:
    tags:
      - 'v*'  # v1.0.0 같은 태그 push시

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'
    - run: dotnet test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'
    - name: Publish
      run: dotnet publish -c Release -o ./publish
    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: app
        path: ./publish

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
    - name: Download Artifact
      uses: actions/download-artifact@v4
      with:
        name: app
    - name: Deploy to Staging
      run: |
        echo "Deploying to staging..."
        # 실제 배포 스크립트

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    steps:
    - name: Download Artifact
      uses: actions/download-artifact@v4
      with:
        name: app
    - name: Deploy to Production
      run: |
        echo "Deploying to production..."
        # 실제 배포 스크립트
```

---

## 6. Jenkins CI/CD (대안)

### 6.1 Jenkinsfile 예시

```groovy
pipeline {
    agent any
    
    environment {
        DOTNET_CLI_HOME = "/tmp/dotnet_cli_home"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Restore') {
            steps {
                sh 'dotnet restore'
            }
        }
        
        stage('Build') {
            steps {
                sh 'dotnet build --no-restore --configuration Release'
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    dotnet test --no-build --configuration Release \
                        --collect:"XPlat Code Coverage" \
                        --logger "trx;LogFileName=TestResults.trx"
                '''
            }
            post {
                always {
                    // 테스트 결과 리포트
                    mstest testResultsFile: '**/TestResults.trx'
                    
                    // 커버리지 리포트
                    publishCoverage adapters: [coberturaAdapter('**/coverage.cobertura.xml')]
                }
            }
        }
        
        stage('Publish') {
            when {
                branch 'main'
            }
            steps {
                sh 'dotnet publish -c Release -o ./publish'
                archiveArtifacts artifacts: 'publish/**', fingerprint: true
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to staging?', ok: 'Deploy'
                echo 'Deploying to staging...'
            }
        }
    }
    
    post {
        failure {
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Check console output: ${env.BUILD_URL}",
                recipientProviders: [culprits()]
            )
        }
    }
}
```

---

## 7. 테스트 품질 관리

### 7.1 테스트 명명 규칙

```csharp
// 좋은 테스트 이름 패턴
[테스트대상]_[시나리오]_[기대결과]

// 예시
Add_TwoPositiveNumbers_ReturnsSum()
Withdraw_InsufficientFunds_ThrowsException()
Login_InvalidCredentials_ReturnsFalse()
```

### 7.2 테스트 코드 리뷰 체크리스트

```markdown
## 테스트 코드 리뷰 체크리스트

### 구조
- [ ] AAA 패턴 (Arrange-Act-Assert) 준수
- [ ] 한 테스트에 하나의 검증만 수행
- [ ] 테스트 간 독립성 보장
- [ ] 적절한 테스트 명명 규칙 사용

### 커버리지
- [ ] 정상 케이스 테스트
- [ ] 예외/에러 케이스 테스트
- [ ] 경계값 테스트
- [ ] null/empty 처리 테스트

### 품질
- [ ] 테스트가 실패하는 이유가 명확한가
- [ ] Mock 객체가 적절히 사용되었는가
- [ ] 불필요한 Setup/Teardown이 없는가
- [ ] 테스트 실행 시간이 적절한가

### 유지보수
- [ ] 중복 코드가 최소화되었는가
- [ ] 매직 넘버 대신 상수 사용
- [ ] 테스트 데이터가 명확한가
```

### 7.3 테스트 안티패턴

```csharp
// ❌ 안티패턴 1: 여러 검증
[Test]
public void Bad_MultipleAssertions()
{
    var calc = new Calculator();
    Assert.That(calc.Add(1, 1), Is.EqualTo(2));
    Assert.That(calc.Subtract(5, 3), Is.EqualTo(2));
    Assert.That(calc.Multiply(2, 3), Is.EqualTo(6));
}

// ✅ 개선: 각각 분리
[Test]
public void Add_TwoNumbers_ReturnsSum()
{
    Assert.That(new Calculator().Add(1, 1), Is.EqualTo(2));
}

// ❌ 안티패턴 2: 테스트 간 의존성
private static int _counter = 0;

[Test]
public void Bad_Test1_IncrementCounter()
{
    _counter++;
    Assert.That(_counter, Is.EqualTo(1));
}

[Test]
public void Bad_Test2_CheckCounter()
{
    Assert.That(_counter, Is.EqualTo(1)); // Test1에 의존
}

// ✅ 개선: 독립적인 테스트
[Test]
public void Good_IndependentTest()
{
    var counter = 0;
    counter++;
    Assert.That(counter, Is.EqualTo(1));
}

// ❌ 안티패턴 3: 불필요한 Mock
[Test]
public void Bad_MockingValueObject()
{
    var mockCalc = new Mock<Calculator>(); // Calculator는 Mock 불필요
}

// ✅ 개선: 실제 객체 사용
[Test]
public void Good_UseRealObject()
{
    var calc = new Calculator();
    Assert.That(calc.Add(1, 1), Is.EqualTo(2));
}
```

---

## 8. 종합 프로젝트: 온라인 주문 시스템

### 8.1 요구사항

```
온라인 주문 시스템
├── 사용자 관리
│   ├── 회원가입/로그인
│   └── 포인트 관리
├── 상품 관리
│   ├── 상품 조회
│   └── 재고 관리
├── 주문 처리
│   ├── 장바구니
│   ├── 주문 생성
│   ├── 결제 처리
│   └── 주문 취소
└── 알림
    ├── 이메일 발송
    └── SMS 발송
```

### 8.2 프로젝트 구조

```
OnlineOrderSystem/
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   ├── OnlineOrder.Domain/           # 도메인 모델
│   ├── OnlineOrder.Application/      # 비즈니스 로직
│   └── OnlineOrder.Infrastructure/   # 외부 연동
├── tests/
│   ├── OnlineOrder.Domain.Tests/
│   ├── OnlineOrder.Application.Tests/
│   └── OnlineOrder.Integration.Tests/
└── OnlineOrderSystem.sln
```

### 8.3 인터페이스 정의

**Domain/Interfaces.cs**
```csharp
namespace OnlineOrder.Domain
{
    public interface IUserRepository
    {
        User GetById(int id);
        User GetByEmail(string email);
        void Save(User user);
    }

    public interface IProductRepository
    {
        Product GetById(int id);
        IEnumerable<Product> GetAll();
        void UpdateStock(int productId, int quantity);
    }

    public interface IOrderRepository
    {
        void Save(Order order);
        Order GetById(int orderId);
        IEnumerable<Order> GetByUserId(int userId);
    }

    public interface IPaymentService
    {
        PaymentResult Process(PaymentRequest request);
        void Refund(string transactionId);
    }

    public interface INotificationService
    {
        Task SendEmailAsync(string to, string subject, string body);
        Task SendSmsAsync(string phoneNumber, string message);
    }
}
```

### 8.4 TDD로 구현할 핵심 테스트

**Application.Tests/OrderServiceTests.cs**
```csharp
using NUnit.Framework;
using Moq;
using OnlineOrder.Application;
using OnlineOrder.Domain;

namespace OnlineOrder.Application.Tests
{
    [TestFixture]
    public class OrderServiceTests
    {
        private Mock<IUserRepository> _mockUserRepo;
        private Mock<IProductRepository> _mockProductRepo;
        private Mock<IOrderRepository> _mockOrderRepo;
        private Mock<IPaymentService> _mockPayment;
        private Mock<INotificationService> _mockNotification;
        private OrderService _orderService;

        [SetUp]
        public void Setup()
        {
            _mockUserRepo = new Mock<IUserRepository>();
            _mockProductRepo = new Mock<IProductRepository>();
            _mockOrderRepo = new Mock<IOrderRepository>();
            _mockPayment = new Mock<IPaymentService>();
            _mockNotification = new Mock<INotificationService>();

            _orderService = new OrderService(
                _mockUserRepo.Object,
                _mockProductRepo.Object,
                _mockOrderRepo.Object,
                _mockPayment.Object,
                _mockNotification.Object);
        }

        #region 주문 생성 테스트

        [Test]
        public async Task CreateOrder_ValidOrder_ReturnsOrderId()
        {
            // Arrange
            var user = new User { Id = 1, Email = "test@test.com", Points = 1000 };
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000, Stock = 10 };
            var orderRequest = new OrderRequest
            {
                UserId = 1,
                Items = new[] { new OrderItemRequest { ProductId = 1, Quantity = 1 } }
            };

            _mockUserRepo.Setup(x => x.GetById(1)).Returns(user);
            _mockProductRepo.Setup(x => x.GetById(1)).Returns(product);
            _mockPayment.Setup(x => x.Process(It.IsAny<PaymentRequest>()))
                .Returns(new PaymentResult { Success = true, TransactionId = "TX123" });

            // Act
            var result = await _orderService.CreateOrderAsync(orderRequest);

            // Assert
            Assert.That(result.Success, Is.True);
            Assert.That(result.OrderId, Is.GreaterThan(0));
            
            _mockOrderRepo.Verify(x => x.Save(It.IsAny<Order>()), Times.Once);
            _mockProductRepo.Verify(x => x.UpdateStock(1, -1), Times.Once);
        }

        [Test]
        public void CreateOrder_InsufficientStock_ThrowsException()
        {
            // Arrange
            var user = new User { Id = 1, Email = "test@test.com" };
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000, Stock = 0 };
            var orderRequest = new OrderRequest
            {
                UserId = 1,
                Items = new[] { new OrderItemRequest { ProductId = 1, Quantity = 1 } }
            };

            _mockUserRepo.Setup(x => x.GetById(1)).Returns(user);
            _mockProductRepo.Setup(x => x.GetById(1)).Returns(product);

            // Act & Assert
            Assert.ThrowsAsync<InsufficientStockException>(
                async () => await _orderService.CreateOrderAsync(orderRequest));
        }

        [Test]
        public async Task CreateOrder_PaymentFailed_RollbackStock()
        {
            // Arrange
            var user = new User { Id = 1, Email = "test@test.com" };
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000, Stock = 10 };
            var orderRequest = new OrderRequest
            {
                UserId = 1,
                Items = new[] { new OrderItemRequest { ProductId = 1, Quantity = 1 } }
            };

            _mockUserRepo.Setup(x => x.GetById(1)).Returns(user);
            _mockProductRepo.Setup(x => x.GetById(1)).Returns(product);
            _mockPayment.Setup(x => x.Process(It.IsAny<PaymentRequest>()))
                .Returns(new PaymentResult { Success = false, ErrorMessage = "결제 실패" });

            // Act
            var result = await _orderService.CreateOrderAsync(orderRequest);

            // Assert
            Assert.That(result.Success, Is.False);
            _mockProductRepo.Verify(x => x.UpdateStock(1, 1), Times.Once); // 재고 롤백
        }

        #endregion

        #region 주문 취소 테스트

        [Test]
        public async Task CancelOrder_ValidOrder_RefundsPayment()
        {
            // Arrange
            var order = new Order
            {
                Id = 1,
                UserId = 1,
                Status = OrderStatus.Paid,
                TransactionId = "TX123",
                Items = new[] { new OrderItem { ProductId = 1, Quantity = 2 } }
            };

            _mockOrderRepo.Setup(x => x.GetById(1)).Returns(order);

            // Act
            await _orderService.CancelOrderAsync(1);

            // Assert
            _mockPayment.Verify(x => x.Refund("TX123"), Times.Once);
            _mockProductRepo.Verify(x => x.UpdateStock(1, 2), Times.Once); // 재고 복구
        }

        [Test]
        public void CancelOrder_ShippedOrder_ThrowsException()
        {
            // Arrange
            var order = new Order
            {
                Id = 1,
                Status = OrderStatus.Shipped
            };

            _mockOrderRepo.Setup(x => x.GetById(1)).Returns(order);

            // Act & Assert
            Assert.ThrowsAsync<InvalidOperationException>(
                async () => await _orderService.CancelOrderAsync(1));
        }

        #endregion

        #region 알림 테스트

        [Test]
        public async Task CreateOrder_Success_SendsConfirmationEmail()
        {
            // Arrange
            var user = new User { Id = 1, Email = "customer@test.com" };
            var product = new Product { Id = 1, Name = "노트북", Price = 1000000, Stock = 10 };
            var orderRequest = new OrderRequest
            {
                UserId = 1,
                Items = new[] { new OrderItemRequest { ProductId = 1, Quantity = 1 } }
            };

            _mockUserRepo.Setup(x => x.GetById(1)).Returns(user);
            _mockProductRepo.Setup(x => x.GetById(1)).Returns(product);
            _mockPayment.Setup(x => x.Process(It.IsAny<PaymentRequest>()))
                .Returns(new PaymentResult { Success = true });

            // Act
            await _orderService.CreateOrderAsync(orderRequest);

            // Assert
            _mockNotification.Verify(
                x => x.SendEmailAsync(
                    "customer@test.com",
                    It.Is<string>(s => s.Contains("주문 확인")),
                    It.IsAny<string>()),
                Times.Once);
        }

        #endregion

        #region 포인트 적립/사용 테스트

        [Test]
        public async Task CreateOrder_WithPoints_AppliesDiscount()
        {
            // Arrange
            var user = new User { Id = 1, Email = "test@test.com", Points = 5000 };
            var product = new Product { Id = 1, Name = "노트북", Price = 100000, Stock = 10 };
            var orderRequest = new OrderRequest
            {
                UserId = 1,
                UsePoints = 5000,
                Items = new[] { new OrderItemRequest { ProductId = 1, Quantity = 1 } }
            };

            _mockUserRepo.Setup(x => x.GetById(1)).Returns(user);
            _mockProductRepo.Setup(x => x.GetById(1)).Returns(product);
            _mockPayment.Setup(x => x.Process(It.IsAny<PaymentRequest>()))
                .Returns(new PaymentResult { Success = true });

            // Act
            await _orderService.CreateOrderAsync(orderRequest);

            // Assert - 95,000원 결제 (100,000 - 5,000 포인트)
            _mockPayment.Verify(
                x => x.Process(It.Is<PaymentRequest>(p => p.Amount == 95000)),
                Times.Once);
        }

        [Test]
        public async Task CreateOrder_Success_EarnsPoints()
        {
            // Arrange
            var user = new User { Id = 1, Email = "test@test.com", Points = 0 };
            var product = new Product { Id = 1, Name = "노트북", Price = 100000, Stock = 10 };
            var orderRequest = new OrderRequest
            {
                UserId = 1,
                Items = new[] { new OrderItemRequest { ProductId = 1, Quantity = 1 } }
            };

            _mockUserRepo.Setup(x => x.GetById(1)).Returns(user);
            _mockProductRepo.Setup(x => x.GetById(1)).Returns(product);
            _mockPayment.Setup(x => x.Process(It.IsAny<PaymentRequest>()))
                .Returns(new PaymentResult { Success = true });

            // Act
            await _orderService.CreateOrderAsync(orderRequest);

            // Assert - 1% 포인트 적립 (1,000 포인트)
            Assert.That(user.Points, Is.EqualTo(1000));
            _mockUserRepo.Verify(x => x.Save(It.Is<User>(u => u.Points == 1000)), Times.Once);
        }

        #endregion
    }
}
```

### 8.5 CI 워크플로우

**.github/workflows/online-order-ci.yml**
```yaml
name: Online Order System CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'

    - name: Restore
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore --configuration Release

    - name: Unit Tests
      run: |
        dotnet test tests/OnlineOrder.Domain.Tests \
          --no-build --configuration Release \
          --collect:"XPlat Code Coverage" \
          --logger "trx"

    - name: Application Tests
      run: |
        dotnet test tests/OnlineOrder.Application.Tests \
          --no-build --configuration Release \
          --collect:"XPlat Code Coverage" \
          --logger "trx"

    - name: Generate Coverage Report
      uses: danielpalme/ReportGenerator-GitHub-Action@5.2.0
      with:
        reports: '**/coverage.cobertura.xml'
        targetdir: 'coveragereport'
        reporttypes: 'HtmlInline;Cobertura'

    - name: Upload Coverage
      uses: actions/upload-artifact@v4
      with:
        name: coverage-report
        path: coveragereport

    - name: Check Coverage Threshold
      run: |
        COVERAGE=$(grep -oP 'lineCoverage="[^"]*' coveragereport/Summary.xml | head -1 | grep -oP '[0-9.]+')
        echo "Coverage: $COVERAGE%"
        if (( $(echo "$COVERAGE < 80" | bc -l) )); then
          echo "::error::Coverage $COVERAGE% is below threshold 80%"
          exit 1
        fi
```

---

## 9. 팀별 발표 및 코드 리뷰

### 9.1 발표 항목

1. **프로젝트 개요** (2분)
   - 구현한 기능 소개
   - 아키텍처 설명

2. **TDD 적용 사례** (5분)
   - Red-Green-Refactor 과정 시연
   - 어려웠던 점과 해결 방법

3. **CI/CD 파이프라인** (3분)
   - GitHub Actions 구성 설명
   - 커버리지 결과 공유

4. **코드 리뷰** (5분)
   - 다른 팀의 코드 리뷰
   - 개선 포인트 제안

### 9.2 평가 기준

| 항목 | 배점 |
|------|------|
| 테스트 커버리지 (80% 이상) | 30점 |
| TDD 프로세스 준수 | 25점 |
| CI/CD 파이프라인 구축 | 20점 |
| 코드 품질 | 15점 |
| 발표 및 문서화 | 10점 |

---

## 10. 핵심 정리

### CI/CD 체크리스트

```markdown
## CI/CD 구축 체크리스트

### 기본 설정
- [ ] 워크플로우 파일 생성 (.github/workflows/*.yml)
- [ ] 트리거 이벤트 설정 (push, PR)
- [ ] 빌드 환경 설정 (.NET 버전)

### 테스트
- [ ] 단위 테스트 실행
- [ ] 커버리지 수집
- [ ] 테스트 결과 리포트

### 품질 관리
- [ ] 코드 스타일 검사 (dotnet format)
- [ ] 정적 분석
- [ ] 커버리지 임계값 설정

### 배포
- [ ] 아티팩트 생성
- [ ] 스테이징 환경 배포
- [ ] 프로덕션 배포 (승인 절차)
```

### 모범 사례

1. **테스트 우선**: 테스트 없이 배포하지 않기
2. **빠른 피드백**: 파이프라인 실행 시간 최적화
3. **실패시 알림**: Slack/Email 알림 설정
4. **작은 변경**: 자주, 작게 배포
5. **롤백 준비**: 언제든 롤백 가능한 상태 유지

---

## 📚 추가 학습 자료

### 공식 문서
- [GitHub Actions 문서](https://docs.github.com/en/actions)
- [.NET CLI 문서](https://docs.microsoft.com/en-us/dotnet/core/tools/)
- [NUnit 문서](https://docs.nunit.org/)

### 추천 도서
- "Clean Code" - Robert C. Martin
- "Test-Driven Development" - Kent Beck
- "Continuous Delivery" - Jez Humble

### 온라인 코스
- [Microsoft Learn: .NET 테스트](https://learn.microsoft.com/en-us/dotnet/core/testing/)
- [Pluralsight: TDD 실전](https://www.pluralsight.com/paths/test-driven-development-in-c-sharp)

---

## 🎉 교육 수료

축하합니다! 3일간의 단위테스트 및 TDD 교육을 완료하셨습니다.

**학습한 내용:**
- ✅ 단위테스트 기본 원리와 FIRST 원칙
- ✅ NUnit/xUnit 프레임워크 활용
- ✅ Mock, Stub, Fake를 활용한 테스트 격리
- ✅ TDD Red-Green-Refactor 사이클
- ✅ 코드 커버리지 분석
- ✅ 예외 처리 및 경계값 테스트
- ✅ GitHub Actions CI/CD 파이프라인
- ✅ 테스트 품질 관리

**다음 단계:**
1. 실무 프로젝트에 TDD 적용
2. 팀 내 테스트 문화 확산
3. 지속적인 커버리지 개선
