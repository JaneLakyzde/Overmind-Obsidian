---
tags:
  - OOP
  - cpp
date: 2026-01-06
status: 🟢 已掌握
aliases:
  - 类
---
# 📘 C++ 类基础概念详解 (Class & OOP)

> [!SUMMARY] 核心概念速览
> **类 (Class)** 是蓝图，**对象 (Object)** 是具体的实例。面向对象编程 (OOP) 的三大支柱是：**[[封装]]** (Encapsulation)、**[[继承]]** (Inheritance) 和 **[[多态]]** (Polymorphism)。

## 1. 什么是类与对象？

### 概念类比
* **类 (Class)**：就像一张 **汽车设计图纸**。它规定了汽车应该有轮子（属性）和能跑（行为），但它本身不能开。
* **对象 (Object)**：就像根据图纸造出来的 **一辆具体的红色法拉利**。它是实实在在存在的。

### 代码结构
```cpp
// 1. 定义类（蓝图）
class Fruit {
public:
    int weight; // 属性：重量
    
    void introduce() { // 行为：自我介绍
        cout << "我是一个水果" << endl;
    }
};

// 2. 创建对象（具体的实例）
Fruit apple; 
apple.weight = 10;
apple.introduce();
````

---

## 2. 成员变量与成员函数

- **成员变量 (Member Variables)**：描述对象的**状态**或特征。
    
    - _例子_：`weight` (重量), `sugar_content` (含糖量)。
        
- **成员函数 (Member Functions)**：描述对象能做什么**动作**。
    
    - _例子_：`display()` (显示信息), `eat()` (吃)。
        

---

## 3. 访问权限 (Access Modifiers)

这是实现 **[[封装]]** 的关键。控制谁能看到房间里的东西。

|**关键字**|**权限范围**|**生活类比**|**谁能访问？**|
|---|---|---|---|
|**`public`**|公有|房子的**门铃**|任何人 (外部代码)|
|**`protected`**|保护|家族的**传家宝**|自己 + 孩子 (子类)|
|**`private`**|私有|自己的**日记本**|只有自己 (类内部)|

> [!TIP] 最佳实践
> 
> 通常将变量设为 private 或 protected，将需要给外部调用的函数设为 public。

---

## 4. 构造与析构 (Lifecycle)

对象的“出生”和“死亡”。

- **构造函数 (Constructor)**：`ClassName()`
    
    - _作用_：对象创建时**自动调用**。就像买新手机开机时的“设置向导”，用来初始化数据。
        
- **析构函数 (Destructor)**：`~ClassName()`
    
    - _作用_：对象销毁时**自动调用**。就像搬家前打扫卫生，用来释放内存或资源。
        

> [!example] 调用顺序
> 
> 创建时：先父类构造 -> 后子类构造 (先打地基，再盖房)
> 
> 销毁时：先子类析构 -> 后父类析构 (先拆房顶，再挖地基)

---

## 5. 继承 (Inheritance)

允许我们在现有类的基础上创建新类。

- **语法**：`class Apple : public Fruit`
    
- **含义**：Apple **Is-a** (是一种) Fruit。
    

代码段

```c++
classDiagram
    class Fruit {
        #weight
        +display()
    }
    class Apple {
        -sugar
        +display()
    }
    class Pear {
        -water
        +display()
    }
    Fruit <|-- Apple : 继承
    Fruit <|-- Pear : 继承
```

---

## 6. 多态与虚函数 (Polymorphism)

**多态**：同一个指令，不同的对象做出不同的反应。

- **关键**：使用 `virtual` 关键字修饰父类的函数。
    
- **场景**：当你用 `Fruit*` (父类指针) 指向 `Apple` (子类对象) 时，系统需要知道调用谁的函数。
    

> [!important] 虚函数的作用
> 
> 如果没有 virtual，编译器只会看指针类型 (Fruit)，调用 Fruit 的方法。
> 
> 加上 virtual，编译器会在运行时看实际对象 (Apple)，调用 Apple 的方法。

---

## 7. 综合代码演示

复制以下代码到 IDE 中运行，配合注释理解。

```c++
#include <iostream>
using namespace std;

// ================= 父类：水果 =================
class Fruit {
protected:
    int weight; // 保护属性：只有水果家族能访问

public:
    // 构造函数
    Fruit(int w) : weight(w) {
        cout << "📦 [Fruit] 构造函数: 创建一个重量为 " << w << " 的水果" << endl;
    }

    // 虚析构函数 (重要！确保子类能被正确销毁)
    virtual ~Fruit() {
        cout << "♻️ [Fruit] 析构函数: 水果被销毁" << endl;
    }

    // 虚函数：允许子类覆盖这个行为
    virtual void display() {
        cout << "我是水果，重量: " << weight << endl;
    }
};

// ================= 子类：苹果 =================
class Apple : public Fruit {
private:
    int sugar; // 私有属性：只有苹果自己知道甜度

public:
    // 初始化列表：先调用父类构造(w)，再初始化自己的sugar(s)
    Apple(int s, int w) : Fruit(w), sugar(s) {
        cout << "🍎 [Apple] 构造函数: 这是一个甜度为 " << s << " 的苹果" << endl;
    }

    ~Apple() {
        cout << "🍂 [Apple] 析构函数: 苹果被销毁" << endl;
    }

    // 重写(Override)父类的 display
    void display() override {
        cout << "--> 我是苹果 | 重量: " << weight << " | 甜度: " << sugar << "%" << endl;
    }
};

// ================= 主函数 =================
int main() {
    cout << "=== 1. 创建过程 ===" << endl;
    // 多态的核心：父类指针指向子类对象
    Fruit* myFruit = new Apple(16, 10); 

    cout << "\n=== 2. 多态调用 ===" << endl;
    // 虽然指针是 Fruit 类型，但因为 display 是虚函数，所以调用的是 Apple 的版本
    myFruit->display(); 

    cout << "\n=== 3. 销毁过程 ===" << endl;
    // 因为父类析构函数是 virtual 的，所以会先调用 ~Apple() 再调用 ~Fruit()
    delete myFruit; 

    return 0;
}
```

### 运行结果预期

Plaintext

```
=== 1. 创建过程 ===
📦 [Fruit] 构造函数: 创建一个重量为 10 的水果
🍎 [Apple] 构造函数: 这是一个甜度为 16 的苹果

=== 2. 多态调用 ===
--> 我是苹果 | 重量: 10 | 甜度: 16%

=== 3. 销毁过程 ===
🍂 [Apple] 析构函数: 苹果被销毁
♻️ [Fruit] 析构函数: 水果被销毁
```

---

## 8. 常见面试题/易错点

1. **为什么析构函数必须是虚函数 (`virtual`)？**
    
    - 如果父类析构函数不是虚函数，当 `delete parentPointer` 时，只会调用父类的析构函数，子类特有的资源（如 Apple 的内存）就不会被释放，导致**内存泄漏**。
        
2. **`override` 关键字有什么用？**
    
    - (C++11 特性) 放在子类函数后面，显式告诉编译器“我正在重写父类函数”。如果你拼写错误，编译器会报错，防止低级错误。
        

---

关联链接：

[[C++ 指针与引用]]

[[3_内存管理]]

[[设计模式基础]]

```

### 如何使用这份笔记：
1.  **Mermaid图表**：Obsidian 原生支持 Mermaid，你会看到一个类图，展示 Fruit 和 Apple 的继承关系。
2.  **Callout**：带颜色的框（如 Summary, Tip, Important）能帮你快速抓住重点。
3.  **代码块**：带有 C++ 语法高亮，方便阅读。
4.  **双向链接**：底部的 `[[...]]` 是 Obsidian 的灵魂，建议你以后建立这些页面来完善你的知识网。
```