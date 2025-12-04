# 🎯 C# 단위 테스트 종합 실습 과제

## 디스플레이 제조 공장 품질 관리 시스템

> **목표**: 디스플레이 패널 생산 라인의 품질 관리 시스템을 TDD 방식으로 개발하며 단위 테스트의 모든 핵심 개념 학습  
> **환경**: Visual Studio 2022, .NET 8.0, NUnit, Moq  
> **예상 소요 시간**: 약 120분 (2시간)  
> **난이도**: ★★★☆☆ (중급)

---

## 📋 목차

| 파트 | 내용 | 예상 시간 | 핵심 개념 |
|------|------|----------|----------|
| Part 1 | 프로젝트 설정 | 10분 | 솔루션 구성, NuGet 패키지 |
| Part 2 | TDD로 도메인 모델 개발 | 20분 | Red-Green-Refactor, AAA 패턴 |
| Part 3 | 검사 서비스 테스트 | 25분 | Mock 객체, 의존성 주입 |
| Part 4 | 예외 처리 테스트 | 15분 | Assert.Throws, 경계값 테스트 |
| Part 5 | 파라미터화 테스트 | 15분 | TestCase, TestCaseSource |
| Part 6 | 고급 Moq 기능 | 20분 | Verify, Callback, Sequence |
| Part 7 | 코드 커버리지 | 10분 | Fine Code Coverage |
| Part 8 | 리팩토링 & 마무리 | 5분 | 테스트 품질 검토 |

---

## 🎯 실습 시나리오

```
┌────────────────────────────────────────────────────────────────┐
│  🖥️ DisplayQC - 디스플레이 패널 품질 관리 시스템               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  [생산 라인 흐름]                                              │
│                                                                │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    │
│  │ Glass   │ → │ TFT     │ → │ Cell    │ → │ Module  │    │
│  │ 공정    │    │ 공정    │    │ 공정    │    │ 공정    │    │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘    │
│       ↓              ↓              ↓              ↓          │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    │
│  │ AOI     │    │ Array   │    │ Mura    │    │ Final   │    │
│  │ 검사    │    │ 검사    │    │ 검사    │    │ 검사    │    │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘    │
│                                                                │
├────────────────────────────────────────────────────────────────┤
│  기능 요구사항:                                                │
│                                                                │
│  1. 패널 관리                                                  │
│     • 패널 등록 (Panel ID, 모델, 크기, 해상도)                │
│     • 공정 이력 추적                                          │
│     • 불량 이력 관리                                          │
│                                                                │
│  2. 검사 처리                                                  │
│     • AOI (자동 광학 검사) 결과 처리                          │
│     • Mura (얼룩) 검사 결과 처리                              │
│     • 불량 유형별 분류 (Dead Pixel, Scratch, Mura 등)         │
│                                                                │
│  3. 품질 판정                                                  │
│     • 등급 판정 (A+, A, B, C, NG)                             │
│     • 불량률 계산                                             │
│     • Yield(수율) 계산                                        │
│                                                                │
│  4. 알람 서비스                                                │
│     • 연속 불량 발생 알람                                     │
│     • 수율 저하 알람                                          │
│     • 설비 이상 알람                                          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 🔧 도메인 용어 설명

```
┌────────────────────────────────────────────────────────────────┐
│  디스플레이 제조 용어                                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  • Panel (패널): 디스플레이 제품의 기본 단위                   │
│  • Lot: 동일 조건으로 생산된 패널 그룹 (보통 20~50장)         │
│  • AOI: Automated Optical Inspection (자동 광학 검사)         │
│  • Mura: 디스플레이의 얼룩/불균일 현상                        │
│  • Dead Pixel: 동작하지 않는 화소                             │
│  • Bright Spot: 항상 켜져 있는 불량 화소                      │
│  • Scratch: 표면 스크래치                                     │
│  • Crack: 유리 기판 균열                                      │
│  • Yield (수율): 양품 비율 (양품수 / 총생산수 × 100)          │
│  • Tact Time: 단위 제품 생산 소요 시간                        │
│  • Spec: 품질 기준 규격                                       │
│  • Grade: 품질 등급 (A+, A, B, C, NG)                         │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

# Part 1: 프로젝트 설정 (10분)

---

## 과제 1.1: 솔루션 생성

### 요구사항

```
┌────────────────────────────────────────────────────────────────┐
│  솔루션 구조 생성                                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  솔루션: DisplayQC                                             │
│  │                                                             │
│  ├── DisplayQC.Domain        [클래스 라이브러리]               │
│  │   └── 도메인 모델 (Panel, Defect, Inspection 등)           │
│  │                                                             │
│  ├── DisplayQC.Services      [클래스 라이브러리]               │
│  │   └── 비즈니스 로직 서비스                                 │
│  │                                                             │
│  └── DisplayQC.Tests         [NUnit 테스트 프로젝트]           │
│      └── 모든 테스트 코드                                     │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 작업 단계

```
1. Visual Studio 2022 실행
2. "새 프로젝트 만들기" → "클래스 라이브러리" → 이름: DisplayQC.Domain
3. 솔루션 우클릭 → 추가 → 새 프로젝트 → "클래스 라이브러리" → 이름: DisplayQC.Services
4. 솔루션 우클릭 → 추가 → 새 프로젝트 → "NUnit 테스트 프로젝트" → 이름: DisplayQC.Tests
5. 프로젝트 참조 설정:
   - DisplayQC.Services → DisplayQC.Domain 참조 추가
   - DisplayQC.Tests → DisplayQC.Domain, DisplayQC.Services 참조 추가
```

---

## 과제 1.2: NuGet 패키지 설치

### DisplayQC.Tests 프로젝트에 추가 설치

```
┌────────────────────────────────────────────────────────────────┐
│  필수 패키지 (테스트 프로젝트)                                 │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  이미 설치됨 (NUnit 템플릿 기본):                              │
│  ✅ NUnit                                                      │
│  ✅ NUnit3TestAdapter                                          │
│  ✅ Microsoft.NET.Test.Sdk                                     │
│  ✅ coverlet.collector                                         │
│                                                                │
│  추가 설치 필요:                                               │
│  📦 Moq (Mock 객체 생성용)                                    │
│                                                                │
│  설치 명령:                                                    │
│  도구 → NuGet 패키지 관리자 → 솔루션용 NuGet 패키지 관리      │
│  → "Moq" 검색 → DisplayQC.Tests에 설치                        │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### ✅ 체크포인트 1

```
□ 솔루션에 3개 프로젝트 생성됨
□ 프로젝트 참조 설정 완료
□ Moq 패키지 설치됨
□ 솔루션 빌드 성공
```

---

# Part 2: TDD로 도메인 모델 개발 (20분)

---

## 과제 2.1: Panel 클래스 - TDD 방식

### 🔴 Red: 먼저 실패하는 테스트 작성

**DisplayQC.Tests/PanelTests.cs** 생성:

```csharp
// TODO: 아래 테스트를 작성하세요
// 파일: DisplayQC.Tests/PanelTests.cs

namespace DisplayQC.Tests;

[TestFixture]
public class PanelTests
{
    #region 과제 2.1.1: 생성자 테스트

    [Test]
    public void Constructor_ValidParameters_CreatesPanel()
    {
        // TODO: Panel 객체 생성 테스트
        // PanelId: "P240101-001"
        // Model: "OLED-55-UHD"
        // Size: 55.0 (인치)
        // Resolution: "3840x2160"
        // LotId: "LOT240101-01"
        
        // Assert: 모든 속성이 올바르게 설정되었는지 확인
        // Assert: 초기 Grade는 Pending, Status는 InProcess
    }

    [Test]
    public void Constructor_EmptyPanelId_ThrowsArgumentException()
    {
        // TODO: PanelId가 빈 문자열일 때 예외 발생 테스트
    }

    [Test]
    public void Constructor_InvalidSize_ThrowsArgumentException()
    {
        // TODO: Size가 0 이하일 때 예외 발생 테스트
    }

    #endregion

    #region 과제 2.1.2: 불량 등록 테스트

    [Test]
    public void AddDefect_ValidDefect_AddsToDefectList()
    {
        // TODO: 불량 추가 테스트
        // Dead Pixel 불량 1개 추가 → DefectCount = 1
    }

    [Test]
    public void AddDefect_MultipleDefects_AccumulatesCount()
    {
        // TODO: 복수 불량 추가 테스트
        // Dead Pixel 2개 + Scratch 1개 → DefectCount = 3
    }

    [Test]
    public void GetDefectsByType_SpecificType_ReturnsFilteredList()
    {
        // TODO: 불량 유형별 필터링 테스트
        // Dead Pixel 2개 + Mura 1개 등록
        // GetDefectsByType(DeadPixel) → 2개 반환
    }

    #endregion

    #region 과제 2.1.3: 등급 판정 테스트

    [Test]
    public void SetGrade_ValidGrade_UpdatesGradeAndStatus()
    {
        // TODO: 등급 설정 테스트
        // Grade.A 설정 → Grade = A, Status = Completed
    }

    [Test]
    public void SetGrade_NGGrade_SetsStatusToRejected()
    {
        // TODO: NG 등급 설정 테스트
        // Grade.NG 설정 → Status = Rejected
    }

    [Test]
    public void SetGrade_AlreadyCompleted_ThrowsInvalidOperationException()
    {
        // TODO: 이미 완료된 패널 등급 재설정 시 예외 테스트
    }

    #endregion
}
```

### 🟢 Green: 테스트를 통과하는 최소 코드 작성

**DisplayQC.Domain/Panel.cs** 생성:

```csharp
// TODO: 테스트를 통과하도록 Panel 클래스를 구현하세요
// 파일: DisplayQC.Domain/Panel.cs

namespace DisplayQC.Domain;

/// <summary>
/// 디스플레이 패널 엔티티
/// </summary>
public class Panel
{
    // TODO: 속성 정의
    // - PanelId (string, 읽기 전용) - 패널 고유 ID
    // - Model (string, 읽기 전용) - 모델명 (예: OLED-55-UHD)
    // - Size (double, 읽기 전용) - 화면 크기 (인치)
    // - Resolution (string, 읽기 전용) - 해상도 (예: 3840x2160)
    // - LotId (string, 읽기 전용) - Lot 번호
    // - Grade (PanelGrade) - 품질 등급
    // - Status (PanelStatus) - 처리 상태
    // - Defects (List<Defect>) - 불량 목록
    // - DefectCount (int) - 총 불량 수
    // - CreatedAt (DateTime) - 생성 시간
    // - CompletedAt (DateTime?) - 완료 시간

    // TODO: 생성자 구현 (유효성 검증 포함)

    // TODO: AddDefect(Defect defect) 메서드 구현

    // TODO: GetDefectsByType(DefectType type) 메서드 구현

    // TODO: SetGrade(PanelGrade grade) 메서드 구현
}

/// <summary>
/// 패널 품질 등급
/// </summary>
public enum PanelGrade
{
    Pending,    // 판정 대기
    APlus,      // A+ 등급 (최상)
    A,          // A 등급
    B,          // B 등급
    C,          // C 등급
    NG          // 불량 (Not Good)
}

/// <summary>
/// 패널 처리 상태
/// </summary>
public enum PanelStatus
{
    InProcess,  // 공정 진행 중
    Completed,  // 완료 (양품)
    Rejected,   // 불량 판정
    OnHold      // 보류
}
```

### 💡 힌트

<details>
<summary>Panel 클래스 구현 힌트 (클릭하여 펼치기)</summary>

```csharp
namespace DisplayQC.Domain;

public class Panel
{
    public string PanelId { get; }
    public string Model { get; }
    public double Size { get; }
    public string Resolution { get; }
    public string LotId { get; }
    public PanelGrade Grade { get; private set; }
    public PanelStatus Status { get; private set; }
    public List<Defect> Defects { get; } = [];
    public int DefectCount => Defects.Count;
    public DateTime CreatedAt { get; }
    public DateTime? CompletedAt { get; private set; }

    public Panel(string panelId, string model, double size, string resolution, string lotId)
    {
        if (string.IsNullOrWhiteSpace(panelId))
            throw new ArgumentException("Panel ID는 필수입니다.", nameof(panelId));
        
        if (string.IsNullOrWhiteSpace(model))
            throw new ArgumentException("모델명은 필수입니다.", nameof(model));
        
        if (size <= 0)
            throw new ArgumentException("크기는 0보다 커야 합니다.", nameof(size));

        PanelId = panelId;
        Model = model;
        Size = size;
        Resolution = resolution;
        LotId = lotId;
        Grade = PanelGrade.Pending;
        Status = PanelStatus.InProcess;
        CreatedAt = DateTime.Now;
    }

    public void AddDefect(Defect defect)
    {
        ArgumentNullException.ThrowIfNull(defect);
        Defects.Add(defect);
    }

    public IEnumerable<Defect> GetDefectsByType(DefectType type)
    {
        return Defects.Where(d => d.Type == type);
    }

    public void SetGrade(PanelGrade grade)
    {
        if (Status == PanelStatus.Completed || Status == PanelStatus.Rejected)
            throw new InvalidOperationException("이미 판정이 완료된 패널입니다.");

        Grade = grade;
        Status = grade == PanelGrade.NG ? PanelStatus.Rejected : PanelStatus.Completed;
        CompletedAt = DateTime.Now;
    }
}
```

</details>

---

## 과제 2.2: Defect 클래스

### 요구사항

```
┌────────────────────────────────────────────────────────────────┐
│  Defect 클래스 요구사항                                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  속성:                                                         │
│  • DefectId (string) - 불량 ID                                │
│  • Type (DefectType enum) - 불량 유형                         │
│  • Severity (DefectSeverity enum) - 심각도                    │
│  • PositionX (int) - X 좌표 (픽셀)                            │
│  • PositionY (int) - Y 좌표 (픽셀)                            │
│  • Size (double) - 불량 크기 (mm)                             │
│  • DetectedAt (DateTime) - 검출 시간                          │
│  • DetectedBy (string) - 검출 설비명                          │
│                                                                │
│  불량 유형 (DefectType):                                       │
│  • DeadPixel - 죽은 화소                                      │
│  • BrightSpot - 밝은 점                                       │
│  • Scratch - 스크래치                                         │
│  • Mura - 얼룩                                                │
│  • Crack - 균열                                               │
│  • Particle - 이물                                            │
│  • ColorDefect - 색상 불량                                    │
│                                                                │
│  심각도 (DefectSeverity):                                      │
│  • Minor - 경미 (B등급 이상 가능)                             │
│  • Major - 주요 (C등급 이상 가능)                             │
│  • Critical - 심각 (NG 판정)                                  │
│                                                                │
│  메서드:                                                       │
│  • IsWithinSpec(spec) - 규격 내 여부 확인                     │
│  • GetGradeImpact() - 등급에 미치는 영향 반환                 │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 테스트 파일 작성

**DisplayQC.Tests/DefectTests.cs** 생성:

```csharp
// TODO: Defect 클래스 테스트 작성
// 파일: DisplayQC.Tests/DefectTests.cs

namespace DisplayQC.Tests;

[TestFixture]
public class DefectTests
{
    #region 과제 2.2.1: 생성자 테스트

    [Test]
    public void Constructor_ValidParameters_CreatesDefect()
    {
        // TODO: 구현
    }

    [Test]
    public void Constructor_NegativePosition_ThrowsArgumentException()
    {
        // TODO: 음수 좌표 예외 테스트
    }

    #endregion

    #region 과제 2.2.2: 규격 판정 테스트

    [Test]
    public void IsWithinSpec_SizeBelowLimit_ReturnsTrue()
    {
        // TODO: 불량 크기가 규격 이내일 때 테스트
        // Dead Pixel 크기 0.1mm, 규격 0.3mm → true
    }

    [Test]
    public void IsWithinSpec_SizeAboveLimit_ReturnsFalse()
    {
        // TODO: 불량 크기가 규격 초과일 때 테스트
        // Scratch 크기 5mm, 규격 3mm → false
    }

    #endregion

    #region 과제 2.2.3: 등급 영향도 테스트

    [Test]
    public void GetGradeImpact_CriticalSeverity_ReturnsNG()
    {
        // TODO: Critical 심각도 → NG 판정
    }

    [Test]
    public void GetGradeImpact_MinorSeverity_ReturnsB()
    {
        // TODO: Minor 심각도 → 최소 B등급 가능
    }

    [Test]
    public void GetGradeImpact_MajorSeverity_ReturnsC()
    {
        // TODO: Major 심각도 → 최소 C등급 가능
    }

    #endregion
}
```

### 도메인 모델 작성

**DisplayQC.Domain/Defect.cs** 생성:

```csharp
// TODO: DefectType, DefectSeverity enum과 Defect 클래스를 구현하세요
```

### ✅ 체크포인트 2

```
□ PanelTests 모든 테스트 통과 (9개)
□ DefectTests 모든 테스트 통과 (6개)
□ Panel, Defect, 관련 enum 클래스 구현 완료
```

---

# Part 3: 검사 서비스 테스트 (25분)

---

## 과제 3.1: 인터페이스 정의

### 저장소(Repository) 및 외부 서비스 인터페이스

**DisplayQC.Domain/Interfaces/** 폴더에 생성:

```csharp
// 파일: DisplayQC.Domain/Interfaces/IPanelRepository.cs
namespace DisplayQC.Domain.Interfaces;

public interface IPanelRepository
{
    Panel? GetById(string panelId);
    IEnumerable<Panel> GetByLotId(string lotId);
    IEnumerable<Panel> GetByModel(string model);
    IEnumerable<Panel> GetByStatus(PanelStatus status);
    IEnumerable<Panel> GetByGrade(PanelGrade grade);
    void Add(Panel panel);
    void Update(Panel panel);
    bool Exists(string panelId);
    int GetTotalCount();
    int GetDefectCount();
}
```

```csharp
// 파일: DisplayQC.Domain/Interfaces/IInspectionEquipment.cs
namespace DisplayQC.Domain.Interfaces;

/// <summary>
/// 검사 설비 인터페이스 (AOI, Mura 검사기 등)
/// </summary>
public interface IInspectionEquipment
{
    string EquipmentId { get; }
    string EquipmentName { get; }
    bool IsOnline { get; }
    
    InspectionResult Inspect(Panel panel);
    bool SelfDiagnosis();
    void Calibrate();
}

/// <summary>
/// 검사 결과
/// </summary>
public class InspectionResult
{
    public string PanelId { get; set; } = string.Empty;
    public string EquipmentId { get; set; } = string.Empty;
    public bool IsPass { get; set; }
    public List<Defect> DetectedDefects { get; set; } = [];
    public DateTime InspectedAt { get; set; }
    public double InspectionTime { get; set; }  // 검사 소요 시간 (초)
}
```

```csharp
// 파일: DisplayQC.Domain/Interfaces/IAlarmService.cs
namespace DisplayQC.Domain.Interfaces;

public interface IAlarmService
{
    void SendDefectAlarm(string panelId, DefectType defectType, DefectSeverity severity);
    void SendYieldAlarm(string lotId, double currentYield, double targetYield);
    void SendEquipmentAlarm(string equipmentId, string message);
    void SendConsecutiveNGAlarm(string lotId, int ngCount);
}
```

```csharp
// 파일: DisplayQC.Domain/Interfaces/IMESInterface.cs
namespace DisplayQC.Domain.Interfaces;

/// <summary>
/// MES (Manufacturing Execution System) 연동 인터페이스
/// </summary>
public interface IMESInterface
{
    void ReportInspectionResult(string panelId, PanelGrade grade, List<Defect> defects);
    void ReportEquipmentStatus(string equipmentId, bool isOnline);
    void UpdatePanelStatus(string panelId, PanelStatus status);
    PanelSpec? GetPanelSpec(string model);
}

/// <summary>
/// 패널 규격 정보
/// </summary>
public class PanelSpec
{
    public string Model { get; set; } = string.Empty;
    public int MaxDeadPixels { get; set; }          // 허용 Dead Pixel 수
    public int MaxBrightSpots { get; set; }         // 허용 Bright Spot 수
    public double MaxMuraLevel { get; set; }        // 허용 Mura 레벨
    public double MaxScratchLength { get; set; }    // 허용 스크래치 길이 (mm)
    public double TargetYield { get; set; }         // 목표 수율 (%)
}
```

---

## 과제 3.2: InspectionService 테스트 (Mock 사용)

### 테스트 파일 작성

**DisplayQC.Tests/InspectionServiceTests.cs** 생성:

```csharp
// 파일: DisplayQC.Tests/InspectionServiceTests.cs
using Moq;
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;
using DisplayQC.Services;

namespace DisplayQC.Tests;

[TestFixture]
public class InspectionServiceTests
{
    private Mock<IPanelRepository> _mockPanelRepository = null!;
    private Mock<IInspectionEquipment> _mockAOI = null!;
    private Mock<IAlarmService> _mockAlarmService = null!;
    private Mock<IMESInterface> _mockMES = null!;
    private InspectionService _inspectionService = null!;

    [SetUp]
    public void SetUp()
    {
        _mockPanelRepository = new Mock<IPanelRepository>();
        _mockAOI = new Mock<IInspectionEquipment>();
        _mockAlarmService = new Mock<IAlarmService>();
        _mockMES = new Mock<IMESInterface>();
        
        _inspectionService = new InspectionService(
            _mockPanelRepository.Object,
            _mockAOI.Object,
            _mockAlarmService.Object,
            _mockMES.Object
        );
    }

    #region 과제 3.2.1: 검사 실행 테스트

    [Test]
    public void ExecuteInspection_ValidPanel_ReturnsInspectionResult()
    {
        // TODO: 정상 패널 검사 테스트
        // Arrange
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT240101-01");
        var inspectionResult = new InspectionResult
        {
            PanelId = panel.PanelId,
            IsPass = true,
            DetectedDefects = []
        };
        
        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.Inspect(panel)).Returns(inspectionResult);

        // Act
        // TODO: _inspectionService.ExecuteInspection(panel.PanelId) 호출

        // Assert
        // TODO: 결과 검증
    }

    [Test]
    public void ExecuteInspection_EquipmentOffline_ThrowsInvalidOperationException()
    {
        // TODO: 설비 오프라인 시 예외 발생 테스트
        _mockAOI.Setup(e => e.IsOnline).Returns(false);
    }

    [Test]
    public void ExecuteInspection_PanelNotFound_ThrowsKeyNotFoundException()
    {
        // TODO: 패널 미존재 시 예외 발생 테스트
    }

    #endregion

    #region 과제 3.2.2: 불량 감지 시 알람 테스트

    [Test]
    public void ExecuteInspection_CriticalDefect_SendsAlarm()
    {
        // TODO: Critical 불량 감지 시 알람 발송 테스트
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT240101-01");
        var criticalDefect = new Defect("D001", DefectType.Crack, DefectSeverity.Critical, 100, 100, 5.0, "AOI-01");
        
        var inspectionResult = new InspectionResult
        {
            PanelId = panel.PanelId,
            IsPass = false,
            DetectedDefects = [criticalDefect]
        };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.Inspect(panel)).Returns(inspectionResult);

        // Act
        // TODO: 검사 실행

        // Assert
        // TODO: _mockAlarmService.Verify로 SendDefectAlarm 호출 확인
    }

    [Test]
    public void ExecuteInspection_MinorDefect_NoAlarm()
    {
        // TODO: Minor 불량은 알람 미발송 테스트
    }

    #endregion

    #region 과제 3.2.3: MES 연동 테스트

    [Test]
    public void ExecuteInspection_Completed_ReportsToMES()
    {
        // TODO: 검사 완료 후 MES 리포트 테스트
        // MES.ReportInspectionResult가 호출되는지 확인
    }

    [Test]
    public void ExecuteInspection_WithDefects_UpdatesPanelInRepository()
    {
        // TODO: 불량 감지 시 패널 정보 업데이트 테스트
        // Repository.Update가 호출되는지 확인
    }

    #endregion
}
```

### 서비스 클래스 구현

**DisplayQC.Services/InspectionService.cs** 생성:

```csharp
// TODO: 테스트를 통과하도록 InspectionService를 구현하세요
// 파일: DisplayQC.Services/InspectionService.cs

using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;

namespace DisplayQC.Services;

public class InspectionService
{
    private readonly IPanelRepository _panelRepository;
    private readonly IInspectionEquipment _inspectionEquipment;
    private readonly IAlarmService _alarmService;
    private readonly IMESInterface _mes;

    public InspectionService(
        IPanelRepository panelRepository,
        IInspectionEquipment inspectionEquipment,
        IAlarmService alarmService,
        IMESInterface mes)
    {
        _panelRepository = panelRepository;
        _inspectionEquipment = inspectionEquipment;
        _alarmService = alarmService;
        _mes = mes;
    }

    // TODO: ExecuteInspection(string panelId) 메서드 구현
    // 1. 설비 온라인 상태 확인
    // 2. 패널 조회
    // 3. 검사 실행
    // 4. 불량 등록
    // 5. Critical 불량 시 알람 발송
    // 6. MES 리포트
    // 7. 패널 정보 업데이트
}
```

---

## 과제 3.3: GradingService 테스트

### 등급 판정 로직

**DisplayQC.Tests/GradingServiceTests.cs**:

```csharp
// 파일: DisplayQC.Tests/GradingServiceTests.cs
using Moq;
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;
using DisplayQC.Services;

namespace DisplayQC.Tests;

[TestFixture]
public class GradingServiceTests
{
    private Mock<IPanelRepository> _mockPanelRepository = null!;
    private Mock<IMESInterface> _mockMES = null!;
    private Mock<IAlarmService> _mockAlarmService = null!;
    private GradingService _gradingService = null!;

    [SetUp]
    public void SetUp()
    {
        _mockPanelRepository = new Mock<IPanelRepository>();
        _mockMES = new Mock<IMESInterface>();
        _mockAlarmService = new Mock<IAlarmService>();
        
        _gradingService = new GradingService(
            _mockPanelRepository.Object,
            _mockMES.Object,
            _mockAlarmService.Object
        );
    }

    #region 과제 3.3.1: 등급 판정 테스트

    [Test]
    public void GradePanel_NoDefects_ReturnsAPlus()
    {
        // TODO: 불량 없는 패널 → A+ 등급
    }

    [Test]
    public void GradePanel_OneMinorDefect_ReturnsA()
    {
        // TODO: Minor 불량 1개 → A 등급
    }

    [Test]
    public void GradePanel_TwoMinorDefects_ReturnsB()
    {
        // TODO: Minor 불량 2개 → B 등급
    }

    [Test]
    public void GradePanel_OneMajorDefect_ReturnsC()
    {
        // TODO: Major 불량 1개 → C 등급
    }

    [Test]
    public void GradePanel_CriticalDefect_ReturnsNG()
    {
        // TODO: Critical 불량 → NG 등급
    }

    [Test]
    public void GradePanel_ExceedsSpec_ReturnsNG()
    {
        // TODO: 규격 초과 불량 → NG 등급
        // Dead Pixel 5개 초과 시 NG
    }

    #endregion

    #region 과제 3.3.2: 연속 NG 알람 테스트

    [Test]
    public void GradePanel_ConsecutiveNG_SendsAlarm()
    {
        // TODO: 동일 Lot에서 연속 3개 NG 발생 시 알람 테스트
    }

    #endregion

    #region 과제 3.3.3: 수율 계산 테스트

    [Test]
    public void CalculateYield_AllPass_Returns100Percent()
    {
        // TODO: 전체 양품 시 수율 100% 테스트
    }

    [Test]
    public void CalculateYield_HalfNG_Returns50Percent()
    {
        // TODO: 절반 불량 시 수율 50% 테스트
    }

    [Test]
    public void CalculateYield_BelowTarget_SendsYieldAlarm()
    {
        // TODO: 목표 수율 미달 시 알람 발송 테스트
        // 목표 95%, 현재 90% → 알람 발송
    }

    #endregion
}
```

### ✅ 체크포인트 3

```
□ InspectionServiceTests 모든 테스트 통과 (7개)
□ GradingServiceTests 모든 테스트 통과 (9개)
□ Mock을 활용한 의존성 주입 이해
□ Verify를 사용한 메서드 호출 검증 이해
```

---

# Part 4: 예외 처리 테스트 (15분)

---

## 과제 4.1: 다양한 예외 상황 테스트

### 테스트 파일 작성

**DisplayQC.Tests/ExceptionTests.cs**:

```csharp
// 파일: DisplayQC.Tests/ExceptionTests.cs
namespace DisplayQC.Tests;

[TestFixture]
public class ExceptionTests
{
    #region 과제 4.1.1: Panel 유효성 검증 예외 테스트

    [Test]
    public void Panel_EmptyModel_ThrowsArgumentExceptionWithMessage()
    {
        // TODO: 예외 메시지 검증 테스트
        var ex = Assert.Throws<ArgumentException>(() => 
            new Panel("P240101-001", "", 55.0, "3840x2160", "LOT001"));
        
        Assert.That(ex.Message, Does.Contain("모델"));
    }

    [Test]
    public void Panel_ZeroSize_ThrowsArgumentExceptionWithParamName()
    {
        // TODO: ParamName 검증 테스트
    }

    #endregion

    #region 과제 4.1.2: 경계값 테스트

    [Test]
    public void Panel_MinimumValidSize_DoesNotThrow()
    {
        // TODO: 최소 유효 크기 (0.1인치) 테스트
        Assert.DoesNotThrow(() => 
            new Panel("P001", "Test", 0.1, "100x100", "LOT001"));
    }

    [Test]
    public void Panel_SizeJustBelowMinimum_ThrowsException()
    {
        // TODO: 최소 크기 미만 (0 이하) 테스트
    }

    [Test]
    public void Defect_PositionAtOrigin_DoesNotThrow()
    {
        // TODO: 좌표 (0, 0)은 유효 - 화면 좌상단
    }

    [Test]
    public void Defect_NegativePosition_ThrowsException()
    {
        // TODO: 음수 좌표는 예외
    }

    #endregion

    #region 과제 4.1.3: 상태 기반 예외 테스트

    [Test]
    public void Panel_AddDefectAfterCompleted_ThrowsException()
    {
        // TODO: 완료된 패널에 불량 추가 시 예외
        var panel = new Panel("P001", "Test", 55.0, "3840x2160", "LOT001");
        panel.SetGrade(PanelGrade.A);  // 완료 상태로 변경
        
        var defect = new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01");
        
        // TODO: Assert.Throws로 예외 확인
    }

    [Test]
    public void Panel_SetGradeTwice_ThrowsException()
    {
        // TODO: 등급 중복 설정 시 예외
    }

    #endregion
}
```

---

## 과제 4.2: 설비 관련 예외 테스트

**DisplayQC.Tests/EquipmentExceptionTests.cs**:

```csharp
// 파일: DisplayQC.Tests/EquipmentExceptionTests.cs
using Moq;
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;
using DisplayQC.Services;

namespace DisplayQC.Tests;

[TestFixture]
public class EquipmentExceptionTests
{
    #region 과제 4.2.1: 설비 오프라인 예외

    [Test]
    public void InspectionService_EquipmentOffline_ThrowsWithEquipmentId()
    {
        // TODO: 오프라인 설비 ID가 예외 메시지에 포함되는지 테스트
    }

    #endregion

    #region 과제 4.2.2: 설비 캘리브레이션 예외

    [Test]
    public void InspectionService_CalibrationRequired_ThrowsException()
    {
        // TODO: 캘리브레이션 필요 시 예외 테스트
    }

    #endregion

    #region 과제 4.2.3: 타임아웃 예외

    [Test]
    public void InspectionService_InspectionTimeout_ThrowsTimeoutException()
    {
        // TODO: 검사 타임아웃 테스트
    }

    #endregion
}
```

### ✅ 체크포인트 4

```
□ ExceptionTests 모든 테스트 통과 (8개)
□ EquipmentExceptionTests 모든 테스트 통과 (3개)
□ Assert.Throws 사용법 이해
□ 경계값 테스트 개념 이해
```

---

# Part 5: 파라미터화 테스트 (15분)

---

## 과제 5.1: TestCase를 활용한 테스트

**DisplayQC.Tests/ParameterizedTests.cs**:

```csharp
// 파일: DisplayQC.Tests/ParameterizedTests.cs
namespace DisplayQC.Tests;

[TestFixture]
public class ParameterizedTests
{
    #region 과제 5.1.1: 등급 판정 기준 테스트

    [TestCase(0, 0, 0, PanelGrade.APlus)]       // 불량 없음 → A+
    [TestCase(1, 0, 0, PanelGrade.A)]           // Minor 1개 → A
    [TestCase(2, 0, 0, PanelGrade.B)]           // Minor 2개 → B
    [TestCase(3, 0, 0, PanelGrade.B)]           // Minor 3개 → B
    [TestCase(0, 1, 0, PanelGrade.C)]           // Major 1개 → C
    [TestCase(1, 1, 0, PanelGrade.C)]           // Minor 1 + Major 1 → C
    [TestCase(0, 0, 1, PanelGrade.NG)]          // Critical 1개 → NG
    [TestCase(5, 0, 0, PanelGrade.NG)]          // Minor 5개 이상 → NG
    public void GradePanel_ByDefectCounts_ReturnsCorrectGrade(
        int minorCount, int majorCount, int criticalCount, PanelGrade expectedGrade)
    {
        // TODO: 구현
        // 불량 수에 따른 등급 판정 테스트
    }

    #endregion

    #region 과제 5.1.2: 수율 계산 테스트

    [TestCase(100, 100, 100.0)]     // 100개 중 100개 양품 = 100%
    [TestCase(100, 95, 95.0)]       // 100개 중 95개 양품 = 95%
    [TestCase(100, 50, 50.0)]       // 100개 중 50개 양품 = 50%
    [TestCase(50, 45, 90.0)]        // 50개 중 45개 양품 = 90%
    [TestCase(1, 1, 100.0)]         // 1개 중 1개 양품 = 100%
    [TestCase(1, 0, 0.0)]           // 1개 중 0개 양품 = 0%
    public void CalculateYield_VariousCounts_ReturnsCorrectPercentage(
        int totalCount, int passCount, double expectedYield)
    {
        // TODO: 구현
    }

    #endregion

    #region 과제 5.1.3: 불량 유형별 심각도 테스트

    [TestCase(DefectType.Crack, DefectSeverity.Critical)]
    [TestCase(DefectType.DeadPixel, DefectSeverity.Minor)]
    [TestCase(DefectType.BrightSpot, DefectSeverity.Minor)]
    [TestCase(DefectType.Scratch, DefectSeverity.Major)]
    [TestCase(DefectType.Mura, DefectSeverity.Major)]
    [TestCase(DefectType.Particle, DefectSeverity.Minor)]
    public void GetDefaultSeverity_ByDefectType_ReturnsCorrectSeverity(
        DefectType defectType, DefectSeverity expectedSeverity)
    {
        // TODO: 구현
        // 불량 유형에 따른 기본 심각도 테스트
    }

    #endregion

    #region 과제 5.1.4: 규격 판정 테스트

    [TestCase(DefectType.DeadPixel, 3, 5, true)]    // Dead Pixel 3개, 허용 5개 → Pass
    [TestCase(DefectType.DeadPixel, 5, 5, true)]    // Dead Pixel 5개, 허용 5개 → Pass (경계)
    [TestCase(DefectType.DeadPixel, 6, 5, false)]   // Dead Pixel 6개, 허용 5개 → Fail
    [TestCase(DefectType.BrightSpot, 2, 3, true)]   // Bright Spot 2개, 허용 3개 → Pass
    [TestCase(DefectType.BrightSpot, 4, 3, false)]  // Bright Spot 4개, 허용 3개 → Fail
    public void CheckSpec_DefectCount_ReturnsCorrectResult(
        DefectType defectType, int actualCount, int maxAllowed, bool expectedResult)
    {
        // TODO: 구현
    }

    #endregion
}
```

---

## 과제 5.2: TestCaseSource를 활용한 복잡한 테스트

**DisplayQC.Tests/TestCaseSourceTests.cs**:

```csharp
// 파일: DisplayQC.Tests/TestCaseSourceTests.cs
namespace DisplayQC.Tests;

[TestFixture]
public class TestCaseSourceTests
{
    #region 과제 5.2.1: 복합 등급 판정 테스트 데이터

    private static IEnumerable<TestCaseData> GradingTestCases()
    {
        // 케이스 1: 무결점 패널
        yield return new TestCaseData(
            new List<Defect>(),
            new PanelSpec { MaxDeadPixels = 5, MaxBrightSpots = 3 },
            PanelGrade.APlus
        ).SetName("무결점_A+등급");

        // 케이스 2: Minor 불량만 있는 패널
        yield return new TestCaseData(
            new List<Defect>
            {
                new("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01"),
                new("D002", DefectType.Particle, DefectSeverity.Minor, 200, 200, 0.05, "AOI-01")
            },
            new PanelSpec { MaxDeadPixels = 5, MaxBrightSpots = 3 },
            PanelGrade.B
        ).SetName("Minor불량2개_B등급");

        // TODO: 케이스 3, 4, 5 추가
        // - Major 불량 케이스
        // - Critical 불량 케이스
        // - 규격 초과 케이스
    }

    [Test]
    [TestCaseSource(nameof(GradingTestCases))]
    public void GradePanel_ComplexCases_ReturnsCorrectGrade(
        List<Defect> defects, PanelSpec spec, PanelGrade expectedGrade)
    {
        // TODO: 구현
    }

    #endregion

    #region 과제 5.2.2: Lot 단위 수율 분석 테스트 데이터

    private static IEnumerable<TestCaseData> LotYieldTestCases()
    {
        // 케이스 1: 전량 양품
        yield return new TestCaseData(
            new List<PanelGrade> 
            { 
                PanelGrade.APlus, PanelGrade.APlus, PanelGrade.A, 
                PanelGrade.A, PanelGrade.B 
            },
            100.0,  // 예상 수율
            false   // 알람 발생 여부
        ).SetName("전량양품_수율100%");

        // 케이스 2: NG 포함
        yield return new TestCaseData(
            new List<PanelGrade> 
            { 
                PanelGrade.A, PanelGrade.A, PanelGrade.B, 
                PanelGrade.NG, PanelGrade.NG 
            },
            60.0,   // 3/5 = 60%
            true    // 목표 95% 미달 → 알람
        ).SetName("NG2개_수율60%_알람발생");

        // TODO: 추가 케이스
    }

    [Test]
    [TestCaseSource(nameof(LotYieldTestCases))]
    public void AnalyzeLotYield_VariousCases_ReturnsCorrectResult(
        List<PanelGrade> grades, double expectedYield, bool shouldAlarm)
    {
        // TODO: 구현
    }

    #endregion

    #region 과제 5.2.3: 불량 위치 분석 테스트

    private static IEnumerable<TestCaseData> DefectPositionTestCases()
    {
        // 화면 영역별 불량 위치 분석
        // 중앙 영역 불량은 더 심각하게 처리

        yield return new TestCaseData(
            1920, 1080,     // 해상도
            960, 540,       // 불량 위치 (정중앙)
            "Center",       // 예상 영역
            true            // 심각도 상향 여부
        ).SetName("정중앙_불량");

        yield return new TestCaseData(
            1920, 1080,
            10, 10,         // 좌상단 코너
            "Corner",
            false
        ).SetName("코너_불량");

        // TODO: 추가 케이스 (Edge 영역 등)
    }

    [Test]
    [TestCaseSource(nameof(DefectPositionTestCases))]
    public void AnalyzeDefectPosition_VariousLocations_ReturnsCorrectArea(
        int resX, int resY, int defectX, int defectY, 
        string expectedArea, bool shouldUpgradeSeverity)
    {
        // TODO: 구현
    }

    #endregion
}
```

### ✅ 체크포인트 5

```
□ ParameterizedTests 모든 테스트 통과 (20개 이상)
□ TestCaseSourceTests 모든 테스트 통과 (10개 이상)
□ TestCase 속성 사용법 이해
□ TestCaseSource 사용법 이해
```

---

# Part 6: 고급 Moq 기능 (20분)

---

## 과제 6.1: Verify 심화

**DisplayQC.Tests/AdvancedMoqTests.cs**:

```csharp
// 파일: DisplayQC.Tests/AdvancedMoqTests.cs
using Moq;
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;
using DisplayQC.Services;

namespace DisplayQC.Tests;

[TestFixture]
public class AdvancedMoqTests
{
    #region 과제 6.1.1: Verify 횟수 검증

    [Test]
    public void ProcessLot_10Panels_CallsInspect10Times()
    {
        // TODO: Lot 처리 시 Inspect가 패널 수만큼 호출되는지 검증
        // Verify(..., Times.Exactly(10))
    }

    [Test]
    public void ProcessPanel_Success_ReportsToMESOnce()
    {
        // TODO: 처리 성공 시 MES 리포트가 정확히 1번 호출되는지 검증
        // Verify(..., Times.Once)
    }

    [Test]
    public void ProcessPanel_ValidationFailed_NeverCallsInspect()
    {
        // TODO: 유효성 검증 실패 시 Inspect가 호출되지 않는지 검증
        // Verify(..., Times.Never)
    }

    #endregion

    #region 과제 6.1.2: 파라미터 검증

    [Test]
    public void SendAlarm_CriticalDefect_CorrectSeverityPassed()
    {
        // TODO: 알람 발송 시 올바른 심각도가 전달되는지 검증
        // It.Is<DefectSeverity>(s => s == DefectSeverity.Critical)
    }

    [Test]
    public void ReportToMES_CorrectPanelId_Passed()
    {
        // TODO: MES 리포트 시 올바른 Panel ID가 전달되는지 검증
    }

    [Test]
    public void SendYieldAlarm_BelowTarget_ContainsLotId()
    {
        // TODO: 수율 알람에 Lot ID가 포함되는지 검증
    }

    #endregion
}
```

---

## 과제 6.2: Callback과 Sequence

**DisplayQC.Tests/CallbackSequenceTests.cs**:

```csharp
// 파일: DisplayQC.Tests/CallbackSequenceTests.cs
using Moq;
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;
using DisplayQC.Services;

namespace DisplayQC.Tests;

[TestFixture]
public class CallbackSequenceTests
{
    #region 과제 6.2.1: Callback으로 검사 이력 추적

    [Test]
    public void ProcessLot_TracksAllInspectedPanels()
    {
        // TODO: Callback을 사용하여 검사된 패널들 추적
        var inspectedPanels = new List<string>();
        var mockEquipment = new Mock<IInspectionEquipment>();
        
        mockEquipment.Setup(e => e.Inspect(It.IsAny<Panel>()))
            .Callback<Panel>(p => inspectedPanels.Add(p.PanelId))
            .Returns((Panel p) => new InspectionResult 
            { 
                PanelId = p.PanelId, 
                IsPass = true 
            });

        // Lot 처리 후 inspectedPanels에 모든 패널 ID가 있는지 확인
    }

    [Test]
    public void ProcessLot_TracksAllAlarms()
    {
        // TODO: 발생한 모든 알람 추적
        var alarms = new List<(string panelId, DefectType type)>();
        
        // Callback으로 알람 정보 수집
    }

    #endregion

    #region 과제 6.2.2: SetupSequence로 연속 검사 시뮬레이션

    [Test]
    public void InspectMultiplePanels_SequentialResults()
    {
        // TODO: 동일 설비로 여러 패널 검사 시 순차적 결과 반환
        var mockEquipment = new Mock<IInspectionEquipment>();
        
        mockEquipment.SetupSequence(e => e.Inspect(It.IsAny<Panel>()))
            .Returns(new InspectionResult { IsPass = true })   // 첫번째 패널: 양품
            .Returns(new InspectionResult { IsPass = true })   // 두번째 패널: 양품
            .Returns(new InspectionResult { IsPass = false })  // 세번째 패널: 불량
            .Returns(new InspectionResult { IsPass = false }); // 네번째 패널: 불량

        // 4개 패널 검사 후 결과 확인
    }

    [Test]
    public void EquipmentStatus_SequentialChanges()
    {
        // TODO: 설비 상태 변화 시뮬레이션
        // Online → Online → Offline (고장) → Online (복구)
        var mockEquipment = new Mock<IInspectionEquipment>();
        
        mockEquipment.SetupSequence(e => e.IsOnline)
            .Returns(true)
            .Returns(true)
            .Returns(false)  // 설비 고장
            .Returns(true);  // 복구
    }

    #endregion

    #region 과제 6.2.3: 재시도 로직 테스트

    [Test]
    public void InspectionWithRetry_FailsThenSucceeds()
    {
        // TODO: 검사 재시도 로직 테스트
        // 첫번째: 타임아웃
        // 두번째: 타임아웃
        // 세번째: 성공
        
        var mockEquipment = new Mock<IInspectionEquipment>();
        
        mockEquipment.SetupSequence(e => e.Inspect(It.IsAny<Panel>()))
            .Throws<TimeoutException>()
            .Throws<TimeoutException>()
            .Returns(new InspectionResult { IsPass = true });

        // 재시도 로직이 3번 시도 후 성공하는지 확인
    }

    [Test]
    public void EquipmentRecovery_AutoCalibration()
    {
        // TODO: 설비 이상 후 자동 캘리브레이션 테스트
        // SelfDiagnosis 실패 → Calibrate 호출 → 재검사
    }

    #endregion
}
```

---

## 과제 6.3: Strict vs Loose Mock

**DisplayQC.Tests/MockBehaviorTests.cs**:

```csharp
// 파일: DisplayQC.Tests/MockBehaviorTests.cs
using Moq;
using DisplayQC.Domain.Interfaces;

namespace DisplayQC.Tests;

[TestFixture]
public class MockBehaviorTests
{
    #region 과제 6.3.1: Strict Mock - 설비 인터페이스

    [Test]
    public void StrictMock_UnexpectedMethodCall_ThrowsException()
    {
        // TODO: Strict Mock에서 Setup되지 않은 메서드 호출 시 예외 확인
        // 설비 인터페이스의 모든 호출을 명시적으로 검증해야 할 때 유용
        var strictMock = new Mock<IInspectionEquipment>(MockBehavior.Strict);
        
        Assert.Throws<MockException>(() => strictMock.Object.Inspect(null!));
    }

    [Test]
    public void StrictMock_AllMethodsSetup_NoException()
    {
        // TODO: 모든 메서드를 Setup한 Strict Mock 테스트
    }

    #endregion

    #region 과제 6.3.2: Loose Mock (기본) - 알람 서비스

    [Test]
    public void LooseMock_UnsetupMethod_NoException()
    {
        // TODO: Loose Mock에서 Setup되지 않은 메서드는 예외 없이 기본값 반환
        var looseMock = new Mock<IAlarmService>();
        
        // Setup 없이 호출해도 예외 발생 안 함
        Assert.DoesNotThrow(() => 
            looseMock.Object.SendDefectAlarm("P001", DefectType.DeadPixel, DefectSeverity.Minor));
    }

    #endregion

    #region 과제 6.3.3: VerifyNoOtherCalls - 불필요한 호출 검증

    [Test]
    public void InspectionService_OnlyCallsExpectedMethods()
    {
        // TODO: 서비스가 예상된 메서드만 호출하는지 확인
        // 불필요한 MES 호출이나 알람 발송이 없는지 검증
        var mockMES = new Mock<IMESInterface>();
        
        // 서비스 호출 후
        
        mockMES.Verify(m => m.ReportInspectionResult(
            It.IsAny<string>(), It.IsAny<PanelGrade>(), It.IsAny<List<Defect>>()), 
            Times.Once);
        mockMES.VerifyNoOtherCalls();  // 다른 메서드 호출 없음 확인
    }

    #endregion
}
```

### ✅ 체크포인트 6

```
□ AdvancedMoqTests 모든 테스트 통과 (6개)
□ CallbackSequenceTests 모든 테스트 통과 (5개)
□ MockBehaviorTests 모든 테스트 통과 (4개)
□ Callback, SetupSequence 사용법 이해
□ Strict vs Loose Mock 차이 이해
```

---

# Part 7: 코드 커버리지 (10분)

---

## 과제 7.1: 커버리지 측정

### Fine Code Coverage로 측정

```
┌────────────────────────────────────────────────────────────────┐
│  커버리지 측정 단계                                            │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. Fine Code Coverage 설치 확인                               │
│     확장 → 확장 관리 → "Fine Code Coverage" 검색              │
│                                                                │
│  2. 테스트 실행                                                │
│     테스트 → 모든 테스트 실행 (Ctrl + R, A)                   │
│                                                                │
│  3. 커버리지 결과 확인                                         │
│     보기 → 다른 창 → Fine Code Coverage                       │
│                                                                │
│  4. 목표 달성 여부 확인                                        │
│     • Panel.cs: 90% 이상                                      │
│     • Defect.cs: 90% 이상                                     │
│     • InspectionService.cs: 85% 이상                          │
│     • GradingService.cs: 85% 이상                             │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 과제 7.2: 미커버 코드 분석

### 커버리지 개선 작업

```csharp
// 파일: DisplayQC.Tests/CoverageImprovementTests.cs

namespace DisplayQC.Tests;

[TestFixture]
public class CoverageImprovementTests
{
    #region 과제 7.2.1: 미커버 경로 테스트 추가

    // TODO: Fine Code Coverage에서 빨간색(미커버)으로 표시된 코드에 대한 테스트 추가

    // 예시: Panel.cs에서 아직 테스트하지 않은 경로
    // - 특정 예외 조건
    // - else 분기
    // - 경계 조건

    #endregion

    #region 과제 7.2.2: 브랜치 커버리지 개선

    // TODO: 조건문의 모든 분기를 테스트하는 코드 추가

    // 예시: 등급 판정 로직의 모든 분기
    // - if (defectCount == 0) → A+
    // - else if (minorOnly && count <= 1) → A
    // - else if (minorOnly && count <= 3) → B
    // - else if (hasMajor && !hasCritical) → C
    // - else → NG

    #endregion
}
```

### ✅ 체크포인트 7

```
□ Fine Code Coverage 설치 및 실행 완료
□ 전체 프로젝트 커버리지 80% 이상 달성
□ 미커버 코드 식별 및 테스트 추가
□ 커버리지 리포트 해석 가능
```

---

# Part 8: 리팩토링 & 마무리 (5분)

---

## 과제 8.1: 테스트 코드 품질 검토

### 체크리스트

```
┌────────────────────────────────────────────────────────────────┐
│  테스트 품질 체크리스트                                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  명명 규칙:                                                    │
│  □ 테스트 이름이 [메서드]_[시나리오]_[예상결과] 형식인가?     │
│  □ 테스트 이름만 보고 무엇을 테스트하는지 알 수 있는가?       │
│  □ 한글 시나리오 설명이 명확한가?                             │
│                                                                │
│  AAA 패턴:                                                     │
│  □ Arrange-Act-Assert가 명확히 구분되어 있는가?               │
│  □ 각 섹션에 주석이 있는가? (// Arrange, // Act, // Assert)   │
│                                                                │
│  테스트 독립성:                                                │
│  □ 각 테스트가 독립적으로 실행 가능한가?                      │
│  □ 테스트 간 상태 공유가 없는가?                              │
│  □ SetUp/TearDown이 적절히 사용되었는가?                      │
│                                                                │
│  Assert 품질:                                                  │
│  □ 의미 있는 Assert가 포함되어 있는가?                        │
│  □ 하나의 테스트에 너무 많은 Assert가 없는가? (3개 이하 권장) │
│                                                                │
│  Mock 사용:                                                    │
│  □ 외부 의존성(설비, MES, 알람)만 Mock하고 있는가?            │
│  □ Verify가 적절히 사용되었는가?                              │
│  □ 과도한 Setup이 없는가?                                     │
│                                                                │
│  도메인 적합성:                                                │
│  □ 실제 공정 시나리오가 반영되었는가?                         │
│  □ 실무에서 발생 가능한 예외 상황이 테스트되었는가?           │
│  □ 규격(Spec) 기반 판정 로직이 정확한가?                      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 과제 8.2: 최종 제출물

### 제출 체크리스트

```
┌────────────────────────────────────────────────────────────────┐
│  최종 제출물 체크리스트                                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  프로젝트 구조:                                                │
│  □ DisplayQC.Domain 프로젝트                                  │
│    □ Panel.cs, PanelGrade.cs, PanelStatus.cs                  │
│    □ Defect.cs, DefectType.cs, DefectSeverity.cs              │
│    □ Interfaces/IPanelRepository.cs                           │
│    □ Interfaces/IInspectionEquipment.cs                       │
│    □ Interfaces/IAlarmService.cs                              │
│    □ Interfaces/IMESInterface.cs                              │
│                                                                │
│  □ DisplayQC.Services 프로젝트                                │
│    □ InspectionService.cs                                     │
│    □ GradingService.cs                                        │
│                                                                │
│  □ DisplayQC.Tests 프로젝트                                   │
│    □ PanelTests.cs                                            │
│    □ DefectTests.cs                                           │
│    □ InspectionServiceTests.cs                                │
│    □ GradingServiceTests.cs                                   │
│    □ ExceptionTests.cs                                        │
│    □ EquipmentExceptionTests.cs                               │
│    □ ParameterizedTests.cs                                    │
│    □ TestCaseSourceTests.cs                                   │
│    □ AdvancedMoqTests.cs                                      │
│    □ CallbackSequenceTests.cs                                 │
│    □ MockBehaviorTests.cs                                     │
│    □ CoverageImprovementTests.cs                              │
│                                                                │
│  테스트 결과:                                                  │
│  □ 모든 테스트 통과 (총 70개 이상)                            │
│  □ 커버리지 80% 이상                                          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

# 📊 평가 기준

```
┌────────────────────────────────────────────────────────────────┐
│  평가 항목 및 배점                                             │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. 프로젝트 구성 (10점)                                       │
│     • 솔루션 구조 올바름 (5점)                                │
│     • 프로젝트 참조 설정 올바름 (5점)                         │
│                                                                │
│  2. 도메인 모델 (20점)                                         │
│     • Panel 클래스 구현 (10점)                                │
│     • Defect 클래스 구현 (10점)                               │
│                                                                │
│  3. TDD 적용 (15점)                                            │
│     • Red-Green-Refactor 사이클 준수 (5점)                    │
│     • AAA 패턴 적용 (5점)                                     │
│     • 테스트 명명 규칙 준수 (5점)                             │
│                                                                │
│  4. Mock 활용 (20점)                                           │
│     • 설비/MES/알람 Mock 사용 (5점)                           │
│     • Verify 활용 (5점)                                       │
│     • Callback/Sequence 활용 (5점)                            │
│     • Strict/Loose Mock 이해 (5점)                            │
│                                                                │
│  5. 예외 처리 테스트 (10점)                                    │
│     • Assert.Throws 활용 (5점)                                │
│     • 경계값/규격 기반 테스트 (5점)                           │
│                                                                │
│  6. 파라미터화 테스트 (10점)                                   │
│     • TestCase 활용 (5점)                                     │
│     • TestCaseSource 활용 (5점)                               │
│                                                                │
│  7. 코드 커버리지 (10점)                                       │
│     • 80% 이상 달성 (5점)                                     │
│     • 미커버 코드 분석 및 개선 (5점)                          │
│                                                                │
│  8. 도메인 적합성 (5점)                                        │
│     • 실제 공정 시나리오 반영 (3점)                           │
│     • 실무 용어 및 규격 적용 (2점)                            │
│                                                                │
│  총점: 100점                                                   │
│  • 90점 이상: 우수                                            │
│  • 80점 이상: 양호                                            │
│  • 70점 이상: 보통                                            │
│  • 70점 미만: 미흡                                            │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

# 🎯 실습 완료 후 학습 내용 정리

```
┌────────────────────────────────────────────────────────────────┐
│  📚 학습 내용 요약                                             │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ✅ TDD (Test-Driven Development)                             │
│     • Red-Green-Refactor 사이클                               │
│     • 테스트 먼저 작성하는 습관                               │
│     • 품질 검사 로직의 명확한 요구사항 정의                   │
│                                                                │
│  ✅ 단위 테스트 기본                                          │
│     • AAA 패턴 (Arrange-Act-Assert)                           │
│     • 테스트 명명 규칙                                        │
│     • SetUp/TearDown                                          │
│                                                                │
│  ✅ NUnit 활용                                                 │
│     • Assert.That 구문                                        │
│     • TestCase, TestCaseSource                                │
│     • Assert.Throws                                           │
│                                                                │
│  ✅ Moq 활용                                                   │
│     • 설비 인터페이스 Mock                                    │
│     • MES 연동 Mock                                           │
│     • 알람 서비스 Mock                                        │
│     • Verify, Callback, SetupSequence                         │
│                                                                │
│  ✅ 제조 도메인 적용                                          │
│     • 패널 검사 프로세스 모델링                               │
│     • 불량 유형 및 심각도 분류                                │
│     • 등급 판정 로직                                          │
│     • 수율 계산 및 알람 발송                                  │
│                                                                │
│  ✅ 실무 적용 포인트                                          │
│     • 설비 연동 시 Mock 활용                                  │
│     • MES 리포트 검증                                         │
│     • 연속 불량 감지 로직 테스트                              │
│     • 규격(Spec) 기반 판정 테스트                             │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

# 💡 실무 적용 가이드

```
┌────────────────────────────────────────────────────────────────┐
│  실제 프로젝트 적용 시 고려사항                                │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. 설비 인터페이스 설계                                       │
│     • 실제 AOI, Mura 검사기 API 래핑                          │
│     • 비동기 통신 고려 (async/await)                          │
│     • 타임아웃 및 재시도 로직                                 │
│                                                                │
│  2. MES 연동                                                   │
│     • 트랜잭션 처리                                           │
│     • 실패 시 롤백 로직                                       │
│     • 로깅 및 감사 추적                                       │
│                                                                │
│  3. 알람 시스템                                                │
│     • 알람 우선순위 관리                                      │
│     • 중복 알람 방지                                          │
│     • 알람 이력 관리                                          │
│                                                                │
│  4. 성능 고려                                                  │
│     • 대량 패널 처리 시 병렬 처리                             │
│     • 메모리 관리 (대용량 이미지 데이터)                      │
│     • 데이터베이스 쿼리 최적화                                │
│                                                                │
│  5. CI/CD 통합                                                 │
│     • 빌드 시 자동 테스트 실행                                │
│     • 커버리지 리포트 생성                                    │
│     • 품질 게이트 설정 (커버리지 80% 미만 시 빌드 실패)       │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

# 🎉 수고하셨습니다!

2시간 동안의 종합 실습을 완료하셨습니다!

이제 여러분은 디스플레이 제조 공정의 품질 관리 시스템에 단위 테스트를 작성하고 유지보수할 수 있는 역량을 갖추셨습니다.

**다음 단계 학습 제안:**
1. 통합 테스트 - 실제 설비 연동 테스트
2. End-to-End 테스트 - 전체 공정 흐름 테스트
3. CI/CD 파이프라인 구축
4. 성능 테스트 및 부하 테스트

---

**Happy Testing! 🧪**
