# 🎲 FPGA Real-Time Dice Board Game (I2C & VGA 기반)

## 🌟 프로젝트 개요 (Overview)

본 프로젝트는 OV7670 카메라 모듈을 통해 실시간으로 주사위 눈금을 인식하고, Xilinx Basys 3 FPGA 보드에서 게임 로직을 구현하여 VGA 모니터에 출력하는 디지털 하드웨어 설계 프로젝트입니다.

특히, I2C 통신 프로토콜**을 활용한 Master-Slave 아키텍처를 적용하여 중앙 제어 로직과 플레이어 상태 제어 로직을 분리 및 연동함으로써, 2인용 보드 게임의 실시간 상태 변화 및 시각적 효과를 구현했습니다.

* 핵심 목표: 실시간 영상 처리를 통한 입력(주사위 눈금) 인식, 하드웨어 통신 프로토콜(I2C) 설계 및 연동, VGA 디스플레이 제어.
* 주요 기능: 주사위 눈금 인식, 자동 말 이동, 사다리/뱀 효과, 플레이어 상태에 따른 실시간 영상 필터 적용.

## ⚙️ 개발 환경 및 기술 스택 (Development Environment)

| 분류 | 내용 | 비고 |
| :--- | :--- | :--- |
| HDL 언어| SystemVerilog (IEEE 1800-2017) | RTL (Register Transfer Level) 설계 언어 |
| EDA Tool | AMD Xilinx Vivado Design Suite 2020.2  | 합성, 구현 및 비트스트림 생성 |
| 개발 보드| Digilent Basys 3 (100 MHz on-board clock)| Xilinx Artix-7 FPGA 기반 |
| 카메라 모듈 | OV7670 | QQVGA (160x120) 해상도, RGB565 포맷 |
| 인터페이스 | I2C Protocol (Master–Slave 게임 데이터 전송) | Master(게임 로직) - Slave(플레이어 1, 2) 통신 |
| |SCCB Protocol (OV7670 레지스터 제어)  | 카메라 초기 설정 및 제어 |

## 📐 시스템 아키텍처 (System Architecture)

]프로젝트는 중앙에서 게임 로직을 제어하는 Master Board와 각 플레이어의 상태 및 시각 효과를 담당하는 Slave 모듈로 구성됩니다.

****

### 1. Game Logic Module (I2C Master)

게임의 핵심 로직을 담당하며, Master Board의 버튼 입력과 주사위 인식 결과를 기반으로 동작합니다. 4개의 FSM(Finite State Machine) 모듈로 구성되어 유기적인 게임 흐름을 제어합니다.

| 모듈 | 역할 | 상세 로직 |
| :--- | :--- | :--- |
| GAME\_STATE | 게임의 전역 상태 관리 |사용자 버튼 입력, 3판 2승제, Time Over 신호 기반 게임 진행 상태 제어 |
| VICTORY\_TRACKER | ]3판 2승제 승부 판정 |최종 승리 조건(총 2점 획득) 확인 및 결과 계산  |
| PLAY\_GAME | 게임 진행 및 위치 계산 | 주사위 값에 따른 플레이어 위치 업데이트, 사다리/벌칙 구간 위치 보정, 승부 비교  |
| TIMER | 게임 시간 관리 | 10분 카운트다운 관리 (Start, Restart, Final 신호 제어)  |
| I2C Master Controller | Slave 통신 제어  | 게임 이벤트(이동, 사다리, 종료) 발생 시, Player 1 → Player 2 순서로 I2C 통신을 통해 Slave 장치에게 정보 전송 |
| VGA Display | 게임 정보 출력  |SCORE, TIMER, 현재 게임 상태, Player Turn, 말의 위치 등 출력  |

### 2. Dice Detector & VGA Display Module

카메라 영상을 실시간으로 처리하여 주사위 눈금을 인식하고 모니터에 출력하는 모듈입니다.

| 주요 기능 | 상세 로직 |
| :--- | :--- |
| 실시간 영상 출력 | OV7670 데이터 캡처 후 FrameBuffer(RAM)에 저장. VGA 타이밍에 맞춰 데이터 읽기. |
| `red_check` | 주사위 눈금을 판별하기 위해, 픽셀의 Red > Green 및 Red > Blue 조건으로 빨간색 픽셀만 출력하고 나머지는 검은색으로 Masking 처리. |
| `Dice_Reader`| 한 프레임 동안 빨간색 픽셀 수를 카운트하여 주사위 눈금을 실시간으로 판독. 최종 카운트 값을 125 단위로 조건을 나누어 눈금(1~6)을 판별]. |

### 3. I2C Slave Module (Game Player Slave)

Master에서 I2C로 전송된 신호를 받아 해당 플레이어의 화면에 실시간 시각 필터를 적용하는 모듈입니다.

| Register | 기능 (Master Write) | Slave 적용 효과 |
| :--- | :--- | :--- |
| Reg0 | 현재 게임 진행 상태 | 앞선 상황: Normal 필터. 뒤쳐진 상황: Gray 필터 적용  |
| Reg2 | 게임 중 상태 변화 (Event) | 사다리(UP): 골든 효과 적용].뱀(DOWN/벌칙): 모자이크 효과 및 흔들림 적용 (2초 지속 후 리셋 로직 설계) |
| Reg1 | 게임 종료  |3판 2승제 결과에 따른 승리자/패배자 화면 표시 |

## 📜 게임 규칙 (Game Rule)

* 게임 진행:** Player 1 → Player 2 순으로 진행.
* 주사위 인식:플레이어가 굴린 주사위의 빨간 눈금을 카메라가 실시간으로 판독.
* 자동 이동:인식된 눈금 수만큼 모니터 상의 말이 자동으로 이동.
* 특수 칸:
   사다리(UP) 칸: 도착 시 사다리를 타고 더 높은 칸으로 전진하며, Slave는 골든 효과를 적용.
   뱀(DOWN) 칸: 도착 시 뱀을 타고 이전 칸으로 후퇴하며, Slave는 모자이크 및 흔들림 효과를 적용.
* 승리 조건 (3판 2승제):
    1.  말이 Finish Line (40) 에 도착하면 해당 라운드가 종료되며 스코어 1점을 획득. 말이 시작 지점으로 자동 이동.
    2.  총 2점의 스코어를 먼저 획득하는 플레이어가 최종 승리.

## 💡 Trouble Shooting 및 해결 (Troubleshooting & Solution)

| 문제 발생 모듈 | 문제 상황 | 원인 분석 | 해결책 및 성과 |
| :--- | :--- | :--- | :--- |
| Dice Reader | `VSYNC` 순간 `final_count` 저장과 리셋이 동시 발생하여 유효 값이 0으로 반영되는 경합 발생. | `VSYNC` 타이밍에서의 비동기적인 저장 및 리셋 로직 충돌. | 유효 데이터 안전장치 추가:빨간 픽셀을 센 후 바로 초기화하지 않고 `pixel count` 값을 업데이트하도록 설정하여 유효성 확보. |
| I2C Controller | 첫 번째 이벤트 외 다음 이벤트들이 무시되는 현상. | I2C 통신이 IDLE 상태로 복귀하는 약 0.2ms에 비해, Game FSM의 이벤트 Trigger 신호 사이 간격(10~20ns)이 너무 짧았음. | 3ms Delay Counter 도입: FSM 로직에서 State를 넘어가기 전 3ms 카운터를 통해 I2C IDLE 상태를 충분히 확보하여 이벤트 무시 문제 해결. |
| Display\_Top |QVGA 해상도 그대로 저장 시 BRAM Over-utilization (114%) 에러 발생 및 Bitstream 생성 실패. | FPGA 내장 BRAM 자원 용량 초과. | Downscaling 및 RAM 최적화:카메라 입력 데이터를 QQVGA (160x120)로 1/4 축소하여 BRAM 용량 절감[. (* rom_style = "distributed")` 속성을 사용하여 Distributed RAM 기능을 활용. |

## 🙋‍♂️ 개인별 기여 (Contribution - 진우석)

| 역할 | 기여 내용 | 성과 |
| :--- | :--- | :--- |
| Back Ground Text 구현 | 실시간 게임 정보(SCORE, 시간, 게임 상태, Player Turn 등)를 VGA 화면에 확대·정렬·테두리 처리하여 표시하는 기능 구현]. | 픽셀 매핑 오류를 신호 흐름 점검 및 좌표 연산 수정을 통해 해결]. VGA 화면 제어 기술 숙련. |
| 영상 편집 | 최종 발표 및 동작 영상 편집 담당. | - |
| 학습 경험 | VGA 제어 기술과 더불어, 실제 하드웨어 문제(픽셀 매핑 오류)를 팀원들과 분석하고 해결하는 과정에서 협업 및 문제 해결 역량 강화]. | - |

---

