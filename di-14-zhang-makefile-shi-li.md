# 第14章 一个简单的 port

这里是一个可以用来创建新 Port 的 **Makefile** 示例。确保删除所有多余的注释（即括号中的注释）。

所示的格式是推荐的变量排序方式，部分之间的空行等。这种格式设计的目的是使最重要的信息容易找到。我们推荐使用 [portlint](https://docs.freebsd.org/en/books/porters-handbook/quick-porting/#porting-portlint) 来检查 **Makefile**。


```makefile
[描述 Port 本身和主站的部分 - PORTNAME 和 PORTVERSION 或 DISTVERSION* 变量总是放在第一位，接着是 CATEGORIES，然后是 MASTER_SITES，后面可以跟 MASTER_SITE_SUBDIR。如果需要，PKGNAMEPREFIX 和 PKGNAMESUFFIX 会排在后面。然后是 DISTNAME，EXTRACT_SUFX 和/或 DISTFILES，然后根据需要是 EXTRACT_ONLY。]
PORTNAME=	xdvi
DISTVERSION=	18.2
CATEGORIES=	print
[如果没有使用 MASTER_SITE_* 宏，别忘了尾部的斜杠（"/"）！]
MASTER_SITES=	${MASTER_SITE_XCONTRIB}
MASTER_SITE_SUBDIR=	applications
PKGNAMEPREFIX=	ja-
DISTNAME=	xdvi-pl18
[如果源代码不是标准的 ".tar.gz" 格式，设置此项]
EXTRACT_SUFX=	.tar.Z

[分发的补丁部分 -- 可以为空]
PATCH_SITES=	ftp://ftp.sra.co.jp/pub/X11/japanese/
PATCHFILES=	xdvi-18.patch1.gz xdvi-18.patch2.gz
[如果分发的补丁不是相对于 ${WRKSRC} 制作的，可能需要调整此项]
PATCH_DIST_STRIP=	-p1

[维护者；*必须的*！这是自愿处理端口更新、构建中断和用户提出问题与错误报告的人。为了保持 Ports Collection 的质量，我们不接受新端口被分配给 "ports@FreeBSD.org"。]
MAINTAINER=	asami@FreeBSD.org
COMMENT=	DVI Previewer for the X Window System
WWW=		http://xdvi.sourceforge.net/

[许可证 -- 不应为空]
LICENSE=	BSD2CLAUSE
LICENSE_FILE=	${WRKSRC}/LICENSE

[依赖关系 -- 可以为空]
RUN_DEPENDS=	gs:print/ghostscript

[如果需要 GNU make，而不是 /usr/bin/make，来进行构建...]
USES= gmake
[如果是 X 应用程序，并且需要运行 "xmkmf -a"...]
USES= imake

[此部分用于其他不属于上述任何类别的标准 bsd.port.mk 变量]
[如果在配置、构建、安装过程中会询问问题...]
IS_INTERACTIVE=	yes
[如果解压到与 ${DISTNAME} 不同的目录...]
WRKSRC=		${WRKDIR}/xdvi-new
[如果需要运行由 GNU autoconf 生成的 "configure" 脚本]
GNU_CONFIGURE=	yes
[等等...]

[如果需要选项，此部分用于选项]
OPTIONS_DEFINE=	DOCS EXAMPLES FOO
OPTIONS_DEFAULT=	FOO
[如果选项会更改 plist 中的文件]
OPTIONS_SUB=yes

FOO_DESC=		启用 foo 支持

FOO_CONFIGURE_ENABLE=	foo

[用于规则中的非标准变量]
MY_FAVORITE_RESPONSE=	"yeah, right"

[然后是特殊规则，按调用顺序排列]
pre-fetch:
	i go fetch something, yeah

post-patch:
	i need to do something after patch, great

pre-install:
	and then some more stuff before installing, wow

[最后是尾声]

.include <bsd.port.mk>
```

