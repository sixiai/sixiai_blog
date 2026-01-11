# STM32的IWDG配置

> 📅 2025-10-27 | 🏷️ stm32

## 什么是 IWDG？

**IWDG (Independent Watchdog)** 独立看门狗，是 STM32 内置的硬件看门狗定时器。当程序跑飞或死循环时，看门狗会自动复位 MCU，保证系统可靠性。

## 工作原理

```
┌─────────────────────────────────────────────────────┐
│                    IWDG 工作原理                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│   LSI (40kHz)  ──→  预分频器  ──→  递减计数器       │
│                        │              │             │
│                        ↓              ↓             │
│                    分频系数      计数值减到 0       │
│                   (4~256)            │             │
│                                      ↓             │
│                               系统复位！            │
│                                                     │
│   喂狗操作：重新加载计数值，防止复位                  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## 超时时间计算

$$T_{out} = \frac{Prescaler \times Reload}{f_{LSI}}$$

其中：
- $f_{LSI}$ ≈ 40 kHz（实际 30~60 kHz）
- Prescaler: 4, 8, 16, 32, 64, 128, 256
- Reload: 0 ~ 4095

### 常用配置

| 预分频 | 重载值 | 超时时间 |
|--------|--------|----------|
| 4 | 4095 | ~409 ms |
| 16 | 4095 | ~1.6 s |
| 64 | 4095 | ~6.5 s |
| 256 | 4095 | ~26 s |

## HAL 库配置

### CubeMX 配置

1. 打开 IWDG 外设
2. 设置预分频系数
3. 设置重载值

### 代码实现

```c
IWDG_HandleTypeDef hiwdg;

void MX_IWDG_Init(void)
{
    hiwdg.Instance = IWDG;
    hiwdg.Init.Prescaler = IWDG_PRESCALER_64;  // 64 分频
    hiwdg.Init.Reload = 4095;                   // 重载值
    
    // 超时时间 = 64 * 4095 / 40000 ≈ 6.5 秒
    
    if (HAL_IWDG_Init(&hiwdg) != HAL_OK)
    {
        Error_Handler();
    }
}
```

### 喂狗操作

```c
// 在主循环中定期调用
void Feed_Dog(void)
{
    HAL_IWDG_Refresh(&hiwdg);
}

int main(void)
{
    HAL_Init();
    SystemClock_Config();
    MX_IWDG_Init();
    
    while (1)
    {
        // 主程序逻辑
        Do_Something();
        
        // 喂狗
        Feed_Dog();
        
        HAL_Delay(100);
    }
}
```

## 标准库配置

```c
void IWDG_Init(void)
{
    // 使能写入
    IWDG_WriteAccessCmd(IWDG_WriteAccess_Enable);
    
    // 设置预分频
    IWDG_SetPrescaler(IWDG_Prescaler_64);
    
    // 设置重载值
    IWDG_SetReload(4095);
    
    // 重载计数器
    IWDG_ReloadCounter();
    
    // 使能 IWDG
    IWDG_Enable();
}

void Feed_Dog(void)
{
    IWDG_ReloadCounter();
}
```

## 寄存器配置

```c
void IWDG_Init_Register(void)
{
    // 解锁写保护
    IWDG->KR = 0x5555;
    
    // 设置预分频 (64 分频)
    IWDG->PR = 4;  // 0:4, 1:8, 2:16, 3:32, 4:64, 5:128, 6:256
    
    // 设置重载值
    IWDG->RLR = 4095;
    
    // 等待寄存器更新
    while (IWDG->SR != 0);
    
    // 重载计数器
    IWDG->KR = 0xAAAA;
    
    // 启动看门狗
    IWDG->KR = 0xCCCC;
}

void Feed_Dog_Register(void)
{
    IWDG->KR = 0xAAAA;
}
```

## 注意事项

### 1. 一旦启动无法关闭

⚠️ IWDG 一旦启动，只能通过复位关闭！调试时要注意。

### 2. 调试模式

在调试时，可以配置 DBGMCU 暂停看门狗：

```c
// 调试时暂停 IWDG
__HAL_DBGMCU_FREEZE_IWDG();
```

### 3. LSI 精度

LSI 时钟精度较低（30~60 kHz），超时时间会有偏差。关键应用建议留有余量。

### 4. 喂狗时机

```c
// ❌ 错误：在中断中喂狗
void SysTick_Handler(void)
{
    Feed_Dog();  // 即使主程序死了，中断还在喂狗
}

// ✅ 正确：在主循环中喂狗
while (1)
{
    Task1();
    Task2();
    Feed_Dog();  // 只有所有任务正常才能喂狗
}
```

## 总结

IWDG 是保证嵌入式系统可靠性的重要手段：

1. **配置简单** - 只需设置预分频和重载值
2. **独立运行** - 使用独立的 LSI 时钟
3. **不可关闭** - 一旦启动只能复位
4. **定期喂狗** - 在主循环中喂狗，不要在中断中喂狗
