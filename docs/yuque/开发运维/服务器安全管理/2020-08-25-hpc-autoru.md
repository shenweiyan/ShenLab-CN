---
title: 一次 HPC 病毒感染与解决经历
urlname: 2020-08-25-hpc-autoru
author: 章鱼猫先生
date: 2020-08-25
updated: "2021-06-30 09:36:57"
---

周一的时候，有同事反馈说，HPC 的项目报告路径正在不断产生 \*.exe 和 \*.pif 文件，怀疑是不是被病毒感染！
![image.png](https://shub.weiyan.tech/yuque/elog-cookbook-img/FruWFOZWN1SO32k60dIGJ0k2RW4C.png)
收到信息，第一时间进去目录，的确发现该目录每个几秒钟就自动生成一个 \*.exe 和 \*.pif 二进制文件。

于是：

1.  检查垃圾文件的属主，ps、htop、top 排查该属主的进程，均没有发现任何正在运行的相关程序。
2.  用 crontab 检查是否有定时运行的任务，也没有任何发现。

为了防止 \*.exe 和 _.pif 垃圾文件的持续生成，占满磁盘，我们在 /etc/passwd 中把有问题的用户进行了注释禁用，同时把正在对发生问题目录进行数据拷贝的所有移动硬盘进行终止，并移除，这样一来发现，_.exe 和 \*.pif 二进制文件自动生成的情况消失了。

由于发生问题所在的目录是通过 Samba 服务与本地的几台 Windows 台式机进行了同步，用于客户数据上传下载的拷贝。我们排查了 HPC 发生问题的时间，以及相关时间段的服务记录，没发现什么异常，服务器 io 也正常，而且生成的 exe 二进制文件正常来说在 Linux 上也是无法执行的，一般只有 Windows 对 exe 可执行文件比较敏感。

于是，初步怀疑基于 Samba 服务的台式电脑存在异常，在不断自我复制产生 \*.exe 和 \*.pif 二进制文件。

接下来，我们一个个去查看当天用于上传下载目标目录的移动硬盘的文件，终于发现一个名字为 autorun.inf 的可疑文件，同时也发现了这个程序里面的 tpkv.exe 程序：

![autorun-inf.png](https://shub.weiyan.tech/yuque/elog-cookbook-img/Fkg9mQR4tnY1isIy18-Katshmgry.png)

针对 autorun.inf，谷歌了一下，发现：

> Autorun 病毒是一种专门针对 Windows 系列操作系统的病毒，它是依靠 Windows 的 Autorun/Autoplay 功能发作传播的。这个功能的目的是在电脑上插入闪存盘等可移动设备或是光盘之后，自动执行一系列的指令。在 Windows XP 之前， Autorun 和 Autoplay 指的是同一个功能，但是在 Windows XP 之后，这两个单词指不同的功能。在中文版的 XP 里，前者被翻译为“自动运行”，而后者叫“自动播放”。其中自动运行是指：
>
> - 对光盘来说，放入光盘后，自动执行 `autorun.inf` 文件中规定的程序。
> - 对其它可移动设备来说（闪存盘、移动硬盘等），**当双击盘符时**，自动执行 `autorun.inf` 文件中规定的程序。
>
> 摘自：《[Autorun 的介绍及彻底防治 U 盘病毒](https://wzyboy.im/post/492.html)》

我们把这两个文件从移动硬盘里面去掉，更换到其他的台式机器上重新执行数据上传下载拷贝，发现再也没有出现不断自我复制产生 \*.exe 和 \*.pif 二进制文件的现象！

最后，简单总结一下。

HPC 服务是一个极其重要且敏感的集群服务，包括了诸多重要数据和程序信息资料，大部分都是基于内部网络环境进行相关数据处理和传输，对于第三方数据上传下载一定要做好安全防护，尽量不要直接与服务器连接进行传输。其次，做好每一个账号的权限、历史记录等相关设置，方便出现异常时进行问题排查。

HPC 管理与维护是一个系统化的工作，对于老旧的服务器更考验一个运维人员的业务水平和能力，以及细心程度，如果你有类似经验，非常欢迎随时和我分享。
