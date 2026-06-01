---
publish: true
---

**GNU** 是一个自由软件项目，其名称是一个递归缩写，全称为 **"GNU's Not Unix!"**（GNU 不是 Unix！）。这个名字本身就揭示了项目的核心目标：创建一个功能上与 Unix 兼容，但完全由**自由软件**组成的操作系统。

这个项目由 **理查德·斯托曼（Richard Stallman）** 于1983年发起，并由他创立的自由软件基金会（Free Software Foundation, FSF）提供支持。

**核心理念：自由软件（Free Software）**

GNU 的基石是“自由软件”的哲学理念。这里的“自由”并非指“免费”，而是指用户的**自由（Liberty）**。它强调用户拥有以下四项基本自由：

1. **自由 0：** 为任何目的运行程序的自由。
    
2. **自由 1：** 研究程序如何工作并根据需要修改它的自由（访问源代码是前提）。
    
3. **自由 2：** 重新分发副本的自由，以便可以帮助他人。
    
4. **自由 3：** 分发您修改过的版本的副本的自由，从而让整个社区有机会从您的改进中受益。
为了在法律上保障这些自由，GNU 项目发布了一系列著名的许可证，其中最核心的是 **GNU通用公共许可证（GPL）**

# 列举 GNU 的产品

要列出“所有”的 GNU 产品几乎是不可能的，因为其项目数量庞大且在不断变化。下面列出的是其中最重要、最知名、影响最深远的产品，并进行了分类。

## 1. 核心操作系统组件

- **GNU Hurd:** GNU 项目官方的内核。它采用微内核架构，开发进度相对缓慢，目前仍处于实验阶段。
    
- **GNU C Library (glibc):** C语言标准库的实现。几乎所有 GNU/Linux 系统都依赖它来与内核交互和执行基本任务。
    
- **Bash (Bourne-Again SHell):** 最流行、功能最强大的命令行解释器（Shell），是绝大多数 GNU/Linux 发行版的默认 Shell。
    
- **Coreutils (Core Utilities):** 包含了最基本的命令行工具，如 ls, cp, mv, rm, cat, chmod 等。
    
- **GRUB (Grand Unified Bootloader):** 强大的多系统启动引导程序，负责在计算机启动时加载操作系统内核（如 Linux 或 Hurd）。
    

## 2. 开发工具

- **GCC (GNU Compiler Collection):** GNU 编译器套件。最初是C语言编译器，现已发展为支持 C++, Objective-C, Fortran, Ada, Go 等多种语言的编译器集合，是自由软件世界和许多商业领域最重要的编译器。
    
- **GDB (GNU Debugger):** 功能强大的命令行调试器，是开发人员调试程序的标准工具。
    
- **GNU Binutils:** 一套二进制工具集，包括链接器 ld、汇编器 as 等。
    
- **GNU Make:** 一个自动化构建工具，通过读取 Makefile 文件来编译和构建软件项目。
    
- **GNU Autotools (Autoconf, Automake, Libtool):** 用于创建可移植源代码包的工具套件，能够让软件在多种类 Unix 系统上轻松编译安装。
    
- **GNU Emacs:** 由理查德·斯托曼亲自开发的、功能极其强大且高度可扩展的文本编辑器。它不仅仅是编辑器，更是一个集成了邮件、日历、Shell 等功能的集成环境。
    

## 3. 桌面环境和应用程序

- **GNOME (GNU Network Object Model Environment):** 一个完整、流行且用户友好的桌面环境。虽然它现在是一个独立的项目，但其初衷是作为 GNU 项目的官方桌面。
    
- **GIMP (GNU Image Manipulation Program):** 功能强大的免费光栅图像编辑器，被誉为“开源世界的 Photoshop”。
    
- **GnuPG (GNU Privacy Guard):** OpenPGP 标准的完整且免费的实现，用于加密、签名数据和通信。
    
- **GNU Nano:** 一个简单易用的命令行文本编辑器，适合新手使用。
    
- **Gnumeric:** 一款电子表格软件。
    
- **GNU Health:** 一个自由的医院信息和健康管理系统。
    

## 4. 系统工具和实用程序

- **grep (Global Regular Expression Print):** 用于在文件中搜索指定模式的强大工具。
    
- **gawk (GNU Awk):** AWK 脚本语言的 GNU 实现版本，用于文本处理和数据提取。
    
- **sed (Stream Editor):** 流编辑器，用于对文本进行非交互式的编辑。
    
- **tar (Tape Archive):** 用于将多个文件打包成一个归档文件的工具。
    
- **Parted (Partition Editor):** 用于创建、删除、调整磁盘分区的工具。
    
- **Wget:** 一个非交互式的网络下载工具。
    

## 5. 法律和哲学“产品”

这些虽然不是软件，但可以说是 GNU 项目最根本、影响最深远的“产品”。

- **GNU GPL (General Public License):** GNU 通用公共许可证。确保软件及其衍生品保持自由的核心许可证。
    
- **GNU LGPL (Lesser General Public License):** GNU 宽通用公共许可证。允许专有软件链接使用该许可证下的库。
    
- **GNU AGPL (Affero General Public License):** 确保在网络服务器上运行的修改版软件也必须提供源代码。
    
- **GNU FDL (Free Documentation License):** 用于文档的自由许可证，确保文档可以自由复制、修改和分发。