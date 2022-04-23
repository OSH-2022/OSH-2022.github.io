# 实验二

!!! Warning "本文仍在编辑中"

在实验一中，我们使用了许多命令行操作来完成一系列工作，如编译 Linux 内核、打包 initrd 等。所有的命令行都由一个叫做 shell 的程序解释并执行。本实验我们将自己编写一个简单的 shell 并理解 Linux shell 程序的工作原理。

## 编写 Shell 程序

首先，大家可以将本页底部助教编写的一个示例程序命名为 `shell.cpp`：

```shell
g++ shell.cpp -o shell
```

以上命令会调用 g++ 编译器编译出一个可执行文件 shell，你可以继续输入 `./shell` 来运行它。这是一个非常简陋的 shell，它会提示你输入命令，你可以输入 `cd` 来切换工作目录，输入 `pwd` 显示当前目录，输入 `export` 导出环境变量，输入 `exit` 退出，或者调用系统中有的其他命令来运行，例如 `ls`、`cat` 等。

你的任务是对这个 shell 程序进行修改升级（当然，你也可以选择自己从头或选择其他语言编写），并完成下面的任务。

### 更健壮（推荐，但不要求）

目前的示例程序非常脆弱，仅随意检查了部分可能的错误。建议你将它改得更健壮，以方便之后的进一步开发和调试。

### 支持管道

形如 `env | wc` 这样的命令利用了「管道」语法，将两条不同的命令对接在一起同时运行。`|` 的意思是将前面的命令 `env`（输出所有环境变量）的标准输出连接到后面命令 `wc`（统计行数）的标准输入（这样就能统计出环境变量的总数）。

请你观察并学习这个语法，为你的 shell 程序实现这一功能。你可能要用到的函数：`pipe`、`close`、`dup`。你可以运行 `man 函数名` 来查看系统自带的文档，或者上网搜索更多信息。

### 支持基本的文件重定向

形如 `ls > out.txt` 会将 `ls` 命令的标准输出（stdout）重定向到 `out.txt` 文件中，具体地说，会将 `out.txt` 关联到程序的标准输出，然后再运行相应的命令。类似的，`ls >> out.txt` 会将输出追加（而不是覆盖）到 `out.txt` 文件，`cat < in.txt` 会将程序的标准输入（stdin）重定向到文件 `in.txt`。

请为你的 shell 程序实现 `>`、`>>` 和 `<` 的功能。你可能要用到的函数：`open`、`close`、`dup`。

#### 支持基于文件描述符的文件重定向、文件重定向组合（选做）

一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：

- 标准输入文件（stdin）：文件描述符为 0，Unix/Linux 程序默认从 stdin 读取数据。
- 标准输出文件（stdout）：文件描述符为 1，Unix/Linux 程序默认向 stdout 输出数据。
- 标准错误文件（stderr）：文件描述符为 2，Unix/Linux 程序会向 stderr 流中写入错误信息。

我们可以手动更改输入输出的文件描述符。请自行查找资料，实现以下这些文件重定向：

1. 形如 `cmd 10> out.txt` 和 `cmd 20< in.txt` 以及 `cmd 10>&20 30< in.txt` 这样的命令会将打开文件描述符 10、20 和 30 并重定向到相应的文件。
2. `cmd <<< text` 会将 `"text\n"` 作为标准输入重定向给 `cmd`。
3. 以下命令会将字符串 `"this\noutput\n"` 作为标准输入重定向给 `cmd`：

```shell
cmd << EOF
this
output
EOF
```

### 处理 Ctrl-C 的按键

在使用 shell 的时候按下 Ctrl-C 可以丢弃当前输入到一半的命令，重新显示提示符并接受新的命令输入。当有程序运行时，按下 Ctrl-C 可以终结运行中的程序，立即回到 shell 开始新的命令输入（shell 没有随程序一起结束）。

例如（`^C` 表示遇到 Ctrl-C 的输入）：

```shell
$ echo taokystrong
taokystrong
$ echo taokystrong^C
$ sleep 5  # 几秒之后
^C
$          # sleep 没有运行完
$ ^C
```

请为你的 shell 实现对 Ctrl-C 的处理。你可能要用到的函数：`signal`、`waitpid`。

提示：当你正确处理第一种情况后（丢弃未输入完的命令），第二种情况（终结运行中的程序）并不需要你做任何工作。

### 支持 History

在使用 shell（这里特指 bash）的时候输入 `history n` 可以显示最近执行的 n 条指令（对于 zsh 用户，`history` 命令会被自动替换为内建的 `fc -l`，显示从 n 条命令开始的记录）。同时我们可以使用 `!n` 重复执行历史记录中的第 n 条命令，该命令不会被记入 history。在 shell 中我们还可以通过上下方向键来切换到不同的历史命令。

例如：

```shell
$ history 3
  843  echo taokystrong
  844  echo taoky
  845  history 3
$ !843
echo taokystrong
taokystrong
```

请为你的 shell 实现对历史命令的处理，支持 `!n`、`history n` 和通过上下键切换到历史命令，且 `history n` 的输出请尽可能与 bash 相同。

提示：可以考虑学习 bash/zsh，在退出程序时将历史命令记录在一个文件中（如 `.bash_history` 和 `.zsh_history`），也可以自己想其他的办法。

可选：实现用 `!!` 命令重复执行上一条命令，例如：

```shell
$ !!
echo taokystrong
taokystrong
```

### 处理 Ctrl-D (EOF) 按键（选做）

在使用 shell 的时候输入 `exit` 即可退出 shell，Ctrl-D (EOF) 在这种场景下等同于 `exit`。

### 支持 Bash 风格的 TCP 重定向（选做）

在精简的 Linux 环境中（如 Docker 容器里），常常是没有 `nc` 命令用来进行原始的 TCP 网络通信的。Bash 和一些其他 shell 支持一种特殊的重定向语法：`/dev/tcp/<host>/<port>`。

通过查看 [Bash 的 man 文档][bash.1]，`REDIRECTION` 一节，当重定向目标是下面几种路径，且操作系统没有提供这个路径时，Bash 会自行处理它们：

```text
/dev/fd/<fd>
/dev/stdin
/dev/stdout
/dev/stderr
/dev/tcp/<host>/<port>
/dev/udp/<host>/<port>
```

阅读相关文档，模拟 Bash 的行为实现 `cmd > /dev/tcp/<host>/<port>` 和 `cmd < /dev/tcp/<host>/<port>` 的重定向。你可能要用到的函数：`socket`, `connect`

方便起见，你只需要处理 `<host>` 是典型的 IPv4 地址（即 `a.b.c.d` 的形式，其中 abcd 均为 0 ~ 255 之间的整数）且 `<port>` 为 1 ~ 65535 之间的整数时的情况。

[bash.1]: https://linux.die.net/man/1/bash

### 更多功能（选做）

我们一般使用的 shell 非常强大，你还可以自行了解下面这些语法的含义：

```shell
echo $SHELL
A=1 env
alias ll='ls -l'
echo ~root
(sleep 10; echo aha) &
if true; then ls; fi
ifdown eth0 && ifup eth0
set -u # check existence of variable
```

请自行选择一个或多个功能并实现它们。

## 编写 tiny strace 工具

Linux 系统中有许多用于监控、追踪系统状态的工具，如下图所示。

![horror image](https://i.stack.imgur.com/ntC1q.png)

`strace` 是一个用于监控进程系统调用的程序，例如，在 Debian 系统中使用 `strace` 追踪 `true` 命令（一个什么都不做并返回 0 的命令），可以看到类似以下输出：

```shell
$ strace true
execve("/usr/bin/true", ["true"], 0x7ffd34ea2230 /* 59 vars */) = 0
brk(NULL)                               = 0x55c95678b000
arch_prctl(0x3001 /* ARCH_??? */, 0x7fff46be7eb0) = -1 EINVAL (Invalid argument)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=223491, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 223491, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f16a8186000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\320\324\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
pread64(3, "\4\0\0\0@\0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0"..., 80, 848) = 80
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0\205vn\235\204X\261n\234|\346\340|q,\2"..., 68, 928) = 68
newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=2463384, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f16a8184000
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
mmap(NULL, 2136752, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f16a7f7a000
mprotect(0x7f16a7fa6000, 1880064, PROT_NONE) = 0
mmap(0x7f16a7fa6000, 1531904, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x2c000) = 0x7f16a7fa6000
mmap(0x7f16a811c000, 344064, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1a2000) = 0x7f16a811c000
mmap(0x7f16a8171000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1f6000) = 0x7f16a8171000
mmap(0x7f16a8177000, 51888, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f16a8177000
close(3)                                = 0
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f16a7f77000
arch_prctl(ARCH_SET_FS, 0x7f16a7f77740) = 0
set_tid_address(0x7f16a7f77a10)         = 185317
set_robust_list(0x7f16a7f77a20, 24)     = 0
rseq(0x7f16a7f780e0, 0x20, 0, 0x53053053) = 0
mprotect(0x7f16a8171000, 12288, PROT_READ) = 0
mprotect(0x55c954a3b000, 4096, PROT_READ) = 0
mprotect(0x7f16a81f2000, 8192, PROT_READ) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
munmap(0x7f16a8186000, 223491)          = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```

这里我们可以看到，strace 列出了 true 这一程序调用的所有 Linux syscall。
接下来，我们将利用 ptrace 这一框架实现一个类似 strace 的工具。

### 文档

在进行程序编写前，请先简要阅读 [ptrace 文档](https://man7.org/linux/man-pages/man2/ptrace.2.html)。
一个示例性的例子可以在[这篇文章](https://nullprogram.com/blog/2018/06/23/)中找到。
我们将围绕这篇文章提供一些解释，然后列出具体需要实现的功能

### 简单追踪

首先要监听一个进程的 syscall，需要有另外一个进程。
这里我们首先用 `fork()` 创建一个子进程，在子进程中开启 ptrace，然后调用 `execvp()` 运行相应的程序：

```c
pid_t pid = fork();
switch (pid) {
  case -1:
    exit(1)
  case 0:
    ptrace(PTRACE_TRACEME, 0, 0, 0);
    execvp(argv[1], argv + 1);
    exit(1)
}
```

子进程在执行到 `PTRACE_TRACEME` 后会停住等待父进程指令。
在父进程中，使用：

```c
waitpid(pid, 0, 0);
```

!!! info 不能使用 `wait()`，因为需要传入 PID

来等待子进程的这次停止。
等到后，使用：

```c
ptrace(PTRACE_SETOPTIONS, pid, 0, PTRACE_O_EXITKILL);
```

来设置一些参数。
这里 `PTRACE_O_EXITKILL` 指示子进程在父进程退出后立即退出。
更多的参数及具体含义可以在前述文档中找到。

现在就可以正式开始监听子进程 syscall 了。
首先在父进程中运行：

```c
ptrace(PTRACE_SYSCALL, pid, 0, 0);
waitpid(pid, 0, 0);
```

来让子进程恢复执行，并让子进程在下一次 syscall 的入口处停住；
`waitpid()` 会让父进程等待到目标时刻再恢复执行。

`waitpid()` 结束后，子进程停在 syscall 入口处。
父进程此时恢复执行。
此时，父进程可以使用：

```c
struct user_regs_struct regs;
ptrace(PTRACE_GETREGS, pid, 0, &regs);
long syscall = regs.orig_rax;
fprintf(stderr, "%ld(%ld, %ld, %ld, %ld, %ld, %ld)", syscall,
  (long)regs.rdi, (long)regs.rsi, (long)regs.rdx,
  (long)regs.r10, (long)regs.r8,  (long)regs.r9
);
```

来读取子进程中进行 syscall 时各个 CPU 寄存器的值。
由于是在 Linux 环境中，function call 遵循 Linux system call convention，参数按顺序排列在 rdi、rsi、rdx、r10、r8、r9 寄存器中，syscall number 置于 rax 寄存器中。
通过读取这些寄存器，我们就可以知道参数的值和子进程具体要调用的 syscall。

!!! info 指针参数

    当参数是指针时，这里从寄存器中读出的值当然是指针的值。
    而不同进程间，指针并不能直接解引用而访问具体数据。
    ptrace 中，可以使用 `PTRACE_PEEKTEXT` 或是 `PTRACE_PEEKDATA`（Linux 上两者等价）来读取指针指向的数据。
    然后，在本次实验中，**并不**要求对指针进行解引用。
    测试中将会主要考察 integer 类型的参数。

在父进程中完成信息获取后，再次调用 `PTRACE_SYSCALL` 来让子进程恢复执行并停在 syscall 出口处：

```c
ptrace(PTRACE_SYSCALL, pid, 0, 0);
waitpid(pid, 0, 0);
```

这里可以看到，`PTRACE_SYSCALL` 有两种作用，能让子进程恢复执行并停留在 syscall 入口或出口处。
需要注意的是具体情况中是停在出口处还是停在入口处是**无法分辨**的，需要自己用代码在父进程中维护状态。
具体可能的流程在前述文档中有详细描写。

在 syscall 出口处时，返回值存在 rax 寄存器中。
我们可以通过读取寄存器 rax 来获取返回值，甚至是修改返回值：

```
ptrace(PTRACE_GETREGS, pid, 0, &regs);
fprintf(stderr, " = %ld\n", (long)regs.rax);
```

!!! info 注意与入口处不同，出口处用的是 `rax` field 而不是 `orig_rax` field

这样，一个可用的类 strace 工具就实现了。

输出时，每一个 syscall 请用 `"%ld(%ld, %ld, %ld, %ld, %ld, %ld)\n"` 的形式输出。
由于测试中会使用正则表达式进行匹配，一些小细节，例如程序开始/结尾额外输出的信息、多余的空格，不会影响答案的测评。
保证 syscall number 和参数正确即可。

### 处理子进程（选做）

在子进程 `fork()` 或是 `clone()` 后，子进程的子进程由于没有进行相应配置，是无法使用上述方法追踪 syscall 的。
ptrace 提供了相应的解决方法。
在 `PTRACE_SETOPTIONS` 中，可以配置更多的选项来自动追踪子进程。
请仔细阅读 ptrace 文档，理解具体方案，提供一个类似上述程序、但能够正确处理子进程的实现。

一些可能有用的信息：

- `PTRACE_SYSCALL` 和 `waitpid()` 可能因 signal 而退出，但此次实现不考虑 signal 的问题
- `PTRACE_O_TRACEFORK` 和 `PTRACE_O_TRACECLONE` 会在 syscall **出口处**进行通知

## 实验要求

请按照以下目录结构组织你的 GitHub 仓库：

```tree
.                         // Git 仓库目录
├── lab2                  // 实验二根目录
│   ├── shell             // 实验的 shell 部分
│   │   ├── shell.cpp     // 你的 shell 的源代码
│   │   ├── other.cpp
│   │   └── Makefile      // （可选）你提供的 Makefile
│   ├── strace            // 实验的 strace 部分
│   │   ├── strace.cpp    // 你的 strace 的源代码
│   │   ├── other.cpp
│   │   └── Makefile      // （可选）你提供的 Makefile
│   └── README.md         // 简单描述的实验报告，不必过长
├── .gitignore
└── README.md
```

本次实验满分 12 分。你需要完成：

- 按照上面的要求组织仓库结构，提交你的 shell 源代码和 strace 源代码
  - 如果你提交的内容无法正常编译，我们会尝试修复并酌情扣除一定分数
  - 实验一中对于 Git 工具的使用要求仍然适用于本实验，即当出现以下情况时，我们会酌情扣除一定分数
    - 很大一部分的 commit 由 GitHub 网页版生成，即通过网页版文件上传的方式提交实验的文件
    - 只有寥寥无几的 commit
    - 上传了大量与实验要求无关的文件
- shell 部分：必做项目共 8 分。对于额外的选做项目，shell 部分总分不超过 10 分：
  - 你的 shell 能够正确处理含有 1 个管道的命令，如 `ls | grep shell`（1 分）
    - 你的 shell 能够正确处理含有多个管道的命令，如 `ls | cat -n | grep shell`（1 分）
  - 你的 shell 支持 `>`, `>>`, `<` 重定向（各 0.5 分）
  - 你的 shell 在遇到 Ctrl-C 信号时不会中断 （0.5 分）
    - 你的 shell 在遇到 Ctrl-C 时能丢弃已经输入一半的命令行，显示 `#` 提示符并重新接受输入（1 分）
  - 你的 shell 支持 `history n` 命令（1 分）
    - 你的 shell 支持通过 `!n` 重复执行历史记录中的第 n 条命令（1 分）
    - 你的 shell 支持通过上下键切换历史命令（1 分）
- strace 部分：必做项目共 3 分。对于额外的选做项目，shell 部分总分不超过 4 分：
  - 你的 strace 程序能够追踪 syscall（3 分）
  - 你的 strace 程序支持追踪 fork 和 clone 后的子进程（各 0.5 分）

本实验可以使用 C/C++ 或 Rust 语言完成。

本实验可以使用 libc, libstdc++, libm 以及 iostream, STL 等 C/C++ 语言标准和常用库。如果你愿意，你也可以使用 readline 和 ncurses 等 Linux 程序常用库。使用此处没有列出的库前请询问助教。

### 关于实验报告

我们推荐你在 `README.md` 中写少量内容：

- 你的 shell 实现可能与系统中的 `bash`（或助教期望的表现）有所不同，简要介绍这些潜在的区别，以免产生误会，导致不必要的扣分。
- 你完成了一些选做项目，也可以简单介绍，方便助教进行更准确的评估

本实验的主要内容为 shell 程序的编写，因此不必花费太多工夫在实验报告上。

### 关于选做项目

在必做部分以外，你可以参考标记为「选做」的几个小节中介绍的 shell 的常见功能并实现来得到选做分数（不限制为本文档列出的功能，见下）。每一项额外功能都会由助教讨论评估，但通常单个项目不会超过 1 分。

我们鼓励进行与操作系统相关的实验探究，因此过度脱离主题的项目可能不会获得加分，例如：

- 过于简单的内置命令，如 `:` (colon), `true`, `false`, `help` 等
- 严重偏离 shell 的基本功能的项目，例如你[模仿 Zsh](https://github.com/johan/zsh/blob/master/Functions/Misc/tetris) 为你的 shell 内置了一个俄罗斯方块游戏

  ![Zsh Tetris](https://i.redd.it/blfzzmopc7j41.png)

作为一个参考基准，GNU Bash 具有的功能大部分都会被认可。

### 关于编译

本实验中，如果你提供了 `Makefile` 或者 `cargo.toml` 文件，我们将使用它来编译你提交的程序；否则，我们将尽可能去尝试编译 `lab2/` 目录下所有的 `*.c`、`*.cpp` 或者 `*.rs` 文件。在任何情况下，你也可以在 `README.md` 中说明编译与运行相关的注意事项。

## 示例程序

```cpp
// IO
#include <iostream>
// std::string
#include <string>
// std::vector
#include <vector>
// std::string 转 int
#include <sstream>
// PATH_MAX 等常量
#include <climits>
// POSIX API
#include <unistd.h>
// wait
#include <sys/wait.h>

std::vector<std::string> split(std::string s, const std::string &delimiter);

int main() {
  // 不同步 iostream 和 cstdio 的 buffer
  std::ios::sync_with_stdio(false);

  // 用来存储读入的一行命令
  std::string cmd;
  while (true) {
    // 打印提示符
    std::cout << "# ";

    // 读入一行。std::getline 结果不包含换行符。
    std::getline(std::cin, cmd);

    // 按空格分割命令为单词
    std::vector<std::string> args = split(cmd, " ");

    // 没有可处理的命令
    if (args.empty()) {
      continue;
    }

    // 更改工作目录为目标目录
    if (args[0] == "cd") {
      if (args.size() <= 1) {
        // 输出的信息尽量为英文，非英文输出（其实是非 ASCII 输出）在没有特别配置的情况下（特别是 Windows 下）会乱码
        // 如感兴趣可以自行搜索 GBK Unicode UTF-8 Codepage UTF-16 等进行学习
        std::cout << "Insufficient arguments\n";
        // 不要用 std::endl，std::endl = "\n" + fflush(stdout)
        continue;
      }

      // 调用系统 API
      int ret = chdir(args[1].c_str());
      if (ret < 0) {
        std::cout << "cd failed\n";
      }
      continue;
    }

    // 显示当前工作目录
    if (args[0] == "pwd") {
      std::string cwd;

      // 预先分配好空间
      cwd.resize(PATH_MAX);

      // std::string to char *: &s[0]（C++17 以上可以用 s.data()）
      // std::string 保证其内存是连续的
      const char *ret = getcwd(&cwd[0], PATH_MAX);
      if (ret == nullptr) {
        std::cout << "cwd failed\n";
      } else {
        std::cout << ret << "\n";
      }
      continue;
    }

    // 设置环境变量
    if (args[0] == "export") {
      for (auto i = ++args.begin(); i != args.end(); i++) {
        std::string key = *i;

        // std::string 默认为空
        std::string value;

        // std::string::npos = std::string end
        // std::string 不是 nullptr 结尾的，但确实会有一个结尾字符 npos
        size_t pos;
        if ((pos = i->find('=')) != std::string::npos) {
          key = i->substr(0, pos);
          value = i->substr(pos + 1);
        }

        int ret = setenv(key.c_str(), value.c_str(), 1);
        if (ret < 0) {
          std::cout << "export failed\n";
        }
      }
    }

    // 退出
    if (args[0] == "exit") {
      if (args.size() <= 1) {
        return 0;
      }

      // std::string 转 int
      std::stringstream code_stream(args[1]);
      int code = 0;
      code_stream >> code;

      // 转换失败
      if (!code_stream.eof() || code_stream.fail()) {
        std::cout << "Invalid exit code\n";
        continue;
      }

      return code;
    }

    // 外部命令
    pid_t pid = fork();

    // std::vector<std::string> 转 char **
    char *arg_ptrs[args.size() + 1];
    for (auto i = 0; i < args.size(); i++) {
      arg_ptrs[i] = &args[i][0];
    }
    // exec p 系列的 argv 需要以 nullptr 结尾
    arg_ptrs[args.size()] = nullptr;

    if (pid == 0) {
      // 这里只有子进程才会进入
      // execvp 会完全更换子进程接下来的代码，所以正常情况下 execvp 之后这里的代码就没意义了
      // 如果 execvp 之后的代码被运行了，那就是 execvp 出问题了
      execvp(args[0].c_str(), arg_ptrs);

      // 所以这里直接报错
      exit(255);
    }

    // 这里只有父进程（原进程）才会进入
    int ret = wait(nullptr);
    if (ret < 0) {
      std::cout << "wait failed";
    }
  }
}

// 经典的 cpp string split 实现
// https://stackoverflow.com/a/14266139/11691878
std::vector<std::string> split(std::string s, const std::string &delimiter) {
  std::vector<std::string> res;
  size_t pos = 0;
  std::string token;
  while ((pos = s.find(delimiter)) != std::string::npos) {
    token = s.substr(0, pos);
    res.push_back(token);
    s = s.substr(pos + delimiter.length());
  }
  res.push_back(s);
  return res;
}
```
