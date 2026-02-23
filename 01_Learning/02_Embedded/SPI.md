## 1. **SPI (Serial Peripheral Interface)**

- **동기식 직렬 통신**
- **Master–Slave 구조**
- **클럭은** **Master가 생성**

![image.png](attachment:b6ca8eb7-07ef-4537-9110-02e22de35604:image.png)

기존 시리얼 통신에서 사용하던 start bit와 stop bit를 사용하지 않고 클럭 선을 두어 라이징(low에서 HIGH로)신호, 또는 폴링(HIGH에서 LOW로)신호에 맞춰 동기식으로 데이터를 읽어 온다.

---

## 2. SPI 신호선

| 신호 | 의미 | 설명 |
| --- | --- | --- |
| **SCK** | Serial Clock | 클럭 (Master → Slave) |
| **MOSI** | Master Out Slave In | Master → Slave 데이터 |
| **MISO** | Master In Slave Out | Slave → Master 데이터 |
| **CS / NSS** | Chip Select | Slave 선택 (보통 Active Low) |
- SCK
    - 이 클럭의 엣지(상승/하강)에 맞춰 데이터 샘플링
    - CPOL / CPHA 설정에 따라 언제 읽고 쓰는지가 달라짐
        - CPOL : 클럭이 idle(대기) 상태일 때의 레벨
            - 0 → 클럭이 Low 상태로 쉬고 있음
            - 1 → 클럭이 High 상태로 쉬고 있음
        - CPHA
            - 클럭의 어느 엣지에서 데이터를 읽을지 정함
            - 0 → 첫 번째 엣지에서 샘플링
            - 1 → 두 번째 엣지에서 샘플링
            - CPOL 기준으로 처음 변하는 엣지
- MOSI
    - Master가 Slave로 보내는 데이터
- MISO
    - Slave가 Master에게 보내는 데이터
    - 센서 값, 상태 레지스터 값 등
- CS / NSS
    - Slave 선택 신호

---

## 3. 동작 개념 (한 줄 요약)

```
CS ↓
SCK 토글
MOSI/MISO 동시에 한 비트씩 교환
CS ↑
```

- **풀듀플렉스** (보내면서 동시에 받음)
- 클럭 1번 = 데이터 1비트

---

## 4. SPI 모드 (CPOL / CPHA)

| Mode | CPOL | CPHA | Idle Clock | 데이터 샘플링 | 비고 |
| --- | --- | --- | --- | --- | --- |
| 0 | 0 | 0 | Low | Rising edge | 가장 흔함 |
| 1 | 0 | 1 | Low | Falling edge |  |
| 2 | 1 | 0 | High | Falling edge |  |
| 3 | 1 | 1 | High | Rising edge |  |

---

## 5. CubeIDE 설정 순서

### 1. SPI 활성화

- `SPI1` or `SPI2`
- **Mode**: Full-Duplex Master
- **Hardware NSS** : disable
    
    → **Software NSS** (CS는 GPIO로 직접 제어)
    

### 2. 파라미터

- Data Size: **8-bit**
- First Bit: **MSB**
- Prescaler: **64 or 128** (처음엔 느리게)
- Clock Polarity / Phase: **Slave 맞춰서**

### 3. GPIO

- SCK / MOSI / MISO → **Alternate Function**
- CS → **GPIO Output**

---

## 6. 코드 흐름 (HAL 기준)

### 기본 송신

```c
uint8_t tx = 0x9A;
HAL_GPIO_WritePin(CS_GPIO_Port, CS_Pin, GPIO_PIN_RESET);
HAL_SPI_Transmit(&hspi1, &tx, 1, HAL_MAX_DELAY);
HAL_GPIO_WritePin(CS_GPIO_Port, CS_Pin, GPIO_PIN_SET);
```

### 송수신

```c
uint8_t tx = 0x00;
uint8_t rx = 0x00;

HAL_GPIO_WritePin(CS_GPIO_Port, CS_Pin, GPIO_PIN_RESET);
HAL_SPI_TransmitReceive(&hspi1, &tx, &rx, 1, HAL_MAX_DELAY);
HAL_GPIO_WritePin(CS_GPIO_Port, CS_Pin, GPIO_PIN_SET);
```

**SPI는 항상 클럭을 줘야 받는다**

→ 그래서 “읽기”도 더미 데이터 전송 필요

---

## 7. 기본 예제

```c
int _write(int file, char *ptr, int len)
{
    HAL_UART_Transmit(&huart2, (uint8_t*)ptr, len, HAL_MAX_DELAY);
    return len;
}

uint8_t tx_data = 0xA5;
uint8_t rx_data = 0x00;
int main(void)
{
	...
  while (1)
  {
		rx_data = 0x00;
	
		HAL_SPI_TransmitReceive(
			&hspi2,
			&tx_data,
			&rx_data,
			1,
			HAL_MAX_DELAY
		);
	
		printf("rx_data: %d\r\n", rx_data);
		HAL_Delay(1000);
  }
  ...
}
```

### HAL_SPI_TransmitReceive 함수

```c
HAL_StatusTypeDef HAL_SPI_TransmitReceive(
    SPI_HandleTypeDef *hspi,         // 사용할 SPI 포트 정보
    uint8_t *pTxData,                // 송신 데이터 버퍼
    uint8_t *pRxData,                // 수신 데이터 버퍼
    uint16_t Size,                   // 송수신 데이터 크기 (Byte)
    uint32_t Timeout                 // 대기시간 (ms), 초과 시 HAL_TIMEOUT 반환
);
```

**반환 값 (HAL_StatusTypeDef )**

| 값 | 의미 |
| --- | --- |
| `HAL_OK` | 정상 통신 |
| `HAL_BUSY` | SPI가 이미 사용 중 |
| `HAL_TIMEOUT` | 시간 초과 |
| `HAL_ERROR` | 기타 오류 |

## 8. SPI 사용 패턴

### 레지스터 읽기 패턴

```c
CS ↓
WRITE: (READ_CMD | addr)
READ: dummy → data
CS ↑
```

### 레지스터 쓰기 패턴

```c
CS ↓
WRITE: (WRITE_CMD | addr)
WRITE: data
CS ↑
```

---