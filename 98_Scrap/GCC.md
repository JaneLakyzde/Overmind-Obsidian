---
aliases:
  - GNU C Compiler
  - CNU Compiler Collection
tags:
  - Code
date: 2026-05-24
---
 Linux 系统中最主流、最标准的编译器。在 Linux 环境下进行 C/C++ 等语言的编程开发，绝大多数情况下默认使用的就是 GCC。

 与 GCC 对应, Mac 环境下通常使用 [[Clang]] 进行编译

 Windows 上通常使用 [[MSYS2]] 来安装 GCC 编译器:
 
1. 打开 `MSYS2 MSYS` 终端，输入 `pacman -Syu` 并回车，按提示更新系统核心包
2. 打开开始菜单里的 `MSYS2 MinGW 64-bit` 
3. 输入`pacman -S mingw-w64-x86_64-gcc`，按提示输入 `y` 确认安装。
4. 配置环境变量, 在 windows 环境变量设置页面通常输入：`C:\msys64\mingw64\bin`
5. 打开 CMD 输入 `gcc -v`，如果显示出版本号即安装成功。
	`g++` 用来编译 `cpp` 程序, `gcc` 用来编译 `c` 程序
	
