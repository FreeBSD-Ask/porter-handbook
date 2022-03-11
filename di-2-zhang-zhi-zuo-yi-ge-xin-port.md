# 第2章 制作一个新 port

如果你想制作一个新的port，或者对一些已有的port改进升级，都是一件很棒的事情！

本章文档接下来叙述如何新建一个FreeBSD port。如果想维护一个已有的port并对它升级, 仍然可以参考这一章，然后再阅读[升级port](https://docs.freebsd.org/en/books/porters-handbook/upgrading/index.html#preamble)章节部分。

如果觉得本章文档描述的不够详细，请参阅 **/usr/ports/Mk/bsd.port.mk**，所有port Makefile都会包含该文件。 通过阅读该文件，即便对Makefile不熟悉人也可以从中获益。

>笔记：
>本章文档仅涵盖 **/usr/ports/Mk/bsd.port.mk** 中的一部分变量(VAR)说明，这些变量说明大部分出现在该文件的开头。需要注意的是，该文件没有使用标准的 tab 缩进设置：Emacs 和 Vim 编辑器在加载文件的时候能够识别设置。文件加载后，[vi(1)](https://www.freebsd.org/cgi/man.cgi?query=vi&sektion=1&format=html) 和 [ex(1)](https://www.freebsd.org/cgi/man.cgi?query=ex&sektion=1&format=html) 可以通过输入 **:set tabstop=4** 命令选项来设置正确的tab缩进值。

如果一时间没想好做点什么，可以看看[port需求列表](https://wiki.freebsd.org/WantedPorts)，也许里面有你能帮上忙的工作。
