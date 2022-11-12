# FreeBSD Port 开发者手冊翻译项目

## 翻译指南

翻译人员应该加入 **QQ 群**  ***512905950***。**注意，此群仅讨论翻译相关事宜，其他事项者将被移出。**

实时翻译预览：[https://porters-handbook.bsdcn.org/](https://porters-handbook.bsdcn.org/)

英文手册地址：[https://docs.freebsd.org/en/books/porters-handbook/book/](https://docs.freebsd.org/en/books/porters-handbook/book/)

基础入门：markdown 语法——https://markdown.com.cn/basic-syntax/ 注意，无需掌握最后三个（图片，转义（所有转义都用\`\`括起来），内嵌 HTLM）。请注意：本项目不支持任何 HTML 语法！请勿使用！所有标题请勿插入 md 语法。

本次项目机器翻译采用 deepl。严禁使用 deepl 以外的翻译网站或程序。附 deepl 网站——https://www.deepl.com/zh/translator#zh/en/ 。**请勿必注意禁止抄袭原有翻译的所有内容，即使看上去和英语原文的意思一模一样。一律使用 deepl，因为原有翻译也存在许多问题。**

所有图片我来操作，你们不要插图，会导致错乱。在需要插图的位置做标记【——————————-此处需要插入图片——————————————-】。并且在提交的时候告知我哪一小节需要插图，请务必主动告知我。

翻译进度预计一天一节。过时视为放弃。

中英文之间前后一律空格。文件和命令以及键盘位，要用\`\`括起来（在数字键1的左边，英文状态下输入）。

第二人称一律使用“你”而不使用“您”。

git/github 相关术语请参考： https://linux.cn/article-12245-1.html

注意 ports 这个单词常常被错误地翻译为“端口”、“港口”，应该改回 ports，请注意不要使用查找替换直接替换，因为有些真的是“端口”。所有“监狱”、“监牢”一律直接替换成“Jail”,“拷贝”->“复制”；“壳、外壳”——>“shell”

ykla 负责整体格式和校对。

会 github 的请直接 pull（我把你加进组，自己提交无需审批），不会的发 md 文件给我即可。拿不准格式可以参考下第三章。github 地址——https://github.com/FreeBSD-Ask/Handbook

Github 提交的用户：大标题和章节（如第 X 章，第 X.X 节）请勿更改，目录我来创建（如果没有请联系我），如需更改请联系我，否则会造成网站错乱。务必注意，不要改动大章节的标题及所有目录结构，否则项目会错乱。请忠实于原文，严禁在正文添加诸如译者备注注释之类的文字或原文没有的段落文字。

一大节应该使用一级标题，如3.4/3.6/3.7.

遇到“tip（技巧）、important（重要）、note（注意）”时，整体使用“>”缩进引用。使用`代码时，请勿添加多余标记，如 bash，如`bash，在本项目中这样是不被认可的。表格请一律居中处理。

## 进度

**翻译进度**

|       章节       |    翻译   |  起始日期|
| :------------: | :-----: | :-: |
|     第1章 简介     | 陈洪翰\_成都 |  /   |
| 第2章 制作一个新 port | 陈洪翰\_成都 |  /   |
|  第3章 简单的 port  |   哈哈哈   | /    |
|    第4章 复杂的 Port            |  HF       |  2022.11.11   |
|    第5章 配置 Makefile            |    哈哈哈     |  2022.11.12   |
|    第6章 特殊情况             |         |     |
|  第7章 Flavors              |         |     |
|  第8章 高级 pkg-plist 实践              |         |     |
|    第9章 pkg-\*            |         |     |
|   第10章 测试 port             |         |     |
|  第11章 升级 port              |         |     |
|    第12章 安全            |         |     |
|  第13章 该做什么和不该做什么              |         |     |
|  第14章 Makefile 示例              |         |     |
|    第15章 在 Port Makefile 中变量的顺序            |      亲爱的翻译官   |     2022.11.12   |
|     第16章 保持同步           |     亲爱的翻译官    |    2022.11.11      |
|   第17章 使用 USES 宏             |     亲爱的翻译官    |     2022.11.12     |
|      第18章 \_\_FreeBSD\_version 的值          |   亲爱的翻译官       |   2022.11.12      |


**校对进度**

|       章节       |  校对  |  其他 |
| :------------: | :--: | :-: |
|     第1章 简介     |   |     |
| 第2章 制作一个新 port |  |     |
|  第3章 简单的 port  |     |     |
|    第4章 复杂的 Port            |         |     |
|    第5章 配置 Makefile            |         |     |
|    第6章 特殊情况             |         |     |
|  第7章 Flavors              |         |     |
|  第8章 高级 pkg-plist 实践              |         |     |
|    第9章 pkg-\*            |         |     |
|   第10章 测试 port             |         |     |
|  第11章 升级 port              |         |     |
|    第12章 安全            |         |     |
|  第13章 该做什么和不该做什么              |         |     |
|  第14章 Makefile 示例              |         |     |
|    第15章 在 Port Makefile 中变量的顺序            |         |     |
|     第16章 保持同步           |         |     |
|   第17章 使用 USES 宏             |         |     |
|      第18章 \_\_FreeBSD\_version 的值          |         |     |



**网站部署与维护**

 - Shengyun
 - ykla

## Q & A

* ~~Q：对 FreeBSD 文档进行简体中文翻译的必要性？~~
  * ~~A：~~

~~俗话说的好，要想富先修路。要想推广和宣传 FreeBSD，也必须先翻译 FreeBSD 文档。~~

~~有些人认为有英语不需要翻译，看不懂的人就不配学习 FreeBSD。此言差矣。FreeBSD 也并非专门为了某些精英而创设，不懂英语是一件很正常的事情，不同的语言有不同的世界观，只有经过翻译才能让更多普通人走进 FreeBSD，发展 FreeBSD，结缘 FreeBSD。你懂英语，别人不一定懂，不能用你的标准去衡量所有人。FreeBSD 社区以及基金会从未说过不懂英语就无法使用 FreeBSD 这种话。~~

~~还有些人认为与其翻译 Handbook 不如我们自己去写原创性的文章，其实翻译 handbook，恰恰是为了更好地撰写原创性的文章，我们的另一个项目——~~[~~《FreeBSD 从入门到跑路》~~](https://github.com/FreeBSD-Ask/FreeBSD-Ask)~~目标就是包括所有 Handbook 有与无的内容。我们认为，翻译文档和贡献代码是同样重要的事情；如果你有原创教程，请参与提交。~~最重要的是，我们的进程已经完成等待校对。现在还在纠结这个问题这无异于在中国哲学史课程快要上完的时候还在讨论中国到底有没有哲学。~~

* Q：本项目为什么不能向 FreeBSD 上游进行合并？
  * A：

~~很简单，我们无法向上游贡献。~~[~~https://wiki.freebsd.org/Doc/Translation~~](https://wiki.freebsd.org/Doc/Translation) ~~中的指南无法具体落实，其翻译进度一直是（99%）,实际上是 0。我们多次与 FreeBSD 简体中文翻译负责人（该负责人从 18 年就进行 FreeBSD handbook 的翻译，但始终未能进行）进行联络，始终无法得到及时回应。但本着 FreeBSD 是一个开源的项目，人人都可以 fork 的态度。并不会影响我们的工作进度。也不会导致本项目产生任何问题。我们认为，踏出第一步永远比空谈任何东西都更为重要！~~

~~现阶段需要先完成校对并且我们现在需要人手来进行校对并向上游进行合并。要求能够对照英文找到中文的翻译。直接翻译（照抄）ykla 给定的 po 文件。另外需要一名或多名 committer 进行配合。目前 FreeBSD 上游的 doc 文档工具链存在 bug，编译出来的中文字体会乱码（疑似没有加载中文字体，只有日文字体），也许要等其修复。~~

我也非常疑惑。

* Q：怎样维护？
  * A：

翻译完毕后约 1 个月整体维护一次。对比官方 Handbook 进行更新。


## 关于

### 简介

来到这里你可能什么也得不到。也将不会失去任何东西。FreeBSD 中文用户社区！恪守古老的法则，追寻真正的自由。BSD 方为真正 UNIX 哲学继承者。加入我们，共同推进 FreeBSD 中国化与世界化。

|       资源      |                            链接                            |
| :-----------: | :------------------------------------------------------: |
|   Telegram 群  |     [https://t.me/freebsdba](https://t.me/freebsdba)     |
|      QQ 群     |                         787969044                        |
| Handbook 最新翻译 | [https://handbook.bsdcn.org](https://handbook.bsdcn.org) |
|  FreeBSD 入门书籍 |     [https://book.bsdcn.org](https://book.bsdcn.org)     |
|  FreeBSD 中文论坛 |      [https://bbs.bsdcn.org](https://bbs.bsdcn.org)      |
|     微信公众号     |                         freebsdzh                        |

扫码关注微信公众号：

![扫码关注微信公众号](./.gitbook/assets/weixin.jpg)

### FreeBSD 中文社区 寄言：

> 每一个人的不经意付出和教程完善的分享，都是极具意义的，理论上会长久存在，你做的每一点一滴的努力与付出都会成为历史的印记和未来某些科技的驱动力的先驱。手册是死的，我们对手册的贡献是一直存活的，存活在那些使用 Handbook 的人的心中。
>
> **“The best time to plant a tree is 20 years ago. The second-best time is now.”（种一棵树最好的时间是十年前，其次是现在）——Dambisa Moyo,** _**dead aid**_
>
> 我希望有更多的人参与进来，一起协作这个开源项目。英语水平和计算机水平固然重要，但是翻译凭借的不是傲慢与偏见，不是苦难哲学，也不是泥潭般的社区，更不是所谓的巨苣，而是我们的心，翻译是用心来完成的！用心，每个人都可以参与，无论是错别字的修正还是大章节的翻译，都是有意义的。


### FreeBSD 中文社区的愿景

我们成立于 2018年3月17日，由贴吧——FreeBSD 吧发展到了 QQ 群（主群 787969044），Telegram 群，乃至于微信群。

我们的成员具有非常大的广泛性和普遍性，能够代表绝大多数 FreeBSD 用户的平均水平：可能根本没有听说过何为 FreeBSD，但这并不影响我们的交流与沟通。也许有人觉得这是浪费时间，但是没有新生力量的培养，何来 FreeBSD 的明天呢？谁不知道新人可能有很多坏习惯呢。

同鲁迅先生说的那样，但愿每个人都是一束光，照亮 FreeBSD 在中国大陆地区前进的光荣的荆棘路。也希望，你可以加入我们，共同组成漫天星光亦或者是星星之火。

无穷的远方，无数的人们，都和我有关。

我曾无数次眺望远山，想要找到一汪清泉，天总是不随人愿，还是没有找到。

我是谁，我们是谁？这个问题永远也不会有结果。

我们选择 FreeBSD，是因为想选择一个清晰、明了、可靠、稳固的一个操作系统在工作上给我们带来收益以及在生活中给我们带来乐趣。当然 FreeBSD 还存在很多问题，有待大家积极发现、探讨、完善，社会在进步，技术在进步，热情丝毫不减在持续，未来越来越美好。
