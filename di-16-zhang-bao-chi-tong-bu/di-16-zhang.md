# 第16章 保持更新

FreeBSD Ports 收藏品在不断变化。以下是一些关于如何跟上的信息。

## 16.1. FreshPorts

一个了解已经提交更新的最简单方法是订阅 FreshPorts。可以监视多个ports。强烈建议维护人员订阅，因为他们将收到不仅是自己更改的通知，还有任何其他 FreeBSD 提交者进行的更改通知。（通常必须跟踪底层ports框架的更改 - 虽然最好收到提交这些更改的人提前通知，但有时会被忽略或不切实际。此外，在某些情况下，更改的性质非常微小。我们期望每个人在这些情况下都能做出最好的判断。）

使用 FreshPorts 需要一个帐户。在 @FreeBSD.org 注册电子邮件地址的人将在网页右侧看到选择加入的链接。那些已经拥有 FreshPorts 帐户但未使用 @FreeBSD.org 电子邮件地址的人可以更改电子邮件地址为 @FreeBSD.org ，订阅，然后再次更改回来。

FreshPorts 亦具备一个健全性测试功能，该功能会自动测试提交至 FreeBSD 1001 树的每个提交。如果订阅了此服务，提交者将会收到 FreshPorts 在测试其提交的健全性测试过程中检测到的任何错误的通知。

## 16.2. 源代码库的 Web 接口

可以通过 web 接口浏览源代码库中的文件。影响整个 1001 系统的更改现在在 CHANGES 文件中有所记录。影响个别 1002 的更改现在在 UPDATING 文件中有所记录。然而，对于任何问题的明确答案无疑是阅读 bsd.1003.mk 和相关文件的源代码。

## 16.3. FreeBSD Ports 邮件列表

作为ports维护者，请考虑订阅 FreeBSD ports 邮件列表。重要更改将在那里宣布，然后提交到 CHANGES。

如果这个邮件列表的信息量太大，请考虑关注只包含公告的 FreeBSD ports 公告邮件列表。

## 16.4. FreeBSD Port 构建集群

FreeBSD 最少为之一的优势是专门为持续构建 Ports 集合的整个机群，为每个主要的操作系统发布版本和每个 Tier-1 架构。

除非明确标有 IGNORE ，否则将构建单独的 ports。仍将尝试标记为 BROKEN 的 Ports，以查看是否已解决潜在问题。（这通过向 port 的 Makefile 传递 TRYBROKEN 来完成。）

## 16.5. Portscout: FreeBSD Ports软件源码扫描器

构建集群专门用于构建每个port的最新版本，已经获取了 distfiles。然而，由于互联网不断变化，distfiles 可能很快就会丢失。Portscout，FreeBSD Ports distfile 扫描器，尝试查询每个下载站点上每个port，以查看每个 distfile 是否仍然可用。Portscout 可以生成 HTML 报告，并向请求的人发送有关新可用ports的电子邮件。除非另有要求，维护者被要求定期检查更改，无论是手动还是使用 RSS 提要。

Portscout 的第一页提供了port维护者的电子邮件地址，维护者负责的ports数量，具有新 distfiles 的ports数量，以及那些ports中过时的百分比。搜索功能允许按电子邮件地址搜索特定维护者，并选择是否仅显示过时的ports。

单击维护者的电子邮件地址后，将显示他们的所有 ports 列表，以及 port 类别、当前版本号、是否有新版本、最后更新时间以及最后检查时间。此页面上的搜索功能允许用户搜索特定的 port。

单击列表中的 port 名称会显示 FreshPorts port 信息。

在 Portscout 存储库中提供了额外的文档。
