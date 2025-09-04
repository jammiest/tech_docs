# 流体力学

### **流体力学理论体系概述**

流体力学（Fluid Mechanics）是研究流体（液体和气体）运动规律及其与边界相互作用的学科，其理论体系可分为**经典流体力学**和**现代流体力学**两大分支。以下是系统性阐述：

---

## **一、基本概念与分类**
### **1. 流体定义**
- **液体**：不可压缩性显著（密度变化$\leq 5\%$），有自由表面。
- **气体**：可压缩性强，无固定形状。

### **2. 研究范畴**
| **分类**          | **研究对象**                     | **典型问题**                  |
|--------------------|----------------------------------|-------------------------------|
| 流体静力学         | 静止流体中的压力分布             | 帕斯卡原理、浮力              |
| 流体动力学         | 运动流体的力学行为               | 纳维-斯托克斯方程、湍流       |
| 空气动力学         | 气体与运动物体的相互作用         | 机翼升力、激波形成            |
| 多相流             | 不同相态流体的混合流动           | 气泡动力学、颗粒悬浮          |

---

## **二、控制方程（经典流体力学核心）**
### **1. 连续性方程（质量守恒）**
$$ \frac{\partial \rho}{\partial t} + \nabla \cdot (\rho \mathbf{u}) = 0 $$
- **不可压缩流体**（$\rho = \text{常量}$）：  
  $$ \nabla \cdot \mathbf{u} = 0 $$

### **2. 纳维-斯托克斯方程（动量守恒）**
$$ \rho \left( \frac{\partial \mathbf{u}}{\partial t} + \mathbf{u} \cdot \nabla \mathbf{u} \right) = -\nabla p + \mu \nabla^2 \mathbf{u} + \mathbf{f} $$
- **符号说明**：  
  - $\mathbf{u}$：速度场  
  - $p$：压强  
  - $\mu$：动力粘度  
  - $\mathbf{f}$：体积力（如重力$\rho \mathbf{g}$）

### **3. 能量方程（热力学第一定律）**
$$ \rho c_p \left( \frac{\partial T}{\partial t} + \mathbf{u} \cdot \nabla T \right) = \nabla \cdot (k \nabla T) + \Phi + Q $$
- $\Phi$：粘性耗散项，$Q$：热源项。

---

## **三、无量纲数（流动特性表征）**
| **数名**         | **公式**                          | **物理意义**                  |
|-------------------|-----------------------------------|-------------------------------|
| 雷诺数（Re）      | $Re = \frac{\rho U L}{\mu}$      | 惯性力与粘性力之比            |
| 马赫数（Ma）      | $Ma = \frac{U}{c}$               | 流速与声速之比（可压缩性）    |
| 弗劳德数（Fr）    | $Fr = \frac{U}{\sqrt{gL}}$       | 惯性力与重力之比              |
| 普朗特数（Pr）    | $Pr = \frac{c_p \mu}{k}$         | 动量扩散与热扩散之比          |

---

## **四、典型流动解析**
### **1. 层流与湍流**
- **层流**：$Re < 2300$（圆管流），流线有序。
- **湍流**：$Re > 4000$，存在涡旋和脉动，需用**雷诺平均方程（RANS）**：  
  $$ \rho \frac{\partial \bar{u}_i}{\partial t} + \rho \bar{u}_j \frac{\partial \bar{u}_i}{\partial x_j} = -\frac{\partial \bar{p}}{\partial x_i} + \frac{\partial}{\partial x_j} \left( \mu \frac{\partial \bar{u}_i}{\partial x_j} - \rho \overline{u'_i u'_j} \right) $$
  （$\overline{u'_i u'_j}$为雷诺应力）

### **2. 边界层理论**
- **普朗特边界层方程**（二维不可压缩）：  
  $$ u \frac{\partial u}{\partial x} + v \frac{\partial u}{\partial y} = -\frac{1}{\rho} \frac{\partial p}{\partial x} + \nu \frac{\partial^2 u}{\partial y^2} $$
- **位移厚度**：  
  $$ \delta^* = \int_0^\infty \left(1 - \frac{u}{U}\right) dy $$

### **3. 可压缩流动**
- **等熵流动关系**（理想气体）：  
  $$ \frac{T_0}{T} = 1 + \frac{\gamma - 1}{2} Ma^2 $$
  （$T_0$：驻点温度，$\gamma$：比热比）

---

## **五、数值方法（计算流体力学，CFD）**
### **1. 离散方法**
- **有限体积法（FVM）**：  
  $$ \int_V \frac{\partial \mathbf{U}}{\partial t} dV + \oint_S \mathbf{F} \cdot d\mathbf{S} = \int_V \mathbf{Q} dV $$
  （$\mathbf{U}$：守恒量，$\mathbf{F}$：通量，$\mathbf{Q}$：源项）

### **2. 湍流模型**
- **$k$-$\epsilon$模型**：  
  $$ \frac{\partial (\rho k)}{\partial t} + \nabla \cdot (\rho k \mathbf{u}) = \nabla \cdot \left[ \left(\mu + \frac{\mu_t}{\sigma_k}\right) \nabla k \right] + P_k - \rho \epsilon $$
  （$k$：湍动能，$\epsilon$：耗散率）

---

## **六、前沿领域**
### **1. 微流体与纳米流体**
- **滑移边界条件**：  
  $$ u_{\text{wall}} = L_s \frac{\partial u}{\partial y} \bigg|_{\text{wall}} $$
  （$L_s$：滑移长度）

### **2. 非牛顿流体**
- **本构方程**（幂律模型）：  
  $$ \tau = K \dot{\gamma}^n $$
  （$n < 1$：剪切稀化，$n > 1$：剪切增稠）

### **3. 量子流体**
- **超流氦（He-II）**：  
  $$ \mathbf{v}_s = \frac{\hbar}{m} \nabla \phi $$
  （$\phi$：宏观波函数相位）

---

## **七、应用领域**
| **领域**         | **典型问题**                      |
|------------------|-----------------------------------|
| 航空航天         | 机翼绕流、激波-边界层相互作用     |
| 能源工程         | 涡轮机械内流、燃烧室湍流混合      |
| 生物医学         | 血液流动、人工心脏瓣膜设计        |
| 环境科学         | 大气环流、海洋洋流模拟            |

---

### **总结**
流体力学通过**连续介质假设**和**守恒定律**构建了描述流动的数学框架，其理论从经典的纳维-斯托克斯方程扩展到现代的多尺度模拟（如分子动力学与CFD耦合）。未来挑战包括**高雷诺数湍流的直接数值模拟（DNS）**和**复杂流体的跨尺度建模**。