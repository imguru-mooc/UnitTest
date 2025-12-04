# 🎯 C# 단위 테스트 종합 실습 과제 (완성 버전)

## 디스플레이 제조 공장 품질 관리 시스템

> **목표**: 디스플레이 패널 생산 라인의 품질 관리 시스템을 TDD 방식으로 개발하며 단위 테스트의 모든 핵심 개념 학습  
> **환경**: Visual Studio 2022, .NET 8.0, NUnit, Moq  
> **예상 소요 시간**: 약 120분 (2시간)  
> **난이도**: ★★★☆☆ (중급)  
> **⚠️ 본 문서는 모든 코드가 완성된 버전입니다. 바로 복사하여 실행할 수 있습니다.**

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

## 🔧 도메인 용어 설명

| 용어 | 설명 |
|------|------|
| Panel (패널) | 디스플레이 제품의 기본 단위 |
| Lot | 동일 조건으로 생산된 패널 그룹 (보통 20~50장) |
| AOI | Automated Optical Inspection (자동 광학 검사) |
| Mura | 디스플레이의 얼룩/불균일 현상 |
| Dead Pixel | 동작하지 않는 화소 |
| Bright Spot | 항상 켜져 있는 불량 화소 |
| Yield (수율) | 양품 비율 (양품수 / 총생산수 × 100) |
| Grade | 품질 등급 (A+, A, B, C, NG) |

---

# Part 1: 프로젝트 설정 (10분)

## 솔루션 구조

```
DisplayQC (솔루션)
├── DisplayQC.Domain        [클래스 라이브러리]
│   ├── Enums.cs
│   ├── Panel.cs
│   ├── Defect.cs
│   └── Interfaces/
│       ├── IPanelRepository.cs
│       ├── IInspectionEquipment.cs
│       ├── IAlarmService.cs
│       └── IMESInterface.cs
├── DisplayQC.Services      [클래스 라이브러리]
│   ├── InspectionService.cs
│   └── GradingService.cs
└── DisplayQC.Tests         [NUnit 테스트 프로젝트]
    ├── PanelTests.cs
    ├── DefectTests.cs
    ├── InspectionServiceTests.cs
    ├── GradingServiceTests.cs
    ├── ExceptionTests.cs
    ├── EquipmentExceptionTests.cs
    ├── ParameterizedTests.cs
    ├── TestCaseSourceTests.cs
    ├── AdvancedMoqTests.cs
    ├── CallbackSequenceTests.cs
    ├── MockBehaviorTests.cs
    └── CoverageImprovementTests.cs
```

## 프로젝트 참조 설정

- DisplayQC.Services → DisplayQC.Domain 참조 추가
- DisplayQC.Tests → DisplayQC.Domain, DisplayQC.Services 참조 추가

## NuGet 패키지 (DisplayQC.Tests)

- NUnit (템플릿 기본 포함)
- NUnit3TestAdapter (템플릿 기본 포함)
- Microsoft.NET.Test.Sdk (템플릿 기본 포함)
- **Moq** (추가 설치 필요)

---

# Part 2: 도메인 모델 (완성 코드)

---

## 📁 DisplayQC.Domain/Enums.cs

```csharp
namespace DisplayQC.Domain;

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

/// <summary>
/// 불량 유형
/// </summary>
public enum DefectType
{
    DeadPixel,      // 죽은 화소
    BrightSpot,     // 밝은 점
    Scratch,        // 스크래치
    Mura,           // 얼룩
    Crack,          // 균열
    Particle,       // 이물
    ColorDefect     // 색상 불량
}

/// <summary>
/// 불량 심각도
/// </summary>
public enum DefectSeverity
{
    Minor,      // 경미 (B등급 이상 가능)
    Major,      // 주요 (C등급 이상 가능)
    Critical    // 심각 (NG 판정)
}
```

---

## 📁 DisplayQC.Domain/Panel.cs

```csharp
namespace DisplayQC.Domain;

/// <summary>
/// 디스플레이 패널 엔티티
/// </summary>
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
        Resolution = resolution ?? string.Empty;
        LotId = lotId ?? string.Empty;
        Grade = PanelGrade.Pending;
        Status = PanelStatus.InProcess;
        CreatedAt = DateTime.Now;
    }

    public void AddDefect(Defect defect)
    {
        ArgumentNullException.ThrowIfNull(defect);

        if (Status == PanelStatus.Completed || Status == PanelStatus.Rejected)
            throw new InvalidOperationException("이미 판정이 완료된 패널에는 불량을 추가할 수 없습니다.");

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

    public bool HasCriticalDefect()
    {
        return Defects.Any(d => d.Severity == DefectSeverity.Critical);
    }

    public int GetDefectCountBySeverity(DefectSeverity severity)
    {
        return Defects.Count(d => d.Severity == severity);
    }
}
```

---

## 📁 DisplayQC.Domain/Defect.cs

```csharp
namespace DisplayQC.Domain;

/// <summary>
/// 불량 정보 엔티티
/// </summary>
public class Defect
{
    public string DefectId { get; }
    public DefectType Type { get; }
    public DefectSeverity Severity { get; }
    public int PositionX { get; }
    public int PositionY { get; }
    public double Size { get; }
    public DateTime DetectedAt { get; }
    public string DetectedBy { get; }

    public Defect(string defectId, DefectType type, DefectSeverity severity,
                  int positionX, int positionY, double size, string detectedBy)
    {
        if (string.IsNullOrWhiteSpace(defectId))
            throw new ArgumentException("불량 ID는 필수입니다.", nameof(defectId));

        if (positionX < 0)
            throw new ArgumentException("X 좌표는 0 이상이어야 합니다.", nameof(positionX));

        if (positionY < 0)
            throw new ArgumentException("Y 좌표는 0 이상이어야 합니다.", nameof(positionY));

        if (size < 0)
            throw new ArgumentException("불량 크기는 0 이상이어야 합니다.", nameof(size));

        DefectId = defectId;
        Type = type;
        Severity = severity;
        PositionX = positionX;
        PositionY = positionY;
        Size = size;
        DetectedBy = detectedBy ?? string.Empty;
        DetectedAt = DateTime.Now;
    }

    /// <summary>
    /// 규격 내 여부 확인
    /// </summary>
    public bool IsWithinSpec(double specLimit)
    {
        return Size <= specLimit;
    }

    /// <summary>
    /// 등급에 미치는 영향 반환
    /// </summary>
    public PanelGrade GetGradeImpact()
    {
        return Severity switch
        {
            DefectSeverity.Critical => PanelGrade.NG,
            DefectSeverity.Major => PanelGrade.C,
            DefectSeverity.Minor => PanelGrade.B,
            _ => PanelGrade.B
        };
    }

    /// <summary>
    /// 불량 유형에 따른 기본 심각도 반환
    /// </summary>
    public static DefectSeverity GetDefaultSeverity(DefectType defectType)
    {
        return defectType switch
        {
            DefectType.Crack => DefectSeverity.Critical,
            DefectType.ColorDefect => DefectSeverity.Critical,
            DefectType.Scratch => DefectSeverity.Major,
            DefectType.Mura => DefectSeverity.Major,
            DefectType.DeadPixel => DefectSeverity.Minor,
            DefectType.BrightSpot => DefectSeverity.Minor,
            DefectType.Particle => DefectSeverity.Minor,
            _ => DefectSeverity.Minor
        };
    }
}
```

---

## 📁 DisplayQC.Domain/Interfaces/IPanelRepository.cs

```csharp
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
    int GetDefectiveCount();
    IEnumerable<Panel> GetRecentNGPanels(string lotId, int count);
}
```

---

## 📁 DisplayQC.Domain/Interfaces/IInspectionEquipment.cs

```csharp
namespace DisplayQC.Domain.Interfaces;

/// <summary>
/// 검사 설비 인터페이스 (AOI, Mura 검사기 등)
/// </summary>
public interface IInspectionEquipment
{
    string EquipmentId { get; }
    string EquipmentName { get; }
    bool IsOnline { get; }
    bool NeedsCalibration { get; }

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
    public DateTime InspectedAt { get; set; } = DateTime.Now;
    public double InspectionTime { get; set; }
}
```

---

## 📁 DisplayQC.Domain/Interfaces/IAlarmService.cs

```csharp
namespace DisplayQC.Domain.Interfaces;

public interface IAlarmService
{
    void SendDefectAlarm(string panelId, DefectType defectType, DefectSeverity severity);
    void SendYieldAlarm(string lotId, double currentYield, double targetYield);
    void SendEquipmentAlarm(string equipmentId, string message);
    void SendConsecutiveNGAlarm(string lotId, int ngCount);
}
```

---

## 📁 DisplayQC.Domain/Interfaces/IMESInterface.cs

```csharp
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
    public int MaxDeadPixels { get; set; } = 5;
    public int MaxBrightSpots { get; set; } = 3;
    public double MaxMuraLevel { get; set; } = 2.0;
    public double MaxScratchLength { get; set; } = 3.0;
    public double TargetYield { get; set; } = 95.0;
}
```

---

# Part 3: 서비스 계층 (완성 코드)

---

## 📁 DisplayQC.Services/InspectionService.cs

```csharp
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

    public InspectionResult ExecuteInspection(string panelId)
    {
        // 1. 설비 상태 확인
        if (!_inspectionEquipment.IsOnline)
        {
            throw new InvalidOperationException(
                $"설비 {_inspectionEquipment.EquipmentId}가 오프라인 상태입니다.");
        }

        if (_inspectionEquipment.NeedsCalibration)
        {
            throw new InvalidOperationException(
                $"설비 {_inspectionEquipment.EquipmentId}의 캘리브레이션이 필요합니다.");
        }

        // 2. 패널 조회
        var panel = _panelRepository.GetById(panelId)
            ?? throw new KeyNotFoundException($"패널 {panelId}를 찾을 수 없습니다.");

        // 3. 검사 실행
        var result = _inspectionEquipment.Inspect(panel);

        // 4. 불량 등록
        foreach (var defect in result.DetectedDefects)
        {
            panel.AddDefect(defect);

            // 5. Critical/Major 불량 시 알람 발송
            if (defect.Severity == DefectSeverity.Critical ||
                defect.Severity == DefectSeverity.Major)
            {
                _alarmService.SendDefectAlarm(panelId, defect.Type, defect.Severity);
            }
        }

        // 6. 패널 정보 업데이트
        _panelRepository.Update(panel);

        // 7. MES 상태 업데이트
        _mes.UpdatePanelStatus(panelId, panel.Status);

        return result;
    }
}
```

---

## 📁 DisplayQC.Services/GradingService.cs

```csharp
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;

namespace DisplayQC.Services;

public class GradingService
{
    private readonly IPanelRepository _panelRepository;
    private readonly IMESInterface _mes;
    private readonly IAlarmService _alarmService;

    private const int ConsecutiveNGThreshold = 3;

    public GradingService(
        IPanelRepository panelRepository,
        IMESInterface mes,
        IAlarmService alarmService)
    {
        _panelRepository = panelRepository;
        _mes = mes;
        _alarmService = alarmService;
    }

    public PanelGrade GradePanel(string panelId)
    {
        var panel = _panelRepository.GetById(panelId)
            ?? throw new KeyNotFoundException($"패널 {panelId}를 찾을 수 없습니다.");

        var spec = _mes.GetPanelSpec(panel.Model);
        var grade = DetermineGrade(panel, spec);

        // 등급 설정
        panel.SetGrade(grade);
        _panelRepository.Update(panel);

        // MES 리포트
        _mes.ReportInspectionResult(panelId, grade, panel.Defects.ToList());

        // NG 판정 시 연속 NG 체크
        if (grade == PanelGrade.NG)
        {
            CheckConsecutiveNG(panel.LotId);
        }

        return grade;
    }

    private PanelGrade DetermineGrade(Panel panel, PanelSpec? spec)
    {
        // Critical 불량이 있으면 NG
        if (panel.HasCriticalDefect())
            return PanelGrade.NG;

        // 규격 초과 체크
        if (spec != null)
        {
            var deadPixelCount = panel.GetDefectsByType(DefectType.DeadPixel).Count();
            if (deadPixelCount > spec.MaxDeadPixels)
                return PanelGrade.NG;

            var brightSpotCount = panel.GetDefectsByType(DefectType.BrightSpot).Count();
            if (brightSpotCount > spec.MaxBrightSpots)
                return PanelGrade.NG;
        }

        var minorCount = panel.GetDefectCountBySeverity(DefectSeverity.Minor);
        var majorCount = panel.GetDefectCountBySeverity(DefectSeverity.Major);

        // Major 불량이 있으면 C등급
        if (majorCount > 0)
            return PanelGrade.C;

        // Minor 불량 개수에 따른 등급
        if (minorCount >= 5)
            return PanelGrade.NG;
        if (minorCount >= 2)
            return PanelGrade.B;
        if (minorCount == 1)
            return PanelGrade.A;

        return PanelGrade.APlus;
    }

    private void CheckConsecutiveNG(string lotId)
    {
        var recentNGPanels = _panelRepository.GetRecentNGPanels(lotId, ConsecutiveNGThreshold - 1);
        var ngCount = recentNGPanels.Count() + 1;  // 현재 패널 포함

        if (ngCount >= ConsecutiveNGThreshold)
        {
            _alarmService.SendConsecutiveNGAlarm(lotId, ngCount);
        }
    }

    public double CalculateYield()
    {
        var totalCount = _panelRepository.GetTotalCount();
        if (totalCount == 0) return 0.0;

        var defectiveCount = _panelRepository.GetDefectiveCount();
        var passCount = totalCount - defectiveCount;

        return (double)passCount / totalCount * 100;
    }

    public void CheckYieldAndAlert(string lotId, double targetYield)
    {
        var currentYield = CalculateYield();

        if (currentYield < targetYield)
        {
            _alarmService.SendYieldAlarm(lotId, currentYield, targetYield);
        }
    }
}
```

---

# Part 4: 테스트 코드 (완성 코드)

---

## 📁 DisplayQC.Tests/PanelTests.cs

```csharp
using DisplayQC.Domain;

namespace DisplayQC.Tests;

[TestFixture]
public class PanelTests
{
    #region 생성자 테스트

    [Test]
    public void Constructor_ValidParameters_CreatesPanel()
    {
        // Arrange
        var panelId = "P240101-001";
        var model = "OLED-55-UHD";
        var size = 55.0;
        var resolution = "3840x2160";
        var lotId = "LOT240101-01";

        // Act
        var panel = new Panel(panelId, model, size, resolution, lotId);

        // Assert
        Assert.That(panel.PanelId, Is.EqualTo(panelId));
        Assert.That(panel.Model, Is.EqualTo(model));
        Assert.That(panel.Size, Is.EqualTo(size));
        Assert.That(panel.Resolution, Is.EqualTo(resolution));
        Assert.That(panel.LotId, Is.EqualTo(lotId));
        Assert.That(panel.Grade, Is.EqualTo(PanelGrade.Pending));
        Assert.That(panel.Status, Is.EqualTo(PanelStatus.InProcess));
    }

    [Test]
    public void Constructor_EmptyPanelId_ThrowsArgumentException()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Panel("", "OLED-55-UHD", 55.0, "3840x2160", "LOT001"));

        Assert.That(ex.ParamName, Is.EqualTo("panelId"));
    }

    [Test]
    public void Constructor_NullPanelId_ThrowsArgumentException()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Panel(null!, "OLED-55-UHD", 55.0, "3840x2160", "LOT001"));

        Assert.That(ex.ParamName, Is.EqualTo("panelId"));
    }

    [Test]
    public void Constructor_EmptyModel_ThrowsArgumentException()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Panel("P240101-001", "", 55.0, "3840x2160", "LOT001"));

        Assert.That(ex.ParamName, Is.EqualTo("model"));
        Assert.That(ex.Message, Does.Contain("모델"));
    }

    [Test]
    public void Constructor_ZeroSize_ThrowsArgumentException()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Panel("P240101-001", "OLED-55-UHD", 0, "3840x2160", "LOT001"));

        Assert.That(ex.ParamName, Is.EqualTo("size"));
    }

    [Test]
    public void Constructor_NegativeSize_ThrowsArgumentException()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Panel("P240101-001", "OLED-55-UHD", -10, "3840x2160", "LOT001"));

        Assert.That(ex.ParamName, Is.EqualTo("size"));
    }

    #endregion

    #region 불량 등록 테스트

    [Test]
    public void AddDefect_ValidDefect_AddsToDefectList()
    {
        // Arrange
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        var defect = new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01");

        // Act
        panel.AddDefect(defect);

        // Assert
        Assert.That(panel.DefectCount, Is.EqualTo(1));
        Assert.That(panel.Defects, Contains.Item(defect));
    }

    [Test]
    public void AddDefect_MultipleDefects_AccumulatesCount()
    {
        // Arrange
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        var defect1 = new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01");
        var defect2 = new Defect("D002", DefectType.DeadPixel, DefectSeverity.Minor, 200, 200, 0.1, "AOI-01");
        var defect3 = new Defect("D003", DefectType.Scratch, DefectSeverity.Major, 300, 300, 2.0, "AOI-01");

        // Act
        panel.AddDefect(defect1);
        panel.AddDefect(defect2);
        panel.AddDefect(defect3);

        // Assert
        Assert.That(panel.DefectCount, Is.EqualTo(3));
    }

    [Test]
    public void AddDefect_NullDefect_ThrowsArgumentNullException()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        Assert.Throws<ArgumentNullException>(() => panel.AddDefect(null!));
    }

    [Test]
    public void GetDefectsByType_SpecificType_ReturnsFilteredList()
    {
        // Arrange
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.AddDefect(new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01"));
        panel.AddDefect(new Defect("D002", DefectType.DeadPixel, DefectSeverity.Minor, 200, 200, 0.1, "AOI-01"));
        panel.AddDefect(new Defect("D003", DefectType.Mura, DefectSeverity.Major, 300, 300, 5.0, "AOI-01"));

        // Act
        var deadPixels = panel.GetDefectsByType(DefectType.DeadPixel).ToList();

        // Assert
        Assert.That(deadPixels, Has.Count.EqualTo(2));
        Assert.That(deadPixels.All(d => d.Type == DefectType.DeadPixel), Is.True);
    }

    [Test]
    public void GetDefectsByType_NoMatchingDefects_ReturnsEmptyList()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.AddDefect(new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01"));

        var scratches = panel.GetDefectsByType(DefectType.Scratch).ToList();

        Assert.That(scratches, Is.Empty);
    }

    #endregion

    #region 등급 판정 테스트

    [Test]
    public void SetGrade_ValidGrade_UpdatesGradeAndStatus()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");

        panel.SetGrade(PanelGrade.A);

        Assert.That(panel.Grade, Is.EqualTo(PanelGrade.A));
        Assert.That(panel.Status, Is.EqualTo(PanelStatus.Completed));
        Assert.That(panel.CompletedAt, Is.Not.Null);
    }

    [Test]
    public void SetGrade_APlusGrade_SetsStatusToCompleted()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");

        panel.SetGrade(PanelGrade.APlus);

        Assert.That(panel.Grade, Is.EqualTo(PanelGrade.APlus));
        Assert.That(panel.Status, Is.EqualTo(PanelStatus.Completed));
    }

    [Test]
    public void SetGrade_NGGrade_SetsStatusToRejected()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");

        panel.SetGrade(PanelGrade.NG);

        Assert.That(panel.Grade, Is.EqualTo(PanelGrade.NG));
        Assert.That(panel.Status, Is.EqualTo(PanelStatus.Rejected));
    }

    [Test]
    public void SetGrade_AlreadyCompleted_ThrowsInvalidOperationException()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.SetGrade(PanelGrade.A);

        var ex = Assert.Throws<InvalidOperationException>(() => panel.SetGrade(PanelGrade.B));
        Assert.That(ex.Message, Does.Contain("이미 판정이 완료"));
    }

    [Test]
    public void SetGrade_AlreadyRejected_ThrowsInvalidOperationException()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.SetGrade(PanelGrade.NG);

        Assert.Throws<InvalidOperationException>(() => panel.SetGrade(PanelGrade.A));
    }

    #endregion

    #region 완료된 패널 불량 추가 테스트

    [Test]
    public void AddDefect_AfterCompleted_ThrowsInvalidOperationException()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.SetGrade(PanelGrade.A);
        var defect = new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01");

        Assert.Throws<InvalidOperationException>(() => panel.AddDefect(defect));
    }

    #endregion
}
```

---

## 📁 DisplayQC.Tests/DefectTests.cs

```csharp
using DisplayQC.Domain;

namespace DisplayQC.Tests;

[TestFixture]
public class DefectTests
{
    #region 생성자 테스트

    [Test]
    public void Constructor_ValidParameters_CreatesDefect()
    {
        var defectId = "D001";
        var type = DefectType.DeadPixel;
        var severity = DefectSeverity.Minor;
        var posX = 100;
        var posY = 200;
        var size = 0.1;
        var detectedBy = "AOI-01";

        var defect = new Defect(defectId, type, severity, posX, posY, size, detectedBy);

        Assert.That(defect.DefectId, Is.EqualTo(defectId));
        Assert.That(defect.Type, Is.EqualTo(type));
        Assert.That(defect.Severity, Is.EqualTo(severity));
        Assert.That(defect.PositionX, Is.EqualTo(posX));
        Assert.That(defect.PositionY, Is.EqualTo(posY));
        Assert.That(defect.Size, Is.EqualTo(size));
        Assert.That(defect.DetectedBy, Is.EqualTo(detectedBy));
    }

    [Test]
    public void Constructor_NegativePositionX_ThrowsArgumentException()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, -1, 100, 0.1, "AOI-01"));

        Assert.That(ex.ParamName, Is.EqualTo("positionX"));
    }

    [Test]
    public void Constructor_NegativePositionY_ThrowsArgumentException()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, -1, 0.1, "AOI-01"));

        Assert.That(ex.ParamName, Is.EqualTo("positionY"));
    }

    [Test]
    public void Constructor_ZeroPosition_IsValid()
    {
        var defect = new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 0, 0, 0.1, "AOI-01");

        Assert.That(defect.PositionX, Is.EqualTo(0));
        Assert.That(defect.PositionY, Is.EqualTo(0));
    }

    [Test]
    public void Constructor_NegativeSize_ThrowsArgumentException()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, -0.1, "AOI-01"));

        Assert.That(ex.ParamName, Is.EqualTo("size"));
    }

    #endregion

    #region 규격 판정 테스트

    [Test]
    public void IsWithinSpec_SizeBelowLimit_ReturnsTrue()
    {
        var defect = new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01");
        Assert.That(defect.IsWithinSpec(0.3), Is.True);
    }

    [Test]
    public void IsWithinSpec_SizeAtLimit_ReturnsTrue()
    {
        var defect = new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.3, "AOI-01");
        Assert.That(defect.IsWithinSpec(0.3), Is.True);
    }

    [Test]
    public void IsWithinSpec_SizeAboveLimit_ReturnsFalse()
    {
        var defect = new Defect("D001", DefectType.Scratch, DefectSeverity.Major, 100, 100, 5.0, "AOI-01");
        Assert.That(defect.IsWithinSpec(3.0), Is.False);
    }

    #endregion

    #region 등급 영향도 테스트

    [Test]
    public void GetGradeImpact_CriticalSeverity_ReturnsNG()
    {
        var defect = new Defect("D001", DefectType.Crack, DefectSeverity.Critical, 100, 100, 10.0, "AOI-01");
        Assert.That(defect.GetGradeImpact(), Is.EqualTo(PanelGrade.NG));
    }

    [Test]
    public void GetGradeImpact_MajorSeverity_ReturnsC()
    {
        var defect = new Defect("D001", DefectType.Scratch, DefectSeverity.Major, 100, 100, 2.0, "AOI-01");
        Assert.That(defect.GetGradeImpact(), Is.EqualTo(PanelGrade.C));
    }

    [Test]
    public void GetGradeImpact_MinorSeverity_ReturnsB()
    {
        var defect = new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01");
        Assert.That(defect.GetGradeImpact(), Is.EqualTo(PanelGrade.B));
    }

    #endregion

    #region 기본 심각도 테스트

    [Test]
    public void GetDefaultSeverity_Crack_ReturnsCritical()
    {
        Assert.That(Defect.GetDefaultSeverity(DefectType.Crack), Is.EqualTo(DefectSeverity.Critical));
    }

    [Test]
    public void GetDefaultSeverity_DeadPixel_ReturnsMinor()
    {
        Assert.That(Defect.GetDefaultSeverity(DefectType.DeadPixel), Is.EqualTo(DefectSeverity.Minor));
    }

    [Test]
    public void GetDefaultSeverity_Mura_ReturnsMajor()
    {
        Assert.That(Defect.GetDefaultSeverity(DefectType.Mura), Is.EqualTo(DefectSeverity.Major));
    }

    #endregion
}
```

---

## 📁 DisplayQC.Tests/InspectionServiceTests.cs

```csharp
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

        _mockAOI.Setup(e => e.EquipmentId).Returns("AOI-01");
        _mockAOI.Setup(e => e.EquipmentName).Returns("AOI 검사기 1호기");

        _inspectionService = new InspectionService(
            _mockPanelRepository.Object,
            _mockAOI.Object,
            _mockAlarmService.Object,
            _mockMES.Object
        );
    }

    #region 검사 실행 테스트

    [Test]
    public void ExecuteInspection_ValidPanel_ReturnsInspectionResult()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT240101-01");
        var inspectionResult = new InspectionResult
        {
            PanelId = panel.PanelId,
            EquipmentId = "AOI-01",
            IsPass = true,
            DetectedDefects = []
        };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.NeedsCalibration).Returns(false);
        _mockAOI.Setup(e => e.Inspect(panel)).Returns(inspectionResult);

        var result = _inspectionService.ExecuteInspection(panel.PanelId);

        Assert.That(result, Is.Not.Null);
        Assert.That(result.PanelId, Is.EqualTo(panel.PanelId));
        Assert.That(result.IsPass, Is.True);
    }

    [Test]
    public void ExecuteInspection_WithDefects_AddsDefectsToPanel()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT240101-01");
        var defects = new List<Defect>
        {
            new("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01"),
            new("D002", DefectType.Scratch, DefectSeverity.Major, 200, 200, 2.0, "AOI-01")
        };

        var inspectionResult = new InspectionResult
        {
            PanelId = panel.PanelId,
            EquipmentId = "AOI-01",
            IsPass = false,
            DetectedDefects = defects
        };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.NeedsCalibration).Returns(false);
        _mockAOI.Setup(e => e.Inspect(panel)).Returns(inspectionResult);

        _inspectionService.ExecuteInspection(panel.PanelId);

        Assert.That(panel.DefectCount, Is.EqualTo(2));
    }

    [Test]
    public void ExecuteInspection_EquipmentOffline_ThrowsInvalidOperationException()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT240101-01");

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(false);

        var ex = Assert.Throws<InvalidOperationException>(() =>
            _inspectionService.ExecuteInspection(panel.PanelId));

        Assert.That(ex.Message, Does.Contain("오프라인"));
    }

    [Test]
    public void ExecuteInspection_PanelNotFound_ThrowsKeyNotFoundException()
    {
        _mockPanelRepository.Setup(r => r.GetById("INVALID")).Returns((Panel?)null);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);

        Assert.Throws<KeyNotFoundException>(() =>
            _inspectionService.ExecuteInspection("INVALID"));
    }

    [Test]
    public void ExecuteInspection_NeedsCalibration_ThrowsInvalidOperationException()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT240101-01");

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.NeedsCalibration).Returns(true);

        var ex = Assert.Throws<InvalidOperationException>(() =>
            _inspectionService.ExecuteInspection(panel.PanelId));

        Assert.That(ex.Message, Does.Contain("캘리브레이션"));
    }

    #endregion

    #region 알람 발송 테스트

    [Test]
    public void ExecuteInspection_CriticalDefect_SendsAlarm()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT240101-01");
        var criticalDefect = new Defect("D001", DefectType.Crack, DefectSeverity.Critical, 100, 100, 5.0, "AOI-01");

        var inspectionResult = new InspectionResult
        {
            PanelId = panel.PanelId,
            EquipmentId = "AOI-01",
            IsPass = false,
            DetectedDefects = [criticalDefect]
        };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.NeedsCalibration).Returns(false);
        _mockAOI.Setup(e => e.Inspect(panel)).Returns(inspectionResult);

        _inspectionService.ExecuteInspection(panel.PanelId);

        _mockAlarmService.Verify(
            a => a.SendDefectAlarm(panel.PanelId, DefectType.Crack, DefectSeverity.Critical),
            Times.Once);
    }

    [Test]
    public void ExecuteInspection_MinorDefect_NoAlarmSent()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT240101-01");
        var minorDefect = new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01");

        var inspectionResult = new InspectionResult
        {
            PanelId = panel.PanelId,
            EquipmentId = "AOI-01",
            IsPass = true,
            DetectedDefects = [minorDefect]
        };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.NeedsCalibration).Returns(false);
        _mockAOI.Setup(e => e.Inspect(panel)).Returns(inspectionResult);

        _inspectionService.ExecuteInspection(panel.PanelId);

        _mockAlarmService.Verify(
            a => a.SendDefectAlarm(It.IsAny<string>(), It.IsAny<DefectType>(), DefectSeverity.Minor),
            Times.Never);
    }

    #endregion

    #region MES 연동 테스트

    [Test]
    public void ExecuteInspection_Completed_UpdatesPanelInRepository()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT240101-01");
        var inspectionResult = new InspectionResult
        {
            PanelId = panel.PanelId,
            EquipmentId = "AOI-01",
            IsPass = true,
            DetectedDefects = []
        };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.NeedsCalibration).Returns(false);
        _mockAOI.Setup(e => e.Inspect(panel)).Returns(inspectionResult);

        _inspectionService.ExecuteInspection(panel.PanelId);

        _mockPanelRepository.Verify(r => r.Update(panel), Times.Once);
    }

    #endregion
}
```

---

## 📁 DisplayQC.Tests/GradingServiceTests.cs

```csharp
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

    #region 등급 판정 테스트

    [Test]
    public void GradePanel_NoDefects_ReturnsAPlus()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        var spec = new PanelSpec { Model = "OLED-55-UHD", MaxDeadPixels = 5 };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockMES.Setup(m => m.GetPanelSpec(panel.Model)).Returns(spec);
        _mockPanelRepository.Setup(r => r.GetRecentNGPanels(panel.LotId, 3)).Returns([]);

        var grade = _gradingService.GradePanel(panel.PanelId);

        Assert.That(grade, Is.EqualTo(PanelGrade.APlus));
    }

    [Test]
    public void GradePanel_OneMinorDefect_ReturnsA()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.AddDefect(new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01"));

        var spec = new PanelSpec { Model = "OLED-55-UHD", MaxDeadPixels = 5 };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockMES.Setup(m => m.GetPanelSpec(panel.Model)).Returns(spec);
        _mockPanelRepository.Setup(r => r.GetRecentNGPanels(panel.LotId, 3)).Returns([]);

        var grade = _gradingService.GradePanel(panel.PanelId);

        Assert.That(grade, Is.EqualTo(PanelGrade.A));
    }

    [Test]
    public void GradePanel_TwoMinorDefects_ReturnsB()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.AddDefect(new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01"));
        panel.AddDefect(new Defect("D002", DefectType.Particle, DefectSeverity.Minor, 200, 200, 0.05, "AOI-01"));

        var spec = new PanelSpec { Model = "OLED-55-UHD", MaxDeadPixels = 5 };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockMES.Setup(m => m.GetPanelSpec(panel.Model)).Returns(spec);
        _mockPanelRepository.Setup(r => r.GetRecentNGPanels(panel.LotId, 3)).Returns([]);

        var grade = _gradingService.GradePanel(panel.PanelId);

        Assert.That(grade, Is.EqualTo(PanelGrade.B));
    }

    [Test]
    public void GradePanel_OneMajorDefect_ReturnsC()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.AddDefect(new Defect("D001", DefectType.Scratch, DefectSeverity.Major, 100, 100, 2.0, "AOI-01"));

        var spec = new PanelSpec { Model = "OLED-55-UHD", MaxDeadPixels = 5 };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockMES.Setup(m => m.GetPanelSpec(panel.Model)).Returns(spec);
        _mockPanelRepository.Setup(r => r.GetRecentNGPanels(panel.LotId, 3)).Returns([]);

        var grade = _gradingService.GradePanel(panel.PanelId);

        Assert.That(grade, Is.EqualTo(PanelGrade.C));
    }

    [Test]
    public void GradePanel_CriticalDefect_ReturnsNG()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.AddDefect(new Defect("D001", DefectType.Crack, DefectSeverity.Critical, 100, 100, 5.0, "AOI-01"));

        var spec = new PanelSpec { Model = "OLED-55-UHD", MaxDeadPixels = 5 };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockMES.Setup(m => m.GetPanelSpec(panel.Model)).Returns(spec);
        _mockPanelRepository.Setup(r => r.GetRecentNGPanels(panel.LotId, 3)).Returns([]);

        var grade = _gradingService.GradePanel(panel.PanelId);

        Assert.That(grade, Is.EqualTo(PanelGrade.NG));
    }

    #endregion

    #region 연속 NG 알람 테스트

    [Test]
    public void GradePanel_ThirdConsecutiveNG_SendsAlarm()
    {
        var panel = new Panel("P240101-003", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.AddDefect(new Defect("D001", DefectType.Crack, DefectSeverity.Critical, 100, 100, 5.0, "AOI-01"));

        var spec = new PanelSpec { Model = "OLED-55-UHD", MaxDeadPixels = 5 };

        var previousNGPanels = new List<Panel>
        {
            new("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001"),
            new("P240101-002", "OLED-55-UHD", 55.0, "3840x2160", "LOT001")
        };

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockMES.Setup(m => m.GetPanelSpec(panel.Model)).Returns(spec);
        _mockPanelRepository.Setup(r => r.GetRecentNGPanels(panel.LotId, 3)).Returns(previousNGPanels);

        _gradingService.GradePanel(panel.PanelId);

        _mockAlarmService.Verify(
            a => a.SendConsecutiveNGAlarm(panel.LotId, 3),
            Times.Once);
    }

    #endregion

    #region 수율 계산 테스트

    [Test]
    public void CalculateYield_AllPass_Returns100Percent()
    {
        _mockPanelRepository.Setup(r => r.GetTotalCount()).Returns(100);
        _mockPanelRepository.Setup(r => r.GetDefectiveCount()).Returns(0);

        var yield = _gradingService.CalculateYield();

        Assert.That(yield, Is.EqualTo(100.0));
    }

    [Test]
    public void CalculateYield_HalfNG_Returns50Percent()
    {
        _mockPanelRepository.Setup(r => r.GetTotalCount()).Returns(100);
        _mockPanelRepository.Setup(r => r.GetDefectiveCount()).Returns(50);

        var yield = _gradingService.CalculateYield();

        Assert.That(yield, Is.EqualTo(50.0));
    }

    [Test]
    public void CheckYieldAndAlert_BelowTarget_SendsAlarm()
    {
        var lotId = "LOT001";
        var targetYield = 95.0;

        _mockPanelRepository.Setup(r => r.GetTotalCount()).Returns(100);
        _mockPanelRepository.Setup(r => r.GetDefectiveCount()).Returns(10);

        _gradingService.CheckYieldAndAlert(lotId, targetYield);

        _mockAlarmService.Verify(
            a => a.SendYieldAlarm(lotId, 90.0, targetYield),
            Times.Once);
    }

    #endregion
}
```

---

## 📁 DisplayQC.Tests/ExceptionTests.cs

```csharp
using DisplayQC.Domain;

namespace DisplayQC.Tests;

[TestFixture]
public class ExceptionTests
{
    #region Panel 유효성 검증 예외 테스트

    [Test]
    public void Panel_EmptyModel_ThrowsArgumentExceptionWithMessage()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Panel("P240101-001", "", 55.0, "3840x2160", "LOT001"));

        Assert.That(ex.Message, Does.Contain("모델"));
        Assert.That(ex.ParamName, Is.EqualTo("model"));
    }

    [Test]
    public void Panel_ZeroSize_ThrowsArgumentExceptionWithParamName()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Panel("P240101-001", "OLED-55-UHD", 0, "3840x2160", "LOT001"));

        Assert.That(ex.ParamName, Is.EqualTo("size"));
    }

    [Test]
    public void Panel_WhitespacePanelId_ThrowsArgumentException()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Panel("   ", "OLED-55-UHD", 55.0, "3840x2160", "LOT001"));

        Assert.That(ex.ParamName, Is.EqualTo("panelId"));
    }

    #endregion

    #region 경계값 테스트

    [Test]
    public void Panel_MinimumValidSize_DoesNotThrow()
    {
        Assert.DoesNotThrow(() =>
            new Panel("P001", "Test", 0.1, "100x100", "LOT001"));
    }

    [Test]
    public void Panel_SizeJustBelowMinimum_ThrowsException()
    {
        Assert.Throws<ArgumentException>(() =>
            new Panel("P001", "Test", 0, "100x100", "LOT001"));
    }

    [Test]
    public void Defect_PositionAtOrigin_DoesNotThrow()
    {
        Assert.DoesNotThrow(() =>
            new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 0, 0, 0.1, "AOI-01"));
    }

    [Test]
    public void Defect_NegativePositionX_ThrowsException()
    {
        Assert.Throws<ArgumentException>(() =>
            new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, -1, 100, 0.1, "AOI-01"));
    }

    [Test]
    public void Defect_NegativePositionY_ThrowsException()
    {
        Assert.Throws<ArgumentException>(() =>
            new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, -1, 0.1, "AOI-01"));
    }

    #endregion

    #region 상태 기반 예외 테스트

    [Test]
    public void Panel_AddDefectAfterCompleted_ThrowsInvalidOperationException()
    {
        var panel = new Panel("P001", "Test", 55.0, "3840x2160", "LOT001");
        panel.SetGrade(PanelGrade.A);

        var defect = new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01");

        var ex = Assert.Throws<InvalidOperationException>(() => panel.AddDefect(defect));
        Assert.That(ex.Message, Does.Contain("판정이 완료"));
    }

    [Test]
    public void Panel_SetGradeTwice_ThrowsInvalidOperationException()
    {
        var panel = new Panel("P001", "Test", 55.0, "3840x2160", "LOT001");
        panel.SetGrade(PanelGrade.A);

        var ex = Assert.Throws<InvalidOperationException>(() => panel.SetGrade(PanelGrade.B));
        Assert.That(ex.Message, Does.Contain("이미 판정이 완료"));
    }

    #endregion
}
```

---

## 📁 DisplayQC.Tests/EquipmentExceptionTests.cs

```csharp
using Moq;
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;
using DisplayQC.Services;

namespace DisplayQC.Tests;

[TestFixture]
public class EquipmentExceptionTests
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

        _mockAOI.Setup(e => e.EquipmentId).Returns("AOI-01");

        _inspectionService = new InspectionService(
            _mockPanelRepository.Object,
            _mockAOI.Object,
            _mockAlarmService.Object,
            _mockMES.Object
        );
    }

    [Test]
    public void InspectionService_EquipmentOffline_ThrowsWithEquipmentId()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(false);

        var ex = Assert.Throws<InvalidOperationException>(() =>
            _inspectionService.ExecuteInspection(panel.PanelId));

        Assert.That(ex.Message, Does.Contain("AOI-01"));
        Assert.That(ex.Message, Does.Contain("오프라인"));
    }

    [Test]
    public void InspectionService_CalibrationRequired_ThrowsException()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.NeedsCalibration).Returns(true);

        var ex = Assert.Throws<InvalidOperationException>(() =>
            _inspectionService.ExecuteInspection(panel.PanelId));

        Assert.That(ex.Message, Does.Contain("캘리브레이션"));
    }

    [Test]
    public void InspectionService_InspectionTimeout_ThrowsTimeoutException()
    {
        var panel = new Panel("P240101-001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");

        _mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.NeedsCalibration).Returns(false);
        _mockAOI.Setup(e => e.Inspect(It.IsAny<Panel>()))
            .Throws(new TimeoutException("검사 시간 초과"));

        var ex = Assert.Throws<TimeoutException>(() =>
            _inspectionService.ExecuteInspection(panel.PanelId));

        Assert.That(ex.Message, Does.Contain("시간 초과"));
    }

    [Test]
    public void InspectionService_PanelNotFound_ThrowsKeyNotFoundException()
    {
        _mockPanelRepository.Setup(r => r.GetById("INVALID")).Returns((Panel?)null);
        _mockAOI.Setup(e => e.IsOnline).Returns(true);
        _mockAOI.Setup(e => e.NeedsCalibration).Returns(false);

        var ex = Assert.Throws<KeyNotFoundException>(() =>
            _inspectionService.ExecuteInspection("INVALID"));

        Assert.That(ex.Message, Does.Contain("INVALID"));
    }
}
```

---

## 📁 DisplayQC.Tests/ParameterizedTests.cs

```csharp
using DisplayQC.Domain;

namespace DisplayQC.Tests;

[TestFixture]
public class ParameterizedTests
{
    #region 등급 판정 기준 테스트

    [TestCase(0, 0, 0, PanelGrade.APlus)]
    [TestCase(1, 0, 0, PanelGrade.A)]
    [TestCase(2, 0, 0, PanelGrade.B)]
    [TestCase(3, 0, 0, PanelGrade.B)]
    [TestCase(4, 0, 0, PanelGrade.B)]
    [TestCase(5, 0, 0, PanelGrade.NG)]
    [TestCase(0, 1, 0, PanelGrade.C)]
    [TestCase(1, 1, 0, PanelGrade.C)]
    [TestCase(0, 0, 1, PanelGrade.NG)]
    public void GradePanel_ByDefectCounts_ReturnsCorrectGrade(
        int minorCount, int majorCount, int criticalCount, PanelGrade expectedGrade)
    {
        var panel = new Panel("P001", "Test", 55.0, "3840x2160", "LOT001");

        for (int i = 0; i < minorCount; i++)
            panel.AddDefect(new Defect($"DM{i}", DefectType.DeadPixel, DefectSeverity.Minor, i * 10, i * 10, 0.1, "AOI"));

        for (int i = 0; i < majorCount; i++)
            panel.AddDefect(new Defect($"DJ{i}", DefectType.Scratch, DefectSeverity.Major, i * 20, i * 20, 2.0, "AOI"));

        for (int i = 0; i < criticalCount; i++)
            panel.AddDefect(new Defect($"DC{i}", DefectType.Crack, DefectSeverity.Critical, i * 30, i * 30, 5.0, "AOI"));

        var grade = DetermineGrade(panel);

        Assert.That(grade, Is.EqualTo(expectedGrade));
    }

    private PanelGrade DetermineGrade(Panel panel)
    {
        if (panel.HasCriticalDefect()) return PanelGrade.NG;

        var minorCount = panel.GetDefectCountBySeverity(DefectSeverity.Minor);
        var majorCount = panel.GetDefectCountBySeverity(DefectSeverity.Major);

        if (majorCount > 0) return PanelGrade.C;
        if (minorCount >= 5) return PanelGrade.NG;
        if (minorCount >= 2) return PanelGrade.B;
        if (minorCount == 1) return PanelGrade.A;

        return PanelGrade.APlus;
    }

    #endregion

    #region 수율 계산 테스트

    [TestCase(100, 100, 100.0)]
    [TestCase(100, 95, 95.0)]
    [TestCase(100, 50, 50.0)]
    [TestCase(50, 45, 90.0)]
    [TestCase(1, 1, 100.0)]
    [TestCase(1, 0, 0.0)]
    [TestCase(0, 0, 0.0)]
    public void CalculateYield_VariousCounts_ReturnsCorrectPercentage(
        int totalCount, int passCount, double expectedYield)
    {
        var yield = totalCount == 0 ? 0.0 : (double)passCount / totalCount * 100;
        Assert.That(yield, Is.EqualTo(expectedYield).Within(0.01));
    }

    #endregion

    #region 불량 유형별 심각도 테스트

    [TestCase(DefectType.Crack, DefectSeverity.Critical)]
    [TestCase(DefectType.ColorDefect, DefectSeverity.Critical)]
    [TestCase(DefectType.DeadPixel, DefectSeverity.Minor)]
    [TestCase(DefectType.BrightSpot, DefectSeverity.Minor)]
    [TestCase(DefectType.Scratch, DefectSeverity.Major)]
    [TestCase(DefectType.Mura, DefectSeverity.Major)]
    [TestCase(DefectType.Particle, DefectSeverity.Minor)]
    public void GetDefaultSeverity_ByDefectType_ReturnsCorrectSeverity(
        DefectType defectType, DefectSeverity expectedSeverity)
    {
        var severity = Defect.GetDefaultSeverity(defectType);
        Assert.That(severity, Is.EqualTo(expectedSeverity));
    }

    #endregion

    #region 규격 판정 테스트

    [TestCase(DefectType.DeadPixel, 3, 5, true)]
    [TestCase(DefectType.DeadPixel, 5, 5, true)]
    [TestCase(DefectType.DeadPixel, 6, 5, false)]
    [TestCase(DefectType.BrightSpot, 2, 3, true)]
    [TestCase(DefectType.BrightSpot, 4, 3, false)]
    public void CheckSpec_DefectCount_ReturnsCorrectResult(
        DefectType defectType, int actualCount, int maxAllowed, bool expectedResult)
    {
        var panel = new Panel("P001", "Test", 55.0, "3840x2160", "LOT001");

        for (int i = 0; i < actualCount; i++)
            panel.AddDefect(new Defect($"D{i}", defectType, DefectSeverity.Minor, i * 10, i * 10, 0.1, "AOI"));

        var count = panel.GetDefectsByType(defectType).Count();
        var isWithinSpec = count <= maxAllowed;

        Assert.That(isWithinSpec, Is.EqualTo(expectedResult));
    }

    #endregion

    #region 크기별 규격 판정 테스트

    [TestCase(0.1, 0.3, true)]
    [TestCase(0.3, 0.3, true)]
    [TestCase(0.31, 0.3, false)]
    [TestCase(2.0, 3.0, true)]
    [TestCase(5.0, 3.0, false)]
    public void IsWithinSpec_BySize_ReturnsCorrectResult(
        double defectSize, double specLimit, bool expectedResult)
    {
        var defect = new Defect("D001", DefectType.Scratch, DefectSeverity.Minor, 100, 100, defectSize, "AOI");
        Assert.That(defect.IsWithinSpec(specLimit), Is.EqualTo(expectedResult));
    }

    #endregion
}
```

---

## 📁 DisplayQC.Tests/TestCaseSourceTests.cs

```csharp
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;

namespace DisplayQC.Tests;

[TestFixture]
public class TestCaseSourceTests
{
    #region 복합 등급 판정 테스트 데이터

    private static IEnumerable<TestCaseData> GradingTestCases()
    {
        yield return new TestCaseData(
            new List<Defect>(),
            new PanelSpec { MaxDeadPixels = 5, MaxBrightSpots = 3 },
            PanelGrade.APlus
        ).SetName("무결점_APlus등급");

        yield return new TestCaseData(
            new List<Defect>
            {
                new("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01")
            },
            new PanelSpec { MaxDeadPixels = 5, MaxBrightSpots = 3 },
            PanelGrade.A
        ).SetName("Minor불량1개_A등급");

        yield return new TestCaseData(
            new List<Defect>
            {
                new("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI-01"),
                new("D002", DefectType.Particle, DefectSeverity.Minor, 200, 200, 0.05, "AOI-01")
            },
            new PanelSpec { MaxDeadPixels = 5, MaxBrightSpots = 3 },
            PanelGrade.B
        ).SetName("Minor불량2개_B등급");

        yield return new TestCaseData(
            new List<Defect>
            {
                new("D001", DefectType.Scratch, DefectSeverity.Major, 100, 100, 2.0, "AOI-01")
            },
            new PanelSpec { MaxDeadPixels = 5, MaxBrightSpots = 3 },
            PanelGrade.C
        ).SetName("Major불량1개_C등급");

        yield return new TestCaseData(
            new List<Defect>
            {
                new("D001", DefectType.Crack, DefectSeverity.Critical, 100, 100, 5.0, "AOI-01")
            },
            new PanelSpec { MaxDeadPixels = 5, MaxBrightSpots = 3 },
            PanelGrade.NG
        ).SetName("Critical불량_NG등급");
    }

    [Test]
    [TestCaseSource(nameof(GradingTestCases))]
    public void GradePanel_ComplexCases_ReturnsCorrectGrade(
        List<Defect> defects, PanelSpec spec, PanelGrade expectedGrade)
    {
        var panel = new Panel("P001", "Test", 55.0, "3840x2160", "LOT001");
        foreach (var defect in defects)
            panel.AddDefect(defect);

        var grade = DetermineGradeWithSpec(panel, spec);

        Assert.That(grade, Is.EqualTo(expectedGrade));
    }

    private PanelGrade DetermineGradeWithSpec(Panel panel, PanelSpec spec)
    {
        if (panel.HasCriticalDefect()) return PanelGrade.NG;

        var deadPixelCount = panel.GetDefectsByType(DefectType.DeadPixel).Count();
        if (deadPixelCount > spec.MaxDeadPixels) return PanelGrade.NG;

        var minorCount = panel.GetDefectCountBySeverity(DefectSeverity.Minor);
        var majorCount = panel.GetDefectCountBySeverity(DefectSeverity.Major);

        if (majorCount > 0) return PanelGrade.C;
        if (minorCount >= 5) return PanelGrade.NG;
        if (minorCount >= 2) return PanelGrade.B;
        if (minorCount == 1) return PanelGrade.A;

        return PanelGrade.APlus;
    }

    #endregion

    #region Lot 단위 수율 분석 테스트 데이터

    private static IEnumerable<TestCaseData> LotYieldTestCases()
    {
        yield return new TestCaseData(
            new List<PanelGrade> { PanelGrade.APlus, PanelGrade.APlus, PanelGrade.A, PanelGrade.A, PanelGrade.B },
            100.0,
            false
        ).SetName("전량양품_수율100%");

        yield return new TestCaseData(
            new List<PanelGrade> { PanelGrade.A, PanelGrade.A, PanelGrade.B, PanelGrade.C, PanelGrade.NG },
            80.0,
            true
        ).SetName("NG1개_수율80%_알람발생");

        yield return new TestCaseData(
            new List<PanelGrade> { PanelGrade.A, PanelGrade.A, PanelGrade.B, PanelGrade.NG, PanelGrade.NG },
            60.0,
            true
        ).SetName("NG2개_수율60%_알람발생");
    }

    [Test]
    [TestCaseSource(nameof(LotYieldTestCases))]
    public void AnalyzeLotYield_VariousCases_ReturnsCorrectResult(
        List<PanelGrade> grades, double expectedYield, bool shouldAlarm)
    {
        var totalCount = grades.Count;
        var passCount = grades.Count(g => g != PanelGrade.NG);
        var targetYield = 95.0;

        var actualYield = (double)passCount / totalCount * 100;
        var actualShouldAlarm = actualYield < targetYield;

        Assert.That(actualYield, Is.EqualTo(expectedYield).Within(0.01));
        Assert.That(actualShouldAlarm, Is.EqualTo(shouldAlarm));
    }

    #endregion
}
```

---

## 📁 DisplayQC.Tests/AdvancedMoqTests.cs

```csharp
using Moq;
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;
using DisplayQC.Services;

namespace DisplayQC.Tests;

[TestFixture]
public class AdvancedMoqTests
{
    [Test]
    public void ProcessLot_MultiplePanels_CallsInspectCorrectTimes()
    {
        var mockPanelRepository = new Mock<IPanelRepository>();
        var mockEquipment = new Mock<IInspectionEquipment>();
        var mockAlarmService = new Mock<IAlarmService>();
        var mockMES = new Mock<IMESInterface>();

        var panels = new List<Panel>();
        for (int i = 1; i <= 5; i++)
        {
            var panel = new Panel($"P00{i}", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
            panels.Add(panel);
            mockPanelRepository.Setup(r => r.GetById($"P00{i}")).Returns(panel);
        }

        mockEquipment.Setup(e => e.EquipmentId).Returns("AOI-01");
        mockEquipment.Setup(e => e.IsOnline).Returns(true);
        mockEquipment.Setup(e => e.NeedsCalibration).Returns(false);
        mockEquipment.Setup(e => e.Inspect(It.IsAny<Panel>()))
            .Returns((Panel p) => new InspectionResult { PanelId = p.PanelId, IsPass = true, DetectedDefects = [] });

        var service = new InspectionService(
            mockPanelRepository.Object, mockEquipment.Object, mockAlarmService.Object, mockMES.Object);

        foreach (var panel in panels)
            service.ExecuteInspection(panel.PanelId);

        mockEquipment.Verify(e => e.Inspect(It.IsAny<Panel>()), Times.Exactly(5));
    }

    [Test]
    public void ProcessPanel_ValidationFailed_NeverCallsInspect()
    {
        var mockPanelRepository = new Mock<IPanelRepository>();
        var mockEquipment = new Mock<IInspectionEquipment>();
        var mockAlarmService = new Mock<IAlarmService>();
        var mockMES = new Mock<IMESInterface>();

        mockPanelRepository.Setup(r => r.GetById("INVALID")).Returns((Panel?)null);
        mockEquipment.Setup(e => e.IsOnline).Returns(true);

        var service = new InspectionService(
            mockPanelRepository.Object, mockEquipment.Object, mockAlarmService.Object, mockMES.Object);

        Assert.Throws<KeyNotFoundException>(() => service.ExecuteInspection("INVALID"));
        mockEquipment.Verify(e => e.Inspect(It.IsAny<Panel>()), Times.Never);
    }

    [Test]
    public void SendAlarm_CriticalDefect_CorrectSeverityPassed()
    {
        var mockPanelRepository = new Mock<IPanelRepository>();
        var mockEquipment = new Mock<IInspectionEquipment>();
        var mockAlarmService = new Mock<IAlarmService>();
        var mockMES = new Mock<IMESInterface>();

        var panel = new Panel("P001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        var criticalDefect = new Defect("D001", DefectType.Crack, DefectSeverity.Critical, 100, 100, 5.0, "AOI-01");

        mockPanelRepository.Setup(r => r.GetById(panel.PanelId)).Returns(panel);
        mockEquipment.Setup(e => e.EquipmentId).Returns("AOI-01");
        mockEquipment.Setup(e => e.IsOnline).Returns(true);
        mockEquipment.Setup(e => e.NeedsCalibration).Returns(false);
        mockEquipment.Setup(e => e.Inspect(panel))
            .Returns(new InspectionResult { PanelId = panel.PanelId, IsPass = false, DetectedDefects = [criticalDefect] });

        var service = new InspectionService(
            mockPanelRepository.Object, mockEquipment.Object, mockAlarmService.Object, mockMES.Object);

        service.ExecuteInspection(panel.PanelId);

        mockAlarmService.Verify(
            a => a.SendDefectAlarm(panel.PanelId, DefectType.Crack, DefectSeverity.Critical),
            Times.Once);
    }
}
```

---

## 📁 DisplayQC.Tests/CallbackSequenceTests.cs

```csharp
using Moq;
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;
using DisplayQC.Services;

namespace DisplayQC.Tests;

[TestFixture]
public class CallbackSequenceTests
{
    [Test]
    public void ProcessLot_TracksAllInspectedPanels()
    {
        var inspectedPanels = new List<string>();
        var mockPanelRepository = new Mock<IPanelRepository>();
        var mockEquipment = new Mock<IInspectionEquipment>();
        var mockAlarmService = new Mock<IAlarmService>();
        var mockMES = new Mock<IMESInterface>();

        var panelIds = new[] { "P001", "P002", "P003" };
        foreach (var panelId in panelIds)
        {
            var panel = new Panel(panelId, "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
            mockPanelRepository.Setup(r => r.GetById(panelId)).Returns(panel);
        }

        mockEquipment.Setup(e => e.EquipmentId).Returns("AOI-01");
        mockEquipment.Setup(e => e.IsOnline).Returns(true);
        mockEquipment.Setup(e => e.NeedsCalibration).Returns(false);
        mockEquipment.Setup(e => e.Inspect(It.IsAny<Panel>()))
            .Callback<Panel>(p => inspectedPanels.Add(p.PanelId))
            .Returns((Panel p) => new InspectionResult { PanelId = p.PanelId, IsPass = true, DetectedDefects = [] });

        var service = new InspectionService(
            mockPanelRepository.Object, mockEquipment.Object, mockAlarmService.Object, mockMES.Object);

        foreach (var panelId in panelIds)
            service.ExecuteInspection(panelId);

        Assert.That(inspectedPanels, Has.Count.EqualTo(3));
        Assert.That(inspectedPanels, Is.EquivalentTo(panelIds));
    }

    [Test]
    public void InspectMultiplePanels_SequentialResults()
    {
        var mockPanelRepository = new Mock<IPanelRepository>();
        var mockEquipment = new Mock<IInspectionEquipment>();
        var mockAlarmService = new Mock<IAlarmService>();
        var mockMES = new Mock<IMESInterface>();

        var panelIds = new[] { "P001", "P002", "P003" };
        foreach (var panelId in panelIds)
        {
            var panel = new Panel(panelId, "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
            mockPanelRepository.Setup(r => r.GetById(panelId)).Returns(panel);
        }

        mockEquipment.Setup(e => e.EquipmentId).Returns("AOI-01");
        mockEquipment.Setup(e => e.IsOnline).Returns(true);
        mockEquipment.Setup(e => e.NeedsCalibration).Returns(false);

        mockEquipment.SetupSequence(e => e.Inspect(It.IsAny<Panel>()))
            .Returns(new InspectionResult { PanelId = "P001", IsPass = true, DetectedDefects = [] })
            .Returns(new InspectionResult { PanelId = "P002", IsPass = true, DetectedDefects = [] })
            .Returns(new InspectionResult { PanelId = "P003", IsPass = false,
                DetectedDefects = [new Defect("D1", DefectType.Crack, DefectSeverity.Critical, 100, 100, 5.0, "AOI")] });

        var service = new InspectionService(
            mockPanelRepository.Object, mockEquipment.Object, mockAlarmService.Object, mockMES.Object);

        var results = panelIds.Select(id => service.ExecuteInspection(id)).ToList();

        Assert.That(results[0].IsPass, Is.True);
        Assert.That(results[1].IsPass, Is.True);
        Assert.That(results[2].IsPass, Is.False);
    }

    [Test]
    public void InspectionWithRetry_FailsThenSucceeds()
    {
        var mockEquipment = new Mock<IInspectionEquipment>();
        var retryCount = 0;

        mockEquipment.SetupSequence(e => e.Inspect(It.IsAny<Panel>()))
            .Throws(new TimeoutException("타임아웃 1"))
            .Throws(new TimeoutException("타임아웃 2"))
            .Returns(new InspectionResult { PanelId = "P001", IsPass = true, DetectedDefects = [] });

        var panel = new Panel("P001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        InspectionResult? result = null;

        for (int i = 0; i < 3; i++)
        {
            try
            {
                retryCount++;
                result = mockEquipment.Object.Inspect(panel);
                break;
            }
            catch (TimeoutException) { }
        }

        Assert.That(retryCount, Is.EqualTo(3));
        Assert.That(result, Is.Not.Null);
        Assert.That(result!.IsPass, Is.True);
    }
}
```

---

## 📁 DisplayQC.Tests/MockBehaviorTests.cs

```csharp
using Moq;
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;

namespace DisplayQC.Tests;

[TestFixture]
public class MockBehaviorTests
{
    [Test]
    public void StrictMock_UnexpectedMethodCall_ThrowsException()
    {
        var strictMock = new Mock<IInspectionEquipment>(MockBehavior.Strict);
        Assert.Throws<MockException>(() => { var _ = strictMock.Object.IsOnline; });
    }

    [Test]
    public void StrictMock_AllMethodsSetup_NoException()
    {
        var strictMock = new Mock<IInspectionEquipment>(MockBehavior.Strict);
        strictMock.Setup(e => e.IsOnline).Returns(true);
        strictMock.Setup(e => e.EquipmentId).Returns("AOI-01");

        Assert.DoesNotThrow(() =>
        {
            var _ = strictMock.Object.IsOnline;
            var __ = strictMock.Object.EquipmentId;
        });
    }

    [Test]
    public void LooseMock_UnsetupMethod_NoException()
    {
        var looseMock = new Mock<IAlarmService>();
        Assert.DoesNotThrow(() =>
            looseMock.Object.SendDefectAlarm("P001", DefectType.DeadPixel, DefectSeverity.Minor));
    }

    [Test]
    public void LooseMock_ReturnTypeMethod_ReturnsDefault()
    {
        var looseMock = new Mock<IPanelRepository>();
        Assert.That(looseMock.Object.GetById("P001"), Is.Null);
        Assert.That(looseMock.Object.GetTotalCount(), Is.EqualTo(0));
    }
}
```

---

## 📁 DisplayQC.Tests/CoverageImprovementTests.cs

```csharp
using Moq;
using DisplayQC.Domain;
using DisplayQC.Domain.Interfaces;
using DisplayQC.Services;

namespace DisplayQC.Tests;

[TestFixture]
public class CoverageImprovementTests
{
    [Test]
    public void Panel_HasCriticalDefect_WithCriticalDefect_ReturnsTrue()
    {
        var panel = new Panel("P001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.AddDefect(new Defect("D001", DefectType.Crack, DefectSeverity.Critical, 100, 100, 5.0, "AOI"));
        Assert.That(panel.HasCriticalDefect(), Is.True);
    }

    [Test]
    public void Panel_HasCriticalDefect_WithoutCriticalDefect_ReturnsFalse()
    {
        var panel = new Panel("P001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.AddDefect(new Defect("D001", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI"));
        Assert.That(panel.HasCriticalDefect(), Is.False);
    }

    [Test]
    public void Panel_GetDefectCountBySeverity_AllSeverities()
    {
        var panel = new Panel("P001", "OLED-55-UHD", 55.0, "3840x2160", "LOT001");
        panel.AddDefect(new Defect("D1", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI"));
        panel.AddDefect(new Defect("D2", DefectType.Particle, DefectSeverity.Minor, 200, 200, 0.05, "AOI"));
        panel.AddDefect(new Defect("D3", DefectType.Scratch, DefectSeverity.Major, 300, 300, 2.0, "AOI"));
        panel.AddDefect(new Defect("D4", DefectType.Crack, DefectSeverity.Critical, 400, 400, 5.0, "AOI"));

        Assert.That(panel.GetDefectCountBySeverity(DefectSeverity.Minor), Is.EqualTo(2));
        Assert.That(panel.GetDefectCountBySeverity(DefectSeverity.Major), Is.EqualTo(1));
        Assert.That(panel.GetDefectCountBySeverity(DefectSeverity.Critical), Is.EqualTo(1));
    }

    [Test]
    public void Defect_EmptyDefectId_ThrowsArgumentException()
    {
        var ex = Assert.Throws<ArgumentException>(() =>
            new Defect("", DefectType.DeadPixel, DefectSeverity.Minor, 100, 100, 0.1, "AOI"));
        Assert.That(ex.ParamName, Is.EqualTo("defectId"));
    }

    [Test]
    public void GradingService_GradePanel_PanelNotFound_ThrowsKeyNotFoundException()
    {
        var mockPanelRepository = new Mock<IPanelRepository>();
        var mockMES = new Mock<IMESInterface>();
        var mockAlarmService = new Mock<IAlarmService>();

        mockPanelRepository.Setup(r => r.GetById("INVALID")).Returns((Panel?)null);
        var service = new GradingService(mockPanelRepository.Object, mockMES.Object, mockAlarmService.Object);

        Assert.Throws<KeyNotFoundException>(() => service.GradePanel("INVALID"));
    }

    [Test]
    public void Panel_SetGrade_AllGradeTypes()
    {
        var panelAPlus = new Panel("P1", "Test", 55.0, "3840x2160", "LOT001");
        panelAPlus.SetGrade(PanelGrade.APlus);
        Assert.That(panelAPlus.Status, Is.EqualTo(PanelStatus.Completed));

        var panelNG = new Panel("P2", "Test", 55.0, "3840x2160", "LOT001");
        panelNG.SetGrade(PanelGrade.NG);
        Assert.That(panelNG.Status, Is.EqualTo(PanelStatus.Rejected));
    }
}
```

---

# 📊 평가 기준 (100점 만점)

| 항목 | 배점 | 세부 내용 |
|------|------|----------|
| 프로젝트 구성 | 10점 | 솔루션 구조 5점, 프로젝트 참조 5점 |
| 도메인 모델 | 20점 | Panel 10점, Defect 10점 |
| TDD 적용 | 15점 | Red-Green-Refactor 5점, AAA 패턴 5점, 명명 규칙 5점 |
| Mock 활용 | 20점 | 기본 Mock 5점, Verify 5점, Callback/Sequence 5점, Strict/Loose 5점 |
| 예외 처리 테스트 | 10점 | Assert.Throws 5점, 경계값 테스트 5점 |
| 파라미터화 테스트 | 10점 | TestCase 5점, TestCaseSource 5점 |
| 코드 커버리지 | 10점 | 80% 이상 달성 5점, 미커버 코드 분석 5점 |
| 도메인 적합성 | 5점 | 실제 공정 시나리오 반영 3점, 실무 용어 적용 2점 |

**등급**: 90점 이상 우수, 80점 이상 양호, 70점 이상 보통, 70점 미만 미흡

---

# 🎉 수고하셨습니다!

본 실습을 통해 디스플레이 제조 공정의 품질 관리 시스템에 단위 테스트를 작성하는 역량을 갖추셨습니다.

**학습 내용 요약**:
- TDD (Red-Green-Refactor)
- NUnit 활용 (Assert.That, TestCase, TestCaseSource, Assert.Throws)
- Moq 활용 (Mock, Verify, Callback, SetupSequence, Strict/Loose)
- 코드 커버리지 측정 및 개선

**다음 단계**: 통합 테스트, End-to-End 테스트, CI/CD 파이프라인 구축
