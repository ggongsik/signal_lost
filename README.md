# SIGNAL LOST

> 언리얼 엔진 5로 제작하는 멀티플레이어 공포 게임.
> 콘솔을 찾아라. 명령어를 입력하라. 함께 탈출하라.

---

## 개요

SIGNAL LOST는 버려진 폐공장을 배경으로 한 협동 공포 게임입니다.
최대 4명의 플레이어가 함께 시설 곳곳에 숨겨진 5개의 콘솔을 찾아 올바른 명령어를 입력하고, 비상 출구를 열어 탈출하는 것이 목표입니다.

괴물은 플레이어들의 발소리와 목소리를 실시간으로 추적하며,
게임 중 녹음된 플레이어들의 말을 그대로 재생하면서 접근합니다.

---

## 게임플레이

**목표**

5개의 콘솔 각각에 정확한 명령어를 입력하여 모두 활성화한 뒤, 괴물에게 잡히기 전에 출구에 도달한다.

**플레이 인원**

1인 ~ 4인 협동 플레이

**조작법**

| 입력 | 동작 |
|---|---|
| 마우스 | 화면 회전 |
| W / A / S / D | 이동 |
| Shift (유지) | 달리기 |
| Ctrl (유지) | 앉기 |
| E | 콘솔 상호작용 |

**스테미나 시스템**

플레이어와 괴물 모두 동일한 스테미나 구조를 가집니다. 달리기를 지속하면 스테미나가 소모되며, 스테미나가 0이 되면 회복될 때까지 달릴 수 없습니다.

**괴물**

- 발소리를 감지하여 실시간으로 추적 방향을 조정
- 마이크를 통해 플레이어의 목소리를 인식하고 음원 방향으로 접근
- 게임 세션 중 녹음된 플레이어의 목소리를 간헐적으로 재생
- 플레이어와 동일한 캐릭터 모델 사용

---

## 기술 설계

이 프로젝트는 객체지향 원칙을 기반으로 설계되었으며, 실제 게임 개발에서 사용되는 소프트웨어 디자인 패턴을 적용합니다.

**네트워크 구조**

Listen Server 방식을 채택합니다. 별도의 전용 서버 없이 방장의 PC가 서버와 클라이언트 역할을 동시에 수행합니다. 세션은 Steam OnlineSubsystem을 통해 비공개로 생성되며, Steam 친구 초대를 수락한 플레이어만 입장할 수 있습니다. 공개 서버 검색은 지원하지 않습니다.

```
방장이 비공개 세션 생성 (USessionSubsystem)
    -> Steam 친구에게 초대 발송 (SendSteamInvite)
        -> 초대 수락한 플레이어가 방장 PC에 직접 접속
            -> 모든 게임 상태는 서버(방장 PC)에서 관리
                -> 클라이언트에 Replication으로 동기화
```

콘솔 명령어 입력은 ServerRPC를 통해 서버에서만 처리되고, 결과는 ClientRPC로 각 플레이어에게 전달됩니다.

**적용된 디자인 패턴**

| 패턴 | 적용 위치 |
|---|---|
| Template Method | ABaseCharacter가 스테미나 로직을 공통 정의, Player와 Monster가 각자의 이동 방식을 오버라이드 |
| Observer | 스테미나 변화 시 IStaminaObserver를 통해 HUD 및 AI 상태에 자동 반영 |
| Singleton | UAudioManager가 발소리 이벤트를 중앙 관리, Monster가 해당 이벤트를 구독 |
| Command | 콘솔 명령어 입력이 Widget -> ConsoleActor -> ConsoleManager 체인으로 처리 |

**클래스 구조**

전체 클래스 다이어그램 및 설계 결정 사항은 [DESIGN.md](./Docs/DESIGN.md)를 참조하세요.

---

## 프로젝트 구조

```
SIGNAL_LOST/
├── Source/
│   ├── Characters/
│   │   ├── ABaseCharacter
│   │   ├── APlayerCharacter
│   │   └── AMonsterCharacter
│   ├── AI/
│   │   ├── UMonsterAIController
│   │   └── BehaviorTree/
│   ├── Console/
│   │   ├── AConsoleActor
│   │   ├── AConsoleManager
│   │   └── UConsoleWidget
│   ├── Systems/
│   │   ├── UAudioManager
│   │   └── UFootstepComponent
│   └── Core/
│       ├── AFactoryGameMode
│       └── AFactoryGameState
├── Content/
│   ├── Maps/
│   ├── Blueprints/
│   └── Audio/
└── Docs/
    ├── DESIGN.md
    ├── DEVLOG.md
    └── diagrams/
```

---

## 개발 환경

| 도구 | 버전 |
|---|---|
| Unreal Engine | 5.x |
| 언어 | C++ / Blueprint |
| IDE | Visual Studio 2022 |
| 버전 관리 | Git / GitHub |

---

## 개발 로드맵

- [ ] 캐릭터 이동 및 스테미나 시스템 구현
- [ ] 멀티플레이어 세션 및 네트워크 동기화
- [ ] 콘솔 상호작용 및 명령어 입력 UI
- [ ] 출구 잠금 해제 로직
- [ ] 괴물 AI: 발소리 추적
- [ ] 괴물 AI: 마이크 캡처 및 음성 재생
- [ ] 소리 반응형 행동 튜닝
- [ ] 맵 레이아웃 및 환경 아트
- [ ] 최종 빌드 및 폴리싱

---

## 문서

| 문서 | 설명 |
|---|---|
| [DESIGN.md](./Docs/DESIGN.md) | 클래스 다이어그램, 아키텍처, 디자인 패턴 분석 |
| [DEVLOG.md](./Docs/DEVLOG.md) | 개발 과정 기록 일지 |

---

## 라이선스

이 프로젝트는 출시 및 포트폴리오 목적으로 제작되었습니다.
