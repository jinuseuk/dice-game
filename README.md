# 🎲 FPGA Real-Time Dice Board Game (I2C & VGA 기반)

## 🌟 프로젝트 개요 (Overview)

[cite_start]본 프로젝트는 **OV7670 카메라 모듈**을 통해 실시간으로 주사위 눈금을 인식하고, **Xilinx Basys 3 FPGA 보드**에서 게임 로직을 구현하여 **VGA 모니터**에 출력하는 디지털 하드웨어 설계 프로젝트입니다[cite: 449, 450, 451].

[cite_start]특히, **I2C 통신 프로토콜**을 활용한 **Master-Slave 아키텍처**를 적용하여 중앙 제어 로직과 플레이어 상태 제어 로직을 분리 및 연동함으로써, 2인용 보드 게임의 실시간 상태 변화 및 시각적 효과를 구현했습니다[cite: 499, 500, 501, 502].

* **핵심 목표:** 실시간 영상 처리를 통한 입력(주사위 눈금) 인식, 하드웨어 통신 프로토콜(I2C) 설계 및 연동, VGA 디스플레이 제어.
* **주요 기능:** 주사위 눈금 인식, 자동 말 이동, 사다리/뱀 효과, 플레이어 상태에 따른 실시간 영상 필터 적용.

## ⚙️ 개발 환경 및 기술 스택 (Development Environment)

| 분류 | 내용 | 비고 |
| :--- | :--- | :--- |
| **HDL 언어** | [cite_start]SystemVerilog (IEEE 1800-2017) [cite: 484, 486] | RTL (Register Transfer Level) 설계 언어 |
| **EDA Tool** | [cite_start]AMD Xilinx Vivado Design Suite 2020.2 [cite: 483] | 합성, 구현 및 비트스트림 생성 |
| **개발 보드** | [cite_start]Digilent Basys 3 (100 MHz on-board clock) [cite: 485, 487] | Xilinx Artix-7 FPGA 기반 |
| **카메라 모듈** | [cite_start]OV7670 [cite: 488, 489] | [cite_start]QQVGA (160x120) 해상도, RGB565 포맷 [cite: 368, 381] |
| **인터페이스** | [cite_start]**I2C Protocol** (Master–Slave 게임 데이터 전송) [cite: 491] | Master(게임 로직) - Slave(플레이어 1, 2) 통신 |
| | [cite_start]**SCCB Protocol** (OV7670 레지스터 제어) [cite: 492, 382] | 카메라 초기 설정 및 제어 |

## 📐 시스템 아키텍처 (System Architecture)

[cite_start]프로젝트는 중앙에서 게임 로직을 제어하는 **Master Board**와 각 플레이어의 상태 및 시각 효과를 담당하는 **Slave 모듈**로 구성됩니다[cite: 493, 494, 495, 496, 497].

****

### 1. Game Logic Module (I2C Master)

[cite_start]게임의 핵심 로직을 담당하며, Master Board의 버튼 입력과 주사위 인식 결과를 기반으로 동작합니다[cite: 523, 524, 525]. [cite_start]4개의 FSM(Finite State Machine) 모듈로 구성되어 유기적인 게임 흐름을 제어합니다[cite: 519, 521].

| 모듈 | 역할 | 상세 로직 |
| :--- | :--- | :--- |
| **GAME\_STATE** | [cite_start]게임의 전역 상태 관리 [cite: 534] | [cite_start]사용자 버튼 입력, 3판 2승제, Time Over 신호 기반 게임 진행 상태 제어 [cite: 532, 533] |
| **VICTORY\_TRACKER** | [cite_start]3판 2승제 승부 판정 [cite: 358] | [cite_start]최종 승리 조건(총 2점 획득) 확인 및 결과 계산 [cite: 357] |
| **PLAY\_GAME** | [cite_start]게임 진행 및 위치 계산 [cite: 366, 379] | [cite_start]주사위 값에 따른 플레이어 위치 업데이트, 사다리/벌칙 구간 위치 보정, 승부 비교 [cite: 366, 379] |
| **TIMER** | 게임 시간 관리 | [cite_start]10분 카운트다운 관리 (Start, Restart, Final 신호 제어) [cite: 367] |
| **I2C Master Controller** | [cite_start]Slave 통신 제어 [cite: 367] | [cite_start]게임 이벤트(이동, 사다리, 종료) 발생 시, Player 1 → Player 2 순서로 I2C 통신을 통해 Slave 장치에게 정보 전송 [cite: 367, 380] |
| **VGA Display** | [cite_start]게임 정보 출력 [cite: 393] | [cite_start]SCORE, TIMER, 현재 게임 상태, Player Turn, 말의 위치 등 출력 [cite: 404] |

### 2. Dice Detector & VGA Display Module

[cite_start]카메라 영상을 실시간으로 처리하여 주사위 눈금을 인식하고 모니터에 출력하는 모듈입니다[cite: 374, 387].

| 주요 기능 | 상세 로직 |
| :--- | :--- |
| **실시간 영상 출력** | OV7670 데이터 캡처 후 FrameBuffer(RAM)에 저장. [cite_start]VGA 타이밍에 맞춰 데이터 읽기[cite: 369, 370, 371, 382, 383, 384]. |
| **`red_check`** | [cite_start]주사위 눈금을 판별하기 위해, 픽셀의 **Red > Green 및 Red > Blue** 조건으로 빨간색 픽셀만 출력하고 나머지는 검은색으로 Masking 처리[cite: 372, 376, 377, 385, 389, 390, 400, 401]. |
| **`Dice_Reader`** | [cite_start]한 프레임 동안 **빨간색 픽셀 수를 카운트**하여 주사위 눈금을 실시간으로 판독[cite: 373, 386]. [cite_start]최종 카운트 값을 **125 단위**로 조건을 나누어 눈금(1~6)을 판별[cite: 392, 403]. |

### 3. I2C Slave Module (Game Player Slave)

[cite_start]Master에서 I2C로 전송된 신호를 받아 해당 플레이어의 화면에 실시간 시각 필터를 적용하는 모듈입니다[cite: 393, 404].

| Register | 기능 (Master Write) | Slave 적용 효과 |
| :--- | :--- | :--- |
| **Reg0** | [cite_start]현재 게임 진행 상태 [cite: 394, 404] | 앞선 상황: Normal 필터. [cite_start]뒤쳐진 상황: Gray 필터 적용 [cite: 394, 404] |
| **Reg2** | [cite_start]게임 중 상태 변화 (Event) [cite: 394, 405] | [cite_start]사다리(UP): **골든 효과** 적용[cite: 354]. [cite_start]뱀(DOWN/벌칙): **모자이크 효과 및 흔들림** 적용 (2초 지속 후 리셋 로직 설계) [cite: 355, 394, 405] |
| **Reg1** | [cite_start]게임 종료 [cite: 394, 405] | [cite_start]3판 2승제 결과에 따른 승리자/패배자 화면 표시 [cite: 394, 405] |

## 📜 게임 규칙 (Game Rule)

* [cite_start]**게임 진행:** Player 1 → Player 2 순으로 진행[cite: 356].
* [cite_start]**주사위 인식:** 플레이어가 굴린 주사위의 **빨간 눈금**을 카메라가 실시간으로 판독[cite: 352, 360].
* [cite_start]**자동 이동:** 인식된 눈금 수만큼 모니터 상의 말이 자동으로 이동[cite: 353, 361].
* **특수 칸:**
    * [cite_start]**사다리(UP) 칸:** 도착 시 사다리를 타고 더 높은 칸으로 전진하며, Slave는 **골든 효과**를 적용[cite: 354, 362].
    * [cite_start]**뱀(DOWN) 칸:** 도착 시 뱀을 타고 이전 칸으로 후퇴하며, Slave는 **모자이크 및 흔들림 효과**를 적용[cite: 355, 363].
* **승리 조건 (3판 2승제):**
    1.  말이 **Finish Line (40)** 에 도착하면 해당 라운드가 종료되며 스코어 1점을 획득. [cite_start]말이 시작 지점으로 자동 이동[cite: 356, 357, 364, 365].
    2.  [cite_start]총 **2점의 스코어**를 먼저 획득하는 플레이어가 최종 승리[cite: 357, 365].

## 💡 Trouble Shooting 및 해결 (Troubleshooting & Solution)

| 문제 발생 모듈 | 문제 상황 | 원인 분석 | 해결책 및 성과 |
| :--- | :--- | :--- | :--- |
| **Dice Reader** | [cite_start]`VSYNC` 순간 `final_count` 저장과 리셋이 동시 발생하여 유효 값이 0으로 반영되는 경합 발생[cite: 394, 395, 396, 405, 406]. | [cite_start]`VSYNC` 타이밍에서의 비동기적인 저장 및 리셋 로직 충돌[cite: 395, 406]. | [cite_start]**유효 데이터 안전장치 추가:** 빨간 픽셀을 센 후 바로 초기화하지 않고 `pixel count` 값을 업데이트하도록 설정하여 유효성 확보[cite: 397, 408]. |
| **I2C Controller** | [cite_start]첫 번째 이벤트 외 다음 이벤트들이 무시되는 현상[cite: 398, 409]. | [cite_start]I2C 통신이 IDLE 상태로 복귀하는 약 0.2ms에 비해, Game FSM의 이벤트 Trigger 신호 사이 간격(10~20ns)이 너무 짧았음[cite: 398, 399, 409, 410]. | [cite_start]**3ms Delay Counter 도입:** FSM 로직에서 State를 넘어가기 전 **3ms 카운터**를 통해 I2C IDLE 상태를 충분히 확보하여 이벤트 무시 문제 해결[cite: 410, 411, 427, 428]. |
| **Display\_Top** | [cite_start]QVGA 해상도 그대로 저장 시 BRAM Over-utilization (114%) 에러 발생 및 Bitstream 생성 실패[cite: 443, 444, 447, 448]. | [cite_start]FPGA 내장 BRAM 자원 용량 초과[cite: 443]. | [cite_start]**Downscaling 및 RAM 최적화:** 카메라 입력 데이터를 **QQVGA (160x120)로 1/4 축소**하여 BRAM 용량 절감[cite: 443, 446, 447]. [cite_start]`(* rom_style = "distributed" *)` 속성을 사용하여 **Distributed RAM 기능**을 활용[cite: 442, 446]. |

## 🙋‍♂️ 개인별 기여 (Contribution - 진우석)

| 역할 | 기여 내용 | 성과 |
| :--- | :--- | :--- |
| **Back Ground Text 구현** | [cite_start]실시간 게임 정보(SCORE, 시간, 게임 상태, Player Turn 등)를 VGA 화면에 확대·정렬·테두리 처리하여 표시하는 기능 구현[cite: 421]. | [cite_start]픽셀 매핑 오류를 신호 흐름 점검 및 좌표 연산 수정을 통해 해결]. VGA 화면 제어 기술 숙련. |
| **영상 편집** | [cite_start]최종 발표 및 동작 영상 편집 담당[cite: 421]. | - |
| **학습 경험** | [cite_start]VGA 제어 기술과 더불어, 실제 하드웨어 문제(픽셀 매핑 오류)를 팀원들과 분석하고 해결하는 과정에서 협업 및 문제 해결 역량 강화[cite: 423, 424, 440, 441, 445]. | - |

---

