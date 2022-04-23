# 实验二

!!! Warning "本文仍在编辑中"

在实验一中，我们使用了许多命令行操作来完成一系列工作，如编译 Linux 内核、打包 initrd 等。所有的命令行都由一个叫做 shell 的程序解释并执行。本实验我们将自己编写一个简单的 shell 并理解 Linux shell 程序的工作原理。

## 编写 Shell 程序

首先，大家可以将本页底部助教编写的一个示例程序命名为 `shell.cpp`：

```shell
g++ shell.cpp -o shell
```

以上命令会调用 g++ 编译器编译出一个可执行文件 `shell`，你可以继续输入 `./shell` 来运行它。这是一个非常简陋的 shell，它会提示你输入命令，你可以输入 `cd` 来切换工作目录，输入 `pwd` 显示当前目录，输入 `export` 导出环境变量，输入 `exit` 退出，或者调用系统中有的其他命令来运行，例如 `ls`、`cat` 等。

你的任务是对这个 shell 程序进行修改升级（当然，你也可以选择自己从头或选择其他语言编写），并完成下面的任务。

### 更健壮（推荐，但不要求）

目前的示例程序非常脆弱，仅随意检查了部分可能的错误。建议你将它改得更健壮，以方便之后的进一步开发和调试。

### 支持管道

形如 `env | wc` 这样的命令利用了「管道」语法，将两条不同的命令对接在一起同时运行。`|` 的意思是将前面的命令 `env`（输出所有环境变量）的标准输出连接到后面命令 `wc`（统计行数）的标准输入（这样就能统计出环境变量的总数）。请你观察并学习这个语法的效果，为你的 `shell` 程序实现这一功能。

你可能要用到的函数：`pipe`、`close`、`dup`。你可以运行 `man 函数名` 来查看系统自带的文档，或者上网搜索更多信息。

### 支持基本的文件重定向

形如 `ls > out.txt` 会将 `ls` 命令的输出重定向到 `out.txt` 文件中，具体地说，会将 `out.txt` 关联到程序的标准输出，然后再运行相应的命令。

类似的，`ls >> out.txt` 会将输出追加（而不是覆盖）到 `out.txt` 文件，`cat < in.txt` 会将程序的标准输入重定向到文件 `in.txt`。

请为你的 `shell` 程序实现 `>`、`>>` 和 `<` 的功能。

你可能要用到的函数：`open`、`close`、`dup`。

#### 支持基于文件描述符的文件重定向、文件重定向组合（选做）

请自行查找资料，实现以下这些文件重定向：

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

请为你的 `shell` 实现对 Ctrl-C 的处理。

提示：当你正确处理第一种情况后（丢弃未输入完的命令），第二种情况（终结运行中的程序）并不需要你做任何工作。

你可能要用到的函数：`signal`、`waitpid`。

### 支持 History

TODO

`history n` 显示最近执行的 n 条指令

注意 bash/zsh 的 fc 命令

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

阅读相关文档，模拟 Bash 的行为实现 `cmd > /dev/tcp/<host>/<port>` 和 `cmd < /dev/tcp/<host>/<port>` 的重定向。

方便起见，你只需要处理 `<host>` 是典型的 IPv4 地址（即 `a.b.c.d` 的形式，其中 abcd 均为 0 ~ 255 之间的整数）且 `<port>` 为 1 ~ 65535 之间的整数时的情况。

你可能要用到的函数：`socket`, `connect`

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

TODO for myl7

Linux 系统中有许多用于监控、追踪系统状态的工具，如下图所示。

![horror image](https://i.stack.imgur.com/ntC1q.png)

`strace` 是一个用于监控进程系统调用的程序，例如，在 Debian 系统中使用 `strace` 追踪 `true` 命令（一个什么都不做并返回 0 的命令），可以看到类似以下输出：

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
- shell 部分：必做项目共 6 分。对于额外的选做项目，shell 部分总分不超过 8 分：
  - 你的 shell 能够正确处理含有 1 个管道的命令，如 `ls | grep shell`（0.5 分）
    - 你的 shell 能够正确处理含有多个管道的命令，如 `ls | cat -n | grep shell`（0.5 分）
  - 你的 shell 支持 `>`, `>>`, `<` 重定向（各 0.5 分）
  - 你的 shell 在遇到 Ctrl-C 信号时不会中断 （0.5 分）
    - 你的 shell 在遇到 Ctrl-C 时能丢弃已经输入一半的命令行，显示 `#` 提示符并重新接受输入（1 分）
  - TODO 你的 shell 支持 history 相关命令（2 分）
- strace 部分：必做项目共 3 分。对于额外的选做项目，shell 部分总分不超过 4 分：
  - TODO

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
