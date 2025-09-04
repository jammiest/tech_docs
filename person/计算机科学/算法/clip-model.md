# CLIP（Contrastive Language-Image Pretraining）模型详解

## 1. 模型概述

CLIP（Contrastive Language-Image Pretraining）是OpenAI提出的多模态模型，通过对比学习将图像和文本映射到同一语义空间。

### 核心思想
$$
\text{最大化匹配的图像-文本对的相似度，最小化不匹配对的相似度}
$$

## 2. 模型架构

### 双编码器结构
$$
\begin{aligned}
\text{图像编码器} & : f_I \colon \mathbb{R}^{H×W×3} → \mathbb{R}^d \\
\text{文本编码器} & : f_T \colon \mathbb{R}^L → \mathbb{R}^d
\end{aligned}
$$

- **图像编码器**：可选ResNet或ViT架构
- **文本编码器**：基于Transformer

## 3. 训练目标

### 对比损失函数
$$
\mathcal{L} = -\frac{1}{N}\sum_{i=1}^N \left[\log \frac{e^{s(I_i,T_i)/\tau}}{\sum_{j=1}^N e^{s(I_i,T_j)/\tau}} + \log \frac{e^{s(T_i,I_i)/\tau}}{\sum_{j=1}^N e^{s(T_j,I_i)/\tau}}\right]
$$

其中：
- $s(I,T) = f_I(I)^⊤ f_T(T)$ 是图像文本相似度
- $\tau$ 是温度参数

## 4. 关键数学原理

### 相似度矩阵
对于batch size为N的样本：
$$
S = \begin{bmatrix}
s(I_1,T_1) & s(I_1,T_2) & \cdots & s(I_1,T_N) \\
s(I_2,T_1) & s(I_2,T_2) & \cdots & s(I_2,T_N) \\
\vdots & \vdots & \ddots & \vdots \\
s(I_N,T_1) & s(I_N,T_2) & \cdots & s(I_N,T_N)
\end{bmatrix}
$$

### 对称交叉熵损失
$$
\mathcal{L}_I = -\frac{1}{N}\sum_{i=1}^N \log \frac{e^{s(I_i,T_i)/\tau}}{\sum_{j=1}^N e^{s(I_i,T_j)/\tau}}
$$
$$
\mathcal{L}_T = -\frac{1}{N}\sum_{i=1}^N \log \frac{e^{s(T_i,I_i)/\tau}}{\sum_{j=1}^N e^{s(T_j,I_i)/\tau}}
$$
$$
\mathcal{L} = \frac{\mathcal{L}_I + \mathcal{L}_T}{2}
$$

## 5. 零样本分类

给定图像$x$和类别标签$\{y_1,...,y_k\}$：
$$
p(y_k|x) = \frac{e^{f_I(x)^⊤ f_T(t_k)/\tau}}{\sum_{i=1}^K e^{f_I(x)^⊤ f_T(t_i)/\tau}}
$$

其中$t_k$是模板文本如"a photo of a {y_k}"

## 6. 模型优势

1. **零样本能力**：无需特定任务微调
2. **多模态对齐**：共享的语义空间
3. **可扩展性**：适用于多种视觉任务

## 7. 应用示例

```python
import clip
import torch

model, preprocess = clip.load("ViT-B/32")
image = preprocess(Image.open("image.jpg")).unsqueeze(0)
text = clip.tokenize(["a dog", "a cat", "a bird"])

with torch.no_grad():
    image_features = model.encode_image(image)
    text_features = model.encode_text(text)
    logits = (image_features @ text_features.T).softmax(dim=-1)
```

CLIP通过大规模对比学习实现了强大的跨模态理解能力，为多模态研究开辟了新方向。