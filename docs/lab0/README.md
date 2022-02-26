# 实验零

## 本学期的实验设计

不出意料的话，本学期的前三个实验和前几届相同，依次为裁剪 Linux 内核、编写 Shell 程序、完成多人聊天室，不过部分实验会在之前的设计基础上有所调整和优化。第四次实验主题仍处于待定状态，会在之后的课程中确定下来。具体的实验细节同学们可以参考前几次的课程实验安排：[OSH-2021](https://osh-2021.github.io)、[OSH-2020](https://osh-2020.github.io) **等**（注意到链接的规律了吗 :3）。所有实验都可以使用 C/C++ 或是 Rust 完成，但是不能用 Go（简直是作弊.jpg）。

### 实验考察范围

前三次实验的考察内容和技能要求如下：

| 实验   | 主要内容        | 考察知识                                  | 所需技能                                   |
| ------ | --------------- | ----------------------------------------- | ------------------------------------------ |
| 实验一 | 裁剪 Linux 内核 | Linux 启动过程、内核模块                  | make、makefile 基础                        |
| 实验二 | 编写 Shell 程序 | fork 等系统调用、I/O 重定向、多线程、中断 | 多线程编程、使用 syscall、中断处理         |
| 实验三 | 编写多人聊天室  | 进程调度、socket 复用、TCP 相关知识       | 多线程、面向 socket 编程、I/O 相关 syscall |

实验所考察的知识和课程进度同步，可以先跟着课程学习，不过所需的编程技能可能需要靠自己掌握。

### Linux 实验环境

OSH 几乎所有实验都需要在 Linux 平台上完成、实现，所以本节期望你有一个可用的 Linux 工作环境（不限于本地）和基本的 Linux 使用能力。如果你是一个熟练的 Linux 用户，你可以跳过本节。

如果你是第一次接触 Linux，你可以在 [USTC Vlab](https://vlab.ustc.edu.cn/vm/) 上得到一个 Linux 的实验环境（注意：这不是虚拟机，是一个 LXC 容器，*可能*有部分实验无法在其上面完成）。

如果你内存充足、或者不太想在物理机上安装 Linux，可以尝试使用虚拟机运行 Linux。具体可以参照[在 Windows 中使用虚拟机（By iBug）](https://ibug.io/cn/2019/02/setup-ubuntu-in-vmware/)、[在 macOS 中使用虚拟机（By Taoky）](https://blog.taoky.moe/2019-02-23/installing-os-on-vm.html)这两篇文章。

Linux 最基本的使用也可以从  [LUG 的 Linux 101 教程](https://101.lug.ustc.edu.cn/)  开始学习（**如果没有时间也请看一下第三章的文件操作、第六章的重定向和第七章 Linux 上的编程**）。

如果想了解更多有关 Linux 的技巧，可以看  [Debian 教程](https://www.debian.org/doc/manuals/debian-reference/ch01.zh-cn.html)或者 [Arch Linux 文章索引](<https://wiki.archlinux.org/index.php/General_recommendations_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>)。

#### 不同平台兼容性

由于无法确定各个实验全貌，目前助教无法*保证*非软件 Linux 虚拟机的平台，如 WSL1/2、BSD、Vlab，能够完成所有的实验。
但我们在设计实验时将尽可能考虑到不同平台下的兼容性问题，并将第一时间更新下方的兼容性 tips 配合公告作为提醒：

| 平台 | 实验 | 出错现象 | 原因分析 | 解决方案 |
| ---- | ---- | -------- | -------- | -------- |

这里有一些基本分析：WSL1（即 WSL 初版）是微软魔改的内核，没有实现完整的 cgroups 和 IPC，如 fakeroot 也不能正常使用，可能对实验有影响；WSL2 是使用 Hyper-V 虚拟机实现、使用了完整的内核，*应该*不会干扰到实验的进行。「FreeBSD 提供了 Linux 32-bit 二进制的兼容，但是不支持 64-bit 的 Linux 二进制程序」，_需参考实际使用状况_。

### 编程语言要求

如实验设计中所述，本轮实验如无特殊声明均允许使用 C、C++、Rust 完成。
在下面几个章节中，我们将简单介绍需要掌握或是了解的 C、C++、Rust 语言知识。
值得注意的是，这些知识并不一定会在实验中用到，但是理解它们或许能够指导你的实际开发过程并减少在各个语言文档中探索的时间。

#### C 的要求

- 了解 C99 标准下几乎所有的 C 用法，这是考虑到 C 的标准语言特性并不太多，更进阶的用法也更倾向于依赖编译器特性或是标准用法的巧用；
- 了解基本的宏知识，例如宏函数中参数使用时需要括号包裹，这是因为调用 Linux API 时需要用到宏；
- 掌握查询 Linux API 文档的方法，无论是使用 `man` `tldr` 或是直接查看网页版 man 文档;

#### C++ 的要求

- 掌握基本的 C 用法；
- 了解 C++11 下的一些重要库/语言特性的用法，在后续章节 Modern C++ 中会有 tips 的说明；
- 宏，如 C 要求所述；
- Linux API 文档，如 C 要求所述;

#### Rust 的要求

Rust 的具体要求可以参考后续章节关于 Rust 编程语言的详细介绍。

### Modern C++

如果你选择使用 C++ 完成实验，那么请不要将 C++ 代码写得宛如 C 一样——Modern 的、现代的、C++11 标准后的 C++ 增添了众多强有力的工具，能够帮助你以更高效、更简洁的手段达成目标。

如需全方位的了解 Modern C++，推荐阅读《Effective C++》这本书籍，其中详细讲解了 Modern C++ 中的一些设计理念和最佳实践。

额外的我们也给出一些 tips，来帮助你尽快地了解一些 C++ 工具（如 STL），以及阐述部分 Modern C++ 概念，或是协助回避困惑之处：

- 不必使用栈上的 `int a[12]` 或是自行 `new int[12]` 来进行堆上分配，可以直接使用 `std::vector`；
- `std::string` 保证内部空间连续，因而可以配合 `s.resize()` 改变内部空间大小，并用 `s.c_str()` 或是 `&s[0]` 进行不可变或是可变访问，可以代替 `chat buf[N]`；
- `<memory>` 如 `shared_ptr` 可以实现智能指针，自动在合适的时刻释放资源，并允许像一个指针一样廉价地拷贝，可以配合 `std::thread` 等使用；
- `std::string` 缺少 split 功能，但网络上有[众多现成的实现](https://stackoverflow.com/questions/14265581/parse-split-a-string-in-c-using-string-delimiter-standard-c)；

### 关于 Rust 编程语言

#### 要求

实验流程中我们在自愿的基础上**鼓励**使用 Rust 语言，但不会做进一步的要求。
如果你选择在实验中使用 Rust 语言，你依然可以从助教处获取一些语言使用上的基础帮助，并与其他同学在同一标准下进行评测。

#### 实验中的优缺点

选择 Rust 可能会在本轮实验中遇见以下优缺点：

优点：

- Rust 的基础库某种程度上比 C++ 的基础库更全面而更易用；
- Rust 的内存安全保证可以在最大程度上让你避免遇到难以 debug 的 segmentation fault 问题；
- Rust 对于一些比较现代的技术，例如 coroutine，有相对更好的支持，在实验中需要实现类似情况时会更方便;

缺点：

- Rust 的学习曲线稍显陡峭，尤其是从原汁原味的 C 出发的话；
- Rust 不能直接使用 Linux API（由众多 C header 文件提供），下文我们将详细讨论这个问题；

#### 教程

配置环境的教程可以在[官网安装界面](https://www.rust-lang.org/tools/install)查看。
而相应的详细语言教程也可以在[官网教程汇总界面](https://www.rust-lang.org/learn)按需查阅。
其中重点是 [Rust Book](https://doc.rust-lang.org/book/) 这份教程，其在高低层次上都对 Rust 进行了详细的解析。
而如果你之前只了解过 C 开发，可以着重关注 Rust Book 中的以下概念：

- Ownership，lifetime，borrow，这些是 Rust 最核心的概念之一；
  - 对应章节是 [Understanding Ownership](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html) 和 [Validating References with Lifetimes](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)
- Cargo 的使用，从而完成 Rust 的依赖管理及编译等工作；
- 智能指针（smart pointer），C++ `<memory>` 库中也有相似实现；
- 并发编程，C++ `<thread>` `<mutex>` ` <condition_variable>` 等库中也有相似实现；
- 函数式编程（functional programming），如 `map` 的用法；
- 模式匹配（pattern matching），如 `match` 和 `if let` 的用法；
- 基本的 unsafe Rust 用法，可能会在调用外部库时用到；
- async Rust，可能会在 coroutine 相关的实验中用到；
- 错误处理（error handling），Rust 错误处理与 Go 有类似之处，如有经验可以对照学习，但本次实验应该不会在错误处理上有较高要求，把握概念即可；
- Traits（有译作特质的），但本次实验应该不会涉及过多或过深；

其他的例如 [Rust by Example](https://doc.rust-lang.org/rust-by-example/) 也是很优质的教程，也能让你迅速找到自己所需要的写法。

#### 关于外部库

如优缺点中所言，尽管 Rust 标准库相对较丰富，但需要调用特定 Linux API 时，Linux 提供的是 C header 文件。
尽管 Rust 可以利用 FFI 和 binding 生成利用这些 C header 文件完成相关调用，但是操作太过繁杂而无必要。
所以如无特地声明，我们默认允许使用 Rust 语言时调用以下这些外部库（后续可能会增充此列表，但不会删减）：

- `nix`
- `libc`

如你不满足于这些外部库、希望使用其他外部库时，请尽量提前询问助教。
无论是否提前询问，在最后验收实验时，助教都会参考以下条件对除上述列表以外的外部库进行评定：

- 此外部库是否仅是 Linux C API FFI？如是，则允许；
- 此外部库是否仅是 Linux C API FFI 并进行了 safe wrap？如是，则允许；
- 此外部库使用方法是否与原本 C 中调用的方法相类似？如是，参考相似程度允许；
- 此外部库是否破坏了原本的实验设计目标？如是，则禁止；如否，参考和实验目标相关程度允许；

如果一个外部库被禁止，实验中对应项目将可能被酬情扣分。
