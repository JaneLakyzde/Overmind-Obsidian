---
aliases:
  - Open Multi-Processing
tags:
date: 2026-04-15
---


**OpenMP 是一套用于共享内存并行编程的标准，它通过 `#pragma omp` 指令、库函数和运行时机制，让 C/C++/Fortran 程序更容易利用多线程并行执行。**

它最常见的用途是：

- 科学计算
- 图像处理
- 数值模拟
- 数据分析
- C / C++ / Fortran 程序加速

下面我用尽量通俗的方式讲清楚它是什么、怎么用、为什么能加速、有什么限制。

---

# 一、OpenMP 到底是什么

OpenMP 全名是：

**Open Multi-Processing**

它是一套标准，主要给这几种语言用：

- C    
- C++
- Fortran

它的作用是让程序员可以比较方便地写出“并行程序”。

这里的“并行”意思是：

**让多个 CPU 核心同时做不同的部分。**

---

# 二、先用生活例子理解它

假设你要把 100 万张试卷上的分数录入电脑。

### 不用 OpenMP

只有你一个人录：

- 你从第 1 张录到第 100 万张
- 很慢

### 用 OpenMP

你找来 8 个人：

- 第 1 个人录第 1 到 12.5 万张
- 第 2 个人录下一段
- ...
- 8 个人同时录

这就叫**并行处理**。

OpenMP 做的事，本质上就是：

1. 帮你“拉来多个工人”（线程）
2. 帮你“分配任务”
3. 最后再把结果汇总

---

# 三、OpenMP 和“线程”是什么关系

OpenMP 背后主要使用的是**线程（thread）**。

小白可以先把线程理解成：

**同一个程序里同时工作的多个小助手。**

比如一个程序启动后：

- 主线程：像总负责人
- 其他线程：像被分配出去做事的工人

OpenMP 最常见的模式叫：

**fork-join 模型**

意思是：

1. 平时先只有一个主线程
2. 遇到并行代码时，主线程“分身”出多个线程
3. 大家同时工作
4. 工作完再汇合回去

可以想成：

- 平时：1 个老板
- 干活时：老板叫来 7 个员工，一共 8 人一起干
- 干完：员工散去，老板继续往下走流程

---

# 四、OpenMP 的核心作用

它主要解决这几个问题：

## 1. 利用多核 CPU 提速

现代 CPU 往往有很多核心，比如 4 核、8 核、16 核。
如果程序只用 1 个核心，其他核心就在闲着。
OpenMP 能让程序尽量把这些核心用起来。

---

## 2. 降低并行编程难度

如果你直接手写线程，会比较麻烦，要自己处理：

- 创建线程
- 同步线程
- 分工    
- 防止打架抢数据

OpenMP 提供了更简单的写法：  
你只要加几行指令，编译器和运行时系统就帮你做很多事。

---

## 3. 适合给循环加速

很多计算程序最耗时的部分就是大循环，比如：

- 遍历大数组
- 大矩阵计算
- 图像像素逐点处理
- 大量重复计算

OpenMP 对这种“每次循环彼此独立”的情况特别合适。

---

# 五、OpenMP 最适合哪些场景

非常适合：

- 大量 for 循环
- 数组处理
- 矩阵运算
- 每次计算互不影响的任务
- 数值模拟

例如：

- 给 1000 万个数字分别求平方    
- 对一张超大图片的每个像素做处理
- 对很多独立数据做统计

---

# 六、不适合哪些场景

不太适合：

- 任务很小，分线程的开销反而不划算
- 各任务之间强依赖，必须前一步做完后一步才能做
- 数据共享很多，线程之间频繁抢资源
- 本身就不是 CPU 密集型任务

比如：

- 一个循环只有 10 次，没必要并行
- 每一步都依赖上一步的结果，也难并行

---

# 七、OpenMP 怎么使用

下面以 C/C++ 为例。

OpenMP 的典型使用方法是：

1. 写普通 C/C++ 代码
2. 在关键位置加 `#pragma omp ...`
3. 编译时打开 OpenMP 选项

---

## 1. 最简单例子：让多个线程一起打印

```cpp
#include <iostream>
#include <omp.h>

int main() {
    #pragma omp parallel
    {
        int id = omp_get_thread_num();
        int total = omp_get_num_threads();
        std::cout << "Hello from thread " << id
                  << " / " << total << std::endl;
    }
    return 0;
}
```

### 这段代码的意思

`#pragma omp parallel` 表示：

**下面这个代码块，让多个线程同时执行。**

如果开了 4 个线程，就会打印 4 次。

### 常见函数

- `omp_get_thread_num()`：当前线程编号
- `omp_get_num_threads()`：当前总线程数

---

## 2. 编译方法

如果你用 gcc 或 g++：

```bash
g++ -fopenmp test.cpp -o test
```

运行：

```bash
./test
```

如果不加 `-fopenmp`，编译器可能会忽略 OpenMP 指令。

---

# 八、最常用场景：并行 for 循环

这是 OpenMP 最重要的用法。

## 普通循环

```cpp
for (int i = 0; i < 100; i++) {
    arr[i] = arr[i] * 2;
}
```

如果每次循环互不影响，就可以改成：

```cpp
#pragma omp parallel for
for (int i = 0; i < 100; i++) {
    arr[i] = arr[i] * 2;
}
```

### 这是什么意思

OpenMP 会自动把这 100 次循环分给多个线程。

比如 4 个线程时，可能变成：

- 线程 0 做 0~24
- 线程 1 做 25~49
- 线程 2 做 50~74
- 线程 3 做 75~99

所以速度可能更快。

---

# 九、一个完整的简单例子

```cpp
#include <iostream>
#include <omp.h>

int main() {
    const int N = 10;
    int arr[N];

    #pragma omp parallel for
    for (int i = 0; i < N; i++) {
        arr[i] = i * i;
    }

    for (int i = 0; i < N; i++) {
        std::cout << arr[i] << " ";
    }

    return 0;
}
```

这个程序会并行计算：

- 0²
- 1²
- 2²
- ...
- 9²
    

---

# 十、OpenMP 的工作原理

这一部分我继续用大白话解释。

## 1. 编译器先看见指令

你写了：

```cpp
#pragma omp parallel for
```

编译器看到后就知道：

“这里不是普通循环，要改造成多线程执行。”

所以编译器会生成适合并行运行的代码。

---

## 2. 运行时系统创建线程

程序运行到这里时，OpenMP 运行时会：

- 创建若干线程
- 决定每个线程做哪一段工作
- 管理同步和收尾

你不用手动写底层线程管理逻辑。

---

## 3. 任务被拆分

比如 1000 次循环，4 个线程。

OpenMP 可能自动拆成：

- 线程 A：0~249    
- 线程 B：250~499
- 线程 C：500~749
- 线程 D：750~999

这就叫**work-sharing**，也就是“分工”。

---

## 4. 所有线程结束后汇合

循环执行完后，一般会有一个隐式等待点。

就是说：

- 线程快的先做完，也得等等慢的
- 所有人都做完后，程序再继续往下走

这能保证结果完整。

---

# 十一、为什么 OpenMP 能加速

因为它把原来“串行”的工作变成“并行”。

### 串行

一个人干 8 小时。

### 并行

8 个人一起干，理想情况下 1 小时干完。

但现实里不会完全 8 倍加速，因为有额外成本：

- 叫人来的成本
- 分工的成本
- 汇总结果的成本
- 线程之间协调的成本

所以真实加速通常是：

**比 1 个线程快，但不一定等于线程数倍。**

---

# 十二、为什么有时反而不会更快

这是新手特别容易踩的坑。

## 原因 1：线程创建也要成本

你可以理解成：  
“叫来工人、安排工位、分任务”本身也花时间。

如果任务很小，这些准备时间可能比干活时间还长。

---

## 原因 2：线程抢同一个数据

比如多个线程同时修改一个变量，就会“打架”。

例如：

```cpp
int sum = 0;

#pragma omp parallel for
for (int i = 0; i < 100; i++) {
    sum += i;
}
```

这段代码**有问题**。

因为多个线程可能同时读写 `sum`，导致结果错乱。

这叫：  
**竞争条件（race condition）**

---

# 十三、什么是竞争条件

通俗说，就是：

**几个人同时改同一份表格，谁先写、谁后写，顺序乱了。**

比如：

- 线程 A 看到 sum=10，想加 1
    
- 线程 B 也看到 sum=10，想加 1
    
- 两人都写回 11
    

本来应该变成 12，结果只变成了 11。

所以并行程序里，最重要的问题之一就是：

**避免多个线程无保护地同时改共享数据。**

---

# 十四、怎么解决竞争条件

OpenMP 提供了多种办法。

## 1. reduction

这是做求和最常见、最推荐的方法。

```cpp
#include <iostream>
#include <omp.h>

int main() {
    int sum = 0;

    #pragma omp parallel for reduction(+:sum)
    for (int i = 1; i <= 100; i++) {
        sum += i;
    }

    std::cout << "sum = " << sum << std::endl;
    return 0;
}
```

### 它的意思

每个线程先算自己的局部 `sum`，  
最后 OpenMP 再把大家的结果加起来。

这就像：

- 每个人先算自己那一摞账
    
- 最后总会计统一合并
    

这是最安全也最高效的方式之一。

---

## 2. critical

表示某一小段代码同一时刻只允许一个线程进入。

```cpp
#pragma omp critical
{
    sum += i;
}
```

意思像厕所一次只能进一个人。

这样能保证正确，但容易变慢，因为大家要排队。

---

## 3. atomic

适合非常简单的一次更新操作。

```cpp
#pragma omp atomic
sum += i;
```

比 `critical` 更轻量一些，但用途更窄。

---

# 十五、变量为什么有“共享”和“私有”之分

在 OpenMP 里，变量有两类很重要：

## shared（共享）

所有线程都能看到同一个变量。

像一个办公室里大家共用一块白板。

## private（私有）

每个线程都有自己的副本。

像每个人手里各有一本自己的笔记本。

---

## 例子

```cpp
#include <iostream>
#include <omp.h>

int main() {
    int x = 10;

    #pragma omp parallel private(x)
    {
        x = omp_get_thread_num();
        std::cout << "thread x = " << x << std::endl;
    }

    std::cout << "main x = " << x << std::endl;
    return 0;
}
```

这里：

- 并行区域里的 `x` 是每个线程自己的
    
- 主线程外面的 `x` 还是原来的 10
    

---

# 十六、常用 OpenMP 指令

下面列最常用的，先知道名字和作用就够了。

## 1. `parallel`

创建并行区域，让多个线程执行同一段代码。

```cpp
#pragma omp parallel
{
    // 多个线程都会执行
}
```

---

## 2. `parallel for`

把 for 循环分给多个线程。

```cpp
#pragma omp parallel for
for (...) {
}
```

最常用。

---

## 3. `sections`

把几块不同的任务分给不同线程。

```cpp
#pragma omp parallel sections
{
    #pragma omp section
    {
        // 任务1
    }

    #pragma omp section
    {
        // 任务2
    }
}
```

适合几个不同的大任务同时做。

---

## 4. `single`

表示这段代码只让一个线程执行一次。

```cpp
#pragma omp single
{
    // 只执行一次
}
```

比如只让一个线程负责打印汇总信息。

---

## 5. `barrier`

表示所有线程在这里集合，谁都不能先走。

```cpp
#pragma omp barrier
```

像团队集合点。

---

## 6. `critical`

同一时间只允许一个线程执行某段代码。

```cpp
#pragma omp critical
{
}
```

---

# 十七、线程数怎么设置

## 方法 1：代码里设置

```cpp
omp_set_num_threads(4);
```

例子：

```cpp
#include <iostream>
#include <omp.h>

int main() {
    omp_set_num_threads(4);

    #pragma omp parallel
    {
        std::cout << "thread " << omp_get_thread_num() << std::endl;
    }

    return 0;
}
```

---

## 方法 2：环境变量设置

Linux/macOS 终端中：

```bash
export OMP_NUM_THREADS=4
./test
```

Windows 里也有类似方式。

---

# 十八、循环任务怎么分配

OpenMP 不只是“分给多线程”，还可以决定**怎么分**。

这叫 `schedule`。

## 1. static

提前平均分配。

```cpp
#pragma omp parallel for schedule(static)
for (int i = 0; i < N; i++) {
    ...
}
```

适合每次循环工作量差不多的情况。

比如每道题难度一样。

---

## 2. dynamic

线程干完一块后，再去领新任务。

```cpp
#pragma omp parallel for schedule(dynamic)
for (int i = 0; i < N; i++) {
    ...
}
```

适合每次循环耗时差异很大。

比如有些题 1 分钟做完，有些题 20 分钟做完。  
动态分配更公平，不容易有人闲着。

---

## 3. guided

也是动态的一种，但开始给大块，后面越来越小。

用于折中优化。

---

# 十九、一个更实用的例子：并行求数组和

```cpp
#include <iostream>
#include <vector>
#include <omp.h>

int main() {
    const int N = 1000000;
    std::vector<int> arr(N, 1);
    long long sum = 0;

    #pragma omp parallel for reduction(+:sum)
    for (int i = 0; i < N; i++) {
        sum += arr[i];
    }

    std::cout << "sum = " << sum << std::endl;
    return 0;
}
```

这个例子里：

- 数组有 100 万个元素
    
- 每个元素都是 1
    
- 最终结果应该是 1000000
    

这就是 OpenMP 很典型的使用方式。

---

# 二十、一个判断标准：什么循环能安全并行

看一个循环能不能用 OpenMP，并不是看“能不能写”，而是看：

**每次循环是否彼此独立。**

## 可以并行的例子

```cpp
for (int i = 0; i < N; i++) {
    a[i] = b[i] * 2;
}
```

每次只处理自己的 `i`，互不影响。

---

## 不容易直接并行的例子

```cpp
for (int i = 1; i < N; i++) {
    a[i] = a[i - 1] + 1;
}
```

这里 `a[i]` 依赖 `a[i-1]`。  
前一步没做完，后一步就没法做。

这类循环不能直接简单并行。

---

# 二十一、OpenMP 的优点

## 上手相对容易

比起手工写线程，简单很多。

## 改动小

很多时候只要给循环加一条指令。

## 适合老代码加速

已有 C/C++/Fortran 代码常常能逐步改造。

## 跨平台

很多主流编译器都支持。

---

# 二十二、OpenMP 的缺点

## 不是所有程序都能加速

有依赖关系的代码不适合。

## 可能出现线程安全问题

共享变量处理不当就会出错。

## 加速不是无限的

受 CPU 核心数、内存带宽、同步开销限制。

## 调试更难

并行 bug 往往比普通 bug 难查。

---

# 二十三、OpenMP 和 GPU 有什么区别

很多人会混淆。

## OpenMP

主要面向 **CPU 多核并行**。

## CUDA / OpenCL

更多是面向 **GPU 并行**。

简单说：

- OpenMP：让 CPU 的多个核心一起工作
    
- CUDA：让 GPU 的海量小核心一起工作
    

所以 OpenMP 不是主要拿来用显卡加速的。

---

# 二十四、OpenMP 和 MPI 有什么区别

这也是常见问题。

## OpenMP

适合**一台机器内部**的多核并行。

## MPI

适合**多台机器之间**协同计算。

你可以理解成：

- OpenMP：一间办公室里多人合作
    
- MPI：很多办公室之间发消息合作
    

大型超算里经常是 MPI + OpenMP 一起用。

---

# 二十五、给小白的最简学习路线

建议按这个顺序学：

## 第一步：先理解并行概念

弄明白：

- 串行是什么
    
- 并行是什么
    
- 线程是什么
    
- 为什么会有竞争条件
    

---

## 第二步：先学 4 个最重要的东西

先只学：

- `#pragma omp parallel`
    
- `#pragma omp parallel for`
    
- `reduction`
    
- `critical`
    

这 4 个够入门了。

---

## 第三步：只改最耗时的大循环

不要一开始整个程序都并行化。

先找最慢的循环，把它改成：

```cpp
#pragma omp parallel for
```

然后验证：

- 结果是否正确
    
- 速度是否真的变快
    

---

## 第四步：学会判断变量是共享还是私有

这是 OpenMP 的核心难点之一。

---

## 第五步：再学调度和优化

比如：

- `schedule(static)`
    
- `schedule(dynamic)`
    
- 线程数设置
    
- 性能测试
    

---

# 二十六、一个完整入门示例

下面给你一个很适合新手看的版本。

```cpp
#include <iostream>
#include <vector>
#include <omp.h>

int main() {
    omp_set_num_threads(4);

    const int N = 20;
    std::vector<int> a(N), b(N), c(N);

    for (int i = 0; i < N; i++) {
        a[i] = i;
        b[i] = i * 10;
    }

    #pragma omp parallel for
    for (int i = 0; i < N; i++) {
        c[i] = a[i] + b[i];
        std::cout << "thread " << omp_get_thread_num()
                  << " handles i = " << i << std::endl;
    }

    std::cout << "\nresult:\n";
    for (int i = 0; i < N; i++) {
        std::cout << c[i] << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

这个程序做了什么：

1. 创建两个数组 `a` 和 `b`
    
2. 并行计算 `c[i] = a[i] + b[i]`
    
3. 输出每个线程负责哪些下标
    

这个例子很适合观察“任务被分给多个线程”的过程。

---

# 二十七、你可以怎么记住 OpenMP

记一个最朴素的版本就够了：

**OpenMP = 给 C/C++/Fortran 程序加“多核并行能力”的简单办法。**

再记 3 个关键点：

1. **最常用于加速大循环**
    
2. **本质是多个线程同时干活**
    
3. **最怕多个线程乱改同一个共享变量**
    

---

# 二十八、给你一个一句话总结

OpenMP 的本质是：

**通过少量编译指令，把原本一个核心慢慢做的工作，拆给多个 CPU 核心同时做，从而提升计算速度。**

---

# 二十九、初学者最容易犯的 5 个错误

## 1. 觉得加了 OpenMP 一定更快

不一定。任务太小可能更慢。

## 2. 忘记编译选项

要加：

```bash
-fopenmp
```

---

## 3. 在并行循环里乱改共享变量

容易导致结果错误。

---

## 4. 没有验证结果正确性

并行程序先看对不对，再看快不快。

---

## 5. 把有依赖的循环强行并行

这样通常会出错。

---

# 三十、适合你的理解方式

如果你是计算机小白，你可以先只把它当成这样：

- CPU 有很多核心
- 默认程序常常只用一个核心    
- OpenMP 帮程序把任务分给多个核心
- 所以很多计算可以更快    
- 但前提是任务能拆开，而且大家不能抢同一份数据
