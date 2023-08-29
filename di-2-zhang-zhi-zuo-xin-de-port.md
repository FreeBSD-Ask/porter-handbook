# 第 2 章 制作新的 port

有兴趣制作新的 port，或升级现有的 port？这很好!

下面是为 FreeBSD 创建新 port 的一些指南。要升级现有的 port，请先阅读本章，然后再阅读[升级 port](https://docs.freebsd.org/en/books/porters-handbook/upgrading/index.html#preamble)。

这份文档有时不够详细，请参考 **/usr/ports/Mk/bsd.port.mk**，所有 port 的 **Makefile** 都包含他。即使那些不是每天都在钻研 **Makefile** 的人也能从其中获得很多知识。此外，具体的问题可以发送到 [FreeBSD ports 邮件列表](https://lists.freebsd.org/subscription/freebsd-ports)。

> **注意**
>
> 本文档仅涵盖了 **/usr/ports/Mk/bsd.port.mk** 中的一部分变量（_VAR_)）的说明，这些变量说明大部分出现在该文件的开头。需要注意的是，该文件没有使用标准的 tab 缩进设置：Emacs 和 Vim 编辑器在加载文件的时候能够进行识别设置。文件加载后，[vi(1)](https://www.freebsd.org/cgi/man.cgi?query=vi&sektion=1&format=html) 和 [ex(1)](https://www.freebsd.org/cgi/man.cgi?query=ex&sektion=1&format=html) 可以通过输入 `:set tabstop=4` 命令选项来设置正确的 tab 缩进值。

想找一些容易上手的东西吗？看看[请求的 port 列表](https://wiki.freebsd.org/WantedPorts)，看看你是否可以在其中一个（或多个）上帮忙。
