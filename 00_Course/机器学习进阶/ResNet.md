---
aliases:
  - Residual Network
  - 残差网络
tags:
  - CNN
---

## 一、先讲个故事：网络越深，为啥反而变笨了？

想象你要组装一个超级复杂的乐高城堡。你一层一层往上搭，理论上层数越多，城堡应该越精致。但实际情况是：

- 搭到 20 层时，城堡还挺像样。
    
- 搭到 50 层时，你发现有些积木拼错了，但已经改不了底下的部分，导致上面越搭越歪。
    
- 搭到 100 层时，城堡直接塌了，甚至还不如 20 层的那个好看。
    

在神经网络里，这就叫**退化问题**：网络层数增加，训练误差不但不降，反而升高。不是过拟合（过拟合是训练好但测试差），而是连训练集上都变差。原因在于深层网络很难把信息原封不动地传到后面，梯度要么消失要么爆炸，优化变得极度困难。

这时候，ResNet 想出了一个天才般的捷径。

---

## 二、ResNet 的核心魔法：抄近道（残差连接）

ResNet 全称是 **Residual Network（残差网络）**，是 2015 年微软亚洲研究院的何恺明等人提出的。它的核心思想用一句话就能概括：

**让网络学习“差多少”，而不是从头学全部。**

什么意思？我打个比方：

你要画一张肖像画，已经有一张草图了，你要把它修改成最终稿。传统方法是撕掉重新画，而 ResNet 的方法是：在草图旁边放一张透明纸，你只画“需要修改的部分”，然后把两张叠在一起，就得到了完美的成品。

在数学上，假设网络的某一层原本要学一个复杂的映射 ( H(x) )，ResNet 让它改学 **残差** ( F(x) = H(x) - x )，那么最终的输出就是 ( F(x) + x )。这里的 ( x ) 就是输入（也就是那张“草图”），它通过一条**跳跃连接（shortcut connection）** 直接抄近道加到输出上。

这样一来，网络不必重新发明轮子，只需要学习“输入和理想输出之间差的那一点点”。如果某一层啥都不做是最优的，它直接把残差学成 0，让输入 ( x ) 原封不动传下去就行。这就轻松解决了网络加深时的退化问题。

---

## 三、ResNet 长什么样？——搭积木的视角

ResNet 的基本单元叫 **残差块（Residual Block）**，长得像这样：

输入 x ——> 卷积层 ——> 激活 ——> 卷积层 ——> 输出 F(x)  
  |                                            |  
  |_______________ 直接相加 ____________________|  
                         |  
                    最终输出 F(x) + x

把它想象成一个高速公路服务区：

- 主路（跳跃连接）直接穿过去，车辆不用停。
    
- 匝道上的服务站（两个卷积层）可以对部分车辆进行加油、维修（学习残差）。
    
- 最终，原车流和整修完的车流汇合，继续前进。
    

多个残差块堆叠起来，就变成了 ResNet-18、ResNet-34、ResNet-50、ResNet-101、ResNet-152……数字代表网络的总层数。50层起，为了节省计算量，会用一种叫“瓶颈块”的变体（1x1卷积降维 → 3x3卷积 → 1x1卷积升维），但核心思想完全不变。

---

## 四、为什么要用 ResNet？它牛在哪？

1. **可以训练非常深的网络**：因为有抄近道，152层的网络也能顺畅训练，梯度直接顺着跳跃连接传回去，不会消失。
    
2. **效果拔群**：2015 年一举拿下 ImageNet 图像识别、检测、定位等多项冠军，错误率比人类还低。
    
3. **泛化性强**：不仅图像分类，在目标检测、语义分割、甚至自然语言处理、语音合成里，ResNet 都作为骨干网络被广泛使用。
    
4. **简单易实现**：PyTorch、TensorFlow 等框架都有预装好的 ResNet，三行代码就能调用。
    

---

## 五、小白如何上手使用 ResNet？（保姆级步骤）

假设你想用 ResNet 做一个猫狗图像分类，用 Python 和 PyTorch，步骤如下：

### 第 0 步：装环境

pip install torch torchvision matplotlib

### 第 1 步：直接下载预训练模型（重点！）

你完全不需要从零训练一个 ResNet，直接站在巨人肩膀上：

import torchvision.models as models  
​  
# 下载预训练好的 ResNet18（在 ImageNet 上训练过，能认出 1000 类物体）  
model = models.resnet18(pretrained=True)

这个模型已经学会了识别边边角角、纹理形状等通用特征，你要做的只是“微调”。

### 第 2 步：改造最后一层，适应你的任务

ImageNet 输出 1000 类，猫狗分类只要 2 类，所以把最后的全连接层换掉：

import torch.nn as nn  
​  
# 获取原模型最后一层的输入特征数  
num_features = model.fc.in_features  
# 换成新的分类层：输出 2 类  
model.fc = nn.Linear(num_features, 2)

### 第 3 步：准备你的猫狗图片

把数据按如下文件夹放好：

data/  
  train/  
    cat/   (里面全是猫图)  
    dog/   (里面全是狗图)  
  val/  
    cat/  
    dog/

用 `torchvision.datasets.ImageFolder` 自动读取，一行代码搞定标签。

### 第 4 步：训练（其实就是微调）

import torch.optim as optim  
​  
# 定义损失函数和优化器  
criterion = nn.CrossEntropyLoss()  
optimizer = optim.Adam(model.parameters(), lr=0.001)  
​  
# 训练几个 epoch  
for epoch in range(5):  
    for images, labels in train_loader:  
        outputs = model(images)  
        loss = criterion(outputs, labels)  
        optimizer.zero_grad()  
        loss.backward()  
        optimizer.step()

因为预训练模型已经很棒，通常只需 5-10 个 epoch，学习率调小一点，就能得到不错的结果。

### 第 5 步：预测新图片

from PIL import Image  
from torchvision import transforms  
​  
# 预处理单张图片  
transform = transforms.Compose([  
    transforms.Resize((224, 224)),  
    transforms.ToTensor(),  
])  
img = Image.open('my_cat.jpg')  
input_tensor = transform(img).unsqueeze(0)  
​  
# 推理  
model.eval()  
with torch.no_grad():  
    output = model(input_tensor)  
_, predicted = torch.max(output, 1)  
print('猫' if predicted.item()==0 else '狗')

---

## 六、入门路线图（从零到能跑通）

如果你是完全零基础，建议按这个顺序走：

1. **学一点 Python 和 PyTorch 基础**：能写简单的张量运算、数据加载、训练循环。
    
2. **理解 CNN 基础**：知道卷积、池化、全连接层是干嘛的，能搭一个 LeNet 级别的网络。
    
3. **手敲一个最简单的 ResNet**：用 PyTorch 自己实现一个残差块，理解跳跃连接是怎么把输入加到输出上的。代码不超过 20 行，但理解后会豁然开朗。
    
4. **跑通上面微调的猫狗分类**：体验“三行代码调模型”的快感，建立信心。
    
5. **读原论文（可选）**：《Deep Residual Learning for Image Recognition》，看图不看公式也行，论文里的对比实验会让你深刻理解它的好。
    
6. **尝试不同的 ResNet 变体**：ResNet-34、50、101，看看参数量和效果的关系。
    
7. **把 ResNet 当骨干，做更酷的事**：比如目标检测（Faster R-CNN + ResNet）或者图像分割。
    

---

## 七、常见误区排雷

- **“ResNet 只能用于图像”**：错。残差连接是一种通用设计，语音识别、NLP（Transformer里也有残差）都有它的影子。
    
- **“预训练权重必须和我的任务一模一样”**：不用。ImageNet 的预训练模型提取的特征非常通用，即使你识别医学影像或卫星图，微调也远比从零训练好。
    
- **“层数越深就一定越好”**：不一定。对于小数据集，ResNet-18 或 34 往往就够，更深容易过拟合，除非你有海量数据。
    

---

把 ResNet 看成是一个**自带记忆补丁的网络**，它的核心无非就是“把输入跳着传过去，只学剩下的那点差值”。一旦你理解了这个，就能自信地把它当作一个强大的基础工具，去解决各种实际问题了。动手跑一遍，你就全明白了！