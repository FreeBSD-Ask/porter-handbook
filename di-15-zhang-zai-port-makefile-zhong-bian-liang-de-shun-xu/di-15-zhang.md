# 第15章 在 Port Makefile 中变量的顺序

Makefile 的第一个部分必须按相同顺序进行。这个标准使得每个人都可以很容易地阅读任何port，而不必按随机顺序搜索变量。

|  | 这里描述的部分和变量在一个普通的port中是必需的。在一个从属port中，许多部分和变量可以被跳过。 |
| -- | ---------------------------------------------------------------------------------------------- |

|  | 每个后续区块必须与前一个区块用一个空行分隔。<br /><br />在下面的区块中，只设置port所需的变量。按照这里显示的顺序定义这些变量。 |
| -- | ------------------------------------------------------------------------------------------------------------------------ |

## 15.1. PORTNAME 区块

这个区块是最重要的。它定义了port名称、版本、分发文件位置和类别。变量必须按照这个顺序：

* [`PORTNAME`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-portname)
* [`PORTVERSION`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)[[1](https://docs.freebsd.org/en/books/porters-handbook/order/#portversion-footnote)]
* [`DISTVERSIONPREFIX`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)
* [`DISTVERSION`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)[[1](https://docs.freebsd.org/en/books/porters-handbook/order/#portversion-footnote)]
* [`DISTVERSIONSUFFIX`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)
* [`PORTREVISION`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-portrevision)
* [`PORTEPOCH`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-portepoch)
* [`CATEGORIES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-categories)
* [`MASTER_SITES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-master_sites)
* MASTER_SITE_SUBDIR (弃用)
* [`PKGNAMEPREFIX`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#porting-pkgnameprefix-suffix)
* [`PKGNAMESUFFIX`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#porting-pkgnameprefix-suffix)
* [`DISTNAME`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-distname)
* [`EXTRACT_SUFX`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-extract_sufx)
* [`DISTFILES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-distfiles-definition)
* [`DIST_SUBDIR`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-dist_subdir)
* [`EXTRACT_ONLY`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-extract_only)

|  | 只能使用 PORTVERSION 和 DISTVERSION 中的一个。 |
| -- | ------------------------------------------------ |

## 15.2. PATCHFILES 区块

这个区块是可选的。变量有:

* [`PATCH_SITES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#porting-patchfiles)
* [`PATCHFILES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#porting-patchfiles)
* [`PATCH_DIST_STRIP`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#porting-patchfiles)

## 15.3. MAINTAINER 区块

此块是强制性的。变量包括：

* [`MAINTAINER`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-maintainer)
* [`COMMENT`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-comment)
* [`WWW`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-www)

## 15.4. LICENSE 块

此块是可选的，尽管强烈推荐。变量包括：

* [`LICENSE`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license)
* [`LICENSE_COMB`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license_comb)
* LICENSE_GROUPS 或 LICENSE_GROUPS_NAME
* LICENSE_NAME 或 LICENSE_NAME_NAME
* LICENSE_TEXT 或 LICENSE_TEXT_NAME
* LICENSE_FILE 或 LICENSE_FILE_NAME
* LICENSE_PERMS 或 LICENSE_PERMS_NAME_
* LICENSE_DISTFILES 或 LICENSE_DISTFILES_NAME

If there are multiple licenses, sort the different LICENSE_VAR_NAME variables by license name.

## 15.5. Generic `BROKEN`/`IGNORE`/`DEPRECATED` Messages

This block is optional. The variables are:

* [`DEPRECATED`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-deprecated)
* [`EXPIRATION_DATE`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-deprecated)
* [`FORBIDDEN`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-noinstall)
* [`BROKEN`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-noinstall)
* [`BROKEN_*`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-noinstall)
* [`IGNORE`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-noinstall)
* [`IGNORE_*`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-noinstall)
* [`ONLY_FOR_ARCHS`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-noinstall)
* [`ONLY_FOR_ARCHS_REASON*`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-noinstall)
* [`NOT_FOR_ARCHS`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-noinstall)
* [`NOT_FOR_ARCHS_REASON*`](https://docs.freebsd.org/en/books/porters-handbook/porting-dads/#dads-noinstall)

|  | BROKEN_* 和 IGNORE_* 可以是任意通用变量，例如， IGNORE_amd64 ， BROKEN_FreeBSD_10 等。除了依赖于 USES 的变量，将其放在 USES 和 USE_x 中。例如，只有当设置 php 时， IGNORE_WITH_PHP 才有效，只有当设置 ssl 时， BROKEN_SSL 才有效。<br /><br />如果port在满足某些条件时标记为 BROKEN，并且只能在包含 bsd.port.options.mk 或 bsd.port.pre.mk 后才能测试这些条件，则应稍后设置这些变量，即其余变量。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

## 15.6. 依赖项块

这个区块是可选的。变量为：

* [`FETCH_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-fetch_depends)
* [`EXTRACT_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-extract_depends)
* [`PATCH_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-patch_depends)
* [`BUILD_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-build_depends)
* [`LIB_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-lib_depends)
* [`RUN_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-run_depends)
* `TEST_DEPENDS`

## 15.7. Flavors

这个区块是可选的。

从定义 FLAVORS 开始这一部分。继续使用可能的 Flavors 助手。查看更多信息，请参阅 FLAVORS 。

构造设置变量，不能使用 .if ${FLAVOR:U} == foo 作为助手的应放在各自的下面部分。

## 15.8. USES 和 USE_x

从定义 USES 开始，然后可能是 USE_x

将相关变量放在一起。例如，如果使用 USE_GITHUB ，请紧接着放 GH_* 变量。

## 15.9. 标准 bsd.port.mk 变量

此部分块是 bsd.port.mk 中可以定义的变量，这些变量不属于任何先前的部分块。

顺序并不重要，但尽量将类似的变量放在一起。例如，uid 和 gid 变量 USERS 和 GROUPS 。配置变量 CONFIGURE_* 和 *_CONFIGURE 。文件和目录列表 PORTDOCS 和 PORTEXAMPLES 。

## 15.10. 选项和辅助工具

如果port使用选项框架，请先定义 OPTIONS_DEFINE 和 OPTIONS_DEFAULT ，然后先定义其他 OPTIONS_* 变量，然后是 *_DESC 描述，最后是选项助手。尝试按字母顺序对所有这些进行排序。

示例 1。选项变量顺序示例

FOO 和 BAR 选项没有标准描述，因此需要编写一个。其他选项已在 Mk/bsd.options.desc.mk 中有描述，因此不需要编写。 DOCS 和 EXAMPLES 使用目标助手安装它们的文件，尽管它们属于目标，但为了完整起见，这里显示它们，因此其他变量和目标可以在它们之前插入。

```
OPTIONS_DEFINE=	DOCS EXAMPLES FOO BAR
OPTIONS_DEFAULT=	FOO
OPTIONS_RADIO=	SSL
OPTIONS_RADIO_SSL=    OPENSSL GNUTLS
OPTIONS_SUB=	yes

BAR_DESC=		Enable bar support
FOO_DESC=		Enable foo support

BAR_CONFIGURE_WITH=	bar=${LOCALBASE}
FOO_CONFIGURE_ENABLE=	foo
GNUTLS_CONFIGURE_ON=	--with-ssl=gnutls
OPENSSL_CONFIGURE_ON=	--with-ssl=openssl

post-install-DOCS-on:
      ${MKDIR} ${STAGEDIR}${DOCSDIR}
      cd ${WRKSRC}/doc && ${COPYTREE_SHARE} . ${STAGEDIR}${DOCSDIR}

post-install-EXAMPLES-on:
      ${MKDIR} ${STAGEDIR}${EXAMPLESDIR}
      cd ${WRKSRC}/ex && ${COPYTREE_SHARE} . ${STAGEDIR}${EXAMPLESDIR}
```

## 15.11. 其余变量

然后是前面未提及的其余变量。

## 15.12. 目标

在定义所有变量之后，可以定义可选的 make(1) 目标。在不同阶段运行时，保持 pre-<strong></strong> ** 在 post- ** 之前，并按照相同顺序。

* `fetch`
* `extract`
* `patch`
* `configure`
* `build`
* `install`
* `test`

```
post-install:
	# install generic bits

post-install-DOCS-on:
	# Install documentation

post-install-X11-on:
	# Install X11 related bits

post-install-X11-off:
	# Install bits that should be there if X11 is disabled
```
