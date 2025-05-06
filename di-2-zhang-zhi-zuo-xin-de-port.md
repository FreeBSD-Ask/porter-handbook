# 第 2 章 制作新的 port

想要创建一个新的 Port，或升级现有的 Port？太好了！

接下来是一些为 FreeBSD 创建新 Port 的指导方针。要升级现有的 Port，请阅读本文，然后阅读[升级 Port](https://docs.freebsd.org/en/books/porters-handbook/upgrading/#port-upgrading)。

当本文档内容不够详细时，请参考 **/usr/ports/Mk/bsd.port.mk**，该文件包含在所有 Port 的 **Makefile** 中。即使不是每天修改 **Makefile** 的人，也能从中获得很多有用的知识。此外，具体的问题可以发送到 [FreeBSD ports 邮件列表](https://lists.freebsd.org/subscription/freebsd-ports)。

>**注意**
>
>本文仅提到了一小部分可以被覆盖的变量（`VAR`）。大部分（如果不是全部的话）已在 **/usr/ports/Mk/bsd.port.mk** 的开头文档中说明；其他的可能也应该被说明。请注意，该文件使用了非标准的制表符设置：Emacs 和 Vim 会在加载该文件时识别该设置。加载文件后，使用 [vi(1)](https://man.freebsd.org/cgi/man.cgi?query=vi&sektion=1&format=html) 和 [ex(1)](https://man.freebsd.org/cgi/man.cgi?query=ex&sektion=1&format=html) 可以通过输入 `:set tabstop=4` 来设置为正确的值。


想找一些简单的入门项目？看看 [请求的 Port 列表](https://wiki.freebsd.org/WantedPorts)，看看你是否能着手做其中一个（或多个）。
