# 实验零

## 本学期的实验设计

不出意料的话，本学期的前三个实验和前几届相同，依次为裁剪 Linux 内核、编写 Shell 程序、完成多人聊天室，不过部分实验会在之前的设计基础上有所调整和优化。第四次实验主题仍处于待定状态，会在之后的课程中确定下来。具体的实验细节同学们可以参考前几次的课程实验安排：[OSH-2021](osh-2021.github.io)、[OSH-2020](osh-2020.github.io) **等**（注意到链接的规律了吗 :3）。所有实验都可以使用 C/C++ 或是 Rust 完成，但是不能用 Go（简直是作弊.jpg）。

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

关于使用 BSD/WSL 完成 OSH 实验：助教们并没有测试过这些平台上的可行性，实验成功与否和实验本身无关，请自行评估。WSL1（即 WSL 初版）是微软魔改的内核，没有实现完整的 cgroups 和 IPC，如 fakeroot 也不能正常使用，可能对实验有影响；WSL2 是使用 Hyper-V 虚拟机实现、使用了完整的内核，*应该*不会干扰到实验的进行。「FreeBSD 提供了 Linux 32-bit 二进制的兼容，但是不支持 64-bit 的 Linux 二进制程序」，_助教也没使用过也不了解_。
