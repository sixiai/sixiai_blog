# SBL布阵定位原理

> 📅 2025-10-28 | 🏷️ knowledge

## 什么是 SBL？

**SBL (Short Baseline)** 短基线定位系统，是一种水下声学定位技术。通过在船底安装多个换能器阵元，测量目标信号到达各阵元的时间差（TDOA），从而计算目标位置。

## 基本原理

### 阵列布置

典型的 SBL 系统采用 3 个或 4 个换能器，布置在船底：

```
      船头
        ↑
    [1]   [2]
        
    [3]   [4]
      船尾
```

### 时间差测量

当水下目标发出声信号时，各换能器接收到信号的时间不同：

$$\Delta t_{12} = t_1 - t_2 = \frac{r_1 - r_2}{c}$$

其中：
- $t_i$ - 第 i 个换能器接收时间
- $r_i$ - 目标到第 i 个换能器的距离
- $c$ - 声速（约 1500 m/s）

### 几何关系

设目标位置为 $(x, y, z)$，换能器位置为 $(x_i, y_i, z_i)$，则：

$$r_i = \sqrt{(x-x_i)^2 + (y-y_i)^2 + (z-z_i)^2}$$

## 定位算法

### 1. 球面交汇法

通过 3 个以上的距离差方程，建立方程组：

$$\begin{cases}
r_1 - r_2 = c \cdot \Delta t_{12} \\
r_1 - r_3 = c \cdot \Delta t_{13} \\
r_1 - r_4 = c \cdot \Delta t_{14}
\end{cases}$$

### 2. 最小二乘解法

将非线性方程组线性化，使用最小二乘法求解：

```matlab
function pos = sbl_positioning(hydrophones, tdoa, c)
    % hydrophones: 4x3 矩阵，每行是一个换能器的 [x,y,z]
    % tdoa: 3x1 向量，时间差 [t12, t13, t14]
    % c: 声速
    
    % 距离差
    d = c * tdoa;
    
    % 构建矩阵
    A = zeros(3, 3);
    b = zeros(3, 1);
    
    for i = 2:4
        A(i-1, :) = 2 * (hydrophones(i,:) - hydrophones(1,:));
        b(i-1) = d(i-1)^2 - norm(hydrophones(i,:))^2 + norm(hydrophones(1,:))^2;
    end
    
    % 最小二乘解
    pos = A \ b;
end
```

### 3. 迭代优化

使用牛顿迭代法提高精度：

```matlab
function pos = sbl_newton(hydrophones, tdoa, c, pos0)
    % pos0: 初始估计位置
    
    pos = pos0;
    max_iter = 100;
    tol = 1e-6;
    
    for iter = 1:max_iter
        % 计算距离
        r = zeros(4, 1);
        for i = 1:4
            r(i) = norm(pos - hydrophones(i,:)');
        end
        
        % 计算残差
        d_meas = c * tdoa;
        d_calc = r(2:4) - r(1);
        residual = d_meas - d_calc;
        
        % 计算雅可比矩阵
        J = zeros(3, 3);
        for i = 2:4
            J(i-1, :) = (pos - hydrophones(i,:)')'/r(i) - ...
                        (pos - hydrophones(1,:)')'/r(1);
        end
        
        % 更新位置
        delta = J \ residual;
        pos = pos + delta;
        
        if norm(delta) < tol
            break;
        end
    end
end
```

## 误差分析

### 主要误差源

| 误差源 | 影响 | 解决方法 |
|--------|------|----------|
| 声速误差 | 距离计算偏差 | 实时测量声速剖面 |
| 时间测量误差 | 定位精度下降 | 提高采样率 |
| 阵元位置误差 | 系统偏差 | 精确标定 |
| 多路径效应 | 虚假目标 | 信号处理滤波 |

### 几何精度因子 (GDOP)

$$GDOP = \sqrt{trace((J^T J)^{-1})}$$

GDOP 越小，定位精度越高。

## 应用场景

- 🤿 潜水员定位
- 🤖 水下机器人导航 (ROV/AUV)
- 🎣 渔业探测
- 🔬 海洋科学研究

## 总结

SBL 定位系统结构简单、成本较低，适合近距离水下定位。理解其原理有助于优化系统设计和提高定位精度。
