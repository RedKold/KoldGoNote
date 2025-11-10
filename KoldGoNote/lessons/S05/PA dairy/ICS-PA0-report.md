## 学会如何提问

yzh 老师说：结合自己在大一时提问和被提问，以及完成 PA0 的经历——但事实上，上 ICS 做 PA 时我已经大三了（计金）。我对 Linux 的接触是从大二的计算机网络课程开始的，在第一届计网课改的新实验中被折腾的死去活来。不过，那段半夜和助教一起看 bug 的日子，确实让我有了很多关于“提问”的思考，尤其在读完 **提问的智慧** 和 **别像弱智一样提问** 之后。

## 完成 PA0 使我学到了那些

PA0 的主要任务是配置开发环境，我是 Mac 用户，ARM 架构不支持本次实验，所以我选择捡起来去年计网用的阿里云服 x86 ubuntu 务器。采取 xfce4+xrdp 的方案远程开发。期间也遇到了一些问题。

### AI 是我的好朋友
如今做 PA 的朋友们有得天独厚的一个优势：更加强大的 AI。无论是配置环境，还是快速了解 vim 的基本用法，AI 都是一个很好的朋友。

举一个例子：

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250901214039.png)

- 当我遇到 xfce4 的桌面无法打开 firefox 浏览器，我首先想到这会不会是由于 `DISPLAY` 没有配置好，导致图形化无法正常运作，为了排除这个问题，我问了 AI 关于启动报错的细节
- AI 迅速给了我了一套解决方案

在 AI 的指示下，我检查了是否 `DISPLAY` `DBUS` 等环境变量一切正常，确认无误后，再回来尝试解决 memlimits 的问题。


### AI 不是万能的，STFW 和 RTFM 永不过时

然而，在我反复询问 GPT 和 Gemini 之后，都没有解决上述问题。这时候我尝试用 google 搜索。

**有一个深刻的体会**：
- **仔细组织你的问题，能节省大量查找时间**

我没有保存搜索记录，回忆一下我大概是这么组织的：

*linux xfce4 xrdp, unable to change memlock limits, ssh*

这仍然是一个很优秀的问题，**但是要素已经足够齐全**，并且的确帮助到了我。
- 首先，外文资料远高于简中社区资料量，it' s a wise way to ask question in English
- 根据 [ryanhanwu/How-To-Ask-Questions-The-Smart-Way: 本文原文由知名 Hacker Eric S. Raymond 所撰寫，教你如何正確的提出技術問題並獲得你滿意的答案。](https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way/tree/main)所说，提出聪明的问题，**需要你**注明自己的环境如何（提供解决问题的相关信息，容易复现）
- 又包括最关键的报错信息（即 `memlock limits` 问题）。
- `ssh` 则是我考虑到可能是由于我用 `ssh` 远程连接操作，**可能存在一些未知的权限问题**。所以加上作为关键字。


我检索到了一系列信息，有的是 Linux 社区很古早的讨论，有的是 github 上有人给 clearlinux 项目提的 issue，在耐心阅读的过程中，我剥离一些和我情况不一样的问题，逐渐清晰自己的目标和面临问题的关系.

- ([Unable to set memlock unlimited · Issue #2372 · clearlinux/distribution](https://github.com/clearlinux/distribution/issues/2372))
![image.png|700](https://kold.oss-cn-shanghai.aliyuncs.com/20250901215436.png)
- 这篇文章虽然没有解决我的问题，但是促使我思考，AI 给出的方案修改 `/etc/security/limits.conf` 是否因为**权限**问题而不能起作用

之后我优化了自己的问题，加上了 **权限** 的提示词，这次我找到[答案](https://serverfault.com/questions/569288/ulimit-n-not-changing-values-limits-conf-has-no-effect)，是在 serverfault 网站上的问题。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250901220700.png)
- **解决问题的同时**，我也通过阅读这篇回答，学到了如何让自己的问题变得更好
	- 他详细列出了自己解决问题的过程
		- 阅读其他人的问题
		- 尝试解决
		- 解决未果（still see...）
		- 操作细节 (after reboot)
		- 操作环境
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250901220854.png)

- 出现了！这篇回答和我的情况一样，SSH **登录**。通过终端操作。

- 我在询问 AI 修改 `UsePrivilegeSeparation no` 可以解决问题的机理后，又得到了 ssh 登录权限问题，可能没有默认加载我的设置文件，所以前面的内存设置修改无效。

- 这是我 PA0 实验一个很小的部分，但充分让我理解到 **提问** 对于解决问题的重要性，包括
	- 要有提问的意识
	- 要问出好问题
	- 要灵活运用 AI 和 Google 等工具
	- 要耐心阅读问题的回答结果，和其他人问的问题（可能对你有启发）

## 结语
PA0让我深刻体会到，在技术的世界里，解决问题的过程本身，**就是最好的学习。**

我从一开始的懵懂，尝试依赖 AI 快速解决，到面对它的局限性，开始自己去摸索和提问。我学会了如何把一个模糊的报错，分解成一个个精确的关键字；如何通过添加 `权限`、`SSH` 等看似细枝末节的词汇，让我的问题变得更有针对性。

我发现，`提问的智慧` 不仅仅是关于如何向别人寻求帮助，更是一种自我反思和梳理思路的方式。我耐心地在社区论坛中剥离出与我情况相似的问题，从别人的提问方式中学习，最终找到了解决问题的关键线索。

这短短的实验过程，让我明白了三个道理：

1. **AI 是强大的助手，但不是万能的拐杖。**
2. **STFW（用搜索引擎找答案）和 RTFM（阅读手册）永远不会过时。**
3. **一个好的问题，本身就代表了对问题深刻的理解。**


### 附录
#### 解决 firefox 无法启动问题
在 PA0 note 中有所介绍。
- `firefox
```
Authorization required, but no authorization protocol specified
Error: cannot open display: : 10.0
```
- 解药
xrdp 的 `sesman` 有时会创建新 session，导致 `XAUTHORITY` 环境变量没带上。你可以手动导出：

`export XAUTHORITY=~/.Xauthority firefox`

为了方便，我们可以把这个命令写入 xfce4 的初始化文件