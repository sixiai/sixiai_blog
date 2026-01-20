# 目录

**1、水下机器人水平面3自由度简化模型**

**2、最小二乘法辨识9个参数**

**3、欠驱动AUV最小二乘参数辨识总结**

**4、欠驱动AUV最小二乘参数辨识（完整版）**

**5、通过力和力矩递推速度**





## 1、水下机器人水平面3自由度简化模型

---

### 一、动力学方程

$$m_u \dot{u} + d_u u + d_{uu}u|u| = \tau_u$$

$$m_v \dot{v} + d_v v + d_{vv}v|v| = \tau_v$$

$$I_r \dot{r} + d_r r + d_{rr}r|r| = \tau_r$$

---

### 二、运动学方程

$$\dot{x} = u\cos\psi - v\sin\psi$$

$$\dot{y} = u\sin\psi + v\cos\psi$$

$$\dot{\psi} = r$$

---

### 三、变量说明

#### 状态变量

| 符号   | 单位  | 含义                             |
| ------ | ----- | -------------------------------- |
| $x$    | m     | 惯性坐标系下x方向位置            |
| $y$    | m     | 惯性坐标系下y方向位置            |
| $\psi$ | rad   | 艏向角（航向角），绕z轴旋转角度  |
| $u$    | m/s   | 体坐标系下前进速度（Surge）      |
| $v$    | m/s   | 体坐标系下横移速度（Sway）       |
| $r$    | rad/s | 体坐标系下艏摇角速度（Yaw rate） |

#### 惯性参数

| 符号                      | 单位  | 含义                                               |
| ------------------------- | ----- | -------------------------------------------------- |
| $m_u = m - X_{\dot{u}}$   | kg    | Surge方向等效质量（刚体质量 + 附加质量）           |
| $m_v = m - Y_{\dot{v}}$   | kg    | Sway方向等效质量（刚体质量 + 附加质量）            |
| $I_r = I_z - N_{\dot{r}}$ | kg·m² | Yaw方向等效转动惯量（刚体转动惯量 + 附加转动惯量） |

#### 阻尼参数

| 符号                 | 单位    | 含义                  |
| -------------------- | ------- | --------------------- |
| $d_u = -X_u$         | kg/s    | Surge方向线性阻尼系数 |
| $d_{uu} = -X_{u|u|}$ | kg/m    | Surge方向二次阻尼系数 |
| $d_v = -Y_v$         | kg/s    | Sway方向线性阻尼系数  |
| $d_{vv} = -Y_{v|v|}$ | kg/m    | Sway方向二次阻尼系数  |
| $d_r = -N_r$         | kg·m²/s | Yaw方向线性阻尼系数   |
| $d_{rr} = -N_{r|r|}$ | kg·m²   | Yaw方向二次阻尼系数   |

#### 控制输入

| 符号     | 单位 | 含义            |
| -------- | ---- | --------------- |
| $\tau_u$ | N    | Surge方向推进力 |
| $\tau_v$ | N    | Sway方向推进力  |
| $\tau_r$ | N·m  | Yaw方向控制力矩 |

---

### 四、坐标系示意

```
        惯性坐标系 (x, y)
              ↑ x
              |
              |    ← 机器人
              |   ╱
              |  ╱ ψ (艏向角)
              | ╱
    ──────────┼──────────→ y
              |
              
        体坐标系 (u, v)
              ↑ u (前进)
              |
              |
              |
    ──────────┼──────────→ v (横移)
              |
              ↺ r (艏摇)
```

---

### 五、模型假设

1. **水平面运动**：忽略 Heave、Roll、Pitch 三个自由度
2. **解耦假设**：忽略各自由度之间的交叉耦合项
3. **对称性假设**：机器人左右对称
4. **恒定参数**：惯性和阻尼参数为常数

---

## 2、最小二乘法辨识9个参数

### 一、基本思路

对每个自由度**分别进行辨识**，每个方向辨识3个参数，共辨识3次。

---

### 二、方程改写

将动力学方程改写为**线性回归形式**：

$$\dot{x} = \theta^T \phi$$

其中 $\theta$ 是待辨识参数，$\phi$ 是回归向量。

---

### 三、三个方向的辨识

#### **1. Surge方向**

原方程：
$$
m_u \dot{u} + d_u u + d_{uu}u|u| = \tau_u
$$

$$
m_u \dot{u} + d_u u + d_{uu}u|u| = \tau_u
$$





改写为：
$$\dot{u} = \frac{1}{m_u}\tau_u - \frac{d_u}{m_u}u - \frac{d_{uu}}{m_u}u|u|$$

定义：
$$\theta_u = \begin{bmatrix} \theta_1 \\ \theta_2 \\ \theta_3 \end{bmatrix} = \begin{bmatrix} 1/m_u \\ d_u/m_u \\ d_{uu}/m_u \end{bmatrix}, \quad \phi_u = \begin{bmatrix} \tau_u \\ -u \\ -u|u| \end{bmatrix}$$

则：
$$\dot{u} = \theta_u^T \phi_u$$

**参数还原：**
$$m_u = \frac{1}{\theta_1}, \quad d_u = \frac{\theta_2}{\theta_1}, \quad d_{uu} = \frac{\theta_3}{\theta_1}$$

---

#### **2. Sway方向**

原方程：
$$m_v \dot{v} + d_v v + d_{vv}v|v| = \tau_v$$

定义：
$$\theta_v = \begin{bmatrix} 1/m_v \\ d_v/m_v \\ d_{vv}/m_v \end{bmatrix}, \quad \phi_v = \begin{bmatrix} \tau_v \\ -v \\ -v|v| \end{bmatrix}$$

**参数还原：**
$$m_v = \frac{1}{\theta_1}, \quad d_v = \frac{\theta_2}{\theta_1}, \quad d_{vv} = \frac{\theta_3}{\theta_1}$$

---

#### **3. Yaw方向**

原方程：
$$I_r \dot{r} + d_r r + d_{rr}r|r| = \tau_r$$

定义：
$$\theta_r = \begin{bmatrix} 1/I_r \\ d_r/I_r \\ d_{rr}/I_r \end{bmatrix}, \quad \phi_r = \begin{bmatrix} \tau_r \\ -r \\ -r|r| \end{bmatrix}$$

**参数还原：**
$$I_r = \frac{1}{\theta_1}, \quad d_r = \frac{\theta_2}{\theta_1}, \quad d_{rr} = \frac{\theta_3}{\theta_1}$$

---

### 四、最小二乘求解

#### 数据收集

进行实验，采集 $N$ 组数据（采样时间 $\Delta t$）：

| 时刻  | 输入        | 状态                     | 加速度（差分计算）                             |
| ----- | ----------- | ------------------------ | ---------------------------------------------- |
| $t_1$ | $\tau(t_1)$ | $u(t_1), v(t_1), r(t_1)$ | $\dot{u}_1 \approx \frac{u_2 - u_1}{\Delta t}$ |
| $t_2$ | $\tau(t_2)$ | $u(t_2), v(t_2), r(t_2)$ | $\dot{u}_2 \approx \frac{u_3 - u_2}{\Delta t}$ |
| ...   | ...         | ...                      | ...                                            |
| $t_N$ | $\tau(t_N)$ | $u(t_N), v(t_N), r(t_N)$ | ...                                            |

#### 构建矩阵（以Surge为例）

$$\Phi = \begin{bmatrix} \tau_u(t_1) & -u(t_1) & -u(t_1)|u(t_1)| \\ \tau_u(t_2) & -u(t_2) & -u(t_2)|u(t_2)| \\ \vdots & \vdots & \vdots \\ \tau_u(t_N) & -u(t_N) & -u(t_N)|u(t_N)| \end{bmatrix}_{N \times 3}$$

$$\dot{U} = \begin{bmatrix} \dot{u}(t_1) \\ \dot{u}(t_2) \\ \vdots \\ \dot{u}(t_N) \end{bmatrix}_{N \times 1}$$

#### 最小二乘解

$$\boxed{\hat{\theta} = (\Phi^T \Phi)^{-1} \Phi^T \dot{U}}$$

---

### 五、完整辨识流程

```
┌─────────────────────────────────────────────────────────┐
│                    实验设计                              │
├─────────────────────────────────────────────────────────┤
│  1. Surge辨识：只施加前进推力 τ_u，τ_v=0, τ_r=0        │
│  2. Sway辨识：只施加横移推力 τ_v，τ_u=0, τ_r=0         │
│  3. Yaw辨识：只施加转向力矩 τ_r，τ_u=0, τ_v=0          │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                    数据采集                              │
├─────────────────────────────────────────────────────────┤
│  • 采样频率：10~100 Hz                                  │
│  • 测量：位置(x,y,ψ)、速度(u,v,r)、推力(τ_u,τ_v,τ_r)   │
│  • 激励信号：阶跃、正弦、随机信号                        │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                    数据预处理                            │
├─────────────────────────────────────────────────────────┤
│  • 滤波去噪（低通滤波）                                  │
│  • 计算加速度（差分或滤波微分）                          │
│  • 去除异常数据                                          │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                    参数辨识                              │
├─────────────────────────────────────────────────────────┤
│  • 构建回归矩阵 Φ 和输出向量 Y                          │
│  • 最小二乘求解 θ = (Φ'Φ)^(-1)Φ'Y                      │
│  • 还原物理参数                                          │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                    验证与修正                            │
├─────────────────────────────────────────────────────────┤
│  • 用辨识参数仿真，与实测数据对比                        │
│  • 计算拟合误差                                          │
│  • 必要时重新实验或调整模型                              │
└─────────────────────────────────────────────────────────┘
```

---

## 3、欠驱动AUV最小二乘参数辨识总结

---

### 一、系统模型

#### 动力学方程（考虑耦合）

$$m_u \dot{u} - m_v vr + d_u u + d_{uu}u|u| = \tau_u$$

$$m_v \dot{v} + m_u ur + d_v v + d_{vv}v|v| = 0$$

$$I_r \dot{r} + (m_u - m_v)uv + d_r r + d_{rr}r|r| = \tau_r$$

#### 运动学方程

$$\dot{x} = u\cos\psi - v\sin\psi$$

$$\dot{y} = u\sin\psi + v\cos\psi$$

$$\dot{\psi} = r$$

#### 控制输入

| 输入 | 说明 |
|------|------|
| $\tau_u$ | 前进推力 |
| $\tau_r$ | 转向力矩 |
| $\tau_v$ | **无**（欠驱动） |

---

### 二、待辨识参数（9个）

| 类别 | Surge | Sway | Yaw |
|------|-------|------|-----|
| **惯性** | $m_u$ | $m_v$ | $I_r$ |
| **线性阻尼** | $d_u$ | $d_v$ | $d_r$ |
| **二次阻尼** | $d_{uu}$ | $d_{vv}$ | $d_{rr}$ |

---

### 三、辨识流程

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Step 1    │     │   Step 2    │     │   Step 3    │
│   Surge     │ ──→ │    Yaw      │ ──→ │    Sway     │
│  直线航行   │     │  低速转向   │     │  曲线航行   │
└─────────────┘     └─────────────┘     └─────────────┘
      ↓                   ↓                   ↓
   m_u, d_u, d_uu     I_r, d_r, d_rr     m_v, d_v, d_vv
                                              ↑
                                         利用已知m_u
```

---

### 四、各步骤详细公式

#### **Step 1：Surge参数辨识**

##### 实验条件
- 直线航行，保持 $v \approx 0$，$r \approx 0$
- 施加变化的推力 $\tau_u$

##### 简化方程
$$m_u \dot{u} + d_u u + d_{uu}u|u| = \tau_u$$

##### 回归形式
$$\dot{u} = \theta_u^T \phi_u$$

$$\theta_u = \begin{bmatrix} 1/m_u \\ d_u/m_u \\ d_{uu}/m_u \end{bmatrix}, \quad \phi_u = \begin{bmatrix} \tau_u \\ -u \\ -u|u| \end{bmatrix}$$

##### 最小二乘求解
$$\hat{\theta}_u = (\Phi_u^T \Phi_u)^{-1} \Phi_u^T \dot{U}$$

其中：
$$\Phi_u = \begin{bmatrix} \tau_u(t_1) & -u(t_1) & -u(t_1)|u(t_1)| \\ \vdots & \vdots & \vdots \\ \tau_u(t_N) & -u(t_N) & -u(t_N)|u(t_N)| \end{bmatrix}, \quad \dot{U} = \begin{bmatrix} \dot{u}(t_1) \\ \vdots \\ \dot{u}(t_N) \end{bmatrix}$$

##### 参数还原
$$\boxed{m_u = \frac{1}{\theta_{u1}}, \quad d_u = \frac{\theta_{u2}}{\theta_{u1}}, \quad d_{uu} = \frac{\theta_{u3}}{\theta_{u1}}}$$

---

#### **Step 2：Yaw参数辨识**

##### 实验条件
- 低速或原地转向，保持 $u$ 较小且恒定
- 施加变化的力矩 $\tau_r$

##### 简化方程
$$I_r \dot{r} + d_r r + d_{rr}r|r| = \tau_r$$

##### 回归形式
$$\dot{r} = \theta_r^T \phi_r$$

$$\theta_r = \begin{bmatrix} 1/I_r \\ d_r/I_r \\ d_{rr}/I_r \end{bmatrix}, \quad \phi_r = \begin{bmatrix} \tau_r \\ -r \\ -r|r| \end{bmatrix}$$

##### 最小二乘求解
$$\hat{\theta}_r = (\Phi_r^T \Phi_r)^{-1} \Phi_r^T \dot{R}$$

其中：
$$\Phi_r = \begin{bmatrix} \tau_r(t_1) & -r(t_1) & -r(t_1)|r(t_1)| \\ \vdots & \vdots & \vdots \\ \tau_r(t_N) & -r(t_N) & -r(t_N)|r(t_N)| \end{bmatrix}, \quad \dot{R} = \begin{bmatrix} \dot{r}(t_1) \\ \vdots \\ \dot{r}(t_N) \end{bmatrix}$$

##### 参数还原
$$\boxed{I_r = \frac{1}{\theta_{r1}}, \quad d_r = \frac{\theta_{r2}}{\theta_{r1}}, \quad d_{rr} = \frac{\theta_{r3}}{\theta_{r1}}}$$

---

#### **Step 3：Sway参数辨识（转向耦合法）**

##### 实验条件
- 曲线航行（S形或圆弧）
- 同时施加 $\tau_u$ 和 $\tau_r$，产生侧向速度 $v$
- 用DVL测量 $v$

##### 耦合方程
$$m_v \dot{v} + m_u ur + d_v v + d_{vv}v|v| = 0$$

##### 回归形式
$$\dot{v} = \theta_v^T \phi_v$$

$$\theta_v = \begin{bmatrix} m_u/m_v \\ d_v/m_v \\ d_{vv}/m_v \end{bmatrix}, \quad \phi_v = \begin{bmatrix} -ur \\ -v \\ -v|v| \end{bmatrix}$$

##### 最小二乘求解
$$\hat{\theta}_v = (\Phi_v^T \Phi_v)^{-1} \Phi_v^T \dot{V}$$

其中：
$$\Phi_v = \begin{bmatrix} -u(t_1)r(t_1) & -v(t_1) & -v(t_1)|v(t_1)| \\ \vdots & \vdots & \vdots \\ -u(t_N)r(t_N) & -v(t_N) & -v(t_N)|v(t_N)| \end{bmatrix}, \quad \dot{V} = \begin{bmatrix} \dot{v}(t_1) \\ \vdots \\ \dot{v}(t_N) \end{bmatrix}$$

##### 参数还原（利用已知 $m_u$）
$$\boxed{m_v = \frac{m_u}{\theta_{v1}}, \quad d_v = \frac{\theta_{v2} \cdot m_u}{\theta_{v1}}, \quad d_{vv} = \frac{\theta_{v3} \cdot m_u}{\theta_{v1}}}$$

---

### 五、数据处理要点

#### 加速度计算

$$\dot{u}(t_i) \approx \frac{u(t_{i+1}) - u(t_{i-1})}{2\Delta t} \quad \text{(中心差分)}$$

#### 滤波去噪

```
原始数据 → 低通滤波 → 计算加速度 → 构建回归矩阵
```

推荐：2阶Butterworth滤波器，截止频率1-5Hz

---

### 六、公式汇总表

| 步骤 | 方程 | 回归向量 $\phi$ | 参数向量 $\theta$ | 还原公式 |
|------|------|-----------------|-------------------|----------|
| **Surge** | $m_u\dot{u} + d_u u + d_{uu}u\|u\| = \tau_u$ | $[\tau_u, -u, -u\|u\|]^T$ | $[1/m_u, d_u/m_u, d_{uu}/m_u]^T$ | $m_u=1/\theta_1$ |
| **Yaw** | $I_r\dot{r} + d_r r + d_{rr}r\|r\| = \tau_r$ | $[\tau_r, -r, -r\|r\|]^T$ | $[1/I_r, d_r/I_r, d_{rr}/I_r]^T$ | $I_r=1/\theta_1$ |
| **Sway** | $m_v\dot{v} + m_u ur + d_v v + d_{vv}v\|v\| = 0$ | $[-ur, -v, -v\|v\|]^T$ | $[m_u/m_v, d_v/m_v, d_{vv}/m_v]^T$ | $m_v=m_u/\theta_1$ |

---

### 七、实验设计建议

| 步骤 | 实验类型 | 激励信号 | 持续时间 | 注意事项 |
|------|----------|----------|----------|----------|
| Step 1 | 直线航行 | 正弦/阶跃变化的 $\tau_u$ | 60-120s | 保持航向稳定 |
| Step 2 | 低速转向 | 正弦/阶跃变化的 $\tau_r$ | 60-120s | 低速或原地 |
| Step 3 | 曲线航行 | 恒定 $\tau_u$ + 正弦 $\tau_r$ | 120-180s | 确保产生足够的 $v$ |

---

### 八、验证方法

辨识完成后，用辨识参数进行仿真，与实测数据对比：

$$\text{拟合误差} = \frac{\|y_{实测} - y_{仿真}\|}{\|y_{实测}\|} \times 100\%$$

若误差较大，可进行迭代修正或检查数据质量。

---
## 4、欠驱动AUV最小二乘参数辨识（完整版）

---

### 一、完整动力学方程

$$m_u \dot{u} - m_v vr + d_u u + d_{uu}u|u| = \tau_u$$

$$m_v \dot{v} + m_u ur + d_v v + d_{vv}v|v| = 0$$

$$I_r \dot{r} + (m_u - m_v)uv + d_r r + d_{rr}r|r| = \tau_r$$

---

### 二、待辨识参数（确实是9个独立参数）

| 类别 | Surge | Sway | Yaw |
|------|-------|------|-----|
| **惯性** | $m_u$ | $m_v$ | $I_r$ |
| **线性阻尼** | $d_u$ | $d_v$ | $d_r$ |
| **二次阻尼** | $d_{uu}$ | $d_{vv}$ | $d_{rr}$ |

**注意**：耦合项中的 $m_u$、$m_v$ 与惯性项中的是**同一参数**，所以总共还是9个独立参数。

---

### 三、问题分析：为什么不能分步简化？

#### 原方案的问题

| 步骤 | 假设 | 问题 |
|------|------|------|
| Step 1 | $v \approx 0, r \approx 0$ | 实际中难以保证，耦合项 $m_v vr$ 被忽略 |
| Step 2 | $u$ 恒定小，忽略 $(m_u-m_v)uv$ | 耦合项被忽略，且依赖Step 1结果 |
| Step 3 | 利用已知 $m_u$ | 误差累积 |

#### 更好的方案：联合辨识

由于三个方程通过 $m_u$、$m_v$ 相互耦合，应该考虑**联合辨识**或**迭代辨识**。

---

### 四、方案一：分步辨识（改进版）

#### **Step 1：Surge参数辨识**

##### 完整方程
$$m_u \dot{u} = m_v vr - d_u u - d_{uu}u|u| + \tau_u$$

##### 回归形式
$$\dot{u} = \theta_u^T \phi_u$$

$$\theta_u = \begin{bmatrix} m_v/m_u \\ d_u/m_u \\ d_{uu}/m_u \\ 1/m_u \end{bmatrix}, \quad \phi_u = \begin{bmatrix} vr \\ -u \\ -u|u| \\ \tau_u \end{bmatrix}$$

##### 数据矩阵
$$\Phi_u = \begin{bmatrix} v_1 r_1 & -u_1 & -u_1|u_1| & \tau_{u,1} \\ \vdots & \vdots & \vdots & \vdots \\ v_N r_N & -u_N & -u_N|u_N| & \tau_{u,N} \end{bmatrix}$$

##### 参数还原
$$m_u = \frac{1}{\theta_{u4}}, \quad m_v^{(1)} = \frac{\theta_{u1}}{\theta_{u4}}, \quad d_u = \frac{\theta_{u2}}{\theta_{u4}}, \quad d_{uu} = \frac{\theta_{u3}}{\theta_{u4}}$$

---

#### **Step 2：Yaw参数辨识**

##### 完整方程
$$I_r \dot{r} = -(m_u - m_v)uv - d_r r - d_{rr}r|r| + \tau_r$$

##### 回归形式
$$\dot{r} = \theta_r^T \phi_r$$

$$\theta_r = \begin{bmatrix} (m_v-m_u)/I_r \\ d_r/I_r \\ d_{rr}/I_r \\ 1/I_r \end{bmatrix}, \quad \phi_r = \begin{bmatrix} uv \\ -r \\ -r|r| \\ \tau_r \end{bmatrix}$$

##### 数据矩阵
$$\Phi_r = \begin{bmatrix} u_1 v_1 & -r_1 & -r_1|r_1| & \tau_{r,1} \\ \vdots & \vdots & \vdots & \vdots \\ u_N v_N & -r_N & -r_N|r_N| & \tau_{r,N} \end{bmatrix}$$

##### 参数还原
$$I_r = \frac{1}{\theta_{r4}}, \quad (m_v - m_u) = \frac{\theta_{r1}}{\theta_{r4}}, \quad d_r = \frac{\theta_{r2}}{\theta_{r4}}, \quad d_{rr} = \frac{\theta_{r3}}{\theta_{r4}}$$

**交叉验证**：
$$m_v^{(2)} = m_u + \frac{\theta_{r1}}{\theta_{r4}}$$

比较 $m_v^{(1)}$ 和 $m_v^{(2)}$ 是否一致！

---

#### **Step 3：Sway参数辨识**

##### 完整方程
$$m_v \dot{v} = -m_u ur - d_v v - d_{vv}v|v|$$

##### 回归形式
$$\dot{v} = \theta_v^T \phi_v$$

$$\theta_v = \begin{bmatrix} m_u/m_v \\ d_v/m_v \\ d_{vv}/m_v \end{bmatrix}, \quad \phi_v = \begin{bmatrix} -ur \\ -v \\ -v|v| \end{bmatrix}$$

##### 参数还原（利用已知 $m_u$）
$$m_v^{(3)} = \frac{m_u}{\theta_{v1}}, \quad d_v = \frac{\theta_{v2} \cdot m_u}{\theta_{v1}}, \quad d_{vv} = \frac{\theta_{v3} \cdot m_u}{\theta_{v1}}$$

---

#### **一致性检验**

从三个步骤得到 $m_v$ 的三个估计值：

| 来源 | 公式 |
|------|------|
| Step 1 (Surge) | $m_v^{(1)} = \theta_{u1}/\theta_{u4}$ |
| Step 2 (Yaw) | $m_v^{(2)} = m_u + \theta_{r1}/\theta_{r4}$ |
| Step 3 (Sway) | $m_v^{(3)} = m_u/\theta_{v1}$ |

**最终估计**：**$m_v^{(3)}$的权重应该给的高点**
$$\hat{m}_v = \frac{m_v^{(1)} + m_v^{(2)} + m_v^{(3)}}{3}$$

或使用加权平均（根据各估计的置信度）。

---

### 五、方案二：全局联合辨识

将三个方程合并为一个大的回归问题。

#### 统一回归形式

定义状态向量的导数：
$$\mathbf{y} = \begin{bmatrix} \dot{u} \\ \dot{v} \\ \dot{r} \end{bmatrix}$$

#### 参数向量（9个参数）
$$\boldsymbol{\theta} = [m_u, m_v, I_r, d_u, d_v, d_r, d_{uu}, d_{vv}, d_{rr}]^T$$

#### 非线性回归

由于参数以非线性形式出现（如 $m_v/m_u$），需要使用：

1. **非线性最小二乘**（如Levenberg-Marquardt算法）
2. **迭代线性化**

#### 目标函数
$$J(\boldsymbol{\theta}) = \sum_{i=1}^{N} \left[ (\dot{u}_i - f_u)^2 + (\dot{v}_i - f_v)^2 + (\dot{r}_i - f_r)^2 \right]$$

其中：
$$f_u = \frac{m_v vr - d_u u - d_{uu}u|u| + \tau_u}{m_u}$$
$$f_v = \frac{-m_u ur - d_v v - d_{vv}v|v|}{m_v}$$
$$f_r = \frac{-(m_u-m_v)uv - d_r r - d_{rr}r|r| + \tau_r}{I_r}$$

---

### 六、方案三：两阶段迭代辨识

```
┌─────────────────────────────────────────────────────────┐
│                    初始化                                │
│  设定初始猜测值 m_u⁰, m_v⁰, I_r⁰ 等                      │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                 迭代 k = 1, 2, ...                       │
├─────────────────────────────────────────────────────────┤
│ Step A: 固定惯性参数，辨识阻尼参数                        │
│         用 m_u^(k-1), m_v^(k-1), I_r^(k-1)               │
│         线性最小二乘求 d_u, d_v, d_r, d_uu, d_vv, d_rr   │
├─────────────────────────────────────────────────────────┤
│ Step B: 固定阻尼参数，辨识惯性参数                        │
│         用新的阻尼参数                                   │
│         非线性最小二乘求 m_u, m_v, I_r                   │
├─────────────────────────────────────────────────────────┤
│ 检查收敛: |θ^(k) - θ^(k-1)| < ε ?                        │
└─────────────────────────────────────────────────────────┘
                          ↓
                    输出最终参数
```

---

### 七、完整公式汇总

#### 三个动力学方程的回归形式

| 方程 | 回归形式 | 回归向量 $\phi$ | 参数向量 $\theta$ |
|------|----------|-----------------|-------------------|
| **Surge** | $\dot{u} = \theta_u^T \phi_u$ | $[vr, -u, -u\|u\|, \tau_u]^T$ | $[m_v/m_u, d_u/m_u, d_{uu}/m_u, 1/m_u]^T$ |
| **Sway** | $\dot{v} = \theta_v^T \phi_v$ | $[-ur, -v, -v\|v\|]^T$ | $[m_u/m_v, d_v/m_v, d_{vv}/m_v]^T$ |
| **Yaw** | $\dot{r} = \theta_r^T \phi_r$ | $[uv, -r, -r\|r\|, \tau_r]^T$ | $[(m_v-m_u)/I_r, d_r/I_r, d_{rr}/I_r, 1/I_r]^T$ |

#### 参数还原公式

| 参数 | 还原公式 | 来源 |
|------|----------|------|
| $m_u$ | $1/\theta_{u4}$ | Surge |
| $m_v$ | $\theta_{u1}/\theta_{u4}$ 或 $m_u + \theta_{r1}/\theta_{r4}$ 或 $m_u/\theta_{v1}$ | 多源 |
| $I_r$ | $1/\theta_{r4}$ | Yaw |
| $d_u$ | $\theta_{u2}/\theta_{u4}$ | Surge |
| $d_v$ | $\theta_{v2} \cdot m_u/\theta_{v1}$ | Sway |
| $d_r$ | $\theta_{r2}/\theta_{r4}$ | Yaw |
| $d_{uu}$ | $\theta_{u3}/\theta_{u4}$ | Surge |
| $d_{vv}$ | $\theta_{v3} \cdot m_u/\theta_{v1}$ | Sway |
| $d_{rr}$ | $\theta_{r3}/\theta_{r4}$ | Yaw |

---

### 八、实验设计（改进版）

为了保证耦合项的可辨识性，需要设计**激励充分**的实验：

| 实验 | 目的 | 激励信号 | 关键要求 |
|------|------|----------|----------|
| **综合机动** | 同时激励所有自由度 | $\tau_u$: 正弦 + 阶跃<br>$\tau_r$: 正弦（不同频率） | 确保 $u, v, r$ 都有足够变化 |
| **S形机动** | 产生侧向速度 | 恒定 $\tau_u$ + 交替 $\tau_r$ | 多次方向变化 |
| **圆周运动** | 稳态耦合 | 恒定 $\tau_u$ + 恒定 $\tau_r$ | 不同半径的圆 |

---

### 九、总结

您的观察是正确的！完整考虑耦合项后：

1. **Surge方程**：包含 $m_v vr$ 耦合项，回归向量变为4维
2. **Yaw方程**：包含 $(m_u - m_v)uv$ 耦合项，回归向量变为4维
3. **Sway方程**：包含 $m_u ur$ 耦合项，回归向量为3维

**独立参数仍是9个**，但辨识过程需要考虑耦合关系，推荐使用**迭代辨识**或**联合辨识**方法。

这一过程在工程上被称为**航位推算（Dead Reckoning）**或**动力学仿真（Dynamic Simulation）**。

一旦您辨识出了这9个参数（$m_u, m_v, I_r$ 和 6个阻尼系数），您就拥有了AUV的完整**数学模型**。只要给定初始状态和控制输入，您就可以通过**数值积分**的方法，一步步推算出未来的所有状态。

以下是具体的递推流程和实现方法：

---



## 5、通过力和力矩递推速度

### 一、 核心逻辑：从加速度到位置

递推的过程其实就是求解微分方程的过程。计算机无法处理连续时间，所以我们需要将时间离散化（例如每 $\Delta t = 0.1s$ 计算一次）。

**核心步骤如下：**

1.  **算力（Force）**：根据当前速度和输入，计算合外力。
2.  **算加速度（Acceleration）**：利用 $F=ma$（动力学方程）算出 $\dot{u}, \dot{v}, \dot{r}$。
3.  **算速度（Velocity）**：利用 $v = v_0 + at$ 更新体坐标系速度 $u, v, r$。
4.  **算位移（Position）**：利用坐标变换（运动学方程）更新大地坐标系位置 $x, y, \psi$。

---

### 二、 详细递推公式

假设当前时刻是 $k$，下一时刻是 $k+1$，时间步长为 $\Delta t$。

#### 1. 计算加速度（动力学层）

利用您辨识出的9个参数，将动力学方程改写为加速度的显式表达式：

$$
\begin{aligned}
\dot{u}_k &= \frac{1}{m_u} \left( \tau_{u,k} + m_v v_k r_k - d_u u_k - d_{uu} u_k |u_k| \right) \\
\dot{v}_k &= \frac{1}{m_v} \left( \quad 0 \quad - m_u u_k r_k - d_v v_k - d_{vv} v_k |v_k| \right) \\
\dot{r}_k &= \frac{1}{I_r} \left( \tau_{r,k} - (m_u - m_v) u_k v_k - d_r r_k - d_{rr} r_k |r_k| \right)
\end{aligned}
$$

> **注意**：这里用到了欠驱动特性，Sway方向输入为 0。

#### 2. 更新体坐标系速度

使用欧拉积分法（最简单）或 龙格-库塔法（RK4，更精确）：

$$
\begin{aligned}
u_{k+1} &= u_k + \dot{u}_k \cdot \Delta t \\
v_{k+1} &= v_k + \dot{v}_k \cdot \Delta t \\
r_{k+1} &= r_k + \dot{r}_k \cdot \Delta t
\end{aligned}
$$

#### 3. 计算大地坐标系速度（运动学层）

需要将体坐标系速度转换到大地坐标系：

$$
\begin{aligned}
\dot{x}_k &= u_k \cos(\psi_k) - v_k \sin(\psi_k) \\
\dot{y}_k &= u_k \sin(\psi_k) + v_k \cos(\psi_k) \\
\dot{\psi}_k &= r_k
\end{aligned}
$$

#### 4. 更新位置和姿态

$$
\begin{aligned}
x_{k+1} &= x_k + \dot{x}_k \cdot \Delta t \\
y_{k+1} &= y_k + \dot{y}_k \cdot \Delta t \\
\psi_{k+1} &= \psi_k + \dot{\psi}_k \cdot \Delta t
\end{aligned}
$$

---
