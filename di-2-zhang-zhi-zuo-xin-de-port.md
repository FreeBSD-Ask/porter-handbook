# 第 2 章 制作新的 port

对制作新port或升级现有ports感兴趣吗？太棒了！

接下来是创建一个适用于 FreeBSD 的新port的几条指南。要升级现有的port，请阅读本文，然后阅读升级Port。

当本文档不够详细时，请参考/usr/ports/Mk/bsd.port.mk，这个文件被所有port Makefile 包含。即使不是每天都在修改 Makefile，也可以从中获取很多知识。此外，具体问题可以发送到 FreeBSD ports邮件列表。

这篇文档中提到的变量只是可以被覆盖的一小部分（ <em>VAR</em> ）。大多数（如果不是所有的话）都在/usr/ports/Mk/bsd.port.mk 的开头进行了文档化；其余的可能也应该如此。请注意，该文件使用非标准的制表符设置：Emacs 和 Vim 在加载文件时将会识别该设置。vi(1)和 ex(1)在加载文件后可以通过键入 :set tabstop=4 来设置使用正确的值。
看看有什么简单的开始吧？看看所请求的ports列表，看看你是否可以处理其中的一个（或多个）。
