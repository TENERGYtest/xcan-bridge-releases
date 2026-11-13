# xCAN-Bridge

CAN / CAN FD 브릿지와 모니터링·자동화 도구입니다. PEAK PCAN, Kvaser 하드웨어로 CAN 버스에 연결해 송수신하고, DBC/SYM 신호를 디코딩해 보여 주며, UDP·INCA·AVL E-Storage 같은 외부 시스템과 신호를 주고받습니다.

- 플랫폼: Windows 10 이상, x64
- 스택: WPF, .NET 8, C#
- 현재 버전: `xCANBridge.csproj`의 `<Version>` 참조 (버전 규칙은 아래 "버전과 배포" 참고)

## 주요 기능

| 영역 | 내용 |
|---|---|
| CAN 연결 | PCAN과 Kvaser를 한 프로젝트에서 동시에 사용, 다중 버스, Bus-Off 자동 복구, CAN 2.0A/B와 CAN FD(BRS) |
| 주기 송신 (TX) | 1 ms 단위 주기, 신호 Ramp, 그룹 일시정지/재개, TX List 편집, Loopback 확인 |
| 수신과 디코딩 (RX) | DBC/SYM 디코딩, 주기 카운트(EMA), 안정 정렬, 신호 값 표시 |
| DB Manager | DBC 편집·저장, SYM 읽기, 프로젝트 가상 신호(Virtual) |
| 신호 표시 | Signal Plotter(실시간 차트, 커서), Signal Monitor, Signal Control |
| 자동화 | Logic Script, Function Table(스크립트·TX List·명령), 트리거 조건 |
| UDP 게이트웨이 | 외부 프로그램과 신호 교환(`gspCAN` 프로토콜), Watchdog 동기화, Ramp 지원, 최대 2채널 |
| 녹화와 재생 | 장시간 raw 녹화(XREC), 시작/종료 트리거, Pre/Post-roll, ASC·TRC·BLF·CSV·MDF3·MDF4 내보내기, Tracer 재생 |
| 외부 연동 | INCA(ASAP3 TCP, `.exp` 가져오기), AVL E-Storage 제어 창 |
| 진단과 운영 | 연결 Fault 알림, 진단 패키지 내보내기, 프로젝트 호환 마이그레이션, 자동 업데이트 |

기능별 설명과 화면은 `Docs\Catalog\`의 제품 카탈로그를 참고하세요.

### UDP 프로토콜 (`gspCAN`)
- 수신(외부 → xCAN-Bridge): `gspCAN,<채널번호>,<value>,<rampSec>;`
- 송신(xCAN-Bridge → 외부): `gspCAN,update,N,슬롯1_watchdog,슬롯2,...;`
  - 슬롯 1은 Alive_Counter(Watchdog) 고정, 슬롯 2부터 TX 바인딩 순서입니다.
- 기본 포트는 RX 13101 / TX 15101입니다. 채널별로 변경할 수 있습니다.

### 파일 형식
`.xcbp`(프로젝트), `.xmt`(TX List), `.xcls`(Logic Script), `.xrec`(raw 녹화)

## 요구 사항

- **실행**: Windows 10 이상 x64. .NET 런타임은 설치파일에 포함되어 있어 따로 설치하지 않아도 됩니다.
- **PCAN**: PEAK-System 드라이버(`PCANBasic.dll`). 설치 마법사가 설치 여부를 확인합니다.
- **Kvaser**: Kvaser 드라이버(CANlib). 네이티브 `canlib32.dll`은 번들하지 않고 드라이버가 설치한 것을 사용합니다.
- 하드웨어 드라이버는 각 제조사에서 받아 설치해야 합니다. PCAN은 PEAK-System, Kvaser는 Kvaser AB의 제품입니다.

## 빌드와 테스트

개발 환경: Windows, .NET 8 SDK, Visual Studio 2022 이상 또는 `dotnet` CLI.

```bash
dotnet build xCANBridge.csproj -c Release
```

루트에 `.csproj`가 여러 개 보일 수 있으므로(`*_wpftmp.csproj`는 임시 파일) **파일명을 반드시 지정**합니다. 빌드 전에 실행 중인 `xCANBridge` 프로세스를 종료하세요.

```bash
dotnet test xCANBridge.Tests
```

- 하드웨어가 필요한 테스트(Kvaser 실장비 등)는 기본으로 건너뜁니다. 하드웨어 없이도 나머지 테스트는 모두 실행됩니다.
- 외부 라이브러리 중 INCA·Kvaser 래퍼는 `libs\`에 포함되어 있습니다. 나머지는 NuGet에서 복원됩니다(AvalonDock, ScottPlot, MahApps IconPacks, XCanDbc, XCanMdf).

## 설치파일 빌드

반드시 `Installer\BuildInstaller.ps1`로 빌드합니다. 전체 테스트 실행 → 단일 파일 `dotnet publish`(win-x64) → 빌드 정보·해시 기록 → Inno Setup 컴파일을 한 번에 수행합니다. Inno Setup 컴파일러(ISCC)는 직접 실행하지 마세요.

```powershell
.\Installer\BuildInstaller.ps1 -Channel Stable -Notes "릴리스 노트 한 줄"
```

- 결과물은 `Installer\Output\`에 만들어지며 Git에는 포함되지 않습니다.
- `-Channel`은 `Internal`(기본), `RC`, `Stable`, `Hotfix` 중 하나입니다. **자동 업데이트용 `latest.json`은 `Stable`일 때만 생성됩니다.**
- 이 스크립트는 `Core\Project\ProjectData.cs`를 직접 읽습니다. 그 파일을 옮기면 스크립트도 함께 수정해야 합니다.

## 버전과 배포

- 버전의 기준은 `xCANBridge.csproj`의 `<Version>` 한 곳이며, 설치파일·정보 창에 자동으로 반영됩니다.
- 앞 두 자리(x.y)는 사람이 정하고, 세 번째 자리는 배포할 때마다 1씩 올립니다.
- 배포 저장소: `tenergyx/xcan-bridge-releases` (설치파일과 `latest.json`만 공개)

### 자동 업데이트
앱이 시작할 때(그리고 Help → Check for Updates에서) 배포 저장소의 `latest.json`을 읽어 새 버전을 알려 줍니다.

- Stable 채널만 대상입니다.
- 받은 설치파일은 SHA-256과 크기를 검증하고, 실행 직전에 한 번 더 검증합니다.
- 녹화·CAN/UDP/INCA/AVL 연결 중에는 설치를 시작하지 않습니다.
- 코드 서명은 하지 않습니다. 따라서 SHA-256은 전송 중 손상을 걸러 내는 용도이며, 배포 저장소 계정의 보안이 곧 배포 보안입니다.

릴리스 올리는 절차:
1. 위 명령으로 설치파일과 `latest.json`을 만듭니다.
2. 배포 저장소에서 새 릴리스를 만듭니다. 태그는 `v{버전}`, 두 파일을 모두 첨부하며 **pre-release로 표시하지 않습니다.**
3. 공개한 뒤 `releases/latest/download/latest.json`의 `version`과 설치파일 주소가 열리는지 확인합니다.

## 프로젝트 구조

폴더는 기능별로 나누지만 **네임스페이스는 폴더와 무관하게** `xCANBridge.Core` / `.ViewModels` / `.Views`로 유지합니다. 파일을 옮겨도 `using`이나 `x:Class`를 고칠 필요가 없습니다.

```
Core/            도메인 로직 (UI와 무관)
  Can/           PcanChannel, Kvaser, TxScheduler, RxMessageService, RampController
  Dbc/           DBC/SYM 파서, SignalKey
  Diagnostics/   진단 서비스, FaultCode, 진단 패키지
  Project/       프로젝트 파일 저장/불러오기, 마이그레이션, 앱 설정
  Recording/     XREC 녹화, 내보내기(ASC/TRC/BLF/CSV/MDF), 재생 엔진
  Scripting/     Logic Script 엔진, 표현식/트리거 평가
  Udp/           UDP 브릿지
  Update/        업데이트 확인, 다운로드, 설치 실행
  Inca/, AvlEStorage/, Common/
ViewModels/      Shell(MainViewModel partial), Can, Project, Recording, Scripting, Udp, Signals, Editors
Views/           Shell, Panels(도킹 패널), Editors, Recording, Dialogs, Converters
Controls/        재사용 컨트롤 (TrendChart, SignalRangeBar)
Installer/       설치파일 빌드 스크립트와 Inno Setup 스크립트
xCANBridge.Tests/  단위·통합 테스트 (Can, Dbc, Project, Recording, Scripting, Udp, Update 등)
Docs/            설계 문서, 제품 카탈로그(Catalog), UI 미리보기(Previews)
```

### 아키텍처 메모
- `MainViewModel`은 partial class로 기능별(`MainViewModel.Pcan.cs`, `.Rx.cs` 등) 나뉩니다.
- RX: 수신 스레드는 pending 버퍼만 갱신하고, 200 ms 타이머가 UI에 일괄 반영합니다.
- TX: `Stopwatch` 기반 1 ms 고정밀 루프(Sleep + SpinWait 혼합). Windows는 실시간 OS가 아니므로 실제 송신 지터는 PC 부하에 따라 달라질 수 있습니다.

## 문서

| 문서 | 내용 |
|---|---|
| `Docs\Catalog\` | 제품 카탈로그 (HTML/PDF/PNG) |
| `Docs\COMPATIBILITY_REPORT.md` | 프로젝트 파일 호환 형식과 지원 버전 |
| `Docs\QUALITY_RELEASE_INCIDENT_SYSTEM_DESIGN.md` | 품질·릴리스·장애 관리 체계 |
| `Docs\FAULT_DIAGNOSTICS_DESIGN.md` | 장애 진단 설계 |
| `Docs\KVASER_INTEGRATION_DESIGN.md` | Kvaser 통합 설계 |
| `Docs\MIGRATION_RELEASE_SYSTEM_DESIGN.md` | 프로젝트 마이그레이션과 릴리스 |
| `Docs\STATUS_BAR_DESIGN.md` | 상태 표시줄 설계 |
| `Docs\INTERNAL_TEST_EXPANSION_DESIGN.md` | 내부 테스트 확장 |

## 라이선스

사내용 도구이며 별도 라이선스를 두지 않습니다. 소스 코드는 공개하지 않습니다.
