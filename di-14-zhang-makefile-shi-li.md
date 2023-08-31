# 第14章 一个简单的 port

这里是一个 **Makefile** 的样本， 可以用来创建一个新的 port。请确保删除所有额外的注释 (括号中的注释)。

所示的格式是推荐的变量排序、章节之间的空行等的格式。这种格式的设计是为了使最重要的信息容易被找到。我们推荐使用 [portlint](https://docs.freebsd.org/en/books/porters-handbook/quick-porting/index.html#porting-portlint) 来检查 **Makefile**。

```shell
[部分来描述 ports 本身和主站点 - PORTNAME
 和 PORTVERSION 或 DISTVERSION* 变量总是放在前面。
 后面是 CATEGORIES， 然后是 MASTER_SITES， 后面可以是
 MASTER_SITE_SUBDIR。 PKGNAMEPREFIX 和 PKGNAMESUFFIX， 如果需要的话。
 将在这之后。 然后是 DISTNAME、 EXTRACT_SUFX 和/或
 DISTFILES， 然后是 EXTRACT_ONLY， 根据需要。]
PORTNAME=	xdvi
DISTVERSION=	18.2
CATEGORIES=	print
[不要忘记尾部的斜线（“/”）！如果没有使用 MASTER_SITE_* 宏，则需要使用 "/"。
 如果不使用 MASTER_SITE_* 宏的话]
MASTER_SITES=	${MASTER_SITE_XCONTRIB}
MASTER_SITE_SUBDIR=	applications
PKGNAMEPREFIX=	ja-
DISTNAME=	xdvi-pl18
[如果源代码不是标准的“.tar.gz”形式，则设置此选项。]
EXTRACT_SUFX=	.tar.Z

[分布式补丁的部分 -- 可以是空的]
PATCH_SITES=	ftp://ftp.sra.co.jp/pub/X11/japanese/
PATCHFILES=	xdvi-18.patch1.gz xdvi-18.patch2.gz
[如果分发的补丁不是相对于 ${WRKSRC} 制作的。
 这可能需要调整一下]
PATCH_DIST_STRIP=	-p1

[维护者；*必须的*!  这个人是自愿的，他负责
 处理 ports 更新、构建故障的人，以及用户可以直接向其提出
 问题和错误报告。 为了保持 Ports 的质量
 尽可能高的质量， 我们不接受那些被分配到
 “ports@FreeBSD.org” 的 Port。]
MAINTAINER=	asami@FreeBSD.org
COMMENT=	DVI Previewer for the X Window System
WWW=		http://xdvi.sourceforge.net/

[许可证 - 不应是空的]
LICENSE=	BSD2CLAUSE
LICENSE_FILE=	${WRKSRC}/LICENSE

[依赖性 -- 可以是空的]
RUN_DEPENDS=	gs:print/ghostscript

[如果它需要 GNU make，而不是 /usr/bin/make，来构建...]
USES= gmake
[如果它是一个 X 应用程序，需要“xmkmf -a”来运行...]
USES= imake

[这一节是为其他标准的 bsd.port.mk 变量准备的，这些变量不属于]
 属于上述任何一种情况]
[如果它在配置、构建、安装时问问题...]
IS_INTERACTIVE=	yes
[如果它解压到 ${DISTNAME}以外的目录...]
WRKSRC=		${WRKDIR}/xdvi-new
[如果它需要运行由 GNU autoconf 生成的“configure”脚本]
GNU_CONFIGURE=	yes
[诸如此类。]

[如果它需要选项，本节是为选项准备的]
OPTIONS_DEFINE=	DOCS EXAMPLES FOO
OPTIONS_DEFAULT=	FOO
[如果选项将改变plist中的文件]
OPTIONS_SUB=yes

FOO_DESC=		Enable foo support

FOO_CONFIGURE_ENABLE=	foo

[在以下规则中使用的非标准变量]
MY_FAVORITE_RESPONSE=	"yeah, right"

[然后是特殊的规则，按照它们被称为的顺序]
pre-fetch:
	i go fetch something, yeah

post-patch:
	i need to do something after patch, great

pre-install:
	and then some more stuff before installing, wow

[然后是尾声]

.include <bsd.port.mk>
```

