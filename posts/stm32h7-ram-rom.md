# STM32H7的RAM、ROM分区和注意事项

> 📅 2025-11-20 | 🏷️ stm32

## 前言

STM32H7 系列是 ST 公司的高性能 MCU，拥有复杂的内存架构。本文详细介绍其 RAM 和 ROM 的分区及使用注意事项。

## 内存架构概览

STM32H7 的内存架构比较复杂，主要包括：

### Flash (ROM)

| 区域 | 地址范围 | 大小 | 说明 |
|------|----------|------|------|
| Bank1 | 0x08000000 - 0x080FFFFF | 1MB | 主 Flash |
| Bank2 | 0x08100000 - 0x081FFFFF | 1MB | 主 Flash |
| System | 0x1FF00000 - 0x1FF1FFFF | 128KB | 系统存储器 |

### RAM

| 区域 | 地址范围 | 大小 | 说明 |
|------|----------|------|------|
| DTCM | 0x20000000 - 0x2001FFFF | 128KB | 数据紧耦合内存 |
| AXI SRAM | 0x24000000 - 0x2407FFFF | 512KB | 主 SRAM |
| SRAM1 | 0x30000000 - 0x3001FFFF | 128KB | D2 域 |
| SRAM2 | 0x30020000 - 0x3003FFFF | 128KB | D2 域 |
| SRAM3 | 0x30040000 - 0x30047FFF | 32KB | D2 域 |
| SRAM4 | 0x38000000 - 0x3800FFFF | 64KB | D3 域 |
| Backup SRAM | 0x38800000 - 0x38800FFF | 4KB | 备份域 |

## 分散加载文件配置

```
LR_IROM1 0x08000000 0x00200000  {
  ER_IROM1 0x08000000 0x00200000  {
    *.o (RESET, +First)
    *(InRoot$$Sections)
    .ANY (+RO)
  }
  
  ; DTCM - 用于关键数据和栈
  RW_DTCM 0x20000000 0x00020000  {
    .ANY (+RW +ZI)
  }
  
  ; AXI SRAM - 用于大数组和 DMA 缓冲区
  RW_AXI_SRAM 0x24000000 0x00080000  {
    *(.AXI_SRAM)
  }
  
  ; D2 SRAM - 用于 DMA 和外设
  RW_D2_SRAM 0x30000000 0x00048000  {
    *(.D2_SRAM)
  }
}
```

## 使用注意事项

### 1. DMA 缓冲区位置

⚠️ **重要**：DMA 缓冲区必须放在 D2 或 D3 域的 SRAM 中！

```c
// 正确 ✅
__attribute__((section(".D2_SRAM"))) uint8_t dmaBuffer[1024];

// 错误 ❌ - DTCM 不能用于 DMA
uint8_t dmaBuffer[1024];  // 默认在 DTCM
```

### 2. Cache 一致性

```c
// DMA 传输前，清除 Cache
SCB_CleanDCache_by_Addr((uint32_t*)buffer, size);

// DMA 传输后，无效化 Cache
SCB_InvalidateDCache_by_Addr((uint32_t*)buffer, size);
```

### 3. MPU 配置

```c
void MPU_Config(void)
{
    MPU_Region_InitTypeDef MPU_InitStruct;
    
    HAL_MPU_Disable();
    
    // 配置 AXI SRAM 为 Write-through
    MPU_InitStruct.Enable = MPU_REGION_ENABLE;
    MPU_InitStruct.BaseAddress = 0x24000000;
    MPU_InitStruct.Size = MPU_REGION_SIZE_512KB;
    MPU_InitStruct.AccessPermission = MPU_REGION_FULL_ACCESS;
    MPU_InitStruct.IsBufferable = MPU_ACCESS_NOT_BUFFERABLE;
    MPU_InitStruct.IsCacheable = MPU_ACCESS_CACHEABLE;
    MPU_InitStruct.IsShareable = MPU_ACCESS_NOT_SHAREABLE;
    MPU_InitStruct.Number = MPU_REGION_NUMBER0;
    MPU_InitStruct.TypeExtField = MPU_TEX_LEVEL0;
    MPU_InitStruct.SubRegionDisable = 0x00;
    MPU_InitStruct.DisableExec = MPU_INSTRUCTION_ACCESS_ENABLE;
    
    HAL_MPU_ConfigRegion(&MPU_InitStruct);
    HAL_MPU_Enable(MPU_PRIVILEGED_DEFAULT);
}
```

## 总结

STM32H7 的内存架构虽然复杂，但合理使用可以发挥其最大性能。关键点：

1. **DTCM** - 用于栈和关键变量，访问最快
2. **AXI SRAM** - 用于大数组，需注意 Cache
3. **D2 SRAM** - 用于 DMA 缓冲区
4. **注意 Cache 一致性问题**
