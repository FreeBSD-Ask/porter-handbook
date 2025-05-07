# 第16章 保持更新

FreeBSD Ports 不断变化。以下是一些保持更新的方式。

## 16.1. FreshPorts

通过订阅 [FreshPorts](https://www.freshports.org/) 是了解已提交更新的最简单方式之一。可以监控多个 Port。强烈建议维护者订阅，因为他们不仅会收到自己更改的通知，还会收到其他 FreeBSD 提交者的更改通知。（这些通常是保持更新 Ports 框架所必需的——尽管提前通知这些更改的提交者是很礼貌的做法，但有时会被忽视或不切实际。此外，在某些情况下，所做的更改是非常小的。我们期望大家在这些情况下使用最佳判断。）

要使用 FreshPorts，需要一个账户。使用 `@FreeBSD.org` 注册邮箱的人会在网页的右侧看到选择加入的链接。如果已有 FreshPorts 账户但没有使用 `@FreeBSD.org` 邮箱地址，可以将邮箱改为 `@FreeBSD.org`，然后订阅，再改回去。

FreshPorts 还具有一个健全性测试功能，会自动测试每次提交到 FreeBSD Ports 树中的文件。如果订阅了此服务，提交者将收到 FreshPorts 在对提交进行健全性测试时发现的任何错误通知。

## 16.2. 源代码库的 Web 界面

可以通过 Web 界面浏览源代码库中的文件。影响整个 Port 系统的更改现在已记录在 [CHANGES](https://cgit.freebsd.org/ports/tree/CHANGES) 文件中。影响单个 Port 的更改记录在 [UPDATING](https://cgit.freebsd.org/ports/tree/UPDATING) 文件中。然而，解决任何问题的权威答案无疑是阅读 [bsd.port.mk](https://cgit.freebsd.org/ports/tree/Mk/bsd.port.mk) 和相关文件的源代码。

## 16.3. FreeBSD Ports 邮件列表

作为 Port 维护者，考虑订阅 [FreeBSD ports 邮件列表](https://lists.freebsd.org/subscription/freebsd-ports)。有关 Ports 工作方式的重要更改将在那里发布，并随后提交到 **CHANGES**。

如果邮件列表的消息量太大，可以考虑关注 [FreeBSD ports announce 邮件列表](https://lists.freebsd.org/subscription/freebsd-ports-announce)，该列表只包含公告。

## 16.4. FreeBSD Port 构建集群

FreeBSD 的一项不太为人所知的强大功能是，有一个集群的机器专门用于持续构建 Ports 集合，涵盖每个主要的操作系统版本以及每个 Tier-1 架构。

除非特别标记为 `IGNORE`，否则会构建每个独立的 Port。标记为 `BROKEN` 的 Port 仍然会尝试构建，以查看底层问题是否已经解决。（这是通过将 `TRYBROKEN` 传递给 Port 的 **Makefile** 来完成的。）

## 16.5. Portscout: FreeBSD Ports Distfile 扫描器

构建集群专门用于构建每个 Port 的最新版本，使用已经抓取的 distfile。然而，随着互联网的不断变化，distfile 很快就会消失。[Portscout](https://portscout.freebsd.org/) 是 FreeBSD Ports 的 distfile 扫描器，它尝试查询每个 Port 的每个下载站点，查看每个 distfile 是否仍然可用。Portscout 可以生成 HTML 报告并向请求的人发送有关新可用 Port 的电子邮件。除非没有订阅，否则建议维护者定期检查更改，可以手动检查或使用 RSS 提要。

Portscout 的首页显示了 Port 维护者的电子邮件地址、维护者负责的 Ports 数量、其中有新 distfile 的 Port 数量，以及这些 Port 中过期的百分比。搜索功能允许按电子邮件地址搜索特定的维护者，并选择仅显示过期的 Ports。

点击维护者的电子邮件地址后，将显示其所有 Port 的列表，包括 Port 分类、当前版本号、是否有新版本、Port 上次更新的时间以及最后检查的时间。此页面上的搜索功能允许用户搜索特定的 Port。

点击列表中的 Port 名称后，会显示该 Port 在 [FreshPorts](https://freshports.org/) 上的信息。

附加文档可在 [Portscout 仓库](https://github.com/freebsd/portscout/) 中找到。
