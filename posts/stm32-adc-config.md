# STM32CubeMX的ADC配置（以H7为例）

> 📅 2025-11-14 | 🏷️ stm32

## 简介

本文介绍如何使用 STM32CubeMX 配置 STM32H7 的 ADC，实现高精度模拟信号采集。

## CubeMX 配置步骤

### 1. 选择 ADC 通道

在 Pinout 视图中，选择需要的 ADC 引脚，例如：
- PA0 → ADC1_INP16
- PA1 → ADC1_INP17

### 2. ADC 参数配置

```
ADC1 配置：
├── Clock Prescaler: Asynchronous clock mode, div 2
├── Resolution: 16 bits
├── Data Alignment: Right alignment
├── Scan Conversion Mode: Enabled
├── Continuous Conversion Mode: Enabled
├── DMA Continuous Requests: Enabled
└── End of Conversion Selection: End of sequence
```

### 3. 采样时间配置

```
Channel 16:
├── Sampling Time: 64.5 Cycles
└── Offset: 0

Channel 17:
├── Sampling Time: 64.5 Cycles
└── Offset: 0
```

## 代码实现

### 初始化代码

```c
ADC_HandleTypeDef hadc1;
DMA_HandleTypeDef hdma_adc1;

uint16_t adcBuffer[2];  // 存储 2 个通道的数据

void MX_ADC1_Init(void)
{
    ADC_MultiModeTypeDef multimode = {0};
    ADC_ChannelConfTypeDef sConfig = {0};
    
    hadc1.Instance = ADC1;
    hadc1.Init.ClockPrescaler = ADC_CLOCK_ASYNC_DIV2;
    hadc1.Init.Resolution = ADC_RESOLUTION_16B;
    hadc1.Init.ScanConvMode = ADC_SCAN_ENABLE;
    hadc1.Init.EOCSelection = ADC_EOC_SEQ_CONV;
    hadc1.Init.LowPowerAutoWait = DISABLE;
    hadc1.Init.ContinuousConvMode = ENABLE;
    hadc1.Init.NbrOfConversion = 2;
    hadc1.Init.DiscontinuousConvMode = DISABLE;
    hadc1.Init.ExternalTrigConv = ADC_SOFTWARE_START;
    hadc1.Init.ExternalTrigConvEdge = ADC_EXTERNALTRIGCONVEDGE_NONE;
    hadc1.Init.ConversionDataManagement = ADC_CONVERSIONDATA_DMA_CIRCULAR;
    hadc1.Init.Overrun = ADC_OVR_DATA_OVERWRITTEN;
    
    HAL_ADC_Init(&hadc1);
    
    // 配置通道 16
    sConfig.Channel = ADC_CHANNEL_16;
    sConfig.Rank = ADC_REGULAR_RANK_1;
    sConfig.SamplingTime = ADC_SAMPLETIME_64CYCLES_5;
    sConfig.SingleDiff = ADC_SINGLE_ENDED;
    sConfig.OffsetNumber = ADC_OFFSET_NONE;
    HAL_ADC_ConfigChannel(&hadc1, &sConfig);
    
    // 配置通道 17
    sConfig.Channel = ADC_CHANNEL_17;
    sConfig.Rank = ADC_REGULAR_RANK_2;
    HAL_ADC_ConfigChannel(&hadc1, &sConfig);
}
```

### 启动 ADC 采集

```c
// 校准 ADC
HAL_ADCEx_Calibration_Start(&hadc1, ADC_CALIB_OFFSET, ADC_SINGLE_ENDED);

// 启动 DMA 传输
HAL_ADC_Start_DMA(&hadc1, (uint32_t*)adcBuffer, 2);
```

### DMA 回调函数

```c
void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef *hadc)
{
    if (hadc->Instance == ADC1)
    {
        // adcBuffer[0] 是通道 16 的值
        // adcBuffer[1] 是通道 17 的值
        
        float voltage0 = adcBuffer[0] * 3.3f / 65535.0f;
        float voltage1 = adcBuffer[1] * 3.3f / 65535.0f;
    }
}
```

## 注意事项

### 1. DMA 缓冲区位置

```c
// 必须放在 D2 SRAM！
__attribute__((section(".D2_SRAM"))) uint16_t adcBuffer[2];
```

### 2. 参考电压

H7 的 VREF+ 建议使用外部精密基准源，如 REF3030。

### 3. 采样时间选择

| 采样时间 | 适用场景 |
|----------|----------|
| 1.5 cycles | 低阻抗信号源 |
| 64.5 cycles | 一般应用 |
| 810.5 cycles | 高阻抗信号源 |

## 总结

STM32H7 的 ADC 功能强大，16 位分辨率可以满足大多数应用需求。配置时注意 DMA 缓冲区位置和采样时间选择。
