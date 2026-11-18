# aide-ecu

보드 A(열관리 제어기)와 보드 B(스마트 냉각팬 ECU)의 펌웨어를 함께 관리하는 저장소입니다.
저장소는 하나, 보드별 프로젝트는 폴더로 나누고, 작업은 작업 단위 브랜치에서 한 뒤 `main`에 합칩니다.

- 보드: NUCLEO-F103RB (두 보드 공통)
- 개발 도구: STM32CubeIDE 1.19

## 폴더 구조

```
aide-ecu/
├── board_a/        ← 보드 A(열관리 제어기) CubeIDE 프로젝트
├── board_b/        ← 보드 B(스마트 냉각팬 ECU) CubeIDE 프로젝트
│   ├── board_b.ioc ← 핀·타이머·CAN 설정
│   ├── Core/       ← CubeIDE 자동 생성 (main.c 등)
│   ├── Drivers/    ← ST HAL (자동 생성, 직접 수정 X)
│   ├── App/        ← 응용: 무엇을 할지 (제어, 고장 판단)
│   ├── Service/    ← 통신: CAN 프레임 조립·송수신
│   ├── EcuAbs/     ← 하드웨어 추상화: 모터, 센서
│   └── Config/     ← 보드별 설정값
├── common/         ← 두 보드가 같이 쓰는 코드 (CAN ID, 메시지 정의)
├── docs/           ← CAN 메시지 명세서 등
├── .gitignore
└── README.md
```

| 폴더 | 담당 | 들어가는 것 |
|---|---|---|
| `board_a/` | 보드 A 담당자 | 보드 A 프로젝트 전체 (내부 구조는 `board_b/`와 동일하게 맞추기 제안) |
| `board_b/` | 김보명 | 보드 B 프로젝트 전체 |
| `common/` | 두 사람 공동 | CAN ID, 상태·고장 코드, 메시지 조립/해석 함수 |
| `docs/` | 두 사람 공동 | 명세서, 배선도 등 문서 |

**계층 규칙:** 위층은 아래층 함수만 부르고, ST HAL은 `Service/`와 `EcuAbs/`에서만 직접 씁니다.
모터나 센서를 바꿔도 `EcuAbs/`만 고치면 되도록 하기 위함입니다.

## 팀 규칙

1. **`main`에 직접 올리지 않기.** 항상 브랜치에서 작업하고 PR로 합칩니다.
2. **내 폴더만 고치기.** 상대 보드 폴더는 보기만 하고, 고칠 일이 있으면 담당자에게 말합니다.
3. **`common/`은 상대 확인 후 합치기.** `common/`을 고친 PR은 상대 담당자의 승인 후에만 합칩니다.
4. **작업 시작 전에 `main` 받기.** 브랜치를 만들기 전에 `git pull`로 최신 `main`을 받습니다.
5. **빌드가 되는 코드만 합치기.** 내 보드에서 빌드와 동작을 확인한 뒤 PR을 올립니다.

## 브랜치 / 커밋 규칙

| 종류 | 이름 규칙 | 예시 |
|---|---|---|
| 기준 | `main` | 동작 확인된 코드만 있는 브랜치 |
| 보드 작업 | `feature/board-<a\|b>-<작업내용>` | `feature/board-b-fan-pwm` |
| 공통 코드 | `feature/common-<작업내용>` | `feature/common-can-ids` |
| 버그 수정 | `fix/<보드>-<내용>` | `fix/board-b-can-timeout` |

작업 브랜치는 `main`에 합친 뒤 삭제합니다.

커밋 메시지는 `<Jira 키> <한 일>` 형식으로 한 줄로 씁니다. 커밋은 작게 자주 합니다.

```
PJT-17 ULN2003 PWM 구동 추가
PJT-17 팬 상태 머신(INIT/STOP/RUN) 추가
PJT-16 can_ids.h에 보드 B ID 임시값 추가
```

## 처음 세팅

1. 저장소를 받습니다: `git clone <저장소 주소> aide-ecu`
2. 내 프로젝트용 브랜치를 만듭니다: `git switch -c feature/board-a-init` (보드 B는 `board-b-init`)
3. CubeIDE: **File → New → STM32 Project**, 위치를 `board_a/`(또는 `board_b/`)로 지정하고
   `App/ Service/ EcuAbs/ Config/` 폴더를 추가합니다. (`.gitkeep`은 삭제해도 됩니다)
4. `git status`로 `Debug/`가 보이지 않는지 확인한 뒤 내 폴더만 커밋·푸시하고 `main`으로 PR을 만듭니다.
5. CubeIDE에서는 **File → Import → Existing Projects into Workspace**로 내 프로젝트만 가져오고,
   `common/`은 프로젝트 속성의 Include Path에 추가합니다.

## 매일 작업 흐름

```bash
git switch main
git pull                                 # 상대가 합친 내용 받기
git switch -c feature/board-b-fan-pwm    # 작업 브랜치

# 작업...
git add board_b                          # 내 폴더만 지정
git commit -m "PJT-17 ULN2003 PWM 구동 추가"
git push -u origin feature/board-b-fan-pwm
# → 웹에서 PR 생성 → 합치기

git switch main
git pull
git branch -d feature/board-b-fan-pwm    # 다 쓴 브랜치 삭제
```

작업 중에 상대가 `common/`을 바꿔 `main`에 합쳤다면 내 브랜치로 가져옵니다.

```bash
git switch main
git pull
git switch feature/board-b-fan-pwm
git merge main
```

## 올릴 것과 안 올릴 것

| 구분 | 파일/폴더 | 이유 |
|---|---|---|
| 올림 | `.ioc`, `.project`, `.cproject` | 핀 설정과 빌드 설정 |
| 올림 | `Core/`, 직접 만든 폴더 | 우리가 쓴 코드 |
| 올림 | `Drivers/` | CubeIDE 버전에 따라 내용이 달라질 수 있어서 같이 올림 |
| 안 올림 | `Debug/`, `Release/`, `*.elf`, `*.o`, `*.map` | 빌드 결과물 |
| 안 올림 | `.settings/` | 개인 편집기 설정 |

## 아직 정할 것

- [ ] 저장소 폴더 뼈대를 누가 만들지
- [ ] `board_a/` 내부 폴더를 `board_b/`와 같은 계층(App / Service / EcuAbs / Config)으로 맞출지
- [ ] AI팀(라즈베리파이) 코드를 같은 저장소(`edge_ai/`)에 둘지 — AI팀과 협의
