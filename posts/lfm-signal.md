# 认识线性调频信号（LFM）

> 📅 2025-10-31 | 🏷️ knowledge

## 什么是 LFM 信号？

**LFM (Linear Frequency Modulation)** 线性调频信号，也称为 Chirp 信号，是一种频率随时间线性变化的信号。广泛应用于雷达、声纳、通信等领域。

## 数学表达式

### 时域表达式

$$s(t) = A \cdot \cos\left(2\pi f_0 t + \pi k t^2\right), \quad 0 \leq t \leq T$$

其中：
- $A$ - 信号幅度
- $f_0$ - 起始频率
- $k = B/T$ - 调频斜率
- $B$ - 带宽
- $T$ - 脉冲宽度

### 瞬时频率

$$f(t) = f_0 + kt$$

频率从 $f_0$ 线性变化到 $f_0 + B$。

### 复数形式

$$s(t) = A \cdot e^{j(2\pi f_0 t + \pi k t^2)}$$

### 重点

LFM（线性调频）既可以是实数（真实物理载波）形式，也可以用复数（解析/基带等效）形式表示——取决于你在做什么。

#### 实数形式（物理发射 / 回放） 
发到换能器/扬声器/射频口的一定是实信号：

$$s_{real}(t)=Acos(2\pi(f_0t+\frac{1}{2}kt^2)+\phi_0),k=\frac{f_1-f_0}{T}$$

看起来就是“振幅几乎不变、零交叉随时间变密或变疏”的正弦波（上扫更密、下扫更疏）。

#### 复数形式（解析/基带等效，算法里常用）
为了简化分析、匹配滤波、上/下变频、避免镜像带，我们常把它写成解析信号：

$$s_{cx}(t)=Ae^{j\phi (t)},\phi (t)=2\pi (f_0t+\frac{1}{2} kt^2)$$
或者把扫频居中到基带（中心频率0，带宽$𝐵=𝑓_1−𝑓_0$，时长𝑇）：

$$s_{bb}(t)=Ae^{j\pi (B/T)t^2},t\in [-T/2,T/2]$$

需要搬到某个载频$𝑓_c$时，再乘上$e^{j2\pi f_ct}$并取实部即可得到可发射的实信号：

$$s_{passband}(t)=\Re \left \{s_{bb}(t)e^{j2\pi f_ct}  \right \} =A\cos(2\pi f_ct+\pi (B/T)t^2)$$

## Matlab 仿真

### 生成 LFM 信号

```matlab
function [s, t] = generate_lfm(f0, B, T, fs)
    % f0: 起始频率 (Hz)
    % B: 带宽 (Hz)
    % T: 脉冲宽度 (s)
    % fs: 采样率 (Hz)
    
    t = 0 : 1/fs : T-1/fs;
    k = B / T;  % 调频斜率
    
    % 生成 LFM 信号
    s = exp(1j * (2*pi*f0*t + pi*k*t.^2));
end
```

### 绘制时频图

```matlab
% 参数设置
f0 = 1e3;    % 起始频率 1kHz
B = 10e3;    % 带宽 10kHz
T = 1e-3;    % 脉宽 1ms
fs = 100e3;  % 采样率 100kHz

% 生成信号
[s, t] = generate_lfm(f0, B, T, fs);

% 绘图
figure;

% 时域波形
subplot(2,2,1);
plot(t*1e3, real(s));
xlabel('Time (ms)');
ylabel('Amplitude');
title('Time Domain');

% 频谱
subplot(2,2,2);
N = length(s);
f = (-N/2:N/2-1) * fs / N;
S = fftshift(fft(s));
plot(f/1e3, abs(S)/max(abs(S)));
xlabel('Frequency (kHz)');
ylabel('Magnitude');
title('Spectrum');

% 时频图 (STFT)
subplot(2,2,[3,4]);
spectrogram(s, 64, 60, 128, fs, 'yaxis');
title('Time-Frequency');
```

## 脉冲压缩

LFM 信号的核心优势是可以进行**脉冲压缩**，在不增加发射功率的情况下提高距离分辨率。

### 匹配滤波

```matlab
function y = pulse_compression(s, ref)
    % s: 接收信号
    % ref: 参考信号（发射信号）
    
    % 匹配滤波 = 与参考信号的共轭时间反转卷积
    h = conj(fliplr(ref));
    y = conv(s, h);
    
    % 或使用频域方法（更快）
    % N = length(s) + length(ref) - 1;
    % y = ifft(fft(s, N) .* conj(fft(ref, N)));
end
```

### 压缩增益

$$G = B \cdot T$$

例如：$B = 10$ kHz，$T = 1$ ms，则 $G = 10$（即 10 dB）。

### 压缩后脉宽

$$T_{compressed} = \frac{1}{B}$$

## 距离分辨率

$$\Delta R = \frac{c}{2B}$$

其中 $c$ 是传播速度（光速或声速）。

| 带宽 | 距离分辨率（雷达） | 距离分辨率（声纳） |
|------|-------------------|-------------------|
| 1 kHz | 150 km | 0.75 m |
| 10 kHz | 15 km | 7.5 cm |
| 100 kHz | 1.5 km | 7.5 mm |

## 应用场景

### 1. 雷达系统
- 脉冲压缩雷达
- SAR（合成孔径雷达）
- 气象雷达

### 2. 声纳系统
- 主动声纳探测
- 水下通信
- 鱼群探测

### 3. 通信系统
- 扩频通信
- 超宽带 (UWB) 通信

## 总结

LFM 信号是现代雷达和声纳系统的基础，其核心优势是：

1. **大时宽带宽积** - 同时获得高能量和高分辨率
2. **脉冲压缩** - 提高信噪比
3. **抗干扰能力强** - 频率变化使干扰难以跟踪

理解 LFM 信号对于从事雷达、声纳、通信领域的工程师非常重要。
