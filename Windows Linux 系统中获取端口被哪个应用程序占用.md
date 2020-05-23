

管理服务程序的时候，可能会查询某个端口当前被哪个进程占用。不仅能找出有问题的进程将其处理掉，也可以用来辅助检查某个程序是否开启了服务并在监听端口。

---

@[TOC](本文内容)

## Windows 系统

Windows 系统上可以使用 PowerShell 命令来查询占用某个端口的程序。

比如，我们需要查询 5000 端口被占用的进程是谁，可以在 PowerShell 中输入命令：

```powershell
Get-Process -Id (Get-NetTCPConnection -LocalPort 5000).OwningProcess
```

![查询占用某端口的进程](https://blog.walterlv.com/static/posts/2020-03-11-15-04-19.png)

## Linux 系统

在终端中输入命令 `lsof` 可以查询占用某个端口的进程。

```powershell
lsof -i:端口号
```

比如，我们需要查询 5000 端口被占用的进程是谁，可以在中断中输入命令：

```bash
walterlv@localhost:~# lsof -i:5000
COMMAND        PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
dotnet_serve   731 root    3u  IPv6  12890      0t0  TCP *:5000
```

或者使用 netstat 查询。

```bash
netstat -tunpl | grep 端口号
```

举例：

```bash
walterlv@localhost:~# netstat -tunpl | grep 35412
tcp6   0   0 :::5000     :::*             731/dotnet_serve
```

---

我的博客会首发于 <https://blog.walterlv.com/>，而 CSDN 会从其中精选发布，但是一旦发布了就很少更新。

如果在博客看到有任何不懂的内容，欢迎交流。我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入。

![知识共享许可协议](https://img-blog.csdnimg.cn/20190406094629787.png)

本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。欢迎转载、使用、重新发布，但务必保留文章署名吕毅（包含链接：<https://walterlv.blog.csdn.net/>），不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请[与我联系](mailto:walter.lv@qq.com)。
