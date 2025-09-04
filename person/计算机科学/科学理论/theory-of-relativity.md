# 相对论

作为理论物理的核心理论之一，相对论可分为狭义相对论(1905)和广义相对论(1915)两大体系。以下从数学表述和物理内涵两个维度进行系统阐述：

### 一、狭义相对论 (Special Relativity)
**基本假设**：
1. 相对性原理：物理定律在所有惯性参考系中形式相同
2. 光速不变原理：真空光速$c$在各惯性系中数值相同

**核心数学结构**：
1. 闵可夫斯基时空：
$$ds^2 = -c^2dt^2 + dx^2 + dy^2 + dz^2 = \eta_{\mu\nu}dx^\mu dx^\nu$$
其中度规张量$\eta_{\mu\nu}=\text{diag}(-1,1,1,1)$

2. Lorentz变换：
$$
\begin{cases}
t' = \gamma(t - \frac{vx}{c^2}) \\
x' = \gamma(x - vt) \\
y' = y \\
z' = z
\end{cases}
$$
其中$\gamma = \frac{1}{\sqrt{1-v^2/c^2}}$

3. 速度合成公式：
$$u' = \frac{u - v}{1 - uv/c^2}$$

**重要推论**：
1. 时间延缓：$\Delta t = \gamma\Delta\tau$
2. 长度收缩：$L = L_0/\gamma$
3. 质能关系：$E = \gamma mc^2$，静止能量$E_0 = mc^2$
4. 四维动量：$p^\mu = (E/c, \vec{p})$

### 二、广义相对论 (General Relativity)
**基本假设**：
1. 等效原理：惯性质量=引力质量
2. 广义协变原理：物理定律在任意坐标变换下形式不变

**数学框架**：
1. 黎曼几何：
$$ds^2 = g_{\mu\nu}dx^\mu dx^\nu$$
曲率张量：
$$R^\rho_{\ \sigma\mu\nu} = \partial_\mu\Gamma^\rho_{\nu\sigma} - \partial_\nu\Gamma^\rho_{\mu\sigma} + \Gamma^\rho_{\mu\lambda}\Gamma^\lambda_{\nu\sigma} - \Gamma^\rho_{\nu\lambda}\Gamma^\lambda_{\mu\sigma}$$

2. Einstein场方程：
$$G_{\mu\nu} = \frac{8\pi G}{c^4}T_{\mu\nu}$$
其中爱因斯坦张量：
$$G_{\mu\nu} = R_{\mu\nu} - \frac{1}{2}Rg_{\mu\nu}$$

**典型解**：
1. Schwarzschild解：
$$ds^2 = -\left(1-\frac{2GM}{c^2r}\right)c^2dt^2 + \left(1-\frac{2GM}{c^2r}\right)^{-1}dr^2 + r^2(d\theta^2 + \sin^2\theta d\phi^2)$$

2. 引力时间延缓：
$$\frac{d\tau}{dt} = \sqrt{1-\frac{2GM}{c^2r}}$$

### 三、实验验证
1. 狭义相对论：
- μ子寿命延长（时间膨胀）
- 粒子加速器中的质量-速度关系

2. 广义相对论：
- 水星近日点进动（43"/世纪）
- 光线偏折（1.75"）
- 引力波探测（LIGO）

### 四、现代发展
1. 相对论量子场论：
$$S = \int d^4x \sqrt{-g} \mathcal{L}(\psi,\nabla_\mu\psi)$$

2. 黑洞热力学：
$$T_H = \frac{\hbar c^3}{8\pi GMk_B}$$

该理论体系深刻改变了人类对时空本质的认知，其数学严谨性和实验精确性已达到$10^{-15}$量级的验证精度。需要更深入的探讨可具体指出方向。