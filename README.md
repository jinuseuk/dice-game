# 🎲 DICE GAME RTL IMPLEMENTATION (주사위 게임 RTL 구현)

## 🎯 프로젝트 개요

본 프로젝트는 SystemVerilog 언어를 사용하여 주사위 게임의 핵심 로직을 RTL(Register Transfer Level)로 설계하고 구현한 것입니다. 단순한 게임 규칙을 디지털 논리 회로로 변환하여, 하드웨어 시뮬레이션 및 실제 FPGA/ASIC 환경에서 동작할 수 있도록 검증하는 것을 목표로 합니다.

  * **주요 목표:** Master-Slave 아키텍처를 활용하여 주사위 굴림(랜덤 값 생성), 점수 계산, 그리고 게임 상태 관리를 효율적으로 분리하여 구현합니다.
  * **사용 언어:** SystemVerilog (RTL 설계), Tcl (시뮬레이션 및 테스트 자동화 스크립트)

## ⚙️ 아키텍처 및 구성

프로젝트는 주요 기능에 따라 **Master**와 **Slave** 두 개의 모듈로 나뉘어 있습니다. 이는 복잡한 시스템을 모듈화하고 계층적으로 관리하기 위한 표준적인 하드웨어 설계 방식입니다.

### 📁 Master 모듈 (`Master` 폴더)

Master 모듈은 게임의 **중앙 제어 장치(Central Control Unit)** 역할을 담당합니다.

| 모듈 역할 | 상세 내용 |
| :--- | :--- |
| **게임 상태 관리** | 게임 시작, 주사위 굴림 대기, 점수 계산, 게임 종료 등의 상태를 관리하는 FSM(Finite State Machine)을 포함합니다. |
| **입력 처리** | 사용자의 'Roll' 버튼 입력 또는 시뮬레이션 환경의 제어 신호를 처리합니다. |
| **점수 계산** | Slave 모듈로부터 받은 주사위 결과를 바탕으로 현재 플레이어의 점수를 누적하거나 규칙에 따라 점수 초기화 로직을 수행합니다. |
| **출력 제어** | 현재 점수, 라운드 정보, 게임 결과 등을 외부 디스플레이 장치(7-Segment 또는 LED)로 전달하기 위한 신호를 생성합니다. |

### 📁 Slave 모듈 (`Slave` 폴더)

Slave 모듈은 Master의 명령에 따라 특정 하위 기능을 수행하는 모듈입니다. 주사위 게임의 경우, 주사위 굴림 자체의 기능을 분리했을 가능성이 높습니다.

| 모듈 역할 | 상세 내용 |
| :--- | :--- |
| **주사위 값 생성** | Master의 요청(Roll 신호)에 따라 1부터 6까지의 임의의 값을 생성합니다. 이는 **LFSR(Linear Feedback Shift Register)** 기반의 Pseudo-Random Number Generator (PRNG)로 구현되었을 것으로 추정됩니다. |
| **결과 전송** | 생성된 주사위 값을 Master 모듈로 전송합니다. |
| **디스플레이 드라이버** (선택적) | 주사위 눈금에 해당하는 그래픽이나 숫자 정보를 디스플레이하기 위한 저수준(Low-level) 로직이 포함될 수 있습니다. |

## 🛠️ 개발 환경 및 기술 스택

| 분류 | 내용 | 비고 |
| :--- | :--- | :--- |
| **HDL 언어** | SystemVerilog (IEEE 1800-2012) | RTL 설계의 표준 언어 |
| **스크립트 언어** | Tcl (Tool Command Language) | 시뮬레이션 컴파일, 실행, 테스트 자동화에 사용됨 |
| **시뮬레이터** | (추정) ModelSim/QuestaSim 또는 VCS | Tcl 스크립트는 시뮬레이션 툴 구동에 주로 활용됨 |
| **합성 툴** | (추정) Synopsys Design Compiler 또는 Vivado/Quartus | 최종 하드웨어 구현을 위한 합성 및 P\&R 툴 |

## 🚀 프로젝트 실행 및 시뮬레이션

본 프로젝트는 하드웨어 시뮬레이터 환경에서 실행 및 검증됩니다.

### 1\. 환경 설정

1.  프로젝트에 사용된 SystemVerilog 모듈 (`.sv` 파일)을 시뮬레이터에 맞게 컴파일합니다.
2.  `Tcl` 스크립트를 사용하여 테스트벤치(`*_tb.sv`)를 로드하고 시뮬레이션을 위한 초기 설정을 완료합니다.

### 2\. 시뮬레이션 실행 (Tcl 스크립트 사용)

```bash
# 예시: Tcl 셸에서 스크립트 실행
# Tcl 파일을 통해 컴파일 및 시뮬레이션을 자동화합니다.
vsim -c -do run_simulation.tcl 
```

### 3\. 검증

테스트벤치(`*_tb.sv`)는 다음과 같은 시나리오를 검증합니다.

  * **Master FSM 검증:** 모든 상태(Idle, Roll, Calculate, End)가 정확히 천이하는지 확인합니다.
  * **Dice Logic 검증:** Slave 모듈이 Master의 요청에 따라 1\~6 사이의 유효한 값을 생성하는지 확인합니다.
  * **Score Logic 검증:** 특정 주사위 결과(예: 1이 나오는 경우, 더블 등)에 따라 점수 계산이 정확하게 이루어지는지 검증합니다.

## 📝 TODO List (향후 개선 방향)

  * [ ] **FPGA Target Integration:** Xilinx 또는 Intel FPGA 개발 환경에 맞춰 핀 제약 조건 및 클럭 설정을 추가합니다.
  * [ ] **Interface Enhancement:** AXI-Lite 또는 APB와 같은 표준 버스 인터페이스를 Master 모듈에 적용하여 호스트 CPU와의 연결성을 확보합니다.
  * [ ] **Advanced Game Rules:** 두 플레이어 간의 경쟁, 베팅 등 더 복잡한 게임 규칙을 FSM에 추가합니다.
  * [ ] **Formal Verification:** 시뮬레이션 외에 Assertion-Based Verification (ABV) 또는 Formal Verification 환경을 구축하여 설계의 무결성을 검증합니다.
