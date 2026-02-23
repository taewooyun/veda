![Image](https://e2e.ti.com/cfs-file/__key/communityserver-discussions-components-files/791/tms.PNG)

![Image](https://cdn.shopify.com/s/files/1/0615/5911/1885/files/7_c3428ffc-af30-474b-bf55-a1d3c07e5820.png?v=1757659099)

![Image](https://www.cdebyte.com/upload/image/20250915/6389355159289149367469970.png)

---

## 1. **I2S (Inter-IC Sound)**

- **동기식 직렬 통신 (오디오 전용)**
- **Master–Slave 구조**
- **클럭은 Master가 생성**
- **디지털 오디오 전송용 인터페이스**

SPI 기반 확장 구조이지만, **음성/음악 전용 규격**

---

기존 UART처럼 start/stop bit 없음

→ **BCLK + LRCK 기준으로 동기화**

좌/우 채널을 구분해서 연속 스트리밍하는 구조

---

## 2. I2S 신호선

| 신호 | 의미 | 설명 |
| --- | --- | --- |
| **BCLK** | Bit Clock | 비트 단위 클럭 |
| **LRCK (WS)** | Word Select | 좌/우 채널 구분 |
| **SD** | Serial Data | 오디오 데이터 |
| **MCLK** | Master Clock | 기준 클럭 (옵션) |

### 주요 특징

### BCLK

- 데이터 비트 동기용 클럭
- 1클럭 = 1비트

### LRCK (WS)

- Left / Right 채널 선택
- `0 → Left`
- `1 → Right`

### SD

- 실제 PCM 데이터
- MSB부터 전송

### MCLK

- 고음질용 기준 클럭
- 대부분 모듈은 없어도 됨

---

## 3. 동작 개념

```
LRCK 전환
BCLK 토글
SD 비트 전송
좌/우 반복
```

---

- 반이중 구조 (오디오 스트리밍용)
- 프레임 단위 전송
- 연속 데이터 흐름

**→ 한 프레임 = Left + Right 샘플**

---

## 4. I2S 데이터 타이밍

### Philips I2S 표준 기준

| 항목 | 내용 |
| --- | --- |
| 비트 순서 | MSB First |
| 시작 위치 | LRCK 전환 후 1bit 뒤 |
| 패딩 | 남는 비트는 0 |

> **LRCK 바뀌고 → 한 비트 쉬고 → 데이터 시작**
> 

이게 SPI랑 가장 큰 차이점

---

## 5. CubeIDE 설정 순서

### 1) SPI → I2S 모드 변경

- SPI2 / SPI3 선택
- Mode → **I2S**

---

### 2) I2S 기본 설정

| 항목 | 설정값 |
| --- | --- |
| Mode | Master Transmit |
| Standard | Philips I2S |
| Data Format | 16-bit |
| Audio Freq | 44.1kHz |
| MCLK | Disable |

---

### 3) GPIO

| 핀 | 설정 |
| --- | --- |
| BCLK | AF |
| LRCK | AF |
| SD | AF |

---

### +) DMA 설정

- SPIx_TX → DMA Enable
- Mode: Circular / Normal
- Priority: High

📌 DMA 없으면 실시간 처리 불가

---

## 6. 코드 흐름 (HAL 기준)

### 기본 전송

```c
HAL_I2S_Transmit(&hi2s2, audio_buf, BUF_SIZE, HAL_MAX_DELAY);
```

→ Blocking 방식 (실무 거의 안 씀)

---

### DMA 전송

```c
HAL_I2S_Transmit_DMA(&hi2s2, audio_buf, BUF_SIZE);
```

→ 보편적인 사용법

---

### 테스트용 버퍼

```c
#define BUF_SIZE 128

int16_t audio_buf[BUF_SIZE];

void make_test_data(void)
{
    for(int i=0;i<BUF_SIZE;i++)
    {
        audio_buf[i] = (i%32)*1000;
    }
}
```

---

## 7. 주요 HAL 함수

### HAL_I2S_Transmit_DMA

```c
HAL_StatusTypeDef HAL_I2S_Transmit_DMA(
    I2S_HandleTypeDef *hi2s,
    uint16_t *pData,
    uint16_t Size
);
```

| 인자 | 의미 |
| --- | --- |
| hi2s | I2S 핸들 |
| pData | PCM 버퍼 |
| Size | 샘플 개수 |

---

### 반환값

| 값 | 의미 |
| --- | --- |
| HAL_OK | 정상 |
| HAL_BUSY | DMA 사용중 |
| HAL_ERROR | 오류 |
| HAL_TIMEOUT | 시간초과 |

---

## 8. I2S 사용 패턴

### ▶ 출력(DAC/앰프)

```
MCU → I2S → DAC/AMP → Speaker
```

```c
사인파/WAV → 버퍼 → DMA → 출력
```

---

### ▶ 입력(MIC)

```
MIC → I2S → MCU → 처리
```

```c
DMA 수신 → 필터 → 저장
```

---

### ▶ 더블 버퍼 구조 (실무 핵심)

```
[BUF A] 전송중
[BUF B] 준비중
→ 교대
```

언더런 방지용 필수 구조

---

## 9. SPI vs I2S 비교

| 항목 | SPI | I2S |
| --- | --- | --- |
| 용도 | 범용 | 오디오 전용 |
| 채널 | 자유 | L/R 고정 |
| 프레임 | 자유 | 고정 |
| 타이밍 | 단순 | 엄격 |
| DMA | 선택 | 필수 |

---

## 10. 자주 터지는 문제

### 소리 안 남

- I2S 아닌 SPI 모드
- 핀 매핑 오류

### 잡음

- 샘플레이트 불일치
- DMA underrun

### 좌우 이상

- LRCK 극성 문제

### 끊김

- 버퍼 크기 부족

---