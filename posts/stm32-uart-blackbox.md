# STM32F103C8T6串口黑匣子

> 📅 2026-01-04 | 🏷️ stm32

## 简介

串口黑匣子是一个用于记录和调试串口通信数据的工具，本文介绍如何在 STM32F103C8T6 上实现。

## 硬件准备

- STM32F103C8T6 最小系统板
- USB 转 TTL 模块
- SD 卡模块（可选，用于存储数据）

## 代码实现

### 1. 串口初始化

```c
void USART1_Init(uint32_t baudrate)
{
    GPIO_InitTypeDef GPIO_InitStruct;
    USART_InitTypeDef USART_InitStruct;
    
    // 使能时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1 | RCC_APB2Periph_GPIOA, ENABLE);
    
    // 配置 TX (PA9)
    GPIO_InitStruct.GPIO_Pin = GPIO_Pin_9;
    GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_InitStruct);
    
    // 配置 RX (PA10)
    GPIO_InitStruct.GPIO_Pin = GPIO_Pin_10;
    GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IN_FLOATING;
    GPIO_Init(GPIOA, &GPIO_InitStruct);
    
    // 配置串口参数
    USART_InitStruct.USART_BaudRate = baudrate;
    USART_InitStruct.USART_WordLength = USART_WordLength_8b;
    USART_InitStruct.USART_StopBits = USART_StopBits_1;
    USART_InitStruct.USART_Parity = USART_Parity_No;
    USART_InitStruct.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
    USART_InitStruct.USART_Mode = USART_Mode_Rx | USART_Mode_Tx;
    USART_Init(USART1, &USART_InitStruct);
    
    // 使能串口
    USART_Cmd(USART1, ENABLE);
}
```

### 2. 数据缓冲区

```c
#define BUFFER_SIZE 1024

typedef struct {
    uint8_t data[BUFFER_SIZE];
    uint16_t head;
    uint16_t tail;
} RingBuffer;

RingBuffer rxBuffer;

void Buffer_Push(RingBuffer *buf, uint8_t data)
{
    buf->data[buf->head] = data;
    buf->head = (buf->head + 1) % BUFFER_SIZE;
}

uint8_t Buffer_Pop(RingBuffer *buf)
{
    uint8_t data = buf->data[buf->tail];
    buf->tail = (buf->tail + 1) % BUFFER_SIZE;
    return data;
}
```

### 3. 中断处理

```c
void USART1_IRQHandler(void)
{
    if (USART_GetITStatus(USART1, USART_IT_RXNE) != RESET)
    {
        uint8_t data = USART_ReceiveData(USART1);
        Buffer_Push(&rxBuffer, data);
        
        // 记录时间戳
        LogTimestamp();
    }
}
```

## 功能特点

| 功能 | 说明 |
|------|------|
| 数据记录 | 自动记录所有串口收发数据 |
| 时间戳 | 每条数据带有精确时间戳 |
| 循环存储 | 环形缓冲区，自动覆盖旧数据 |
| SD卡导出 | 支持导出到 SD 卡 |

## 总结

串口黑匣子在调试嵌入式系统时非常有用，可以帮助我们追踪通信问题。
