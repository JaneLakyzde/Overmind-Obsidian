---
aliases:
  - sklearn
tags:
  - Python
---
`Scikit-learn` 是构建于 `NumPy`, `SciPy`, `Matplotlib` 之上的知名机器学习库，以其一致且简洁的 API 设计，在机器学习领域被广泛使用[](https://www.ibm.com/cn-zh/think/topics/scikit-learn#2086344953)。

- **主要功能**
    
    - **丰富的算法库**：几乎涵盖所有经典算法，如线性/逻辑回归、SVM、决策树、K-Means、PCA等
        
    - **数据预处理工具**：提供标准化、归一化、编码分类特征等全套数据准备工具
        
    - **模型选择与评估**：内置交叉验证、网格搜索、混淆矩阵等功能，用于模型调优和性能评估
        
    - **统一的Estimator API**：所有模型都遵循 `fit()`/`predict()` 的统一接口，学习成本极低，便于快速原型开发
        
- **核心概念**  
    `Estimator`（估计器）是其核心概念，任何可以基于数据学习参数的算法对象都被称为Estimator
