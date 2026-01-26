---
title: 记一次在 Windows IIS 中配置 FTP 站点并为 Windows 防火墙添加例外规则
abbrlink: 714d
categories: Windows
tags:
  - 系統折騰
  - FTP
  - 網絡
date: 2026-01-23 22:50:58
---
### 前言

笔者要调试安卓程序，过程中就不免经常在安卓和 PC 之间交换文件，交换文件能直接用 USB，也能用 Local Send 这样的工具——但是 USB 不但插拔很麻烦，还要占用掉一个 USB 口，让我笔记本本就稀缺的 USB 口雪上加霜；Local Send 这种工具呢，又不太原生，需要额外下两个软件不说，每次使用还要同时在手机和电脑上打开这个软件，要点很多次；而且如果传送大文件，得益于安卓狗屎的后台保留，手机息屏有概率杀后台，如果把 Local Send 程序杀了，辛辛苦苦传半天的文件就飞烟灭了！虽然不是每次都这样吧，但这种事情只要经历过一次，心里肯定就不那么好受。

所以就只剩下两种原生的网络传输方式了：SMB 和 FTP，好巧不巧 MT 管理器免费版还不支持 SMB，那就只能用 FTP 了。

按理说在现在的时代，在一个主流系统上开启一个主流的文件传输协议，应该并不难——笔者之前也是这么想的，直到自己真的动手，才发现自己错了，**WIndows 不管 IIS 还是防火墙，控制台都非常老旧，配置起来简直一步一个坑！** 所以写了这篇博客，是为吐槽，也是记录，又能为找到这篇文章的后人避坑：
### 开启 IIS

控制面板 > 程式集 > 程序和功能 > 开启或关闭 Windows 功能：

![](https://img.sokny.eu.org/file/posts/V6h6qggZ.webp)

弹出的窗口中勾选：
- Internet Information Server 
	- > FTP 伺服器
		- > **FTP 伺服器**
		- > **FTP 扩充性**
	- > Web 管理工具
		- > **IIS 管理主控台**
			

![](https://img.sokny.eu.org/file/posts/xpB0dKnJ.webp)

稍等片刻，套用完成后就可以在开始菜单中搜寻到 IIS 了

![](https://img.sokny.eu.org/file/posts/pIjYIa59.webp)

到这里先别急着下一步，接下来是第一个坑： Windows FTP 使用的密码只能使用 Windows 账户的密码，没法专门为 Windows FTP 创建一个密码，所以如果你的 Windows 账户没有密码，请先**新建一个新的带有密码的 Windows 账户**再进行接下来的操作：

*这里可能有读者觉得创建账户有什么好教的，笔者之前也是这么想的，真创建才发现真是一步一个坑*
### 新建带有密码的 Windows 账户

*如果你的 Windows 账户有密码，可以跳过这一步*

设定 > 账户 > 家人与其他使用者 > 将其他人新增至此电脑：

![](https://img.sokny.eu.org/file/posts/gPJNgsN8.webp)

坑来了：这里要求我们创建线上账户，因为我们只是要创建一个本地账户给 FTP 用，所以要点击「我没有这位人员的登入资讯」：

![](https://img.sokny.eu.org/file/posts/eB1fO2TV.webp)

等加载完成后，再点击「新增没有 Microsoft 账户的使用者」：

![](https://img.sokny.eu.org/file/posts/ucmHyhnY.webp)

终于到创建账户的界面了，填上你的名字（建议使用 「用户名」 + -FTP 的格式，比较规范）：

![](https://img.sokny.eu.org/file/posts/7rMV5sMg.webp)

这里坑爹的是三个安全问题全要填写不能空着，笔者只能编了几个答案跳过：

![](https://img.sokny.eu.org/file/posts/a65Xudys.webp)

等加载完，新账户就出现在这了

![](https://img.sokny.eu.org/file/posts/Dkdw5iat.webp)

### 创建 FTP 仓库

FTP 需要一整个文件夹，所以新建一个文件夹：

![](https://img.sokny.eu.org/file/posts/s17n5V5E.webp)

#### 让此仓库可以用于 FTP：

右键 > 内容：

![](https://img.sokny.eu.org/file/posts/IRxddN5d.webp)

顶部菜单点击：共用 > 共用：



下拉菜单中找到新建的账户，点击「读取」，下来菜单中改成「读取/写入」：

![](https://img.sokny.eu.org/file/posts/EmJ8iP8X.webp)


加载完就成功共用了：

![](https://img.sokny.eu.org/file/posts/ykgN9BYV.webp)


### 在 IIS 新增 FTP 站台

依次点击：「你的 PC 名字」 > 站台 > 新增 FTP 站台：

![](https://img.sokny.eu.org/file/posts/va0nM6Z1.webp)

命名，下面的框选择上一步创建的文件夹：

![](https://img.sokny.eu.org/file/posts/jwsuYhTv.webp)


在进行下一步之前，先在终端中输入 `ipconfig` 查看本地 IP：

![](https://img.sokny.eu.org/file/posts/zsFEAGWk.webp)


上面的框输入本地 IP，下面的框因为只是自用所以选择「没有 SSL」：

![](https://img.sokny.eu.org/file/posts/hjvk513t.webp)

这里要像我一样这么选：

![](https://img.sokny.eu.org/file/posts/e8T2snfg.webp)

单击「完成」，我们的 FTP 站点就建好了，可以直接在资源管理器输入 `ftp://<你的本地 IP>` 测试

![](https://img.sokny.eu.org/file/posts/UfkMjKnq.webp)

### 配置 Windows 防火墙

如果你的 Windows 防火墙是关的，那么在局域网内的所有设备都可以直接连接到 FTP 服务，但是这样做如果只是测试的话还好，在生产环境中使用简直是\*门大开，所以即使 Windows 防火墙管理器老旧的 GUI 配置起来相当麻烦也得保持它开启，只为 FTP 设置例外规则

开始吧：

然后打开设置 > 更新与安全性 > Windows 安全性 > 防火墙与网络防护：

![](https://img.sokny.eu.org/file/posts/WiMvg1zy.webp)

转到「防火墙与网络保护」 > 「进阶设定」：

![](https://img.sokny.eu.org/file/posts/cfqQGk4i.webp)

右键「输入规则」> 点击「新建规则」：

![](https://img.sokny.eu.org/file/posts/goRMjjlt.webp)

点击「连接埠」，下一步

![](https://img.sokny.eu.org/file/posts/CEFwkMbY.webp)

在这里键入「21」，

![](https://img.sokny.eu.org/file/posts/915bZMAr.webp)

一路下一步到这里，记得只勾选你希望生效的设定档：

![](https://img.sokny.eu.org/file/posts/h3GjQIct.webp)

名称随意，点「完成」

![](https://img.sokny.eu.org/file/posts/jRM7MENK.webp)

然后回到防火墙控制面板主界面，点击「输入规则」，找到图中三条全部依次启用：

![](https://img.sokny.eu.org/file/posts/8NPbMqSu.webp)

记得确认这里的项目，确保只勾选自己希望生效的设定档：

![](https://img.sokny.eu.org/file/posts/9wyRLV27.webp)

笔者当时第一次搞到这里的时候，网上的教程就都告诉我开放成功了，但是笔者用安卓设备连接 FTP 测试还是失败，

经过笔者探索，原来是开这么多东西还不行，**如果是「被动模式」的话，还要设置 FTP 被动模式端口的范围，再把“svchost.exe“这个程序添加到白名单**

还是要打开 IIS 控制台 > FTP 防火墙支援：

![](https://img.sokny.eu.org/file/posts/bnBp7Kno.webp)

在这里填入你要开放的端口范围（笔者填入 5000 - 5100），点保存

![](https://img.sokny.eu.org/file/posts/AJM6G8Ma.webp)

然后在终端输入以下命令

```

# 开放被动模式端口
netsh advfirewall firewall add rule name="FTP 端口範圍" ir=in action=allow protocol=TCP localport=5000-5100 

# 让 svchost.exe 通过防火墙
netsh advfirewall firewall add rule name="FTP 程序" dir=in action=allow program="%SystemRoot%\System32\svchost.exe" service=ftpsvc
```

打开 MT 传一个文件夹测试，终于成功了！

![](https://img.sokny.eu.org/file/posts/M6hvvD1Z.webp)





