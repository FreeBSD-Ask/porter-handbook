# 第 15 章 在 Port Makefile 中变量的顺序

**Makefile** 的开头各段必须始终按固定顺序排列。这个标准让任何人都能轻松阅读任意一个 Port，而无需在混乱顺序中查找变量。

>**注意**
>
>此处所述的各段和变量在一个普通 Port 中是强制性的。在一个从属 Port 中，许多段和变量可以省略。 

>**重要**
>
>每个以下代码块之间必须以一个空行分隔。
>
>在每个代码块中，仅设置该 Port 所必需的变量。请按照下列顺序定义这些变量。 

## 15.1. `PORTNAME` 段

这是最重要的一段。它定义了 Port 的名称、版本、源代码分发文件的位置和分类。变量必须按如下顺序排列：

* [`PORTNAME`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-portname)
  \* [`PORTVERSION`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)\[[1](https://docs.freebsd.org/en/books/porters-handbook/order/#portversion-footnote)]
* [`DISTVERSIONPREFIX`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)
  \* [`DISTVERSION`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)\[[1](https://docs.freebsd.org/en/books/porters-handbook/order/#portversion-footnote)]
* [`DISTVERSIONSUFFIX`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)
* [`PORTREVISION`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-portrevision)
* [`PORTEPOCH`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-portepoch)
* [`CATEGORIES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-categories)
* [`MASTER_SITES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-master_sites)
* [`MASTER_SITE_SUBDIR`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-master_sites-shorthand)（已弃用）
* [`PKGNAMEPREFIX`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#porting-pkgnameprefix-suffix)
* [`PKGNAMESUFFIX`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#porting-pkgnameprefix-suffix)
* [`DISTNAME`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-distname)
* [`EXTRACT_SUFX`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-extract_sufx)
* [`DISTFILES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-distfiles-definition)
* [`DIST_SUBDIR`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-dist_subdir)
* [`EXTRACT_ONLY`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-extract_only)

>**重要**
>
>`PORTVERSION` 和 `DISTVERSION` 只能择一而用。 


## 15.2. `PATCHFILES` 段

此段为可选项。变量如下：

* [`PATCH_SITES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#porting-patchfiles)
* [`PATCHFILES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#porting-patchfiles)
* [`PATCH_DIST_STRIP`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#porting-patchfiles)

## 15.3. `MAINTAINER` 段

此段为强制项。变量如下：

* [`MAINTAINER`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-maintainer)
* [`COMMENT`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-comment)
* [`WWW`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-www)

## 15.4. `LICENSE` 段

这一段是可选的，但强烈建议使用。变量包括：

* [`LICENSE`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license)
* [`LICENSE_COMB`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license_comb)
* [`LICENSE_GROUPS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license_groups) 或 `LICENSE_GROUPS_NAME`
* [`LICENSE_NAME`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license_name) 或 `LICENSE_NAME_NAME`
* [`LICENSE_TEXT`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license_text) 或 `LICENSE_TEXT_NAME`
* [`LICENSE_FILE`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license_file) 或 `LICENSE_FILE_NAME`
* [`LICENSE_PERMS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license_perms) 或 `LICENSE_PERMS_NAME_`
* [`LICENSE_DISTFILES`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license_distfiles) 或 `LICENSE_DISTFILES_NAME`

如果存在多个许可证，需按许可证名称对各个 `LICENSE_VAR_NAME` 变量进行排序。

## 15.5. 通用 `BROKEN` / `IGNORE` / `DEPRECATED` 信息段

这一段是可选的。变量包括：

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

>**重要**
>
> `BROKEN_*` 和 `IGNORE_*` 可以是任意通用变量，例如 `IGNORE_amd64`、`BROKEN_FreeBSD_10` 等。除了依赖于 [`USES`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses) 的变量以外，后者应放在 [`USES` 和 `USE_x`](https://docs.freebsd.org/en/books/porters-handbook/order/#porting-order-uses) 中。例如，`IGNORE_WITH_PHP` 仅在设置了 [`php`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-php) 时才生效，`BROKEN_SSL` 仅在设置了 [`ssl`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-ssl) 时才生效。
>
>如果 Port 仅在满足某些条件时才应标记为 `BROKEN`，并且这些条件只能在包含 **bsd.port.options.mk** 或 **bsd.port.pre.mk** 之后才能判断，那么应在 [其余变量段](https://docs.freebsd.org/en/books/porters-handbook/order/#porting-order-rest) 中设置这些变量。 

## 15.6. 依赖关系段

这一段是可选的。变量包括：

* [`FETCH_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-fetch_depends)
* [`EXTRACT_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-extract_depends)
* [`PATCH_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-patch_depends)
* [`BUILD_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-build_depends)
* [`LIB_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-lib_depends)
* [`RUN_DEPENDS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-run_depends)
* `TEST_DEPENDS`

## 15.7. Flavors

这一段是可选的。

以定义 `FLAVORS` 开始此段，然后继续定义可用的 Flavors 助手。请参阅 [使用 FLAVORS](https://docs.freebsd.org/en/books/porters-handbook/flavors/#flavors-using) 了解更多信息。

对于不能通过助手设置的变量，应使用 `.if ${FLAVOR:U} == foo` 结构，并将其置于各自所属的段落中。

## 15.8. `USES` 和 `USE_x`

以定义 `USES` 开始本段，然后继续定义可能的 `USE_x`。

请将相关变量紧密排列。例如使用 [`USE_GITHUB`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-master_sites-github) 时，务必将所有 `GH_*` 变量紧跟其后。

## 15.9. 标准 bsd.port.mk 变量

这一段用于定义可以在 **bsd.port.mk** 中使用、但不属于前述各段的变量。

虽然变量的顺序并不重要，但请尽量将相似的变量归类在一起。例如，用户和用户组变量 `USERS` 和 `GROUPS`；配置变量 `CONFIGURE_*` 与 `*_CONFIGURE`；文件与目录列表变量如 `PORTDOCS` 与 `PORTEXAMPLES`。

## 15.10. Options 与 Helpers

如果该 Port 使用了 [选项框架](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-options)，请先定义 `OPTIONS_DEFINE` 和 `OPTIONS_DEFAULT`，然后是其它 `OPTIONS_*` 变量，再之后是 `*_DESC` 描述，最后是选项 helpers。尽量将上述所有内容按字母顺序排列。

**示例 1：Options 变量排序示例**

`FOO` 和 `BAR` 选项没有标准描述，因此需要编写。其他选项已经在 **Mk/bsd.options.desc.mk** 中定义好了描述，无需重复编写。`DOCS` 和 `EXAMPLES` 使用目标 helpers 来安装它们的文件，在此处也给出以示完整，尽管它们应归属 [目标段](https://docs.freebsd.org/en/books/porters-handbook/order/#porting-order-targets)，因此其前可插入其他变量和目标。

```makefile
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

然后定义所有未在上述各段中提及的其它变量。

## 15.12. 目标

在所有变量定义之后，可以定义可选的 [make(1)](https://man.freebsd.org/cgi/man.cgi?query=make&sektion=1&format=html) 目标。请确保 `pre-<strong></strong>`\*\* 排在 `post-`\*\* 之前，并按照不同阶段的执行顺序排列：

* `fetch`
* `extract`
* `patch`
* `configure`
* `build`
* `install`
* `test`

>**技巧**
>
>使用选项 helpers 时，保持它们按字母顺序排序，但确保 `-on` 排在 `-off` 之前。若同时使用主目标，则主目标应排在可选目标之前：
>
>```markfile
>post-install:
>	# 安装通用部分
>
>post-install-DOCS-on:
>	# 安装文档
>
>post-install-X11-on:
>	# 安装与 X11 相关的部分
>
>post-install-X11-off:
>	# 安装在 X11 禁用时应存在的部分
>```


