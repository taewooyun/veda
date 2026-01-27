좋아 👍 그 선택 **아주 RTOS답고 정공법**이야.
그럼 **CMSIS-RTOS2 + ST-LINK VCP** 기준으로
👉 *과하지 않지만 “구조가 보이는” Logger Task*를 딱 만들어보자.

아래는 **바로 써먹는 최소 정석**이야.

---

## ✅ 목표 구조 (한 줄 요약)

```text
[Task들] ──(문자열)──> MessageQueue ──> LoggerTask ──> UART(ST-LINK VCP)
```

* Task들은 **UART를 전혀 모름**
* LoggerTask만 **UART 독점**
* 지금은 blocking UART, 나중에 DMA로 바로 교체 가능

---

## 1️⃣ 로그 메시지 타입 정의

```c
// log.h
#pragma once
#include "cmsis_os2.h"

#define LOG_BUF_SIZE 128
#define LOG_QUEUE_LEN 16

typedef struct {
    uint16_t len;
    char buf[LOG_BUF_SIZE];
} log_msg_t;

extern osMessageQueueId_t logQueue;

void log_init(void);
void log_printf(const char *fmt, ...);
```

---

## 2️⃣ 로그 큐 + printf 대체 함수

```c
// log.c
#include "log.h"
#include <stdio.h>
#include <stdarg.h>
#include <string.h>

osMessageQueueId_t logQueue;

void log_init(void)
{
    logQueue = osMessageQueueNew(
        LOG_QUEUE_LEN,
        sizeof(log_msg_t),
        NULL
    );
}
```

```c
void log_printf(const char *fmt, ...)
{
    log_msg_t msg;
    va_list ap;

    va_start(ap, fmt);
    int n = vsnprintf(msg.buf, sizeof(msg.buf), fmt, ap);
    va_end(ap);

    if (n <= 0) return;

    msg.len = (n >= LOG_BUF_SIZE) ? (LOG_BUF_SIZE - 1) : (uint16_t)n;

    // block 안 함 → RTOS 안정성 우선
    (void)osMessageQueuePut(logQueue, &msg, 0, 0);
}
```

📌 여기서 중요한 포인트

* `printf` ❌
* `vsnprintf`만 사용 (포맷만 담당)
* 큐 꽉 차면 **로그 버림** → 실시간성 우선

---

## 3️⃣ Logger Task 구현 (UART 독점)

```c
// logger_task.c
#include "log.h"
#include "usart.h"   // huartX
#include <string.h>

void LoggerTask(void *arg)
{
    log_msg_t msg;

    for (;;)
    {
        if (osMessageQueueGet(logQueue, &msg, NULL, osWaitForever) == osOK)
        {
            HAL_UART_Transmit(
                &huart2,                       // ST-LINK VCP
                (uint8_t *)msg.buf,
                msg.len,
                HAL_MAX_DELAY                  // blocking OK (격리됨)
            );
        }
    }
}
```

📌 **여기만 블로킹**
→ 다른 Task는 전혀 영향 없음

---

## 4️⃣ RTOS 초기화 순서 (중요)

```c
int main(void)
{
    HAL_Init();
    SystemClock_Config();
    MX_USART2_UART_Init();   // ST-LINK VCP
    osKernelInitialize();

    log_init();

    osThreadNew(LoggerTask, NULL,
        &(osThreadAttr_t){
            .name = "Logger",
            .priority = osPriorityLow,
            .stack_size = 512
        });

    // 다른 Task들 생성
    osKernelStart();
}
```

📌 LoggerTask는 **낮은 우선순위**가 정석

---

## 5️⃣ 이제 Task에서는 이렇게만 쓰면 됨

```c
void SensorTask(void *arg)
{
    for (;;)
    {
        log_printf("Temp=%d\r\n", temp);
        osDelay(500);
    }
}
```

* UART 신경 ❌
* mutex ❌
* printf ❌
* RTOS 구조 깨짐 ❌

---

## 🔥 지금 구조의 레벨

| 항목       | 평가         |
| -------- | ---------- |
| RTOS 안전성 | ✅          |
| 구조 분리    | ✅          |
| 포트폴리오    | ✅          |
| DMA 전환   | ✅ (한 줄 교체) |
| 과설계      | ❌          |

---

## 면접에서 쓰기 좋은 한 줄

> “CMSIS-RTOS2 환경에서
> Logger Task를 두어 UART 접근을 단일화했고,
> 로그 포맷과 전송을 분리해
> RTOS 실시간성을 유지했습니다.”

---

다음 단계

1️⃣ **DMA로 전환** (HAL_UART_Transmit_DMA)
2️⃣ **로그 레벨 매크로** (INFO/WARN/ERR)
3️⃣ **ISR 로그 → 이벤트 코드화**

