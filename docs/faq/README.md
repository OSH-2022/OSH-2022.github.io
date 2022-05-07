# 常见问题

往年的 FAQ 亦可作参考，可以参见[实验零中的描述](https://osh-2022.github.io/lab0/#%E6%9C%AC%E5%AD%A6%E6%9C%9F%E7%9A%84%E5%AE%9E%E9%AA%8C%E8%AE%BE%E8%AE%A1)到往年主页进行查看。

## 实验零

**Q1：可以使用 WSL/Vlab 完成实验吗？**

A1：参见[实验零中的不同平台兼容性描述](https://osh-2022.github.io/lab0/#%E4%B8%8D%E5%90%8C%E5%B9%B3%E5%8F%B0%E5%85%BC%E5%AE%B9%E6%80%A7)

## 实验二

**Q1：strace 第一行的 exec\* 没有 trace 到怎么办？**

A1：可以料想到，第一行 exec* 之后需要 trace 的程序才启动了。第一行的 exec* 不考虑即可。

**Q2：为什么我的 getline/cin 读不到东西？**

A2：先检查一下看子进程是不是套了两层。

**Q3：Ctrl+C 读不到？/Ctrl+D 没法马上起效**

A3：Ctrl+C 发 signal，而 Ctrl+D 则是输入 EOF。
