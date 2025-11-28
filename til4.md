> MCU에서 프로그램이 실행된다는 건 **코드가 메모리로 올라오고 → CPU가 레지스터를 통해 명령을 실행 → SFR을 통해 하드웨어를 제어**한다는 뜻

**각 항목은 이 3개의 축에 귀속된다:**

1. **코드 → 바이너리 → 메모리 배치 (컴파일·ELF·부팅)**
2. **실행기(Execution) → CPU 아키텍처(레지스터·PSR·파이프라인)**
3. **HW 제어 → Peripheral(SFR, Interrupt, Timer, DMA, GPIO)**

이제 각 번호를 이 3개의 축 위에서 연결하자.

---

# 1. 컴파일 4단계 = CPU가 읽을 수 있는 **Instruction**을 만드는 과정

> MCU는 **기계어(Instruction)** 외에는 아무것도 이해 못 한다.

* 전처리: include 확장, macro 치환
* 컴파일: C → Assembly
* 어셈블: Assembly → Machine Code
* 링크: 여러 .o → ELF

**이 기계어가 이후 .text 섹션을 구성하여 Flash에 올라간다**

**(3 MCU 부팅 / 2 메모리 구조 / 4 Instruction)과 직접 연결**

---

# 2. ELF = 코드가 메모리의 어디에 놓일지 정의

> ELF는 단순 “실행 파일”이 아니라 **메모리 배치를 정의한 설계도**

예:

* `.text` → 명령어
* `.rodata` → 읽기 전용 상수
* `.data` → 초기값 있는 전역
* `.bss` → 초기값 없는 전역 (0 초기화)
* heap / stack → 런타임 동적 공간

🔗 **부팅 시 Startup Code가 `.data`,` .bss`를 RAM에 올리는 이유가 여기 있음**

➡️ (3 MCU 부팅 / 16 storage class / 2-1 MCU vs OS)와 연결

---

# 🚀 3. MCU 부팅 = ELF를 **런타임 메모리**로 전개하는 과정

MCU(특히 Cortex-M)는 OS 없이 직접 시작.

1. Reset Vector Table 로드

   * 첫 Word = 초기 SP
   * 두번째 Word = Reset_Handler 주소
2. Startup Code 실행

   * `.data` → Flash → RAM 복사
   * `.bss` = RAM 0 초기화
3. `main()` 호출

즉:

> **ELF 설계도 → Startup이 메모리를 구성 → main()에서 실행 시작**

🔗 (2 ELF 구조 / 5 Register / 8 SFR / 16 volatile)

---

# 🧬 4. Instruction vs Assembly = CPU 실행 단위

* Assembly: 사람이 읽는 표현
* Instruction: CPU 파이프라인이 실제 실행

예:

```
ADD R0, R1, R2
```

이 Instruction은:

1. Fetch (PC)
2. Decode (레지스터 해석)
3. Execute(ALU)
4. Memory(op 필요시)
5. Write-back(R0에 결과)

🔗 (7 Pipeline / 5 PSR / 6 ARM 명령어)와 연결

---

# 🔩 5. Cortex-M Register = Execution의 핵심

MCU는 RAM을 직접 연산하지 못함
➡️ **레지스터에 Load → 연산 → Store**

| 레지스터    | 의미        |
| ------- | --------- |
| r0~r12  | 일반 연산     |
| r13(SP) | 스택        |
| r14(LR) | return 주소 |
| r15(PC) | 다음 명령 주소  |

즉:

```
RAM → CPU 연산 → 결과 저장
```

은 반드시

```
RAM → Rn → 연산 → Rm → 메모리
```

형태여야 함.

🔗 (6 Load/Store / 7 Pipeline / 12 GPIO 직접 접근)

---

# 🧾 6. ARM 명령어 패턴 = CPU가 HW를 다루는 언어

### 핵심: Load/Store 구조

> CPU는 메모리에 대해 “연산 불가”, 오직 레지스터 대상.

* `LDR` : Mem → reg
* `STR` : reg → Mem

➡️ SFR도 메모리처럼 보일 뿐, 결국 Load/Store 대상

```
LDR R0, [GPIOA_ODR]
STR R1, [GPIOA_BSRR]
```

🔗 (8 SFR / 12 GPIO 설정 / 13 NVIC / 14 Timer)

---

# 📦 7. Pipeline = Instruction 실행의 성능

MCU는 단일 명령을 “겹쳐서” 처리

```
IF → ID → EXE → MEM → WB
```

즉 **현재 명령이 MEM일 때, 다음 명령은 이미 IF 상태**
속도 ↑

HW 개발에서 중요한 이유:

* **branch(조건문) → 파이프라인 깨짐 → 성능 저하**
* **인터럽트 진입 시 → 파이프라인 flush**

🔗 (13 Interrupt / 9 MPU / 5 Register)

---

# 🧱 8. SFR = Peripheral 제어의 핵심 메모리 영역

> MCU의 모든 하드웨어는 **레지스터 집합(SFR)**

주소를 가지는 **메모리 맵핑된 HW 제어 블록**

예:

```
GPIOA->MODER
RCC->AHB1ENR
TIM2->CCR1
USART1->DR
```

결론:

> **Peripheral 제어 = 특수 주소(레지스터)에 값 쓰기**

🔗 (12 GPIO / 14 Timer / 15 DMA / 13 Interrupt)

---

# 🔐 9. MPU vs MMU = 접근 권한

* **MPU**: 보호 (영역 단위)
* **MMU**: 주소 변환 + 보호 (페이지 단위)

**STM32 = MPU only**

왜 중요?

➡️ **RTOS task를 Non-privileged로 돌려 오동작 격리**

🔗 (5 Register / 12 GPIO / system fault)

---

# 💾 10. MCU 메모리 종류 = 동작 가치

* SRAM: 빠른 휘발성
* Flash: 느리지만 영구 저장
* EEPROM: 설정값 저장

> MCU = Flash에 프로그램 저장 → 실행 시 RAM 사용

🔗 (3 Startup / 2 ELF / 16 const)

---

# 🧠 11. Bus = CPU ↔ Peripheral 연결 방법

* Address bus → “어디”
* Data bus → “무엇”
* Control bus → “언제/방향”

**SFR 접근 = Address bus + Data bus**

I2C / SPI / CAN 은 **통신 프로토콜**
= CPU 내부 버스와 완전 다름

🔗 (12 GPIO / 14 Timer / 15 DMA)

---

# 🔌 12. GPIO = 가장 단순한 Peripheral 제어 실전

> 코드 → SFR 접근 → 물리 핀 변화

과정:

1. RCC clock enable
2. MODER 설정
3. IDR/ODR/BSRR 사용

핵심:

* Schmitt trigger: Digital input 안정화
* Pull-up/down: Floating 방지
* Push-pull vs open-drain

🔗 (8 SFR / 11 Bus / 5 Register)

---

# 🔔 13. Interrupt = CPU가 이벤트에 반응하는 방식

순서:

1. External/Peripheral event 발생
2. EXTI/NVIC에서 Pending
3. CPU 현재 실행 중단
4. Context push(stack)
5. ISR 실행

핵심:

* **Handler mode = Privileged**
* NVIC 우선순위 / 마스킹

🔗 (3 Startup / 5 Register / 7 Pipeline / 15 DMA)

---

# ⏱️ 14. Timer = 시간 생성 하드웨어

> CPU가 루프 돌며 delay하는 게 아니라 **HW counter**

* Prescaler → tick 생성
* ARR → 주기
* CCR → duty(PWM)

산업에서 가장 많이 쓰는 Peripheral.

🔗 (DMA / Interrupt / GPIO)

---

# 🚛 15. DMA = CPU 없이 데이터 전송

> Peripheral ↔ Memory 자체 씀

원리:

* DMA REQ
* BUS REQ
* BUS ACK
* DMA ACK

추론:

> Timer → ADC → DMA → RAM → CPU post-processing

🔗 (인터럽트 부하 감소 / 실시간성 ↑)

---

# 🧾 16. Storage Class & Type Qualifier = 코드 ↔ 메모리 직접 연결

* `static` → .data/.bss (부팅 시 초기화)
* `const` → .rodata(Flash)
* `volatile` → 최적화 금지 (SFR, flag)

Embedded에서 가장 중요한 키워드:

```
volatile uint32_t GPIOA_ODR;
```

없는 코드:

* 컴파일러는 “변수 안 변하네?” → 캐싱 → HW 변화 못 읽음 → 버그

🔗 (8 SFR / 5 Register / 13 Interrupt)

---

# 📌 전체를 한 문장으로 연결

> **C 코드는 ELF가 되어 Flash에 저장되고, Startup이 RAM 메모리 모델을 구성한다.
> CPU는 파이프라인/레지스터/PSR 기반으로 Instruction을 실행하며 Memory-mapped SFR을 통해 GPIO·Timer·Interrupt·DMA를 제어한다.**

---

# 📎 실제 MCU 한 사이클 예시

1. main: 버튼 상태 읽기
2. CPU: PC→IF
3. LDR: `GPIOC_IDR` (SFR) → r0
4. CMP: r0 vs 0
5. BNE: branch → pipeline flush
6. STR: LED_ON 값 → `GPIOA_BSRR`
7. 다음 instruction fetch

이 사이의 모든 요소가 상호 연결되어 동작함:

* **레지스터로 load/store**
* **파이프라인 branch**
* **SFR 주소 접근**
* **저전력 peripheral 동작**

---

# 🔥 결론

👉 **각 항목은 따로 존재하는 지식이 아니라, MCU 실행의 한 흐름을 구성하는 링크다.**

* **컴파일 → ELF → 부팅 → 메모리 구성 → 명령 실행 → HW 접근**
* 이 위에

  * **Interrupt**
  * **Timer**
  * **DMA**
  * **GPIO**
    가 모두 얹힌다.

---

원하면 다음 단계로:

📘 **“부팅 직후 Reset_Handler → main() → 첫 GPIO 토글”까지 실제 어셈블리 라인을 따라 설명**
또는
📗 **USART 송수신 + NVIC + DMA end-to-end 흐름**
도 만들어줄 수 있어.
