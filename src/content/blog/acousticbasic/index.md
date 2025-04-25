---
title: PMUT声学基础
publishDate: 2025-02-17
description: '解释了PMUT领域涉及的声学名词尤其是阻抗匹配'
tags:
  - Acoustic
  - PMUT
  - Impedance Matching
heroImage: { src: './acoustic.jpeg', color: '#4891B2' }
language: '中文'
---
# 基本原理

## 声学基础

由于基础物理课非常忽视声学，这里有必要在学习 PMUT 之前补一下声学的相关知识。

### 一些声学名词

解释声学名词之前，需要先列一下流体的波动方程，对流体的一小块受力分析，其中 $u$ 为质点位移，$v$ 为质点振速，$p$ 为声压（相对静态压强的变化量），$\rho$ 为介质密度：

$$
p|_{x}- p|_{x+dx}=\rho \ dx \ \frac{\partial v}{\partial t}
$$

化简得到 $p$ 与 $v$ 的关系：

$$
\frac{\partial p}{\partial x}+\rho \ \frac{\partial v}{\partial t}=0
$$

对于绝热模型，$P_{tot}V^{\gamma}=Const$，则：

$$
\frac{dP_{tot}}{P_{tot}}+\frac{\gamma dV}{V}=0
$$

注意这里 $P_{tot}$ 是总压强（为声压 + 静态压强，在一阶近似下即声压远小于静态压强时等于静态压强），$dP_{tot}$ 是相对静态压强的变化量（一阶近似下就是 $p$），则声压 $p$ 满足：

$$
p=-P_{tot}\frac{\gamma dV}{V}=-\gamma P_{tot}\frac{\partial u}{\partial x}
$$

代入声压 $p$ 得到波动方程：

$$
\frac{\partial^{2} u}{\partial t^{2}}+\frac{\gamma P_{tot}}{\rho}\frac{\partial^{2} u}{\partial x^{2}}=0
$$

这里可以看出气体绝热模型下声速 $c=\sqrt{\frac{\gamma P_{tot}}{\rho}}$。此外，根据声压和质点振速关系可得质点振速的表达式：

$$
v=-\int \frac{1}{\rho}\frac{\partial p}{ \partial x}dt
$$

---

**1. 声压（Sound Pressure, SP）：**  
声波作用下流体中的局部压力变化，注意声压一般指交变量，即相对于静态压力（一般为大气压）的变化量部分。公式计算如下，其中 $v$ 为质点震速：

$$
p = \rho c v
$$

推导如下，将波动方程的解 $p=p_{0}e^{j(\omega t-kx)}$ 代入声压和质点振速关系得：

$$
v=-\int \frac{\partial p}{\rho \partial x}dt=\int \frac{kp_{0}}{\rho}e^{j(\omega t-kx)}dt= \frac{kp_{0}}{\rho \omega}e^{j(\omega t-kx)}=\frac{p}{\rho c}
$$

---

**2. 声压级（Sound Pressure Level, SPL）：**  
声压级是描述声压的对数量级，通常相对于基准声压（人耳听阈）进行归一化，单位为分贝（dB）。计算公式如下，其中基准声压 $p_{0}=20\mu Pa$：

$$
L_{p} = 20 \log_{10} \left( \frac{p}{p_0} \right)
$$

从这个式子可以看出当人耳听到声压 $p_{0}=20\mu Pa$ 时声压级为 0dB，为人耳最低可听分贝。

---

**3. 相对声压级：**  
两个声压值的相对比值，以分贝表示，计算公式如下：

$$
\Delta L_{p} = 20 \log_{10} \left( \frac{p_{1}}{p_{2}} \right)
$$

---

**4. 声强（Sound Intensity）：**  
单位面积上通过的声功率，即平均声能量流密度。声强是矢量，方向与波的传播方向一致，单位 $W/m^{2}$，公式计算如下：

$$
I=\frac{P_{\text{功率}}}{A}=\frac{\bar{p^{2}}}{\rho c}
$$

推导如下，声能量流密度意味着单位面积声压所做功的平均值：

$$
I=\frac{1}{T}\int_{0}^{T}pvdt=\frac{1}{T}\int_{0}^{T}p\frac{p}{\rho c}dt=\frac{\bar{p^{2}}}{\rho c}
$$

---

**5. 声强级（Sound Intensity Level, SIL）：**  
声强级是描述声强的对数量级，参考声强 $I_{0}=10^{-12}W/m^{2}$，计算公式如下：

$$
L=10\ \log_{10}\left(\frac{I}{I_{0}}\right)
$$

---

**6. 声功率：**  
声源单位时间内辐射出的总能量，单位为瓦（$W$），也是一定面积所通过的声能量，根据声强部分公式可以得到声功率公式。

---

**7. 声阻（Sound Impedance）：**  
声阻是介质对声波传播的阻碍程度，定义为声压和质点振速的比值，单位为 Rayl：

$$
Z=\frac{p}{v}=\rho c
$$

> 注：这对于任意波形都是成立的，虽然推导采用的特定频率复数正余弦波，但是任意波形可以展开成正余弦波求和。另外考虑耗散的介质时，声阻为复数（相当于乘了 $e^{j\beta x}$）。

---

### 声阻匹配

#### 1. 折射与透射系数

考虑声波从声阻 $Z_{1}$ 射入声阻 $Z_{2}$ 的材料，入射波 $p_{i}=p_{0i}e^{j(\omega t-kx)}$，反射波 $p_{r}=p_{0r}e^{j(\omega t+kx)}$，折射波 $p_{t}=p_{0t}e^{j(\omega t-kx)}$，质点速度分别为：

- $v_{i}=\frac{p_{i}}{Z_{1}}$
- $v_{r}=\frac{p_{r}}{Z_{1}}$
- $v_{t}=\frac{p_{t}}{Z_{2}}$

声压连续条件：

$$
p_{i}+p_{r}=p_{t}
$$

速度连续条件（假设 $S_{1}=S_{2}$）：

$$
v_{i}-v_{r}=v_{t}
$$

代入 $x=0$ 处声压和速度条件解得：

$$
p_{r}=\frac{Z_{2}-Z_{1}}{Z_{2}+Z_{1}}p_{i},\quad p_{t}=\frac{2Z_{2}}{Z_{1}+Z_{2}}p_{i}
$$

得到反射系数：

$$
R=\frac{p_{r}}{p_{i}}=\frac{Z_{2}-Z_{1}}{Z_{2}+Z_{1}}
$$

---

#### 2. 透射匹配

透射系数为：

$$
T=\frac{2Z_{2}}{Z_{1}+Z_{2}}
$$

若换能器直接贴敷在介质（水、空气或生物组织），因为换能器声阻 $Z_{1}$ 很大而介质声阻 $Z_{2}$ 很小，导致透射率低。此时需要在两者间添加一个匹配层，设：

- 换能器声阻 $Z_{p}$
- 匹配层声阻 $Z_{m}$
- 工作介质声阻 $Z_{2}$

透射系数变为：

$$
T=\frac{2Z_{m}}{Z_{m}+Z_{p}}\cdot\frac{2Z_{2}}{Z_{m}+Z_{2}}=\frac{4Z_{2}}{Z_{m}+\frac{Z_{p}Z_{2}}{Z_{m}}+Z_{p}+Z_{2}}
$$

当 $Z_{m}=\sqrt{Z_{p}Z_{2}}$ 时透射率最大：

$$
T=\frac{4Z_{2}}{(\sqrt{Z_{p}}+\sqrt{Z_{2}})^2}
$$

---

#### 3. 具体实施例

在 Lu et al. 的经典 PMUT 指纹传感器论文中[^1]，PMUT 上的耦合层声阻为 $Z_{1} = 1.3$ MRayl。对比：

- 指纹隆起部分（皮组织）$Z_{2}=1.5$ MRayl，反射率 $R_{ridge}=7\%$
- 指纹凹陷部分（空气）$Z_{2}=430$ Rayl，反射率 $R_{valley}\simeq 100\%$

因此 PMUT 发出的超声波在遇到指纹凹陷部分时几乎完全反射，而隆起部分由于面积更大（为凹陷部分的三倍），散布损失为 $\frac{1}{3}$。最终对比度约为：

$$
5:1=(100\%):(7\% \times 3)
$$

---

[^1]: Y. Lu, et al., "Ultrasonic fingerprint sensor using a piezoelectric micromachined ultrasonic transducer array integrated with complementary metal oxide semiconductor electronics," *Applied Physics Letters*, 2015.