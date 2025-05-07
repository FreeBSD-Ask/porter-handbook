# 第6章 特殊情况

## 6.1. 拆分长文件

有时，Port 的 **Makefile** 可能会非常长。例如，Rust  Port 的 `CARGO_CRATES` 列表可能很长，或者 **Makefile** 中可能包含根据架构不同而变化的代码。在这种情况下，将原始的 **Makefile** 拆分成多个文件可能会很方便。**bsd.port.mk** 会自动包含一些类型的 **Makefile** 到主 Port **Makefile** 中。

框架会自动处理以下文件类型（如果发现它们的话）：

* **Makefile.crates**。可以在 [audio/ebur128](https://cgit.freebsd.org/ports/tree/audio/ebur128/) 中找到示例。
* **Makefile.inc**。可以在 [net/ntp](https://cgit.freebsd.org/ports/tree/net/ntp/) 中找到示例。
* **Makefile.\${ARCH}-\${OPSYS}**
* **Makefile.\${OPSYS}**。可以在 [net/cvsup-static](https://cgit.freebsd.org/ports/tree/net/cvsup-static/) 中找到示例。
* **Makefile.\${ARCH}**
* **Makefile.local**

通常的做法是，如果包装列表在不同架构之间变化很大，或者依赖于所选的 flavor，那么将 Port 的包装列表拆分成多个文件。在这种情况下，每个架构的 **pkg-plist** 文件命名遵循 **pkg-plist.\${ARCH}** 或 **pkg-plist.\${FLAVOR}** 的模式。如果存在多个 **pkg-plist** 文件，框架不会自动创建包装列表。Port 作者需要选择适当的 **pkg-plist** 文件并将其分配给 `PLIST` 变量。关于如何处理这个问题的示例可以在 [audio/logitechmediaserver](https://cgit.freebsd.org/ports/tree/audio/logitechmediaserver/) 和 [deskutils/libportal](https://cgit.freebsd.org/ports/tree/deskutils/libportal/) 中找到。

## 6.2. Staging

**bsd.port.mk** 要求 Port 使用 "阶段目录"。这意味着 Port 必须将文件安装到一个单独的目录，而不是直接安装到常规的目标目录（例如，`PREFIX`）。然后从这个阶段目录构建包，并将包安装到系统中。在许多情况下，这不需要 root 权限，从而使得可以作为非特权用户构建包。通过 staging，Port 被构建并安装到阶段目录 `STAGEDIR` 中。然后从该目录创建包，并安装到系统中。自动化工具称这一概念为 `DESTDIR`，但在 FreeBSD 中，`DESTDIR` 有不同的含义（见 [`PREFIX` 和 `DESTDIR`](https://docs.freebsd.org/en/books/porters-handbook/testing/#porting-prefix)）。

>**重要**
>
>  没有任何 Port *真正* 需要 root 权限。通过使用 [`USES=uidfix`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-uidfix) 大多数情况下可以避免。如果 Port 仍然运行像 [chown(8)](https://man.freebsd.org/cgi/man.cgi?query=chown&sektion=8&format=html)、[chgrp(1)](https://man.freebsd.org/cgi/man.cgi?query=chgrp&sektion=1&format=html)，或者通过 [install(1)](https://man.freebsd.org/cgi/man.cgi?query=install&sektion=1&format=html) 强制设置文件的所有者或组，则应使用 [`USES=fakeroot`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-fakeroot) 来伪造这些调用。此时可能需要对 Port 的 **Makefiles** 进行一些补丁。 


元 Port（即不安装文件，而是依赖于其他 Port 的 Port）必须避免不必要地将 [mtree(8)](https://man.freebsd.org/cgi/man.cgi?query=mtree&sektion=8&format=html) 提取到阶段目录。因为这是包的基本目录布局，这些空目录会被视为孤立的。为了防止 [mtree(8)](https://man.freebsd.org/cgi/man.cgi?query=mtree&sektion=8&format=html) 提取，可以添加这一行：

```
NO_MTREE=	yes
```

>**技巧**
>
>  元 Port 应该使用 [`USES=metaport`](https://docs.freebsd.org/en/books/porters-handbook/special/#uses-metaport)。它为不获取、构建或安装任何东西的 Port 设置默认值。


Staging 通过在 `pre-install`、`do-install` 和 `post-install` 目标中使用 `STAGEDIR` 前缀来启用（参见本书中的示例）。通常，这包括 `PREFIX`、`ETCDIR`、`DATADIR`、`EXAMPLESDIR`、`DOCSDIR` 等。目录应该作为 `post-install` 目标的一部分创建。尽量避免使用绝对路径。

>**技巧**
>
> 安装内核模块的 Port 必须默认在其目标路径前添加 `STAGEDIR`，例如 **/boot/modules**。

### 6.2.1. 处理符号链接

在创建符号链接时，强烈建议使用相对链接。使用 `${RLN}` 可以自动创建相对符号链接。它在后台使用 [install(1)](https://man.freebsd.org/cgi/man.cgi?query=install&sektion=1&format=html)，自动计算要创建的相对链接。

**示例 1. 自动创建相对符号链接**

`${RLN}` 使用 [install(1)](https://man.freebsd.org/cgi/man.cgi?query=install&sektion=1&format=html) 的相对符号链接功能，使得创建者不必手动计算相对路径。

```sh
${RLN} ${STAGEDIR}${PREFIX}/lib/libfoo.so.42 ${STAGEDIR}${PREFIX}/lib/libfoo.so
${RLN} ${STAGEDIR}${PREFIX}/libexec/foo/bar ${STAGEDIR}${PREFIX}/bin/bar
${RLN} ${STAGEDIR}/var/cache/foo ${STAGEDIR}${PREFIX}/share/foo
```

将生成：

```sh
% ls -lF ${STAGEDIR}${PREFIX}/lib
lrwxr-xr-x  1 nobody  nobody    181 Aug  3 11:27 libfoo.so@ -> libfoo.so.42
-rwxr-xr-x  1 nobody  nobody     15 Aug  3 11:24 libfoo.so.42*
% ls -lF ${STAGEDIR}${PREFIX}/bin
lrwxr-xr-x  1 nobody  nobody    181 Aug  3 11:27 bar@ -> ../libexec/foo/bar
% ls -lF ${STAGEDIRDIR}${PREFIX}/share
lrwxr-xr-x  1 nobody  nobody    181 Aug  3 11:27 foo@ -> ../../../var/cache/foo
```

## 6.3. 捆绑的库

本节解释了为什么捆绑的依赖库被认为是不好的，以及如何处理它们。

### 6.3.1. 为什么捆绑的库不好

一些软件要求开发者找到第三方库并将所需的依赖添加到 Port 中。其他软件则将所有必需的库打包在分发文件中。第二种方法看起来一开始更容易，但存在一些严重的缺点：

以下列表 loosely 基于 [Fedora](https://fedoraproject.org/wiki/Packaging:No_Bundled_Libraries) 和 [Gentoo](https://wiki.gentoo.org/wiki/Why_not_bundle_dependencies) 的维基页面，均由 [CC-BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) 许可证授权。

**安全性**

如果在上游库中发现漏洞并进行修复，可能不会在与 Port 捆绑的库中修复这些漏洞。一个原因可能是作者没有意识到这个问题。这意味着开发者必须修复它们，或升级到一个无漏洞的版本，并向作者提交补丁。这一过程需要时间，导致软件在必要时被暴露在漏洞风险中。这也使得协调修复变得更加困难，且可能会泄漏漏洞的细节。

**错误**

这个问题类似于上一段提到的安全问题，但一般不如安全问题严重。

**分叉**

库被捆绑后，作者更容易对上游库进行分叉。虽然从表面上看，这样做更方便，但这意味着代码与上游分叉，使得解决安全问题或其他问题变得更加困难。原因之一是补丁变得更难以应用。

分叉的另一个问题是，由于代码与上游分叉，bug 需要重复解决，而不是集中在一个地方解决。这违背了开源软件的初衷。

**符号冲突**

当系统中已安装了某个库时，它可能会与捆绑版本发生冲突。这可能会导致编译或链接时立即出错，也可能导致运行时错误，后者可能更难追踪。后一种问题可能是因为两个库的版本不兼容。

**许可问题**

当捆绑来自不同来源的项目时，许可问题可能更容易出现，尤其是当许可不兼容时。

**资源浪费**

捆绑的库在多个层面上浪费资源。特别是当这些库已经存在于系统上时，构建实际应用的时间会变得更长。在运行时，它们可能占用不必要的内存，而系统全局库已经被其他程序加载，而捆绑的库却由另一个程序加载。

**努力的浪费**

当一个库需要为 FreeBSD 打补丁时，这些补丁必须再次应用到捆绑库中。这浪费了开发者的时间，因为这些补丁可能无法干净地应用。并且很难发现这些补丁其实是必需的。

### 6.3.2. 如何处理捆绑的库

尽可能使用未捆绑版本的库，通过向 Port 添加 `LIB_DEPENDS` 来指定依赖。如果这种 Port 还不存在，考虑创建一个新的 Port 。

仅在上游有良好的安全记录并且使用未捆绑版本会导致过于复杂的补丁时，才使用捆绑库。

>**重要**
>
> 在一些非常特殊的情况下，例如仿真器（如 Wine）， Port 必须捆绑库，因为这些库属于不同架构，或者它们已被修改以适应软件的需求。在这种情况下，这些库不应该暴露给其他 Port 进行链接。可以在 Port 的 **Makefile** 中添加 `BUNDLE_LIBS=yes`。这将告诉 [pkg(8)](https://man.freebsd.org/cgi/man.cgi?query=pkg&sektion=8&format=html) 不计算提供的库。在将此添加到 Port 之前，请始终向 Ports 管理团队 <[portmgr@FreeBSD.org](mailto:portmgr@FreeBSD.org)> 询问。 


## 6.4. 共享库

如果 Port 安装一个或多个共享库，定义 `USE_LDCONFIG` make 变量，指示 **bsd.port.mk** 在 `post-install` 目标阶段运行 `${LDCONFIG} -m`，该命令将在新库安装的目录（通常是 **PREFIX/lib**）上运行，以将其注册到共享库缓存中。定义此变量时，它还会在 **pkg-plist** 中添加适当的 `@exec /sbin/ldconfig -m` 和 `@unexec /sbin/ldconfig -R`，以便用户安装软件包后能立即使用共享库，并且卸载时不会让系统仍然认为库存在。

```sh
USE_LDCONFIG=	yes
```

可以通过将 `USE_LDCONFIG` 设置为共享库安装目录的列表，来覆盖默认目录。例如，如果 Port 将共享库安装到 **PREFIX/lib/foo** 和 **PREFIX/lib/bar**，则在 **Makefile** 中使用以下设置：

```sh
USE_LDCONFIG=	${PREFIX}/lib/foo ${PREFIX}/lib/bar
```

请仔细检查，通常这并不总是必要的，或者可以通过 `-rpath` 或在链接时设置 `LD_RUN_PATH` 来避免（参见 [lang/mosml](https://cgit.freebsd.org/ports/tree/lang/mosml/) 示例），或者通过像 [www/seamonkey](https://cgit.freebsd.org/ports/tree/www/seamonkey/) 那样的外壳脚本包装器，先设置 `LD_LIBRARY_PATH` 后再调用二进制文件。

在 64 位系统上安装 32 位库时，使用 `USE_LDCONFIG32`。

如果软件使用 [autotools](https://docs.freebsd.org/en/books/porters-handbook/special/#using-autotools)，特别是 `libtool`，则添加 [`USES=libtool`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-libtool)。

当库的主版本号在新版本 Port 更新时增加时，所有链接到受影响库的其他 Port 必须增加 `PORTREVISION`，以强制重新编译该库的新版本。

## 6.5. 有分发限制或法律问题的 Ports

许可证各不相同，其中一些对应用程序如何打包、是否可以盈利性销售等方面有一定限制。

>**重要**
>
> Port 维护者有责任阅读软件的许可条款，并确保 FreeBSD 项目不会因通过 FTP/HTTP 或 CD-ROM 再分发源代码或已编译的二进制文件而违反相关条款。如果有疑问，请联系 [FreeBSD ports 邮件列表](https://lists.freebsd.org/subscription/freebsd-ports)。


在这种情况下，可以设置下节中说明的变量。

### 6.5.1. `NO_PACKAGE`

此变量表示我们不能生成应用程序的二进制包。例如，许可协议可能禁止二进制再分发，或者可能禁止从修补过的源代码创建的包的分发。

然而， Port 的 `DISTFILES` 可以自由地在 FTP/HTTP 上镜像。它们也可以分发到 CD-ROM（或类似媒体），除非同时设置了 `NO_CDROM`。

如果二进制包没有普遍的用途，并且必须始终从源代码编译应用程序，请使用 `NO_PACKAGE`。例如，如果应用程序在编译时硬编码了与站点特定的配置信息，请设置 `NO_PACKAGE`。

设置 `NO_PACKAGE` 为说明为什么不能生成包的字符串。

### 6.5.2. `NO_CDROM`

仅此变量表示，尽管我们允许生成二进制包，但我们不能将这些包或 Port 的 `DISTFILES` 放到 CD-ROM（或类似媒体）上进行转售。然而，二进制包和 Port 的 `DISTFILES` 仍然可以通过 FTP/HTTP 进行分发。

如果同时设置了 `NO_PACKAGE` 和 `NO_CDROM`，则只有 Port 的 `DISTFILES` 可用，并且仅通过 FTP/HTTP 分发。

设置 `NO_CDROM` 为说明为什么不能在 CD-ROM 上重新分发该 Port 的字符串。例如，如果该 Port 的许可证仅允许“非商业”使用，可以使用此设置。

### 6.5.3. `NOFETCHFILES`

在 `NOFETCHFILES` 中定义的文件不能从任何 `MASTER_SITES` 获取。例如，当文件由供应商提供在 CD-ROM 上时，就是一个这样的文件。

检查这些文件是否在 `MASTER_SITES` 上可用的工具需要忽略这些文件，并且不报告它们的可用性。

### 6.5.4. `RESTRICTED`

如果应用程序的许可既不允许镜像应用程序的 `DISTFILES` 也不允许以任何方式分发二进制包，请单独设置此变量。

不要与 `RESTRICTED` 一起设置 `NO_CDROM` 或 `NO_PACKAGE`，因为后者变量已经包含了前者的含义。

将 `RESTRICTED` 设置为说明为何该 Port 不能再分发的字符串。通常，这表示该 Port 包含专有软件，用户需要手动下载 `DISTFILES`，可能需要在注册软件或同意接受最终用户许可协议（EULA）的条款后进行下载。

### 6.5.5. `RESTRICTED_FILES`

当设置 `RESTRICTED` 或 `NO_CDROM` 时，此变量默认值为 `${DISTFILES} ${PATCHFILES}`，否则为空。如果只有部分分发文件受到限制，可以设置此变量列出它们。

### 6.5.6. `LEGAL_TEXT`

如果 Port 有上述变量未涵盖的法律问题，请将 `LEGAL_TEXT` 设置为解释该问题的字符串。例如，如果 FreeBSD 获得了特别许可来重新分发二进制文件，此变量必须注明这一点。

### 6.5.7. **/usr/ports/LEGAL** 和 `LEGAL`

设置上述任何变量的 Port 必须同时添加到 **/usr/ports/LEGAL**。第一列是匹配受限 `distfiles` 的通配符，第二列是 Port 的来源，第三列是 `make -VLEGAL` 的输出。

### 6.5.8. 示例

声明“此 Port 的 distfiles 必须手动获取”的首选方式如下：

```sh
.if !exists(${DISTDIR}/${DISTNAME}${EXTRACT_SUFX})
IGNORE=	may not be redistributed because of licensing reasons. Please visit some-website to accept their license and download ${DISTFILES} into ${DISTDIR}
.endif
```

这不仅通知用户，还在用户的机器上设置了适当的元数据，供自动化程序使用。

请注意，此段落必须在包含 **bsd.port.pre.mk** 后添加。

## 6.6. 构建机制

### 6.6.1. 并行构建 Port 

FreeBSD 的 Ports 框架支持通过使用多个 `make` 子进程来进行并行构建，这使得 SMP 系统可以利用所有可用的 CPU 功率，从而加快 Port 构建的速度和效果。

这是通过将 `-jX` 标志传递给在供应商代码上运行的 [make(1)](https://man.freebsd.org/cgi/man.cgi?query=make&sektion=1&format=html) 来实现的。这是 Port 的默认构建行为。不幸的是，并非所有 Port 都能很好地处理并行构建，因此可能需要显式禁用此功能，方法是添加 `MAKE_JOBS_UNSAFE=yes` 变量。当一个 Port 因竞争条件而导致间歇性构建失败时，就需要使用这个变量。

>**重要**
>
>设置 `MAKE_JOBS_UNSAFE` 时，必须非常重要地在 **Makefile** 中进行说明，或者至少在提交信息中说明 *为什么* 启用时 Port 无法构建。否则，在以后提交更新时，几乎不可能修复问题或测试问题是否已经解决。


### 6.6.2. `make`、`gmake` 和 `imake`

存在几种不同的 `make` 实现。移植的软件通常需要特定的实现，例如 GNU `make`，在 FreeBSD 中称为 `gmake`。

如果 Port 使用 GNU make，请将 `gmake` 添加到 `USES` 中。

`MAKE_CMD` 可以用来引用 Port  **Makefile** 中通过 `USES` 设置配置的特定命令。仅在应用程序 **Makefile** 中的 `WRKSRC` 内部使用 `MAKE_CMD` 来调用 Port 期望的软件的 `make` 实现。

如果 Port 是使用 imake 来从 **Imakefile** 生成 **Makefile** 的 X 应用程序，则应设置 `USES= imake`。有关更多详细信息，请参阅 [使用 `USES` 宏](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses)。

如果 Port 的源 **Makefile** 具有不同于 `all` 的主要构建目标，请相应地设置 `ALL_TARGET`。同样，`install` 和 `INSTALL_TARGET` 也应该如此。

### 6.6.3. `configure` 脚本

如果 Port 使用 `configure` 脚本从 **Makefile.in** 生成 **Makefile**，请设置 `GNU_CONFIGURE=yes`。要为 `configure` 脚本提供额外的参数（默认参数为 `--prefix=${PREFIX} --infodir=${PREFIX}/${INFO_PATH} --mandir=${PREFIX}/man --build=${CONFIGURE_TARGET}`），请在 `CONFIGURE_ARGS` 中设置这些额外的参数。可以使用 `CONFIGURE_ENV` 传递额外的环境变量。

| 变量                 | 含义                                                        |
| ------------------ | --------------------------------------------------------- |
| `GNU_CONFIGURE`    |  Port 使用 `configure` 脚本进行构建准备。                                |
| `HAS_CONFIGURE`    | 与 `GNU_CONFIGURE` 相同，只是默认的配置目标没有添加到 `CONFIGURE_ARGS` 中。   |
| `CONFIGURE_ARGS`   | 传递给 `configure` 脚本的额外参数。                                  |
| `CONFIGURE_ENV`    | 为 `configure` 脚本运行设置的额外环境变量。                              |
| `CONFIGURE_TARGET` | 重写默认的配置目标。默认值为 `${MACHINE_ARCH}-portbld-freebsd${OSREL}`。 |

### 6.6.4. 使用 `cmake`

对于使用 CMake 的 Port ，定义 `USES= cmake`。

| 变量                  | 含义                                                                                                                                                                  |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CMAKE_ARGS`        | 传递给 `cmake` 二进制文件的 Port 特定 CMake 标志。                                                                                                                                    |
| `CMAKE_ON`          | 对于 `CMAKE_ON` 中的每个条目，都会将启用的布尔值添加到 `CMAKE_ARGS` 中。请参见 [`CMAKE_ON` 和 `CMAKE_OFF`](https://docs.freebsd.org/en/books/porters-handbook/special/#using-cmake-example2)。  |
| `CMAKE_OFF`         | 对于 `CMAKE_OFF` 中的每个条目，都会将禁用的布尔值添加到 `CMAKE_ARGS` 中。请参见 [`CMAKE_ON` 和 `CMAKE_OFF`](https://docs.freebsd.org/en/books/porters-handbook/special/#using-cmake-example2)。 |
| `CMAKE_BUILD_TYPE`  | 构建类型（CMake 预定义的构建配置）。默认值为 `Release`，如果设置了 `WITH_DEBUG`，则为 `Debug`。                                                                                                  |
| `CMAKE_SOURCE_PATH` | 源目录的路径。默认值为 `${WRKSRC}`。                                                                                                                                            |
| `CONFIGURE_ENV`     | 为 `cmake` 二进制文件设置的额外环境变量。                                                                                                                                           |

**表 3. 用户可以为 `cmake` 构建定义的变量**

| 变量              | 说明                                                 |
| --------------- | -------------------------------------------------- |
| `CMAKE_NOCOLOR` | 禁用彩色构建输出。默认未设置，除非设置了 `BATCH` 或 `PACKAGE_BUILDING`。 |

CMake 支持以下构建配置：`Debug`、`Release`、`RelWithDebInfo` 和 `MinSizeRel`。`Debug` 和 `Release` 配置会遵循系统 `*FLAGS`，`RelWithDebInfo` 和 `MinSizeRel` 会分别将 `CFLAGS` 设置为 `-O2 -g` 和 `-Os -DNDEBUG`。`CMAKE_BUILD_TYPE` 的小写值会导出到 `PLIST_SUB`，并且如果 Port 安装 **\*.cmake** 依赖于构建类型（参见 [devel/kf5-kcrash](https://cgit.freebsd.org/ports/tree/devel/kf5-kcrash/) 示例），必须使用该值。请注意，一些项目可能会定义自己的构建配置和/或通过在 **CMakeLists.txt** 中设置 `CMAKE_BUILD_TYPE` 强制特定的构建类型。为了让这样的项目的 Port 尊重 `CFLAGS` 和 `WITH_DEBUG`，必须删除这些文件中的 `CMAKE_BUILD_TYPE` 定义。

大多数基于 CMake 的项目支持源外构建方法。 Port 的源外构建是默认设置。可以通过使用 `:insource` 后缀请求源内构建。在源外构建中，`CONFIGURE_WRKSRC`、`BUILD_WRKSRC` 和 `INSTALL_WRKSRC` 会被设置为 `${WRKDIR}/.build`，并且该目录将用于存放在配置和构建阶段生成的所有文件，保持源代码目录完好。

**示例 2. `USES= cmake` 示例**

该片段演示了如何在 Port 中使用 CMake。通常不需要设置 `CMAKE_SOURCE_PATH`，但当源代码不在顶层目录中，或者 Port 只打算构建项目的一部分时，可以设置该值。

```sh
USES=			cmake
CMAKE_SOURCE_PATH=	${WRKSRC}/subproject
```

**示例 3. `CMAKE_ON` 和 `CMAKE_OFF`**

当向 `CMAKE_ARGS` 添加布尔值时，使用 `CMAKE_ON` 和 `CMAKE_OFF` 变量更为简便。例如：

```sh
CMAKE_ON=	VAR1 VAR2
CMAKE_OFF=	VAR3
```

等同于：

```sh
CMAKE_ARGS=	-DVAR1:BOOL=TRUE -DVAR2:BOOL=TRUE -DVAR3:BOOL=FALSE
```

>**重要**
>
> 这仅适用于 `CMAKE_ARGS` 的默认值关闭。 [`OPT_CMAKE_BOOL` 和 `OPT_CMAKE_BOOL_OFF`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#options-cmake_bool) 中说明的辅助工具使用相同的语义，但适用于可选值。

### 6.6.5. 使用 `scons`

如果 Port 使用 SCons，请定义 `USES=scons`。

为了让第三方 **SConstruct** 尊重传递给 SCons 的所有环境变量（即最重要的 `CC/CXX/CFLAGS/CXXFLAGS`），请修改 **SConstruct**，使得构建 `Environment` 如下构建：

```
env = Environment(**ARGUMENTS)
```

之后，可以使用 `env.Append` 和 `env.Replace` 进行修改。

### 6.6.6. 使用 `cargo` 构建 Rust 应用程序

对于使用 Cargo 的 Port ，定义 `USES=cargo`。

**表 4. 用户可以为 `cargo` 构建定义的变量**

| 变量                   | 默认值                      | 说明                                                                                                                                                                                                                                                                |
| -------------------- | ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CARGO_CRATES`       |                          |  Port 依赖的 crate 列表。每个条目需要具有 `cratename-semver` 的格式，例如 `libc-0.2.40`。 Port 维护者可以通过 `make cargo-crates` 从 **Cargo.lock** 生成此列表。手动更新 crate 版本是可能的，但要注意传递依赖。如果 `make cargo-crates` 生成的列表很大，建议将其放在顶层 Port 目录中的 `Makefile.crates` 文件中。如果存在，该文件会自动被 Ports 框架引入。这有助于保持主 Port  Makefile 的可管理大小。 |
| `CARGO_FEATURES`     |                          | 要构建的应用程序功能列表（以空格分隔）。要停用所有默认功能，请将特殊标记 `--no-default-features` 添加到 `CARGO_FEATURES`。不需要手动将其传递给 `CARGO_BUILD_ARGS`、`CARGO_INSTALL_ARGS` 和 `CARGO_TEST_ARGS`。                                                                                                         |
| `CARGO_CARGOTOML`    | `${WRKSRC}/Cargo.toml`   | 使用的 **Cargo.toml** 的路径。                                                                                                                                                                                                                                           |
| `CARGO_CARGOLOCK`    | `${WRKSRC}/Cargo.lock`   | 用于 `make cargo-crates` 的 **Cargo.lock** 的路径。必要时，可以指定多个锁定文件。                                                                                                                                                                                                       |
| `CARGO_ENV`          |                          | 要传递给 Cargo 的环境变量列表，类似于 `MAKE_ENV`。                                                                                                                                                                                                                                |
| `RUSTFLAGS`          |                          | 传递给 Rust 编译器的标志。                                                                                                                                                                                                                                                  |
| `CARGO_CONFIGURE`    | `yes`                    | 使用默认的 `do-configure`。                                                                                                                                                                                                                                             |
| `CARGO_UPDATE_ARGS`  |                          | 在配置阶段传递给 Cargo 的额外参数。有效的参数可以通过 `cargo update --help` 查找。                                                                                                                                                                                                          |
| `CARGO_BUILDDEP`     | `yes`                    | 添加对 [lang/rust](https://cgit.freebsd.org/ports/tree/lang/rust/) 的构建依赖。                                                                                                                                                                                            |
| `CARGO_CARGO_BIN`    | `${LOCALBASE}/bin/cargo` | `cargo` 二进制文件的位置。                                                                                                                                                                                                                                                 |
| `CARGO_BUILD`        | `yes`                    | 使用默认的 `do-build`。                                                                                                                                                                                                                                                 |
| `CARGO_BUILD_ARGS`   |                          | 在构建阶段传递给 Cargo 的额外参数。有效的参数可以通过 `cargo build --help` 查找。                                                                                                                                                                                                           |
| `CARGO_INSTALL`      | `yes`                    | 使用默认的 `do-install`。                                                                                                                                                                                                                                               |
| `CARGO_INSTALL_ARGS` |                          | 在安装阶段传递给 Cargo 的额外参数。有效的参数可以通过 `cargo install --help` 查找。                                                                                                                                                                                                         |
| `CARGO_INSTALL_PATH` | `.`                      | 要安装的 crate 的路径。这通过 `--path` 参数传递给 `cargo install`。当指定多个路径时，会多次运行 `cargo install`。                                                                                                                                                                                 |
| `CARGO_TEST`         | `yes`                    | 使用默认的 `do-test`。                                                                                                                                                                                                                                                  |
| `CARGO_TEST_ARGS`    |                          | 在测试阶段传递给 Cargo 的额外参数。有效的参数可以通过 `cargo test --help` 查找。                                                                                                                                                                                                            |
| `CARGO_TARGET_DIR`   | `${WRKDIR}/target`       | Cargo 输出目录的位置。                                                                                                                                                                                                                                                    |
| `CARGO_DIST_SUBDIR`  | **rust/crates**          | 相对于 `DISTDIR` 的目录，存储 crate 分发文件。                                                                                                                                                                                                                                  |
| `CARGO_VENDOR_DIR`   | `${WRKSRC}/cargo-crates` | 存放所有 crate 解压目录的供应商目录的位置。请尽量将其放在 `PATCH_WRKSRC` 下，以便于应用补丁。                                                                                                                                                                                                        |
| `CARGO_USE_GITHUB`   | `no`                     | 启用通过 `GH_TUPLE` 从 GitHub 获取锁定到特定 Git 提交的 crates。这将尝试修补 `WRKDIR` 下的所有 **Cargo.toml**，以便指向离线源，而不是在构建期间从 Git 仓库获取它们。                                                                                                                                                 |
| `CARGO_USE_GITLAB`   | `no`                     | 与 `CARGO_USE_GITHUB` 相同，但适用于 GitLab 实例和 `GL_TUPLE`。                                                                                                                                                                                                               |

**示例 4. 创建一个简单的 Rust 应用程序 Port**

创建一个基于 Cargo 的 Port 是一个三阶段的过程。首先，我们需要提供一个 Port 模板来获取应用程序分发文件：

```sh
PORTNAME=	tokei
DISTVERSIONPREFIX=	v
DISTVERSION=	7.0.2
CATEGORIES=	devel

MAINTAINER=	tobik@FreeBSD.org
COMMENT=	Display statistics about your code
WWW=		https://github.com/XAMPPRocky/tokei/

USES=		cargo
USE_GITHUB=	yes
GH_ACCOUNT=	Aaronepower

.include <bsd.port.mk>
```

生成初始的 **distinfo**：

```sh
% make makesum
=> Aaronepower-tokei-v7.0.2_GH0.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch https://codeload.github.com/Aaronepower/tokei/tar.gz/v7.0.2?dummy=/Aaronepower-tokei-v7.0.2_GH0.tar.gz
fetch: https://codeload.github.com/Aaronepower/tokei/tar.gz/v7.0.2?dummy=/Aaronepower-tokei-v7.0.2_GH0.tar.gz: size of remote file is not known
Aaronepower-tokei-v7.0.2_GH0.tar.gz                     45 kB  239 kBps 00m00s
```

现在，分发文件已经准备好使用，我们可以继续从捆绑的 **Cargo.lock** 中提取 crate 依赖：

```sh
% make cargo-crates
CARGO_CRATES=   aho-corasick-0.6.4 \
                ansi_term-0.11.0 \
                arrayvec-0.4.7 \
                atty-0.2.9 \
                bitflags-1.0.1 \
                byteorder-1.2.2 \
                [...]
```

此命令的输出需要直接粘贴到 Makefile 中：

```sh
PORTNAME=	tokei
DISTVERSIONPREFIX=	v
DISTVERSION=	7.0.2
CATEGORIES=	devel

MAINTAINER=	tobik@FreeBSD.org
COMMENT=	Display statistics about your code
WWW=		https://github.com/XAMPPRocky/tokei/

USES=		cargo
USE_GITHUB=	yes
GH_ACCOUNT=	Aaronepower

CARGO_CRATES=   aho-corasick-0.6.4 \
                ansi_term-0.11.0 \
                arrayvec-0.4.7 \
                atty-0.2.9 \
                bitflags-1.0.1 \
                byteorder-1.2.2 \
                [...]

.include <bsd.port.mk>
```

需要重新生成 **distinfo** 以包含所有的 crate 分发文件：

```sh
% make makesum
=> rust/crates/aho-corasick-0.6.4.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch https://crates.io/api/v1/crates/aho-corasick/0.6.4/download?dummy=/rust/crates/aho-corasick-0.6.4.tar.gz
rust/crates/aho-corasick-0.6.4.tar.gz         100% of   24 kB 6139 kBps 00m00s
=> rust/crates/ansi_term-0.11.0.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch https://crates.io/api/v1/crates/ansi_term/0.11.0/download?dummy=/rust/crates/ansi_term-0.11.0.tar.gz
rust/crates/ansi_term-0.11.0.tar.gz           100% of   16 kB   21 MBps 00m00s
=> rust/crates/arrayvec-0.4.7.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch https://crates.io/api/v1/crates/arrayvec/0.4.7/download?dummy=/rust/crates/arrayvec-0.4.7.tar.gz
rust/crates/arrayvec-0.4.7.tar.gz             100% of   22 kB 3237 kBps 00m00s
=> rust/crates/atty-0.2.9.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch https://crates.io/api/v1/crates/atty/0.2.9/download?dummy=/rust/crates/atty-0.2.9.tar.gz
rust/crates/atty-0.2.9.tar.gz                 100% of 5898  B   81 MBps 00m00s
=> rust/crates/bitflags-1.0.1.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
[...]
```

该 Port 现在已准备好进行测试构建，并可以像正常情况一样进行进一步的调整，例如创建 plist、编写说明、添加许可信息、选项等。

如果你没有在像 poudriere 这样的干净环境中测试 Port ，记得在任何测试之前运行 `make clean`。

**示例 5. 启用额外的应用程序功能**

一些应用程序在其 **Cargo.toml** 中定义了额外的功能。可以通过在 Port 中设置 `CARGO_FEATURES` 来编译它们。

在这里，我们启用 Tokei 的 `json` 和 `yaml` 功能：

```sh
CARGO_FEATURES=	json yaml
```

**示例 6. 将应用程序功能编码为 Port 选项**

**Cargo.toml** 中的一个 `[features]` 部分可能如下所示：

```sh
[features]
pulseaudio_backend = ["librespot-playback/pulseaudio-backend"]
portaudio_backend = ["librespot-playback/portaudio-backend"]
default = ["pulseaudio_backend"]
```

`pulseaudio_backend` 是一个默认功能，除非我们显式地通过将 `--no-default-features` 添加到 `CARGO_FEATURES` 来关闭它。这里，我们将 `portaudio_backend` 和 `pulseaudio_backend` 功能转化为 Port 选项：

```sh
CARGO_FEATURES=	--no-default-features

OPTIONS_DEFINE=	PORTAUDIO PULSEAUDIO

PORTAUDIO_VARS=		CARGO_FEATURES+=portaudio_backend
PULSEAUDIO_VARS=	CARGO_FEATURES+=pulseaudio_backend
```

**示例 7. 列出 Crate 许可证**

每个 Crate 都有自己的许可证。在为 Port 添加 `LICENSE` 块时，了解它们非常重要（参见 [许可证](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses)）。辅助目标 `cargo-crates-licenses` 将尝试列出 `CARGO_CRATES` 中定义的所有 Crate 的许可证。

```sh
% make cargo-crates-licenses
aho-corasick-0.6.4  Unlicense/MIT
ansi_term-0.11.0    MIT
arrayvec-0.4.7      MIT/Apache-2.0
atty-0.2.9          MIT
bitflags-1.0.1      MIT/Apache-2.0
byteorder-1.2.2     Unlicense/MIT
[...]
```

>**注意**
>
>  `make cargo-crates-licenses` 输出的许可证名称是 SPDX 2.1 许可证表达式，可能与 Ports 框架中定义的许可证名称不匹配。需要将它们翻译为 [预定义许可证列表](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#licenses-license-list) 中的名称。 

### 6.6.7. 使用 `meson`

对于使用 Meson 的 Port ，请定义 `USES=meson`。

**表格 5. 使用 `meson` 的 Port 变量**

| 变量                | 说明                                 |
| ----------------- | ---------------------------------- |
| `MESON_ARGS`      | 要传递给 `meson` 二进制文件的 Port 特定 Meson 标志。  |
| `MESON_BUILD_DIR` | 相对于 `WRKSRC` 的构建目录路径。默认是 `_build`。 |

**示例 8. `USES=meson` 示例**

此代码片段演示了在 Port 中使用 Meson。

```sh
USES=		meson
MESON_ARGS=	-Dfoo=enabled
```

### 6.6.8. 构建 Go 应用程序

对于使用 Go 的 Port ，定义 `USES=go`。请参考 [`go`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-go) 获取可以设置的变量列表，以控制构建过程。

**示例 9. 创建一个基于 Go 模块的应用程序 Port**

在大多数情况下，将 `GO_MODULE` 变量设置为 `go.mod` 中 `module` 指令指定的值就足够了：

```sh
PORTNAME=       hey
DISTVERSIONPREFIX=	v
DISTVERSION=    0.1.4
CATEGORIES=     benchmarks

MAINTAINER=     dmgk@FreeBSD.org
COMMENT=        Tiny program that sends some load to a web application
WWW=            https://github.com/rakyll/hey/

LICENSE=        APACHE20
LICENSE_FILE=   ${WRKSRC}/LICENSE

USES=           go:modules
GO_MODULE=      github.com/rakyll/hey

PLIST_FILES=    bin/hey

.include <bsd.port.mk>
```

如果“简易”方法不适用，或者需要对依赖项进行更多控制，可以按照下面说明的完整 Port 过程。

创建基于 Go 的 Port 是一个五阶段的过程。首先，我们需要提供一个 Port 模板来获取应用程序分发文件：

```sh
PORTNAME=	ghq
DISTVERSIONPREFIX=	v
DISTVERSION=	0.12.5
CATEGORIES=	devel

MAINTAINER=	tobik@FreeBSD.org
COMMENT=	Remote repository management made easy
WWW=		https://github.com/x-motemen/ghq/

USES=		go:modules
USE_GITHUB=	yes
GH_ACCOUNT=	motemen

.include <bsd.port.mk>
```

生成初步的 **distinfo** 文件：

```sh
% make makesum
===>  License MIT accepted by the user
=> motemen-ghq-v0.12.5_GH0.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch https://codeload.github.com/motemen/ghq/tar.gz/v0.12.5?dummy=/motemen-ghq-v0.12.5_GH0.tar.gz
fetch: https://codeload.github.com/motemen/ghq/tar.gz/v0.12.5?dummy=/motemen-ghq-v0.12.5_GH0.tar.gz: size of remote file is not known
motemen-ghq-v0.12.5_GH0.tar.gz                          32 kB  177 kBps    00s
```

现在，分发文件已经准备好使用，我们可以提取所需的 Go 模块依赖项。此步骤需要安装 [ports-mgmt/modules2tuple](https://cgit.freebsd.org/ports/tree/ports-mgmt/modules2tuple/)：

```sh
% make gomod-vendor
[...]
GH_TUPLE=	\
		Songmu:gitconfig:v0.0.2:songmu_gitconfig/vendor/github.com/Songmu/gitconfig \
		daviddengcn:go-colortext:186a3d44e920:daviddengcn_go_colortext/vendor/github.com/daviddengcn/go-colortext \
		go-yaml:yaml:v2.2.2:go_yaml_yaml/vendor/gopkg.in/yaml.v2 \
		golang:net:3ec191127204:golang_net/vendor/golang.org/x/net \
		golang:sync:112230192c58:golang_sync/vendor/golang.org/x/sync \
		golang:xerrors:3ee3066db522:golang_xerrors/vendor/golang.org/x/xerrors \
		motemen:go-colorine:45d19169413a:motemen_go_colorine/vendor/github.com/motemen/go-colorine \
		urfave:cli:v1.20.0:urfave_cli/vendor/github.com/urfave/cli
```

该命令的输出需要直接粘贴到 Makefile 中：

```sh
PORTNAME=	ghq
DISTVERSIONPREFIX=	v
DISTVERSION=	0.12.5
CATEGORIES=	devel

MAINTAINER=	tobik@FreeBSD.org
COMMENT=	Remote repository management made easy
WWW=		https://github.com/x-motemen/ghq/

USES=		go:modules
USE_GITHUB=	yes
GH_ACCOUNT=	motemen
GH_TUPLE=	Songmu:gitconfig:v0.0.2:songmu_gitconfig/vendor/github.com/Songmu/gitconfig \
		daviddengcn:go-colortext:186a3d44e920:daviddengcn_go_colortext/vendor/github.com/daviddengcn/go-colortext \
		go-yaml:yaml:v2.2.2:go_yaml_yaml/vendor/gopkg.in/yaml.v2 \
		golang:net:3ec191127204:golang_net/vendor/golang.org/x/net \
		golang:sync:112230192c58:golang_sync/vendor/golang.org/x/sync \
		golang:xerrors:3ee3066db522:golang_xerrors/vendor/golang.org/x/xerrors \
		motemen:go-colorine:45d19169413a:motemen_go_colorine/vendor/github.com/motemen/go-colorine \
		urfave:cli:v1.20.0:urfave_cli/vendor/github.com/urfave/cli

.include <bsd.port.mk>
```

**distinfo** 需要重新生成，以包含所有分发文件：

```sh
% make makesum
=> Songmu-gitconfig-v0.0.2_GH0.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch https://codeload.github.com/Songmu/gitconfig/tar.gz/v0.0.2?dummy=/Songmu-gitconfig-v0.0.2_GH0.tar.gz
fetch: https://codeload.github.com/Songmu/gitconfig/tar.gz/v0.0.2?dummy=/Songmu-gitconfig-v0.0.2_GH0.tar.gz: size of remote file is not known
Songmu-gitconfig-v0.0.2_GH0.tar.gz                    5662  B  936 kBps    00s
=> daviddengcn-go-colortext-186a3d44e920_GH0.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch https://codeload.github.com/daviddengcn/go-colortext/tar.gz/186a3d44e920?dummy=/daviddengcn-go-colortext-186a3d44e920_GH0.tar.gz
fetch: https://codeload.github.com/daviddengcn/go-colortext/tar.gz/186a3d44e920?dummy=/daviddengcn-go-colortext-186a3d44e920_GH0.tar.gz: size of remote file is not known
daviddengcn-go-colortext-186a3d44e920_GH0.tar.        4534  B 1098 kBps    00s
[...]
```

现在， Port 已准备好进行测试构建，并可进行进一步调整，如创建 plist、编写说明、添加许可证信息、选项等，像平常一样进行。

如果你没有在像 poudriere 这样的干净环境中测试 Port ，请记得在任何测试之前运行 `make clean`。

**示例 10. 设置输出二进制名称或安装路径**

有些 Port 需要将生成的二进制文件安装到不同的名称或路径下，而不是默认的 `${PREFIX}/bin`。可以通过使用 `GO_TARGET` 元组语法来实现，例如：

```sh
GO_TARGET=  ./cmd/ipfs:ipfs-go
```

这将把 `ipfs` 二进制文件安装为 `${PREFIX}/bin/ipfs-go`，而

```sh
GO_TARGET=  ./dnscrypt-proxy:${PREFIX}/sbin/dnscrypt-proxy
```

将把 `dnscrypt-proxy` 安装到 `${PREFIX}/sbin`。

### 6.6.9. 使用 `cabal` 构建 Haskell 应用程序

对于使用 Cabal 的 Ports，构建系统定义了 `USES=cabal`。参阅 [`cabal`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-cabal) 获取可以设置的变量列表，以控制构建过程。

**示例 11. 创建一个 Hackage 托管的 Haskell 应用程序的 Port**

在准备一个 Haskell Cabal Port 时，必须先安装 [devel/hs-cabal-install](https://cgit.freebsd.org/ports/tree/devel/hs-cabal-install/) 和 [ports-mgmt/hs-cabal2tuple](https://cgit.freebsd.org/ports/tree/ports-mgmt/hs-cabal2tuple/) 程序。首先，我们需要定义一些常见的 Port 变量，以便 cabal-install 获取包的分发文件：

```makefile
PORTNAME=	ShellCheck
DISTVERSION=	0.6.0
CATEGORIES=	devel

MAINTAINER=	haskell@FreeBSD.org
COMMENT=	Shell script analysis tool
WWW=		https://www.shellcheck.net/

USES=		cabal

.include <bsd.port.mk>
```

这个最简 Makefile 使用 `cabal-extract` 辅助目标来获取分发文件：

```sh
% make cabal-extract
[...]
Downloading the latest package list from hackage.haskell.org
cabal get ShellCheck-0.6.0
Downloading  ShellCheck-0.6.0
Downloaded   ShellCheck-0.6.0
Unpacking to ShellCheck-0.6.0/
```

现在，我们已经在 `${WRKSRC}` 下得到了 ShellCheck.cabal 包说明文件，可以使用 `cabal-configure` 生成构建计划：

```sh
% make cabal-configure
[...]
Resolving dependencies...
Build profile: -w ghc-8.10.7 -O1
In order, the following would be built (use -v for more details):
 - Diff-0.4.1 (lib) (requires download & build)
 - OneTuple-0.3.1 (lib) (requires download & build)
[...]
```

完成后，可以生成所需依赖关系的列表：

```sh
% make make-use-cabal
USE_CABAL=	QuickCheck-2.12.6.1 \
		hashable-1.3.0.0 \
		integer-logarithms-1.0.3 \
[...]
```

Haskell 包可能包含版本修订，就像 FreeBSD 的 Port 一样。修订仅影响 **.cabal** 文件。请注意 `_` 后面的附加版本号。将新生成的 `USE_CABAL` 列表替换旧的。

最后，需要重新生成 **distinfo** 以包含所有分发文件：

```sh
% make makesum
=> ShellCheck-0.6.0.tar.gz doesn't seem to exist in /usr/local/poudriere/ports/git/distfiles/cabal.
=> Attempting to fetch https://hackage.haskell.org/package/ShellCheck-0.6.0/ShellCheck-0.6.0.tar.gz
ShellCheck-0.6.0.tar.gz                                136 kB  642 kBps    00s
=> QuickCheck-2.12.6.1/QuickCheck-2.12.6.1.tar.gz doesn't seem to exist in /usr/local/poudriere/ports/git/distfiles/cabal.
=> Attempting to fetch https://hackage.haskell.org/package/QuickCheck-2.12.6.1/QuickCheck-2.12.6.1.tar.gz
QuickCheck-2.12.6.1/QuickCheck-2.12.6.1.tar.gz          65 kB  361 kBps    00s
[...]
```

现在， Port 已准备好进行测试构建，并且可以根据需要进一步调整，例如创建 plist、编写说明、添加许可证信息、选项等。

如果您没有在像 poudriere 这样的干净环境中测试 Port ，请记得在任何测试前运行 `make clean`。

某些 Haskell  Port 会将各种数据文件安装到 `share/${PORTNAME}` 下。对于此类情况， Port 侧需要进行特殊处理。 Port 应定义 `CABAL_WRAPPER_SCRIPTS` 变量，列出将使用数据文件的每个可执行文件。此外，在少数情况下，被移植的程序使用其他 Haskell 包的数据文件，此时 `FOO_DATADIR_VARS` 变量会派上用场。

**示例 12. 处理 Haskell Port 中的数据文件**

`devel/hs-profiteur` 是一个 Haskell 应用程序，它生成一个包含内容的单页 HTML。

```sh
PORTNAME=	profiteur

[...]

USES=		cabal

USE_CABAL=	OneTuple-0.3.1_2 \
		QuickCheck-2.14.2 \
		[...]

.include <bsd.port.mk>
```

它将 HTML 模板安装到 `share/profiteur` 下，因此我们需要添加 `CABAL_WRAPPER_SCRIPTS` 选项：

```sh
[...]

USE_CABAL=	OneTuple-0.3.1_2 \
		QuickCheck-2.14.2 \
		[...]


CABAL_WRAPPER_SCRIPTS=		${CABAL_EXECUTABLES}

.include <bsd.port.mk>
```

该程序还尝试访问 `jquery.js` 文件，这是 `js-jquery-3.3.1` Haskell 包的一部分。为了能够找到该文件，我们需要使包装脚本也在 `share/profiteur` 中查找 `js-jquery` 数据文件。我们使用 `profiteur_DATADIR_VARS` 来实现这一点：

```sh
[...]

CABAL_WRAPPER_SCRIPTS=		${CABAL_EXECUTABLES}
profiteur_DATADIR_VARS=		js-jquery

.include <bsd.port.mk>
```

现在，该 Port 将把实际的二进制文件安装到 `libexec/cabal/profiteur`，并将脚本安装到 `bin/profiteur`。

除了运行程序并检查一切是否正常工作外，没有简单的方法来确定 `FOO_DATADIR_VARS` 选项的正确值。幸运的是，使用 `FOO_DATADIR_VARS` 的情况非常少见。

另一个在移植复杂的 Haskell 程序时可能遇到的特殊情况是 `cabal.project` 文件中存在 VCS 依赖项。

**示例 13. 移植具有 VCS 依赖项的 Haskell 应用程序**

`net-p2p/cardano-node` 是一个非常复杂的软件。在它的 `cabal.project` 中，有很多类似这样的块：

```ini
[...]
source-repository-package
  type: git
  location: https://github.com/input-output-hk/cardano-crypto
  tag: f73079303f663e028288f9f4a9e08bcca39a923e
[...]
```

`source-repository-package` 类型的依赖项会在构建过程中自动由 `cabal` 拉取。不幸的是，这使得在 `fetch` 阶段之后需要使用网络。这是 Ports 框架不允许的。因此，这些源需要在 Port 的 Makefile 中列出。`make-use-cabal` 辅助目标可以使 GitHub 上托管的包更容易处理。在常规的 `cabal-extract` 和 `cabal-configure` 后运行这个目标，不仅会生成 `USE_CABAL` 选项，还会生成 `GH_TUPLE`：

```
% make make-use-cabal
USE_CABAL=	Diff-0.4.1 \
		Glob-0.10.2_3 \
		HUnit-1.6.2.0 \
		[...]

GH_TUPLE=		input-output-hk:cardano-base:0f3a867493059e650cda69e20a5cbf1ace289a57:cardano_base/dist-newstyle/src/cardano-b_-c8db9876882556ed \
		input-output-hk:cardano-crypto:f73079303f663e028288f9f4a9e08bcca39a923e:cardano_crypto/dist-newstyle/src/cardano-c_-253fd88117badd8f \
		[...]
```

将 `make-use-cabal` 生成的 `GH_TUPLE` 条目与其他条目分开可能会更有用，这样可以方便地更新 Port ：

```
GH_TUPLE=	input-output-hk:cardano-base:0f3a867493059e650cda69e20a5cbf1ace289a57:cardano_base/dist-newstyle/src/cardano-b_-c8db9876882556ed \
		input-output-hk:cardano-crypto:f73079303f663e028288f9f4a9e08bcca39a923e:cardano_crypto/dist-newstyle/src/cardano-c_-253fd88117badd8f \
		[...]

GH_TUPLE+=	bitcoin-core:secp256k1:ac83be33d0956faf6b7f61a60ab524ef7d6a473a:secp
```

对于具有 VCS 依赖项的 Haskell  Port ，目前还需要以下修复：

```
BINARY_ALIAS=	git=true
```

## 6.7. 使用 GNU Autotools

如果一个 Port 需要任何 GNU Autotools 软件，添加 `USES=autoreconf`。有关更多信息，请参阅 [`autoreconf`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-autoreconf)。

## 6.8. 使用 GNU `gettext`

### 6.8.1. 基本用法

如果 Port 需要 `gettext`，请设置 `USES=gettext`，该 Port 将继承来自 [devel/gettext](https://cgit.freebsd.org/ports/tree/devel/gettext/) 的 **libintl.so** 依赖项。有关 `gettext` 用法的其他值，请参阅 [`USES=gettext`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-gettext)。

一个比较常见的情况是 Port 使用 `gettext` 和 `configure`。通常，GNU `configure` 应该能够自动定位 `gettext`。

```
USES=	gettext
GNU_CONFIGURE=	yes
```

如果自动定位失败，可以通过 `CPPFLAGS` 和 `LDFLAGS` 传递 `gettext` 的位置，使用 `localbase` 如下：

```
USES=	gettext localbase:ldflags
GNU_CONFIGURE=	yes
```

### 6.8.2. 可选用法

一些软件产品允许禁用 NLS。例如，通过向 `configure` 传递 `--disable-nls`。在这种情况下， Port 必须根据 `NLS` 选项的状态有条件地使用 `gettext`。对于低到中等复杂度的 Port ，可以使用以下惯用法：

```
GNU_CONFIGURE=		yes

OPTIONS_DEFINE=		NLS
OPTIONS_SUB=		yes

NLS_USES=		gettext
NLS_CONFIGURE_ENABLE=	nls

.include <bsd.port.mk>
```

或者使用较旧的选项方法：

```sh
GNU_CONFIGURE=		yes

OPTIONS_DEFINE=		NLS

.include <bsd.port.options.mk>

.if ${PORT_OPTIONS:MNLS}
USES+=			gettext
PLIST_SUB+=		NLS=""
.else
CONFIGURE_ARGS+=	--disable-nls
PLIST_SUB+=		NLS="@comment "
.endif

.include <bsd.port.mk>
```

接下来需要做的工作是安排使消息目录文件条件地包含在打包列表中。**Makefile** 部分的这个任务已经由惯用法提供。它在 [高级 **pkg-plist** 实践](https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-sub) 部分中进行了说明。简而言之，**pkg-plist** 中每个出现的 `%%NLS%%` 将在 NLS 被禁用时替换为 "`@comment `"，或者在 NLS 启用时替换为空字符串。因此，如果 NLS 关闭，以 `%%NLS%%` 为前缀的行将在最终的打包列表中变成注释；否则，前缀将直接被省略。然后，在 **pkg-plist** 中每个消息目录文件路径之前插入 `%%NLS%%`。例如：

```sh
%%NLS%%share/locale/fr/LC_MESSAGES/foobar.mo
%%NLS%%share/locale/no/LC_MESSAGES/foobar.mo
```

在高复杂度的情况下，可能需要更高级的技术，例如 [动态打包列表生成](https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-dynamic)。

### 6.8.3. 处理消息目录

有一点需要注意的是，安装消息目录文件时，目标目录必须位于 **LOCALBASE/share/locale** 下，并且这些目录不应由 Port 创建或删除。最常见的语言有各自的目录列在 **PORTSDIR/Templates/BSD.local.dist** 中。许多其他语言的目录由 [devel/gettext](https://cgit.freebsd.org/ports/tree/devel/gettext/)  Port 管理。请查阅其 **pkg-plist** 并查看该 Port 是否会为独特的语言安装消息目录文件。

## 6.9. 使用 Perl

如果 `MASTER_SITES` 设置为 `CPAN`，通常会自动选择正确的子目录。如果默认子目录错误，可以使用 `CPAN/Module` 来更改它。`MASTER_SITES` 也可以设置为旧的 `MASTER_SITE_PERL_CPAN`，然后 `MASTER_SITE_SUBDIR` 的首选值是顶级层次名称。例如，推荐的 `p5-Module-Name` 的值是 `Module`。可以在 [cpan.org](https://cpan.org/modules/by-module/) 上查看顶级层次结构。这会在模块的作者更改时保持 Port 的正常工作。

这个规则的例外情况是相关目录不存在，或者该目录中没有 distfile。在这种情况下，允许使用作者的 ID 作为 `MASTER_SITE_SUBDIR`。可以使用 `CPAN:AUTHOR` 宏，它将转换为哈希作者目录。例如，`CPAN:AUTHOR` 将转换为 `authors/id/A/AU/AUTHOR`。

当 Port 需要 Perl 支持时，必须设置 `USES=perl5`，并根据 [perl5 USES 说明](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-perl5) 使用可选的 `USE_PERL5`。

**表 6. 使用 Perl 的 Port 只读变量**

| 只读变量           | 说明                                                                                                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `PERL`         | Perl 5 解释器的完整路径，无论是系统自带的还是从 Port 安装的，但不带版本号。当软件需要 Perl 解释器的路径时使用此变量。要替换脚本中的 "`#!`" 行，请使用 [`shebangfix`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-shebangfix)。 |
| `PERL_VERSION` | 安装的 Perl 的完整版本（例如，`5.8.9`）。                                                                                                                                                    |
| `PERL_LEVEL`   | 安装的 Perl 版本，形式为 `MNNNPP` 的整数（例如，`500809`）。                                                                                                                                     |
| `PERL_ARCH`    | Perl 存储架构相关库的位置。默认为 `${ARCH}-freebsd`。                                                                                                                                         |
| `PERL_PORT`    | 已安装的 Perl  Port 的名称（例如，`perl5`）。                                                                                                                                                   |
| `SITE_PERL`    | 存放特定于站点的 Perl 包的目录名称。此值会被添加到 `PLIST_SUB` 中。                                                                                                                                    |

>**注意**
>
> 没有官方网站的 Perl 模块 Port 必须在 **Makefile** 的 WWW 行中链接到 `cpan.org`。推荐的 URL 形式是 `https://search.cpan.org/dist/Module-Name/`（包括尾部斜杠）。


>**注意**
>
> 不要在依赖声明中使用 `${SITE_PERL}`。这样做假设已经包含了 **perl5.mk**，但并非总是如此。如果 Port 的文件在升级过程中被移动，依赖于该 Port 的其他 Port 可能会有错误的依赖关系。正确声明 Perl 模块依赖关系的方法如下所示。 


**示例 14. Perl 依赖示例**

```sh
p5-IO-Tee>=0.64:devel/p5-IO-Tee
```

对于安装手册页的 Perl  Port ，可以在 **pkg-plist** 中使用宏 `PERL5_MAN3` 和 `PERL5_MAN1`。例如，

```sh
lib/perl5/5.14/man/man1/event.1.gz
lib/perl5/5.14/man/man3/AnyEvent::I3.3.gz
```

可以替换为

```sh
%%PERL5_MAN1%%/event.1.gz
%%PERL5_MAN3%%/AnyEvent::I3.3.gz
```

>**注意**
>
> 对于其他部分（`2` 和 `4` 到 `9`）没有 `PERL5_MAN_x_` 宏，因为这些文件会安装到常规目录中。


**示例 15. 仅在构建时需要 Perl 的 Port**

由于默认的 USE\_PERL5 值为 `build` 和 `run`，应设置为：

```sh
USES=		perl5
USE_PERL5=	build
```

**示例 16. 还需要 Perl 进行补丁处理的 Port** 

有时，使用 [sed(1)](https://man.freebsd.org/cgi/man.cgi?query=sed&sektion=1&format=html) 进行补丁处理不足以满足要求。当使用 [perl(1)](https://man.freebsd.org/cgi/man.cgi?query=perl&sektion=1&format=html) 更加简便时，使用：

```sh
USES=		perl5
USE_PERL5=	patch build run
```

**示例 17. 需要 `ExtUtils::MakeMaker` 来构建的 Perl 模块**

大多数 Perl 模块带有 **Makefile.PL** 配置脚本。在这种情况下，设置：

```sh
USES=		perl5
USE_PERL5=	configure
```

**示例 18. 需要 `Module::Build` 来构建的 Perl 模块**

当 Perl 模块带有 **Build.PL** 配置脚本时，它可能需要 `Module::Build`，此时设置：

```sh
USES=		perl5
USE_PERL5=	modbuild
```

如果它需要的是 `Module::Build::Tiny`，设置：

```sh
USES=		perl5
USE_PERL5=	modbuildtiny
```

## 6.10. 使用 X11

### 6.10.1. X.Org 组件

在 Ports 中提供的 X11 实现是 X.Org。如果应用程序依赖于 X 组件，添加 `USES= xorg` 并设置 `USE_XORG` 为所需的组件列表。完整的组件列表可以在 [`xorg`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-xorg) 中找到。

Mesa 项目旨在提供免费的 OpenGL 实现。要指定对该项目的不同组件的依赖，使用 `USES= gl` 和 `USE_GL`。有关可用组件的完整列表，请参见 [`gl`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-gl)。为了兼容性，`yes` 的值会映射为 `glu`。

**示例 19. `USE_XORG` 示例**

```sh
USES=		gl xorg
USE_GL=		glu
USE_XORG=	xrender xft xkbfile xt xaw
```

|               | 变量用于使用 X 的 Port                                   |
| ------------- | --------------------------------------------- |
| `USES= imake` | 该 Port 使用 `imake`。                                |
| `XMKMF`       | 如果 `xmkmf` 不在 `PATH` 中，设置为其路径。默认为 `xmkmf -a`。 |

**示例 20. 使用 X11 相关变量**

```sh
# 使用一些 X11 库
USES=		xorg
USE_XORG=	x11 xpm
```

### 6.10.2. 需要 Motif 的 Port 

如果 Port 需要 Motif 库，在 **Makefile** 中定义 `USES= motif`。默认的 Motif 实现是 [x11-toolkits/open-motif](https://cgit.freebsd.org/ports/tree/x11-toolkits/open-motif/)。用户可以通过在 **make.conf** 中设置 `WANT_LESSTIF` 来选择 [x11-toolkits/lesstif](https://cgit.freebsd.org/ports/tree/x11-toolkits/lesstif/) 作为替代实现。同样，用户也可以通过设置 `WANT_OPEN_MOTIF_DEVEL` 来选择 [x11-toolkits/open-motif-devel](https://cgit.freebsd.org/ports/tree/x11-toolkits/open-motif-devel/)。

`MOTIFLIB` 将由 **motif.mk** 设置，指向合适的 Motif 库。请修改 Port 的源代码，以便在原始 **Makefile** 或 **Imakefile** 中引用 Motif 库的位置时使用 `${MOTIFLIB}`。

有两种常见情况：

* 如果 Port 在其 **Makefile** 或 **Imakefile** 中引用 Motif 库为 `-lXm`，请将其替换为 `${MOTIFLIB}`。
* 如果 Port 在其 **Imakefile** 中使用 `XmClientLibs`，请将其替换为 `${MOTIFLIB} ${XTOOLLIB} ${XLIB}`。

请注意，`MOTIFLIB`（通常）会展开为 `-L/usr/local/lib -lXm -lXp` 或 `/usr/local/lib/libXm.a`，因此无需在前面加上 `-L` 或 `-l`。

### 6.10.3. X11 字体

如果 Port 安装了 X Window System 字体，请将它们放置在 **LOCALBASE/lib/X11/fonts/local** 中。

### 6.10.4. 使用 Xvfb 获取虚拟 `DISPLAY`

某些应用程序需要一个工作的 X11 显示器来成功编译。这对于没有图形界面的机器来说是一个问题。当使用此变量时，构建基础设施将启动虚拟帧缓冲 X 服务器。然后，工作 `DISPLAY` 会传递给构建过程。有关可能的参数，请参见 [`USES=display`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-display)。

```sh
USES=	display
```

### 6.10.5. 桌面条目

桌面条目（[Freedesktop 标准](https://standards.freedesktop.org/desktop-entry-spec/latest/)）提供了一种在安装新程序时自动调整桌面功能的方法，无需用户干预。例如，新安装的程序会自动出现在兼容的桌面环境的应用程序菜单中。桌面条目起源于 GNOME 桌面环境，但现在已经成为标准，并且可以与 KDE 和 Xfce 一起使用。这种自动化功能对用户来说是一个真正的便利，建议可以在桌面环境中使用的应用程序采用桌面条目。

#### 6.10.5.1. 使用预定义的 **.desktop** 文件

包含预定义 **\*.desktop** 文件的 Port 必须在 **pkg-plist** 中包含这些文件，并将它们安装到 **\$LOCALBASE/share/applications** 目录中。[`INSTALL_DATA` 宏](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#install-macros) 可以用来安装这些文件。

#### 6.10.5.2. 更新桌面数据库

如果 Port 的 **portname.desktop** 中包含 MimeType 条目，安装和卸载后必须更新桌面数据库。为此，定义 `USES= desktop-file-utils`。

#### 6.10.5.3. 使用 `DESKTOP_ENTRIES` 创建桌面条目

可以通过使用 `DESKTOP_ENTRIES` 来轻松为应用程序创建桌面条目。一个名为 **name.desktop** 的文件将自动创建、安装，并添加到 **pkg-plist** 中。语法如下：

```sh
DESKTOP_ENTRIES=	"NAME" "COMMENT" "ICON" "COMMAND" "CATEGORY" StartupNotify
```

可用的类别列表可以在 [Freedesktop 网站](https://standards.freedesktop.org/menu-spec/latest/apa.html) 上找到。`StartupNotify` 表示该应用程序是否支持启动通知。启动通知通常是一个图形指示器，如出现在鼠标指针、菜单或面板上的时钟，表示程序正在启动。兼容启动通知的程序在启动后会清除指示器，而不兼容的程序则永远不会清除该指示器（可能会让用户困惑和恼火），因此必须将 `StartupNotify` 设置为 `false`，以便根本不显示该指示器。

**示例：**

```sh
DESKTOP_ENTRIES=	"ToME" "基于 JRR Tolkien 作品的 Roguelike 游戏" \
			"${DATADIR}/xtra/graf/tome-128.png" \
			"tome -v -g" "Application;Game;RolePlaying;" \
			false
```

`DESKTOP_ENTRIES` 会安装到由 `DESKTOPDIR` 变量指向的目录中。`DESKTOPDIR` 默认值为 **\${PREFIX}/share/applications**

## 6.11. 使用 GNOME

### 6.11.1. 介绍

本章解释了 Port 中使用的 GNOME 框架。该框架可以大致分为基础组件、GNOME 桌面组件和一些简化 Port 维护者工作的特殊宏。

### 6.11.2. 使用 `USE_GNOME`

将此变量添加到 Port 中，允许使用在 **bsd.gnome.mk** 中定义的宏和组件。**bsd.gnome.mk** 中的代码会添加所需的构建时、运行时或库依赖项，或者处理特殊文件。在 FreeBSD 中，GNOME 应用程序使用 `USE_GNOME` 基础设施。包括所有所需的组件，作为以空格分隔的列表。`USE_GNOME` 组件分为以下几个虚拟列表：基本组件、GNOME 3 组件和遗留组件。如果 Port 只需要 GTK3 库，这是定义它的最简便方式：

```sh
USE_GNOME=	gtk30
```

`USE_GNOME` 组件会自动添加它们所需的依赖项。有关所有 `USE_GNOME` 组件的详细列表，以及它们暗含的其他组件和依赖项，请参阅 [GNOME 组件](https://docs.freebsd.org/en/books/porters-handbook/special/#gnome-components)。

以下是一个使用了本文件中概述的许多技术的 GNOME  Port 的 **Makefile** 示例。请将其作为创建新 Port 的指南。

```sh
PORTNAME=	regexxer
DISTVERSION=	0.10
CATEGORIES=	devel textproc gnome
MASTER_SITES=	GNOME

MAINTAINER=	kwm@FreeBSD.org
COMMENT=	Interactive tool for performing search and replace operations
WWW=		http://regexxer.sourceforge.net/

USES=		gettext gmake localbase:ldflags pathfix pkgconfig tar:xz
GNU_CONFIGURE=	yes
USE_GNOME=	gnomeprefix intlhack gtksourceviewmm3

GLIB_SCHEMAS=	org.regexxer.gschema.xml

.include <bsd.port.mk>
```

>**注意**
>
>  `USE_GNOME` 宏没有任何参数时不会向 Port 添加任何依赖项。`USE_GNOME` 不能在 **bsd.port.pre.mk** 之后设置。

### 6.11.3. 变量

本节解释了哪些宏是可用的，以及如何使用它们。就像在上述示例中使用的那样。更多关于 `GNOME 组件` 的详细说明可以在 [GNOME Components](https://docs.freebsd.org/en/books/porters-handbook/special/#gnome-components) 中找到。必须设置 `USE_GNOME` 才能使这些宏生效。

`GLIB_SCHEMAS`
列出 Port 安装的所有 glib 模式文件。该宏会将文件添加到 Port 的 **pkg-plist** 中，并在安装和卸载时处理这些文件的注册。

glib 模式文件以 XML 格式编写，并以 **gschema.xml** 扩展名结尾。它们安装在 **share/glib-2.0/schemas/** 目录下。这些模式文件包含所有应用程序配置值及其默认设置。实际由应用程序使用的数据库是通过 glib-compile-schema 构建的，该工具由 `GLIB_SCHEMAS` 宏运行。

```
GLIB_SCHEMAS=foo.gschema.xml
```

>**注意**
>
>  不要将 glib 模式文件添加到 **pkg-plist** 中。如果它们列在 **pkg-plist** 中，则不会被注册，应用程序可能无法正常工作。

`GCONF_SCHEMAS`
列出所有 gconf 模式文件。该宏会将模式文件添加到 Port 的 **pkg-plist** 中，并在安装和卸载时处理这些文件的注册。

GConf 是 GNOME 应用程序用于存储设置的基于 XML 的数据库。这些文件安装在 **etc/gconf/schemas** 目录下。该数据库由已安装的模式文件定义，这些文件用于生成 **%gconf.xml** 键文件。每个由 Port 安装的模式文件都必须在 **Makefile** 中有一个条目：

```
GCONF_SCHEMAS=my_app.schemas my_app2.schemas my_app3.schemas
```

>**注意**
>
> GConf 模式文件应该列在 `GCONF_SCHEMAS` 宏中，而不是 **pkg-plist** 中。如果它们列在 **pkg-plist** 中，则不会被注册，应用程序可能无法正常工作。 


`INSTALLS_OMF`
开源元数据框架（OMF）文件通常由 GNOME 2 应用程序使用。这些文件包含应用程序帮助文件的信息，并需要通过 ScrollKeeper/rarian 进行特殊处理。为了在从软件包安装 GNOME 应用程序时正确注册 OMF 文件，请确保 `omf` 文件列在 `pkg-plist` 中，并且 Port 的 **Makefile** 中定义了 `INSTALLS_OMF`：

```sh
INSTALLS_OMF=yes
```

设置后，**bsd.gnome.mk** 会自动扫描 **pkg-plist**，并为每个 **.omf** 文件添加适当的 `@exec` 和 `@unexec` 指令，以便在 OMF 注册数据库中进行跟踪。

## 6.12. GNOME 组件

要获得更多有关 GNOME  Port 的帮助，可以参考一些 [现有的 Port ](https://ports.freebsd.org/) 示例。如果需要更多帮助，访问 [FreeBSD GNOME 页面](https://www.freebsd.org/gnome/) 查找联系信息。这些组件分为当前正在使用的 GNOME 组件和旧版组件。如果组件支持参数，它们将列在说明中的括号中。第一个参数为默认值。如果组件默认添加到构建和运行依赖项中，则会显示“Both”。

| 组件                      | 相关程序                            | 说明                                                                                                       |
| ----------------------- | ------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `atk`                   | accessibility/atk               | 辅助工具包 (ATK)                                                                                              |
| `atkmm`                 | accessibility/atkmm             | atk 的 C++ 绑定                                                                                             |
| `cairo`                 | graphics/cairo                  | 支持跨设备输出的矢量图形库                                                                                            |
| `cairomm`               | graphics/cairomm                | cairo 的 C++ 绑定                                                                                           |
| `dconf`                 | devel/dconf                     | 配置数据库系统（同时用于构建和运行）                                                                                       |
| `evolutiondataserver3`  | databases/evolution-data-server | Evolution 集成邮件/PIM 套件的数据后端                                                                               |
| `gdkpixbuf2`            | graphics/gdk-pixbuf2            | GTK+ 图形库                                                                                                 |
| `glib20`                | devel/glib20                    | GNOME 核心库 `glib20`                                                                                       |
| `glibmm`                | devel/glibmm                    | glib20 的 C++ 绑定                                                                                          |
| `gnomecontrolcenter3`   | sysutils/gnome-control-center   | GNOME 3 控制中心                                                                                             |
| `gnomedesktop3`         | x11/gnome-desktop               | GNOME 3 桌面 UI 库                                                                                          |
| `gsound`                | audio/gsound                    | 用于播放系统声音的 GObject 库（同时用于构建和运行）                                                                           |
| `gtk-update-icon-cache` | graphics/gtk-update-icon-cache  | 来自 Gtk+ 工具包的 Gtk-update-icon-cache 工具                                                                    |
| `gtk20`                 | x11-toolkits/gtk20              | Gtk+ 2 工具包                                                                                               |
| `gtk30`                 | x11-toolkits/gtk30              | Gtk+ 3 工具包                                                                                               |
| `gtkmm20`               | x11-toolkits/gtkmm20            | gtk20 工具包的 C++ 绑定 2.0                                                                                    |
| `gtkmm24`               | x11-toolkits/gtkmm24            | gtk20 工具包的 C++ 绑定 2.4                                                                                    |
| `gtkmm30`               | x11-toolkits/gtkmm30            | gtk30 工具包的 C++ 绑定 3.0                                                                                    |
| `gtksourceview2`        | x11-toolkits/gtksourceview2     | 为 GtkTextView 添加语法高亮的组件                                                                                  |
| `gtksourceview3`        | x11-toolkits/gtksourceview3     | 为 GtkTextView 组件添加语法高亮的文本组件                                                                              |
| `gtksourceviewmm3`      | x11-toolkits/gtksourceviewmm3   | gtksourceview3 库的 C++ 绑定                                                                                 |
| `gvfs`                  | devel/gvfs                      | GNOME 虚拟文件系统                                                                                             |
| `intltool`              | textproc/intltool               | 国际化工具（另见 intlhack）                                                                                       |
| `introspection`         | devel/gobject-introspection     | 基本的 introspection 绑定和生成 introspection 绑定的工具。大多数时候，\:build 足够了，\:both/\:run 仅在应用程序使用 introspection 绑定时需要。 |
| `libgda5`               | databases/libgda5               | 提供对不同数据源的统一访问                                                                                            |
| `libgda5-ui`            | databases/libgda5-ui            | 来自 libgda5 库的 UI 库                                                                                       |
| `libgdamm5`             | databases/libgdamm5             | libgda5 库的 C++ 绑定                                                                                        |
| `libgsf`                | devel/libgsf                    | 处理结构化文件格式的可扩展 I/O 抽象                                                                                     |
| `librsvg2`              | graphics/librsvg2               | 用于解析和渲染 SVG 矢量图形文件的库                                                                                     |
| `libsigc++20`           | devel/libsigc++20               | C++ 的回调框架                                                                                                |
| `libxml++26`            | textproc/libxml++26             | libxml2 库的 C++ 绑定                                                                                        |
| `libxml2`               | textproc/libxml2                | XML 解析库（同时用于构建和运行）                                                                                       |
| `libxslt`               | textproc/libxslt                | XSLT C 库（同时用于构建和运行）                                                                                      |
| `metacity`              | x11-wm/metacity                 | GNOME 的窗口管理器                                                                                             |
| `nautilus3`             | x11-fm/nautilus                 | GNOME 文件管理器                                                                                              |
| `pango`                 | x11-toolkits/pango              | 用于国际化文本的布局和渲染的开源框架                                                                                       |
| `pangomm`               | x11-toolkits/pangomm            | pango 库的 C++ 绑定                                                                                          |
| `py3gobject3`           | devel/py3-gobject3              | Python 3，GObject 3.0 绑定                                                                                  |
| `pygobject3`            | devel/py-gobject3               | Python 2，GObject 3.0 绑定                                                                                  |
| `vte3`                  | x11-toolkits/vte3               | 带有改进的可访问性和国际化支持的终端组件                                                                                     |

## 6.13. GNOME 宏组件

| 组件              | 说明                                                                |
| --------------- | ----------------------------------------------------------------- |
| `gnomeprefix`   | 为 `configure` 提供一些默认的路径。                                          |
| `intlhack`      | 与 intltool 相同，但进行了修补，以确保使用 **share/locale/**。仅在 `intltool` 不足时使用。 |
| `referencehack` | 该宏用于帮助将 API 或参考文档拆分为自己的 Port 。                                        |

## 6.14. GNOME 旧版组件

| 组件                   | 相关程序                            | 说明                             |
| -------------------- | ------------------------------- | ------------------------------ |
| `atspi`              | accessibility/at-spi            | 辅助技术服务提供者接口                    |
| `esound`             | audio/esound                    | Enlightenment 声音包              |
| `gal2`               | x11-toolkits/gal2               | 来自 GNOME 2 gnumeric 的组件集合      |
| `gconf2`             | devel/gconf2                    | GNOME 2 的配置数据库系统               |
| `gconfmm26`          | devel/gconfmm26                 | gconf2 的 C++ 绑定                |
| `gdkpixbuf`          | graphics/gdk-pixbuf             | GTK+ 图形库                       |
| `glib12`             | devel/glib12                    | glib 1.2 核心库                   |
| `gnomedocutils`      | textproc/gnome-doc-utils        | GNOME 文档工具                     |
| `gnomemimedata`      | misc/gnome-mime-data            | GNOME 2 的 MIME 和应用程序数据库        |
| `gnomesharp20`       | x11-toolkits/gnome-sharp20      | GNOME 2 接口，用于 .NET 运行时         |
| `gnomespeech`        | accessibility/gnome-speech      | GNOME 2 的语音合成 API              |
| `gnomevfs2`          | devel/gnome-vfs                 | GNOME 2 虚拟文件系统                 |
| `gtk12`              | x11-toolkits/gtk12              | Gtk+ 1.2 工具包                   |
| `gtkhtml3`           | www/gtkhtml3                    | 轻量级 HTML 渲染/打印/编辑引擎            |
| `gtkhtml4`           | www/gtkhtml4                    | 轻量级 HTML 渲染/打印/编辑引擎            |
| `gtksharp20`         | x11-toolkits/gtk-sharp20        | 用于 .NET 运行时的 GTK+ 和 GNOME 2 接口 |
| `gtksourceview`      | x11-toolkits/gtksourceview      | 为 GtkTextView 添加语法高亮的组件        |
| `libartgpl2`         | graphics/libart\_lgpl           | 高性能 2D 图形库                     |
| `libbonobo`          | devel/libbonobo                 | GNOME 2 的组件和复合文档系统             |
| `libbonoboui`        | x11-toolkits/libbonoboui        | GNOME 2 libbonobo 组件的 GUI 前端   |
| `libgda4`            | databases/libgda4               | 提供对不同数据源的统一访问                  |
| `libglade2`          | devel/libglade2                 | GNOME 2 glade 库                |
| `libgnome`           | x11/libgnome                    | GNOME 2 库，GNU 桌面环境             |
| `libgnomecanvas`     | graphics/libgnomecanvas         | GNOME 2 图形库                    |
| `libgnomekbd`        | x11/libgnomekbd                 | GNOME 2 键盘共享库                  |
| `libgnomeprint`      | print/libgnomeprint             | GNOME 2 打印支持库                  |
| `libgnomeprintui`    | x11-toolkits/libgnomeprintui    | GNOME 2 打印支持库                  |
| `libgnomeui`         | x11-toolkits/libgnomeui         | GNOME 2 GUI 库，GNU 桌面环境         |
| `libgtkhtml`         | www/libgtkhtml                  | 轻量级 HTML 渲染/打印/编辑引擎            |
| `libgtksourceviewmm` | x11-toolkits/libgtksourceviewmm | GtkSourceView 的 C++ 绑定         |
| `libidl`             | devel/libIDL                    | 用于创建 CORBA IDL 文件树的库           |
| `libsigc++12`        | devel/libsigc++12               | C++ 的回调框架                      |
| `libwnck`            | x11-toolkits/libwnck            | 用于编写分页器和任务列表的库                 |
| `libwnck3`           | x11-toolkits/libwnck3           | 用于编写分页器和任务列表的库                 |
| `orbit2`             | devel/ORBit2                    | 高性能 CORBA ORB，支持 C 语言          |
| `pygnome2`           | x11-toolkits/py-gnome2          | GNOME 2 的 Python 绑定            |
| `pygobject`          | devel/py-gobject                | Python 2，GObject 2.0 绑定        |
| `pygtk2`             | x11-toolkits/py-gtk2            | GTK+ 的 Python 绑定               |
| `pygtksourceview`    | x11-toolkits/py-gtksourceview   | GtkSourceView 2 的 Python 绑定    |
| `vte`                | x11-toolkits/vte                | 带有改进的可访问性和国际化支持的终端组件           |

**表11 废弃组件：请勿使用**

| 组件              | 说明                                    |
| --------------- | ------------------------------------- |
| `pangox-compat` | `pangox-compat` 已被废弃，并从 pango 包中分离出来。 |

## 6.13. 使用 Qt

>**注意**
>
>有关 Qt 本身的一些 Port ，请参阅 [`qt-dist`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-qt-dist)。 

### 6.13.1. 需要 Qt 的 Port 

Ports 提供了对 Qt 5 和 Qt 6 的支持，分别通过 `USES+=qt:5` 和 `USES+=qt:6`。将 `USE_QT` 设置为所需的 Qt 组件（库、工具、插件）列表。

Qt 框架导出了一些可以供 Port 使用的变量，下面列出了一些：

**表12. 使用 Qt 的 Port 提供的变量**

| 变量             | 说明                  |
| -------------- | ------------------- |
| `QMAKE`        | `qmake` 可执行文件的完整路径。 |
| `LRELEASE`     | `lrelease` 工具的完整路径。 |
| `MOC`          | `moc` 的完整路径。        |
| `RCC`          | `rcc` 的完整路径。        |
| `UIC`          | `uic` 的完整路径。        |
| `QT_INCDIR`    | Qt 的包含目录。           |
| `QT_LIBDIR`    | Qt 的库路径。            |
| `QT_PLUGINDIR` | Qt 的插件路径。           |


### 6.13.2. 组件选择

每个 Qt 工具和库的依赖必须在 `USE_QT` 中指定。每个组件可以加上 `_build` 或 `_run` 后缀，后缀表示该组件的依赖是在构建时还是运行时。如果没有后缀，表示该组件在构建时和运行时都会依赖。通常，库组件没有后缀，工具组件通常带有 `_build` 后缀，而插件组件带有 `_run` 后缀。常用的组件如下所示（所有可用组件列在 `_USE_QT_ALL` 中，该变量由 **/usr/ports/Mk/Uses/qt.mk** 中的 `_USE_QT_COMMON` 和 `_USE_QT[56]_ONLY` 生成）：

**表 13. 可用的 Qt 库组件**

| 名称                 | 说明                                      |
| ------------------ | --------------------------------------- |
| `3d`               | Qt3D 模块                                 |
| `5compat`          | Qt 5 兼容模块，用于 Qt 6                       |
| `assistant`        | Qt 5 文档浏览器                              |
| `base`             | Qt 6 基础模块                               |
| `canvas3d`         | Qt canvas3d 模块                          |
| `charts`           | Qt 5 图表模块                               |
| `concurrent`       | Qt 多线程模块                                |
| `connectivity`     | Qt 连接性（蓝牙/NFC）模块                        |
| `core`             | Qt 核心非图形模块                              |
| `datavis3d`        | Qt 5 3D 数据可视化模块                         |
| `dbus`             | Qt D-Bus 进程间通信模块                        |
| `declarative`      | Qt 声明式框架，用于动态用户界面                       |
| `designer`         | Qt 5 图形用户界面设计器                          |
| `diag`             | 用于报告 Qt 及其环境的诊断信息的工具                    |
| `doc`              | Qt 5 文档                                 |
| `examples`         | Qt 5 示例源代码                              |
| `gamepad`          | Qt 5 游戏手柄模块                             |
| `graphicaleffects` | Qt Quick 图形效果                           |
| `gui`              | Qt 图形用户界面模块                             |
| `help`             | Qt 在线帮助集成模块                             |
| `l10n`             | Qt 本地化消息                                |
| `languageserver`   | Qt 6 语言服务器协议实现                          |
| `linguist`         | Qt 5 翻译工具                               |
| `location`         | Qt 位置模块                                 |
| `lottie`           | Qt 6 QML API，用于渲染图形和动画                  |
| `multimedia`       | Qt 音频、视频、广播和相机支持模块                      |
| `network`          | Qt 网络模块                                 |
| `networkauth`      | Qt 网络认证模块                               |
| `opengl`           | Qt 5 兼容的 OpenGL 支持模块                    |
| `paths`            | QStandardPaths 的命令行客户端                  |
| `phonon4`          | KDE 多媒体框架                               |
| `pixeltool`        | Qt 5 屏幕放大镜                              |
| `plugininfo`       | Qt 5 插件元数据转储工具                          |
| `positioning`      | Qt 6 定位 API，支持卫星、WiFi 或文本文件等数据源         |
| `printsupport`     | Qt 打印支持模块                               |
| `qdbus`            | Qt D-Bus 命令行接口                          |
| `qdbusviewer`      | Qt 5 D-Bus 图形界面                         |
| `qdoc`             | Qt 文档生成器                                |
| `qdoc-data`        | QDoc 配置文件                               |
| `qev`              | Qt QWidget 事件分析工具                       |
| `qmake`            | Qt Makefile 生成工具                        |
| `quickcontrols`    | 构建完整界面的 Qt Quick 控件集合                   |
| `quickcontrols2`   | 构建完整界面的 Qt Quick 控件集合                   |
| `remoteobjects`    | Qt 5 SXCML 模块                           |
| `script`           | Qt 4 兼容的脚本模块                            |
| `scripttools`      | Qt 脚本附加组件                               |
| `scxml`            | Qt 5 SXCML 模块                           |
| `sensors`          | Qt 传感器模块                                |
| `serialbus`        | Qt 工具，用于访问工业总线系统                        |
| `serialport`       | Qt 工具，用于访问串口                            |
| `shadertools`      | Qt 6 跨平台 Qt 着色器管道工具                     |
| `speech`           | Qt 5 辅助功能模块                             |
| `sql`              | Qt SQL 数据库集成模块                          |
| `sql-ibase`        | Qt InterBase/Firebird 数据库插件             |
| `sql-mysql`        | Qt MySQL 数据库插件                          |
| `sql-odbc`         | Qt Open Database Connectivity 插件        |
| `sql-pgsql`        | Qt PostgreSQL 数据库插件                     |
| `sql-sqlite2`      | Qt SQLite 2 数据库插件                       |
| `sql-sqlite3`      | Qt SQLite 3 数据库插件                       |
| `sql-tds`          | Qt TDS 数据库连接插件                          |
| `svg`              | Qt SVG 支持模块                             |
| `testlib`          | Qt 单元测试模块                               |
| `tools`            | Qt 6 各种工具                               |
| `translations`     | Qt 6 翻译模块                               |
| `uiplugin`         | Qt Designer 的自定义 Qt 控件插件接口              |
| `uitools`          | Qt Designer UI 表单支持模块                   |
| `virtualkeyboard`  | Qt 5 虚拟键盘模块                             |
| `wayland`          | Qt 5 Wayland 包装器                        |
| `webchannel`       | Qt 5 用于将 C++/QML 与 HTML/js 客户端集成的库      |
| `webengine`        | Qt 5 用于渲染 Web 内容的库                      |
| `webkit`           | QtWebKit，具有更现代的 WebKit 代码库              |
| `websockets`       | Qt WebSocket 协议实现                       |
| `websockets-qml`   | Qt WebSocket 协议实现（QML 绑定）               |
| `webview`          | Qt 组件，用于显示 Web 内容                       |
| `widgets`          | Qt C++ 控件模块                             |
| `x11extras`        | Qt 为 X11 系统提供的特定平台功能                    |
| `xml`              | Qt SAX 和 DOM 实现                         |
| `xmlpatterns`      | Qt 对 XPath、XQuery、XSLT 和 XML Schema 的支持 |

**表 14. 可用的 Qt 工具组件**

| 名称              | 说明                              |
| --------------- | ------------------------------- |
| `buildtools`    | 构建工具（`moc`，`rcc`），几乎每个 Qt 应用都需要 |
| `linguisttools` | 本地化工具：`lrelease`，`lupdate`      |
| `qmake`         | Makefile 生成工具/构建工具              |

**表 15. 可用的 Qt 插件组件**

| 名称             | 说明                        |
| -------------- | ------------------------- |
| `imageformats` | 支持 TGA、TIFF 和 MNG 图像格式的插件 |

**示例 21. 选择 Qt 5 组件**

在这个示例中，移植的应用程序使用 Qt 5 图形用户界面库、Qt 5 核心库、所有 Qt 5 代码生成工具以及 Qt 5 的 Makefile 生成器。由于 `gui` 库已经暗示了对核心库的依赖，因此不需要单独指定 `core`。Qt 5 代码生成工具 `moc`、`uic` 和 `rcc`，以及 Makefile 生成器 `qmake` 只在构建时需要，因此它们被指定为带有 `_build` 后缀：

```sh
USES=	qt:5
USE_QT=	gui buildtools_build qmake_build
```

### 6.13.3. 使用 `qmake`

如果应用程序提供了 qmake 项目文件（**\*.pro**），则定义 `USES= qmake` 和 `USE_QT`。`USES= qmake` 已经隐含了对 `qmake` 的构建依赖，因此可以省略 `USE_QT` 中的 qmake 组件。类似于 [CMake](https://docs.freebsd.org/en/books/porters-handbook/special/#using-cmake)，qmake 支持源代码外构建，可以通过指定 `outsource` 参数来启用（参见 [`USES= qmake` 示例](https://docs.freebsd.org/en/books/porters-handbook/special/#using-qmake-example)）。另请参见 [`USES qmake` 的可能参数](https://docs.freebsd.org/en/books/porters-handbook/special/#using-qmake-arguments)。

**表 16. `USES= qmake` 的可能参数**

| 变量             | 说明                                                                                                            |
| -------------- | ------------------------------------------------------------------------------------------------------------- |
| `no_configure` | 不添加配置目标。这由 `HAS_CONFIGURE=yes` 和 `GNU_CONFIGURE=yes` 隐含。当构建仅需要通过 `USES= qmake` 设置环境，但其他时候自行运行 `qmake` 时需要此参数。 |
| `no_env`       | 禁止修改配置和构建环境。仅当 `qmake` 用于配置软件且构建无法理解 `USES= qmake` 设置的环境时才需要。                                                 |
| `norecursive`  | 不向 `qmake` 传递 `-recursive` 参数。                                                                                |
| `outsource`    | 执行源代码外构建。                                                                                                     |

**表 17. 使用 `qmake` 的 Ports 变量**

| 变量                  | 说明                                                        |
| ------------------- | --------------------------------------------------------- |
| `QMAKE_ARGS`        | 传递给 `qmake` 二进制文件的 Port 特定 `qmake` 标志。                        |
| `QMAKE_ENV`         | 设置给 `qmake` 二进制文件的环境变量。默认值为 `${CONFIGURE_ENV}`。           |
| `QMAKE_SOURCE_PATH` | qmake 项目文件（**.pro**）的路径。如果请求源代码外构建，默认值为 `${WRKSRC}`，否则为空。 |

使用 `USES= qmake` 时，将应用以下设置：

```sh
CONFIGURE_ARGS+=	--with-qt-includes=${QT_INCDIR} \
			--with-qt-libraries=${QT_LIBDIR} \
			--with-extra-libs=${LOCALBASE}/lib \
			--with-extra-includes=${LOCALBASE}/include

CONFIGURE_ENV+=	QTDIR="${QT_PREFIX}" QMAKE="${QMAKE}" \
		MOC="${MOC}" RCC="${RCC}" UIC="${UIC}" \
		QMAKESPEC="${QMAKESPEC}"

PLIST_SUB+=	QT_INCDIR=${QT_INCDIR_REL} \
		QT_LIBDIR=${QT_LIBDIR_REL} \
		QT_PLUGINDIR=${QT_PLUGINDIR_REL}
```

某些配置脚本不支持上述参数。要禁止修改 `CONFIGURE_ENV` 和 `CONFIGURE_ARGS`，请设置 `USES= qmake:no_env`。

**示例 22. `USES= qmake` 示例**

这个示例展示了如何在 Qt 5 port 中使用 `qmake`：

```sh
USES=	qmake:outsource qt:5
USE_QT=	buildtools_build
```

Qt 应用程序通常是跨平台编写的，很多时候 X11/Unix 并不是它们的开发平台，这会导致一些问题，比如：

* *缺少附加的包含路径*。许多应用程序都支持系统托盘图标，但却没有在 X11 目录中查找包含文件和/或库。为了通过命令行将目录添加到 `qmake` 的包含和库搜索路径中，可以使用：

  ```sh
  QMAKE_ARGS+=	INCLUDEPATH+=${LOCALBASE}/include \
  		LIBS+=-L${LOCALBASE}/lib
  ```
* *错误的安装路径*。有时一些数据（如图标或 `.desktop` 文件）默认安装到 XDG 兼容应用程序不会扫描的目录中。[editors/texmaker](https://cgit.freebsd.org/ports/tree/editors/texmaker/) 就是一个例子 - 查看该 port 中 **files** 目录下的 **patch-texmaker.pro** 文件，了解如何在 `qmake` 项目文件中直接修复此问题。

## 6.14. 使用 KDE

### 6.14.1. KDE 变量定义

如果应用程序依赖于 KDE，请设置 `USES+=kde:5` 并将 `USE_KDE` 设置为所需组件的列表。可以使用 `_build` 和 `_run` 后缀来强制指定组件的依赖类型（例如，`baseapps_run`）。如果没有设置后缀，则会使用默认的依赖类型。要强制两种类型的依赖，可以分别添加该组件的两次（例如，`ecm_build ecm_run`）。可用的组件列出在以下位置（最新的组件也列在 **/usr/ports/Mk/Uses/kde.mk**）：

**表 18. 可用的 KDE 组件**

| 名称                     | 说明                                  |
| ---------------------- | ----------------------------------- |
| `activities`           | KF5 运行时和库，用于在不同活动中组织工作              |
| `activities-stats`     | KF5 活动统计                            |
| `activitymanagerd`     | 管理用户活动、跟踪使用模式的系统服务                  |
| `akonadi`              | KDE-Pim 的存储服务器                      |
| `akonadicalendar`      | Akonadi 日历集成                        |
| `akonadiconsole`       | Akonadi 管理和调试控制台                    |
| `akonadicontacts`      | 实现 Akonadi 联系人管理的库和守护进程             |
| `akonadiimportwizard`  | 从其他邮件客户端导入数据到 KMail                 |
| `akonadimime`          | 实现基本电子邮件处理的库和守护进程                   |
| `akonadinotes`         | 用于访问 MBox 格式的邮件存储的 KDE 库            |
| `akonadisearch`        | 实现 Akonadi 中搜索功能的库和守护进程             |
| `akregator`            | KDE 的 Feed 阅读器                      |
| `alarmcalendar`        | KDE 用于 KAlarm 提醒的 API               |
| `apidox`               | KF5 API 文档工具                        |
| `archive`              | KF5 提供处理归档格式的类库                     |
| `attica`               | KDE5 版本的开放协作服务 API 库                |
| `attica5`              | KDE5 版本的开放协作服务 API 库                |
| `auth`                 | KF5 对系统策略和认证功能的抽象                   |
| `baloo`                | KF5 框架，用于搜索和管理用户元数据                 |
| `baloo-widgets`        | BalooWidgets 库                      |
| `baloo5`               | KF5 框架，用于搜索和管理用户元数据                 |
| `blog`                 | KDE API 用于博客访问                      |
| `bookmarks`            | KF5 图书书签和 XBEL 格式库                  |
| `breeze`               | Plasma5 的 Breeze 视觉样式艺术品、样式和资产      |
| `breeze-gtk`           | Plasma5 Breeze 视觉样式的 Gtk 版          |
| `breeze-icons`         | KDE 的 Breeze 图标主题                   |
| `calendarcore`         | KDE 日历访问库                           |
| `calendarsupport`      | KDEPim 日历支持库                        |
| `calendarutils`        | KDE 用于访问日历的实用程序和用户界面功能              |
| `codecs`               | KF5 字符串操作库                          |
| `completion`           | KF5 文本自动完成帮助程序和小部件                  |
| `config`               | KF5 配置对话框小部件                        |
| `configwidgets`        | KF5 配置对话框小部件                        |
| `contacts`             | KDE API 管理联系人信息                     |
| `coreaddons`           | KF5 对 QtCore 的附加功能                  |
| `crash`                | KF5 库，用于处理应用程序崩溃分析和错误报告             |
| `dbusaddons`           | KF5 对 QtDBus 的附加功能                  |
| `decoration`           | Plasma5 用于创建窗口装饰的库                  |
| `designerplugin`       | KF5 在 Qt Designer/Creator 中集成框架小部件  |
| `discover`             | Plasma5 包管理工具                       |
| `dnssd`                | KF5 对系统 DNSSD 功能的抽象                 |
| `doctools`             | KF5 从 docbook 生成文档的工具               |
| `drkonqi`              | Plasma5 崩溃处理程序                      |
| `ecm`                  | CMake 的额外模块和脚本                      |
| `emoticons`            | KF5 库，用于转换表情符号                      |
| `eventviews`           | KDEPim 事件视图库                        |
| `filemetadata`         | KF5 库，用于提取文件元数据                     |
| `frameworkintegration` | KF5 工作区和跨框架集成插件                     |
| `gapi`                 | 基于 KDE 的库，用于访问谷歌服务                  |
| `globalaccel`          | KF5 库，用于添加对全局工作区快捷键的支持              |
| `grantlee-editor`      | Grantlee 主题的编辑器                     |
| `grantleetheme`        | KDE PIM Grantlee 主题                 |
| `gravatar`             | Gravatar 支持库                        |
| `guiaddons`            | KF5 对 QtGui 的附加功能                   |
| `holidays`             | KDE 日历假期库                           |
| `hotkeys`              | Plasma5 热键库                         |
| `i18n`                 | KF5 高级国际化框架                         |
| `iconthemes`           | KF5 库，用于处理应用程序中的图标                  |
| `identitymanagement`   | KDE PIM 身份管理                        |
| `idletime`             | KF5 库，用于监控用户活动                      |
| `imap`                 | KDE API 用于 IMAP 支持                  |
| `incidenceeditor`      | KDEPim 事件编辑器库                       |
| `infocenter`           | Plasma5 提供系统信息的实用工具                 |
| `init`                 | KF5 进程启动器，用于加速 KDE 应用程序启动           |
| `itemmodels`           | KF5 用于 Qt 模型/视图系统的小部件模型             |
| `itemviews`            | KF5 用于 Qt 模型/视图的小部件附加功能             |
| `jobwidgets`           | KF5 小部件，用于跟踪 KJob 实例                |
| `js`                   | KF5 提供 ECMAScript 解释器的库             |
| `jsembed`              | KF5 库，用于将 JavaScript 对象绑定到 QObjects |
| `kaddressbook`         | KDE 联系人管理器                          |
| `kalarm`               | 个人提醒调度程序                            |
| `kalarm`               | 个人提醒调度程序                            |
| `kate`                 | KDE 系统的基本编辑器框架                      |
| `kcmutils`             | KF5 用于处理 KCModules 的实用程序            |
| `kde-cli-tools`        | Plasma5 非交互式系统工具                    |
| `kde-gtk-config`       | Plasma5 GTK2 和 GTK3 配置器             |
| `kdeclarative`         | KF5 提供 QML 和 KDE 框架集成的库             |
| `kded`                 | KF5 可扩展的守护进程，用于提供系统级服务              |
| `kdelibs4support`      | KF5 用于从 KDELibs4 迁移的支持              |
| `kdepim-addons`        | KDE PIM 附加功能                        |
| `kdepim-apps-libs`     | KDE PIM 邮件相关库                       |
| `kdepim-runtime5`      | KDE PIM 工具和服务                       |
| `kdeplasma-addons`     | Plasma5 改善 Plasma 体验的附加功能           |
| `kdesu`                | KF5 与 `su` 集成，以获取提升的权限              |
| `kdewebkit`            | KF5 提供 QtWebKit 集成的库                |
| `kgamma5`              | Plasma5 显示器的伽玛设置                    |
| `khtml`                | KF5 KTHML 渲染引擎                      |
| `kimageformats`        | KF5 库，用于支持额外的图像格式                   |
| `kio`                  | KF5 资源和网络访问抽象                       |
| `kirigami2`            | 基于 QtQuick 的组件集                     |
| `kitinerary`           | 用于旅行预订信息的数据模型和提取系统                  |
| `kmail`                | KDE 邮件客户端                           |
| `kmail`                | KDE 邮件客户端                           |
| `kmail-account-wizard` | KDE 邮件帐户向导                          |
| `kmenuedit`            | Plasma5 菜单编辑器                       |
| `knotes`               | 弹出便签                                |
| `kontact`              | KDE 个人信息管理器                         |
| `kontact`              | KDE 个人信息管理器                         |
| `kontactinterface`     | 将 KParts 嵌入到 Kontact 中的 KDE 粘合剂     |
| `korganizer`           | 日历和调度程序                             |
| `kpimdav`              | 实现 DAV 协议的 KJobs                    |
| `kpkpass`              | 处理 Apple Wallet pass 文件的库           |
| `kross`                | KF5 多语言应用程序脚本                       |
| `kscreen`              | Plasma5 屏幕管理库                       |
| `kscreenlocker`        | Plasma5 安全锁屏架构                      |
| `ksmtp`                | 基于作业的库，用于通过 SMTP 服务器发送邮件            |
| `ksshaskpass`          | Plasma5 ssh-add 前端                  |
| `ksysguard`            | Plasma5 用于跟踪和控制正在运行进程的实用工具          |
| `kwallet-pam`          | Plasma5 KWallet PAM 集成              |
| `kwayland-integration` | 用于基于 Wayland 桌面的集成插件                |
| `kwin`                 | Plasma5 窗口管理器                       |
| `kwrited`              | Plasma5 守护进程，用于监听墙和写消息              |
| `ldap`                 | KDE LDAP 访问 API                     |
| `libkcddb`             | KDE CDDB 库                          |
| `libkcompactdisc`      | 用于与音频 CD 接口的 KDE 库                  |
| `libkdcraw`            | 用于 KDE 的 LibRaw 接口                  |
| `libkdegames`          | KDE 游戏使用的库                          |
| `libkdepim`            | KDE PIM 库                           |
| `libkeduvocdocument`   | 读取和写入词汇文件的库                         |
| `libkexiv2`            | 用于 KDE 的 Exiv2 库接口                  |
| `libkipi`              | KDE 图像插件接口                          |
| `libkleo`              | KDE 证书管理器                           |
| `libksane`             | KDE 用于 SANE 库的接口                    |
| `libkscreen`           | Plasma5 屏幕管理库                       |
| `libksieve`            | KDEPim 的 Sieve 库                    |
| `libksysguard`         | Plasma5 库，用于跟踪和控制正在运行的进程            |
| `mailcommon`           | KDEPim 的通用库                         |
| `mailimporter`                | 将 mbox 文件导入到 KMail             |
| `mailtransport`               | 用于管理邮件传输的 KDE 库                |
| `marble`                      | KDE 的虚拟地球和世界地图                 |
| `mbox`                        | 用于访问 MBox 格式邮件存储的 KDE 库        |
| `mbox-importer`               | 将 mbox 文件导入到 KMail             |
| `mediaplayer`                 | KF5 媒体播放器功能的插件接口               |
| `messagelib`                  | 处理消息的库                         |
| `milou`                       | Plasma5 用于搜索的 Plasmoid         |
| `mime`                        | 处理 MIME 数据的库                   |
| `newstuff`                    | KF5 库，用于从网络下载应用程序资产            |
| `notifications`               | KF5 系统通知的抽象                    |
| `notifyconfig`                | 用于 KNotify 的 KF5 配置系统          |
| `okular`                      | KDE 通用文档查看器                    |
| `oxygen`                      | Plasma5 Oxygen 样式              |
| `oxygen-icons5`               | KDE 的 Oxygen 图标主题              |
| `package`                     | 用于加载和安装包的 KF5 库                |
| `parts`                       | KF5 面向文档的插件系统                  |
| `people`                      | 提供访问联系人功能的 KF5 库               |
| `pim-data-exporter`           | 导入和导出 KDE PIM 设置               |
| `pimcommon`                   | KDEPim 的通用库                    |
| `pimtextedit`                 | 用于 PIM 特定文本编辑的 KDE 库           |
| `plasma-browser-integration`  | Plasma5 组件，用于将浏览器集成到桌面         |
| `plasma-desktop`              | Plasma5 Plasma 桌面              |
| `plasma-framework`            | 用于编写用户界面的 KF5 插件基础 UI 运行时      |
| `plasma-integration`          | 为 Plasma 工作区提供 Qt 平台主题集成插件     |
| `plasma-pa`                   | Plasma5 的 PulseAudio 混音器       |
| `plasma-sdk`                  | 用于 Plasma 开发的 Plasma5 应用程序     |
| `plasma-workspace`            | Plasma5 Plasma 工作空间            |
| `plasma-workspace-wallpapers` | Plasma5 壁纸                     |
| `plotting`                    | KF5 轻量级绘图框架                    |
| `polkit-kde-agent-1`          | 提供 Polkit 认证 UI 的 Plasma5 守护进程 |
| `powerdevil`                  | Plasma5 用于管理电源消耗设置的工具          |
| `prison`                      | 用于生成条形码的 API                   |
| `pty`                         | KF5 pty 抽象                     |
| `purpose`                     | 提供特定目的的可用操作                    |
| `qqc2-desktop-style`          | 用于 KDE 的 Qt QuickControl2 样式   |
| `runner`                      | KF5 并行化查询系统                    |
| `service`                     | KF5 高级插件和服务内省                  |
| `solid`                       | KF5 硬件集成和检测                    |
| `sonnet`                      | 基于插件的 KF5 拼写检查库                |
| `syndication`                 | KDE RSS 源处理库                   |
| `syntaxhighlighting`          | KF5 结构化文本和代码的语法高亮引擎            |
| `systemsettings`              | Plasma5 系统设置                   |
| `texteditor`                  | 可嵌入的 KF5 高级文本编辑器               |
| `textwidgets`                 | KF5 高级文本编辑小部件                  |
| `threadweaver`                | KF5 QtDBus 的附加组件               |
| `tnef`                        | KDE API 用于处理 TNEF 数据           |
| `unitconversion`              | KF5 单位转换库                      |
| `user-manager`                | Plasma5 用户管理器                  |
| `wallet`                      | KF5 安全且统一的用户密码容器               |
| `wayland`                     | KF5 Wayland 客户端和服务器库封装         |
| `widgetsaddons`               | KF5 QtWidgets 的附加组件            |
| `windowsystem`                | KF5 用于访问窗口系统的库                 |
| `xmlgui`                      | KF5 用户可配置的主窗口                  |
| `xmlrpcclient`                | KF5 与 XMLRPC 服务的交互             |

**示例 23. `USE_KDE` 示例**

这是一个简单的 KDE Port 示例。`USES= cmake` 指示 Port 使用 CMake，CMake 是一个被 KDE 项目广泛使用的配置工具（详细用法请参见 [使用 `cmake`](https://docs.freebsd.org/en/books/porters-handbook/special/#using-cmake)）。`USE_KDE` 会引入对 KDE 库的依赖。所需的 KDE 组件和其他依赖项可以通过配置日志来确定。`USE_KDE` 并不意味着 `USE_QT`。如果 Port 需要一些 Qt 组件，请在 `USE_QT` 中指定它们。

```sh
USES=		cmake kde:5 qt:5
USE_KDE=	ecm
USE_QT=		core buildtools_build qmake_build
```

## 6.15. 使用 LXQt

依赖 LXQt 的应用程序应设置 `USES+= lxqt` 并将 `USE_LXQT` 设置为所需组件的列表，如下表所示。

**表 19. 可用的 LXQt 组件**

| 名称           | 说明                            |
| ------------ | ----------------------------- |
| `buildtools` | 用于额外 CMake 模块的辅助工具            |
| `libfmqt`    | Libfm Qt 绑定                   |
| `lxqt`       | LXQt 核心库                      |
| `qtxdg`      | freedesktop.org XDG 规范的 Qt 实现 |

**示例 24. `USE_LXQT` 示例**

这是一个简单的示例，`USE_LXQT` 会添加 LXQt 库的依赖。所需的 LXQt 组件和其他依赖项可以通过配置日志来确定。

```
USES=	cmake lxqt qt:5 tar:xz
USE_QT=		core dbus widgets buildtools_build qmake_build
USE_LXQT=	buildtools libfmqt
```

## 6.16. 使用 Java

### 6.16.1. 变量定义

如果 Port 需要 Java™ 开发工具包（JDK™）来构建、运行或甚至提取 distfile，则需要定义 `USE_JAVA`。

Ports 中有多个 JDK，来自不同的供应商，并且有多个版本。如果 Port 必须使用特定版本，请使用 `JAVA_VERSION` 变量指定它。当前版本是 [java/openjdk18](https://cgit.freebsd.org/ports/tree/java/openjdk18/)，同时还提供有 [java/openjdk17](https://cgit.freebsd.org/ports/tree/java/openjdk17/)、[java/openjdk16](https://cgit.freebsd.org/ports/tree/java/openjdk16/)、[java/openjdk15](https://cgit.freebsd.org/ports/tree/java/openjdk15/)、[java/openjdk14](https://cgit.freebsd.org/ports/tree/java/openjdk14/)、[java/openjdk13](https://cgit.freebsd.org/ports/tree/java/openjdk13/)、[java/openjdk12](https://cgit.freebsd.org/ports/tree/java/openjdk12/)、[java/openjdk11](https://cgit.freebsd.org/ports/tree/java/openjdk11/)、[java/openjdk8](https://cgit.freebsd.org/ports/tree/java/openjdk8/) 和 [java/openjdk7](https://cgit.freebsd.org/ports/tree/java/openjdk7/) 可用。

**表 20. 使用 Java 的 Ports 可以设置的变量**

| 变量             | 含义                                                                                     |
| -------------- | -------------------------------------------------------------------------------------- |
| `USE_JAVA`     | 定义后，其他变量才会生效                                                                           |
| `JAVA_VERSION` | 空格分隔的适用于该 Port 的 Java 版本列表。可选的 `+` 表示版本范围（允许值：`8[+] 11[+] 17[+] 18[+] 19[+] 20[+] 21[+]`）。 |
| `JAVA_OS`      | 空格分隔的适用于该 Port 的 JDK 操作系统列表（允许值：`native linux`）。                                           |
| `JAVA_VENDOR`  | 空格分隔的适用于该 Port 的 JDK 供应商列表（允许值：`openjdk oracle`）。                                          |
| `JAVA_BUILD`   | 设置后，将选定的 JDK  Port 添加到构建依赖项中。                                                              |
| `JAVA_RUN`     | 设置后，将选定的 JDK  Port 添加到运行时依赖项中。                                                             |
| `JAVA_EXTRACT` | 设置后，将选定的 JDK  Port 添加到提取依赖项中。                                                              |

下面是设置 `USE_JAVA` 后， Port 将接收到的所有设置列表。

**表 21. 使用 Java 的 Ports 提供的变量**

| 变量                             | 值                                                                                                |
| ------------------------------ | ------------------------------------------------------------------------------------------------ |
| `JAVA_PORT`                    | JDK  Port 的名称（例如，`java/openjdk6`）。                                                                   |
| `JAVA_PORT_VERSION`            | JDK  Port 的完整版本（例如，`1.6.0`）。只需要该版本号的前两位数字，使用 `${JAVA_PORT_VERSION:C/^([0-9])\.([0-9])(.*)$/\1.\2/}`。 |
| `JAVA_PORT_OS`                 | JDK  Port 使用的操作系统（例如，`'native'`）。                                                                    |
| `JAVA_PORT_VENDOR`             | JDK  Port 的供应商（例如，`'openjdk'`）。                                                                      |
| `JAVA_PORT_OS_DESCRIPTION`     | JDK  Port 使用的操作系统说明（例如，`'Native'`）。                                                                  |
| `JAVA_PORT_VENDOR_DESCRIPTION` | JDK  Port 供应商的说明（例如，`'OpenJDK BSD Porting Team'`）。                                                   |
| `JAVA_HOME`                    | JDK 安装目录的路径（例如，**'/usr/local/openjdk6'**）。                                                       |
| `JAVAC`                        | 要使用的 Java 编译器路径（例如，**'/usr/local/openjdk6/bin/javac'**）。                                         |
| `JAR`                          | 要使用的 `jar` 工具的路径（例如，**'/usr/local/openjdk6/bin/jar'** 或 **'/usr/local/bin/fastjar'**）。           |
| `APPLETVIEWER`                 | `appletviewer` 工具的路径（例如，**'/usr/local/openjdk6/bin/appletviewer'**）。                             |
| `JAVA`                         | `java` 可执行文件的路径。用于执行 Java 程序（例如，**'/usr/local/openjdk6/bin/java'**）。                             |
| `JAVADOC`                      | `javadoc` 工具程序的路径。                                                                               |
| `JAVAH`                        | `javah` 程序的路径。                                                                                   |
| `JAVAP`                        | `javap` 程序的路径。                                                                                   |
| `JAVA_KEYTOOL`                 | `keytool` 工具程序的路径。                                                                               |
| `JAVA_N2A`                     | `native2ascii` 工具的路径。                                                                            |
| `JAVA_POLICYTOOL`              | `policytool` 程序的路径。                                                                              |
| `JAVA_SERIALVER`               | `serialver` 工具程序的路径。                                                                             |
| `RMIC`                         | RMI 存根/骨架生成器 `rmic` 的路径。                                                                         |
| `RMIREGISTRY`                  | RMI 注册程序 `rmiregistry` 的路径。                                                                      |
| `RMID`                         | RMI 守护程序 `rmid` 的路径。                                                                             |
| `JAVA_CLASSES`                 | 包含 JDK 类文件的存档路径，**\${JAVA\_HOME}/jre/lib/rt.jar**。                                               |

使用 `java-debug` make 目标可以获取调试 Port 所需的信息。它将显示之前列出许多变量的值。

此外，定义了这些常量，以便所有 Java  Port 可以一致地安装：

**表 22. 为使用 Java 的 Ports 定义的常量**

| 常量             | 值                                                            |
| -------------- | ------------------------------------------------------------ |
| `JAVASHAREDIR` | 与 Java 相关的所有内容的基本目录。默认值：**\${PREFIX}/share/java**。           |
| `JAVAJARDIR`   | 安装 JAR 文件的目录。默认值：**\${JAVASHAREDIR}/classes**。               |
| `JAVALIBDIR`   | 其他 Port 安装的 JAR 文件所在目录。默认值：**\${LOCALBASE}/share/java/classes**。 |

相关条目在 `PLIST_SUB`（文档参见 [基于 Make 变量修改 pkg-plist](https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-sub)）和 `SUB_LIST` 中定义。

### 6.16.2. 使用 Ant 构建

当 Port 需要使用 Apache Ant 构建时，它必须定义 `USE_ANT`。Ant 被认为是子构建命令。当 Port 未定义 `do-build` 目标时，系统将设置一个默认的目标，该目标根据 `MAKE_ENV`、`MAKE_ARGS` 和 `ALL_TARGET` 运行 Ant。这与 [构建机制](https://docs.freebsd.org/en/books/porters-handbook/special/#building) 中文档说明的 `USES= gmake` 机制类似。

### 6.16.3. 最佳实践

在移植 Java 库时， Port 需要将 JAR 文件安装到 **\${JAVAJARDIR}**，并将其他所有内容安装到 **\${JAVASHAREDIR}/\${PORTNAME}** 目录下（文档除外，见下文）。为了减小打包文件的大小，应在 **Makefile** 中直接引用 JAR 文件。可以使用如下语句（其中 **myport.jar** 是 Port 安装的 JAR 文件名称）：

```sh
PLIST_FILES+=	${JAVAJARDIR}/myport.jar
```

在移植 Java 应用程序时， Port 通常会将所有内容安装到单一目录下（包括其 JAR 依赖项）。在这种情况下，强烈推荐使用 **\${JAVASHAREDIR}/\${PORTNAME}**。具体是否将额外的 JAR 依赖项安装到此目录下，还是使用已经安装的 JAR（来自 **\${JAVAJARDIR}**），由移植者决定。

当移植需要应用服务器（如 [www/tomcat7](https://cgit.freebsd.org/ports/tree/www/tomcat7/)）来运行服务的 Java™ 应用程序时，供应商通常会分发一个 **.war** 文件。**.war** 是 Web 应用程序存档文件，在应用程序调用时会被解压。避免将 **.war** 添加到 **pkg-plist** 中，这不是最佳实践。应用服务器会展开 war 存档，但如果 Port 被移除，它不会正确清理该存档。更好的做法是先解压存档文件，再安装文件，最后将这些文件添加到 **pkg-plist** 中。

```sh
TOMCATDIR=	${LOCALBASE}/apache-tomcat-7.0
WEBAPPDIR=	myapplication

post-extract:
	@${MKDIR} ${WRKDIR}/${PORTDIRNAME}
	@${TAR} xf ${WRKDIR}/myapplication.war -C ${WRKDIR}/${PORTDIRNAME}

do-install:
	cd ${WRKDIR} && \
	${INSTALL} -d -o ${WWWOWN} -g ${WWWGRP} ${TOMCATDIR}/webapps/${PORTDIRNAME}
	cd ${WRKDIR}/${PORTDIRNAME} && ${COPYTREE_SHARE} \* ${WEBAPPDIR}/${PORTDIRNAME}
```

无论 Port 是库还是应用程序，额外的文档都会安装在与其他 Port 相同的位置。`javadoc` 工具已知会根据所使用的 JDK 版本生成不同的文件集。因此，对于不强制使用特定 JDK 的 Port ，指定打包清单（**pkg-plist**）是一个复杂的任务。这也是为什么强烈建议移植者使用 `PORTDOCS` 的原因之一。此外，即使可以预测由 `javadoc` 生成的文件集，生成的 **pkg-plist** 的大小也支持使用 `PORTDOCS`。

`DATADIR` 的默认值是 **\${PREFIX}/share/\${PORTNAME}**。对于 Java  Port ，最好将 `DATADIR` 重写为 **\${JAVASHAREDIR}/\${PORTNAME}**。确实，`DATADIR` 会自动添加到 `PLIST_SUB` 中（文档参见 [基于 Make 变量修改 pkg-plist](https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-sub)），因此可以在 **pkg-plist** 中直接使用 `%%DATADIR%%`。

至于从源代码构建 Java  Port 还是直接从二进制分发安装，目前没有明确的政策。然而，来自 [FreeBSD Java 项目](https://www.freebsd.org/java/) 的成员鼓励移植者在任务简单时从源代码构建他们的 Port 。

本节介绍的所有功能都在 **bsd.java.mk** 中实现。如果 Port 需要更复杂的 Java 支持，请首先查看 [bsd.java.mk Git 日志](https://cgit.freebsd.org/ports/tree/Mk/bsd.java.mk)，因为通常需要一些时间才能记录最新功能。然后，如果缺少的支持对许多其他 Java  Port 有益，请随时在 freebsd-java 上讨论。

尽管存在一个 `java` 类别用于 PR，但它指的是来自 FreeBSD Java 项目的 JDK 移植工作。因此，除非问题与 JDK 实现或 **bsd.java.mk** 相关，否则应像处理其他 Port 一样，在 `ports` 类别中提交 Java  Port 。

类似地，对于 Java  Port 的 `CATEGORIES`，有一个定义明确的政策，详情请见 [分类](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-categories)。

## 6.17. Web 应用程序、Apache 和 PHP

### 6.17.1. Apache

| 变量                 | 说明                                                                                                                                                                               |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `USE_APACHE`       |  Port 需要 Apache。可能的值：`yes`（获取任意版本），`22`，`24`，`22-24`，`22+` 等。默认的 Apache 版本是 `22`。更多细节请参见 **ports/Mk/bsd.apache.mk** 以及 [wiki.freebsd.org/Apache/](https://wiki.freebsd.org/Apache/)。 |
| `APXS`             | `apxs` 可执行文件的完整路径，可以在 Port 中重写。                                                                                                                                                      |
| `HTTPD`            | `httpd` 可执行文件的完整路径，可以在 Port 中重写。                                                                                                                                                     |
| `APACHE_VERSION`   | 当前安装的 Apache 版本（只读变量）。该变量只有在包含 **bsd.port.pre.mk** 后才能使用。可能的值：`22`，`24`。                                                                                                         |
| `APACHEMODDIR`     | Apache 模块的目录。此变量会在 **pkg-plist** 中自动展开。                                                                                                                                          |
| `APACHEINCLUDEDIR` | Apache 头文件的目录。此变量会在 **pkg-plist** 中自动展开。                                                                                                                                         |
| `APACHEETCDIR`     | Apache 配置文件的目录。此变量会在 **pkg-plist** 中自动展开。                                                                                                                                        |

### 6.17.2. Web 应用程序

Web 应用程序必须安装到 **PREFIX/www/appname** 目录下。这个路径可以在 **Makefile** 和 **pkg-plist** 中作为 `WWWDIR` 使用，相对于 `PREFIX` 的路径可以在 **Makefile** 中作为 `WWWDIR_REL` 使用。

Web 服务器进程的用户和组信息可以通过 `WWWOWN` 和 `WWWGRP` 获取，以便在需要更改某些文件的所有权时使用。两者的默认值为 `www`。如果 Port 需要不同的值，可以使用 `WWWOWN?= myuser` 和 `WWWGRP?= mygroup`。这允许用户轻松地覆盖这些值。

>**重要**
>
>谨慎使用 `WWWOWN` 和 `WWWGRP`。请记住，Web 服务器可以写入的每个文件都是一个潜在的安全风险。


除非 Web 应用程序明确需要 Apache，否则不要依赖 Apache。请尊重用户可能希望在 Apache 以外的 Web 服务器上运行 Web 应用程序。

### 6.17.3. PHP

PHP Web 应用程序通过 `USES=php` 声明对 PHP 的依赖。更多信息请参考 [`php`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-php)。

### 6.17.4. PEAR 模块

移植 PEAR 模块是一个非常简单的过程。

在 Port 的 **Makefile** 中添加 `USES=pear`。该框架将把相关文件安装到正确的位置，并在安装时自动生成 plist。

**示例 25. PEAR 类的 Makefile 示例**

```sh
PORTNAME=       Date
DISTVERSION=	1.4.3
CATEGORIES=	devel www pear

MAINTAINER=	someone@example.org
COMMENT=	PEAR Date and Time Zone Classes
WWW=		https://pear.php.net/package/Date/

USES=	pear

.include <bsd.port.mk>
```

>**技巧**
>
>PEAR 模块将使用 [PHP flavors](https://docs.freebsd.org/en/books/porters-handbook/flavors/#flavors-auto-php) 自动进行 flavor 化。 


>**注意**
>
> 如果使用非默认的 `PEAR_CHANNEL`，则构建时和运行时的依赖项将自动添加。


>**重要**
>
>PEAR 模块不需要定义 `PKGNAMESUFFIX`，该值会自动使用 `PEAR_PKGNAMEPREFIX` 填充。如果 Port 需要添加 `PKGNAMEPREFIX`，则必须同时使用 `PEAR_PKGNAMEPREFIX` 来区分不同的 flavor。

#### 6.17.4.1. Horde 模块

同样，移植 Horde 模块是一个简单的过程。

在 Port 的 **Makefile** 中添加 `USES=horde`。该框架将把相关文件安装到正确的位置，并在安装时自动生成 plist。

`USE_HORDE_BUILD` 和 `USE_HORDE_RUN` 变量可以用于添加构建时和运行时对其他 Horde 模块的依赖。有关可用模块的完整列表，请参见 **Mk/Uses/horde.mk**。

**示例 26. Horde 模块的 Makefile 示例**

```sh
PORTNAME=	Horde_Core
DISTVERSION=	2.14.0
CATEGORIES=	devel www pear

MAINTAINER=	horde@FreeBSD.org
COMMENT=	Horde Core Framework libraries
WWW=		https://pear.horde.org/

OPTIONS_DEFINE=	KOLAB SOCKETS
KOLAB_DESC=	Enable Kolab server support
SOCKETS_DESC=	Depend on sockets PHP extension

USES=	horde
USE_PHP=	session

USE_HORDE_BUILD=	Horde_Role
USE_HORDE_RUN=	Horde_Role Horde_History Horde_Pack \
		Horde_Text_Filter Horde_View

KOLAB_USE=	HORDE_RUN=Horde_Kolab_Server,Horde_Kolab_Session
SOCKETS_USE=	PHP=sockets

.include <bsd.port.mk>
```

>**技巧**
>
> 由于 Horde 模块也是 PEAR 模块，它们将使用 [PHP flavors](https://docs.freebsd.org/en/books/porters-handbook/flavors/#flavors-auto-php) 自动进行 flavor 化。 

### 6.18. 使用 Python

Ports 支持多个 Python 版本的并行安装。 Port 必须根据用户可设置的 `PYTHON_VERSION` 使用正确的 `python` 解释器。最重要的是，这意味着将脚本中的 `python` 可执行文件路径替换为 `PYTHON_CMD` 的值。

将文件安装到 `PYTHON_SITELIBDIR` 下的 Port 必须使用 `pyXY-` 包名前缀，这样它们的包名就包含了 Python 的版本号。

```sh
PKGNAMEPREFIX=	${PYTHON_PKGNAMEPREFIX}
```

**表 25. 使用 Python 的 Port 最有用的变量**

| `USES=python`             | 该 Port 需要 Python。可以通过类似 `3.10+` 的值指定最小要求的版本。也可以通过使用短横线分隔两个版本号来指定版本范围，如 `USES=python:3.8-3.9`。请注意，`USES=python` *不* 包含 Python 2.7，需明确要求使用 `USES=python:2.7+`。                           |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `USE_PYTHON=distutils`    | 使用 Python distutils 进行配置、编译和安装。当 Port 带有 **setup.py** 时，必需设置此选项。这会覆盖 `do-build` 和 `do-install` 目标，并且如果没有定义 `GNU_CONFIGURE`，也可能会覆盖 `do-configure`。此外，它还暗示 `USE_PYTHON=flavors`。         |
| `USE_PYTHON=autoplist`    | 自动创建包装清单。此选项还需要设置 `USE_PYTHON=distutils`。                                                                                                                                          |
| `USE_PYTHON=concurrent`   | 该 Port 将使用独特的前缀，通常是 `PYTHON_PKGNAMEPREFIX`，用于某些目录，如 `EXAMPLESDIR` 和 `DOCSDIR`，并且将为二进制文件和脚本添加一个后缀，即 `PYTHON_VER` 中的 Python 版本号。这样可以同时安装多个 Python 版本的 Port ，避免冲突文件的安装。                       |
| `USE_PYTHON=flavors`      | 该 Port 不使用 distutils，但仍然支持多个 Python 版本。`FLAVORS` 将被设置为支持的 Python 版本。有关更多信息，请参见 [使用 Python 和 Flavors](https://docs.freebsd.org/en/books/porters-handbook/flavors/#flavors-auto-python)。 |
| `USE_PYTHON=optsuffix`    | 如果当前 Python 版本不是默认版本，则该 Port 将获得 `PKGNAMESUFFIX=${PYTHON_PKGNAMESUFFIX}`。仅在使用 flavors 时有用。                                                                                             |
| `PYTHON_PKGNAMEPREFIX`    | 用作 `PKGNAMEPREFIX` 来区分不同 Python 版本的包。例如：`py27-`。                                                                                                                                   |
| `PYTHON_SITELIBDIR`       | 站点包树的位置，包含 Python 的安装路径（通常是 `LOCALBASE`）。在安装 Python 模块时，`PYTHON_SITELIBDIR` 非常有用。                                                                                                  |
| `PYTHONPREFIX_SITELIBDIR` | `PYTHON_SITELIBDIR` 的 PREFIX 清理版本。始终在 **pkg-plist** 中使用 `%%PYTHON_SITELIBDIR%%`。`%%PYTHON_SITELIBDIR%%` 的默认值为 `lib/python%%PYTHON_VERSION%%/site-packages`。                        |
| `PYTHON_CMD`              | Python 解释器命令行，包括版本号。                                                                                                                                                               |

**表 26. Python 模块依赖帮助程序**

| `PYNUMERIC`      | 数值扩展的依赖项                                                                                           |
| ---------------- | ----------------------------------------------------------------------------------------------------- |
| `PYNUMPY`        | 新的数值扩展 numpy 的依赖项。（`PYNUMERIC` 已被上游供应商废弃）。                                                            |
| `PYXML`          | XML 扩展的依赖项（对于 Python 2.0 及更高版本，因其也在基础发行版中，故不需要）。                                                      |
| `PY_ENUM34`      | 根据 Python 版本，条件依赖于 [devel/py-enum34](https://cgit.freebsd.org/ports/tree/devel/py-enum34/)。           |
| `PY_ENUM_COMPAT` | 根据 Python 版本，条件依赖于 [devel/py-enum-compat](https://cgit.freebsd.org/ports/tree/devel/py-enum-compat/)。 |
| `PY_PATHLIB`     | 根据 Python 版本，条件依赖于 [devel/py-pathlib](https://cgit.freebsd.org/ports/tree/devel/py-pathlib/)。         |
| `PY_IPADDRESS`   | 根据 Python 版本，条件依赖于 [net/py-ipaddress](https://cgit.freebsd.org/ports/tree/net/py-ipaddress/)。         |
| `PY_FUTURES`     | 根据 Python 版本，条件依赖于 [devel/py-futures](https://cgit.freebsd.org/ports/tree/devel/py-futures/)。         |

完整的可用变量列表可以在 **/usr/ports/Mk/Uses/python.mk** 中找到。

>**重要**
>
>所有使用 [Python flavors](https://docs.freebsd.org/en/books/porters-handbook/flavors/#flavors-auto-python) 的 Python  Port 的依赖项（无论是使用 `USE_PYTHON=distutils` 还是 `USE_PYTHON=flavors`）都必须使用 `@${PY_FLAVOR}` 将 Python flavor 添加到它们的 origin 中。请参见 [简单的 Python 模块 Makefile](https://docs.freebsd.org/en/books/porters-handbook/special/#python-Makefile)。

**示例 27. 简单 Python 模块的 Makefile**

```sh
PORTNAME=	sample
DISTVERSION=	1.2.3
CATEGORIES=	devel

MAINTAINER=	fred.bloggs@example.com
COMMENT=	Python sample module
WWW=		https://example.com/project/sample/

RUN_DEPENDS=	${PYTHON_PKGNAMEPREFIX}six>0:devel/py-six@${PY_FLAVOR}

USES=		python
USE_PYTHON=	autoplist distutils

.include <bsd.port.mk>
```

一些 Python 应用程序声称支持 `DESTDIR`（这是 staging 所需的），但实际上它是破损的（例如，Mailman 版本 2.1.16 之前的版本）。可以通过重新编译脚本来解决此问题。可以在 `post-build` 目标中实现这一点。例如，假设 Python 脚本应该安装到 `PYTHONPREFIX_SITELIBDIR`，则可以应用以下解决方案：

```sh
(cd ${STAGEDIR}${PREFIX} \
  && ${PYTHON_CMD} ${PYTHON_LIBDIR}/compileall.py \
   -d ${PREFIX} -f ${PYTHONPREFIX_SITELIBDIR:S;${PREFIX}/;;})
```

这将使用相对于阶段目录的路径重新编译源代码，并通过 `-d` 选项将 `PREFIX` 的值添加到字节编译输出文件的文件名中。需要使用 `-f` 强制重新编译，`-S;${PREFIX}/;;` 将 `PYTHONPREFIX_SITELIBDIR` 的前缀去除，使其相对于 `PREFIX`。

## 6.19. 使用 Tcl/Tk

Ports 支持多个 Tcl/Tk 版本的并行安装。Ports 应该至少支持默认的 Tcl/Tk 版本和更高版本，可以通过 `USES=tcl` 来指定。也可以通过附加版本号 `:_xx_` 来指定所需的 `tcl` 版本，例如 `USES=tcl:85`。

**表 27. 使用 Tcl/Tk 的 Ports 最有用的只读变量**

| `TCL_VER`              | 选择的 Tcl 版本号（主版本.次版本） |
| ---------------------- | -------------------- |
| `TCLSH`                | Tcl 解释器的完整路径         |
| `TCL_LIBDIR`           | Tcl 库文件的路径           |
| `TCL_INCLUDEDIR`       | Tcl C 头文件的路径         |
| `TCL_PKG_LIB_PREFIX`   | 库前缀，按 TIP595 标准      |
| `TCL_PKG_STUB_POSTFIX` | Stub 库后缀             |
| `TK_VER`               | 选择的 Tk 版本号（主版本.次版本）  |
| `WISH`                 | Tk 解释器的完整路径          |
| `TK_LIBDIR`            | Tk 库文件的路径            |
| `TK_INCLUDEDIR`        | Tk C 头文件的路径          |

有关这些变量的完整说明，请参阅 [`USES=tcl`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-tcl) 和 [`USES=tk`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-tk) 以及 [Using `USES` Macros](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses)。这些变量的完整列表可在 **/usr/ports/Mk/Uses/tcl.mk** 中找到。

## 6.20. 使用 SDL

`USE_SDL` 用于自动配置使用 SDL 基础库的 Ports，例如 [devel/sdl12](https://cgit.freebsd.org/ports/tree/devel/sdl12/) 和 [graphics/sdl\_image](https://cgit.freebsd.org/ports/tree/graphics/sdl_image/)。

SDL 1.2 版本的这些库被识别：

* sdl: [devel/sdl12](https://cgit.freebsd.org/ports/tree/devel/sdl12/)
* console: [devel/sdl\_console](https://cgit.freebsd.org/ports/tree/devel/sdl_console/)
* gfx: [graphics/sdl\_gfx](https://cgit.freebsd.org/ports/tree/graphics/sdl_gfx/)
* image: [graphics/sdl\_image](https://cgit.freebsd.org/ports/tree/graphics/sdl_image/)
* mixer: [audio/sdl\_mixer](https://cgit.freebsd.org/ports/tree/audio/sdl_mixer/)
* mm: [devel/sdlmm](https://cgit.freebsd.org/ports/tree/devel/sdlmm/)
* net: [net/sdl\_net](https://cgit.freebsd.org/ports/tree/net/sdl_net/)
* pango: [x11-toolkits/sdl\_pango](https://cgit.freebsd.org/ports/tree/x11-toolkits/sdl_pango/)
* sound: [audio/sdl\_sound](https://cgit.freebsd.org/ports/tree/audio/sdl_sound/)
* ttf: [graphics/sdl\_ttf](https://cgit.freebsd.org/ports/tree/graphics/sdl_ttf/)

SDL 2.0 版本的这些库被识别：

* sdl: [devel/sdl20](https://cgit.freebsd.org/ports/tree/devel/sdl20/)
* gfx: [graphics/sdl2\_gfx](https://cgit.freebsd.org/ports/tree/graphics/sdl2_gfx/)
* image: [graphics/sdl2\_image](https://cgit.freebsd.org/ports/tree/graphics/sdl2_image/)
* mixer: [audio/sdl2\_mixer](https://cgit.freebsd.org/ports/tree/audio/sdl2_mixer/)
* net: [net/sdl2\_net](https://cgit.freebsd.org/ports/tree/net/sdl2_net/)
* ttf: [graphics/sdl2\_ttf](https://cgit.freebsd.org/ports/tree/graphics/sdl2_ttf/)

因此，如果一个 port 依赖于 [net/sdl\_net](https://cgit.freebsd.org/ports/tree/net/sdl_net/) 和 [audio/sdl\_mixer](https://cgit.freebsd.org/ports/tree/audio/sdl_mixer/)，其语法为：

```sh
USE_SDL=	net mixer
```

依赖项 [devel/sdl12](https://cgit.freebsd.org/ports/tree/devel/sdl12/)，这是 [net/sdl\_net](https://cgit.freebsd.org/ports/tree/net/sdl_net/) 和 [audio/sdl\_mixer](https://cgit.freebsd.org/ports/tree/audio/sdl_mixer/) 所必需的，将会自动添加。

使用 `USE_SDL` 配置 SDL 1.2 时，它会自动：

* 将对 sdl12-config 的依赖添加到 `BUILD_DEPENDS`；
* 将变量 `SDL_CONFIG` 添加到 `CONFIGURE_ENV`；
* 将所选库的依赖项添加到 `LIB_DEPENDS`。

使用 `USE_SDL` 配置 SDL 2.0 时，它会自动：

* 将对 sdl2-config 的依赖添加到 `BUILD_DEPENDS`；
* 将变量 `SDL2_CONFIG` 添加到 `CONFIGURE_ENV`；
* 将所选库的依赖项添加到 `LIB_DEPENDS`。

## 6.21. 使用 wxWidgets

本节介绍了 wxWidgets 库在 Ports 树中的状态以及与 Ports 系统的集成。

### 6.21.1. 介绍

wxWidgets 库有多个版本，它们之间存在冲突（安装的文件同名）。在 Ports 树中，这个问题通过为每个版本使用版本号后缀来解决，使每个版本安装在不同的路径下。

显而易见的缺点是每个应用程序都需要修改，以找到预期的版本。幸运的是，大多数应用程序会调用 `wx-config` 脚本来确定所需的编译器和链接器标志。该脚本的名称在每个可用版本中都是不同的。大多数应用程序会尊重环境变量，或接受配置参数来指定要调用的 `wx-config` 脚本。否则，它们必须进行补丁处理。

### 6.21.2. 版本选择

为了让 port 使用特定版本的 wxWidgets，提供了两个可供定义的变量（如果只定义了其中一个，另一个将设置为默认值）：

**表 28. 选择 wxWidgets 版本的变量**

| 变量           | 说明          | 默认值    |
| ------------ | ----------- | ------ |
| `USE_WX`     |  Port 可以使用的版本列表 | 所有可用版本 |
| `USE_WX_NOT` |  Port 不能使用的版本列表 | 无      |

可用的 wxWidgets 版本及其在树中的对应 Port 如下：

**表 29. 可用的 wxWidgets 版本**

| 版本    |  Port                                                                                 |
| ----- | --------------------------------------------------------------------------------- |
| `2.8` | [x11-toolkits/wxgtk28](https://cgit.freebsd.org/ports/tree/x11-toolkits/wxgtk28/) |
| `3.0` | [x11-toolkits/wxgtk30](https://cgit.freebsd.org/ports/tree/x11-toolkits/wxgtk30/) |

[选择 wxWidgets 版本的变量](https://docs.freebsd.org/en/books/porters-handbook/special/#wx-ver-sel-table)可以设置为这些组合之一，多个版本之间用空格分隔：

**表 30. wxWidgets 版本规范**

| 说明          | 示例        |
| ----------- | --------- |
| 单一版本        | `2.8`     |
| 升序范围        | `2.8+`    |
| 降序范围        | `3.0-`    |
| 完整范围（必须是升序） | `2.8-3.0` |

还有一些变量用于从可用版本中选择首选版本。它们可以设置为一个版本列表，列表中的第一个版本将具有更高的优先级。

**表 31. 选择首选 wxWidgets 版本的变量**

| 名称            | 适用对象 |
| ------------- | ---- |
| `WANT_WX_VER` | port |
| `WITH_WX_VER` | 用户   |

### 6.21.3. 组件选择

还有一些应用程序，尽管不是 wxWidgets 库，但与它们相关。这些应用程序可以在 `WX_COMPS` 中指定。可用的组件如下：

**表 32. 可用的 wxWidgets 组件**

| 名称        | 说明                  | 版本限制      |
| --------- | ------------------- | --------- |
| `wx`      | 主库                  | 无         |
| `contrib` | 贡献的库                | 无         |
| `python`  | wxPython（Python 绑定） | `2.8-3.0` |

每个组件的依赖类型可以通过添加后缀来选择，后缀与组件名称之间用分号分隔。如果没有指定后缀，则会使用默认类型（请参见 [默认 wxWidgets 依赖类型](https://docs.freebsd.org/en/books/porters-handbook/special/#wx-def-dep-types)）。可用的依赖类型如下：

**表 33. 可用的 wxWidgets 依赖类型**

| 名称      | 说明                          |
| ------- | --------------------------- |
| `build` | 组件用于构建，相当于 `BUILD_DEPENDS`  |
| `run`   | 组件用于运行，相当于 `RUN_DEPENDS`    |
| `lib`   | 组件用于构建和运行，相当于 `LIB_DEPENDS` |

组件的默认依赖类型在下表中列出：

**表 34. 默认 wxWidgets 依赖类型**

| 组件        | 依赖类型  |
| --------- | ----- |
| `wx`      | `lib` |
| `contrib` | `lib` |
| `python`  | `run` |
| `mozilla` | `lib` |
| `svg`     | `lib` |

**示例 28. 选择 wxWidgets 组件**

以下片段对应一个使用 wxWidgets 版本 `2.4` 及其贡献库的 port：

```sh
USE_WX=		2.8
WX_COMPS=	wx contrib
```

### 6.21.4. 检测已安装的版本

要检测已安装的版本，可以定义 `WANT_WX`。如果没有设置为特定版本，则组件将带有版本后缀。检测后，`HAVE_WX` 将会被填充。

**示例 29. 检测已安装的 wxWidgets 版本和组件**

此片段可以用于一个使用 wxWidgets 的 port，如果它已安装，或者选择了某个选项。

```sh
WANT_WX=	yes

.include <bsd.port.pre.mk>

.if defined(WITH_WX) || !empty(PORT_OPTIONS:MWX) || !empty(HAVE_WX:Mwx-2.8)
USE_WX=			2.8
CONFIGURE_ARGS+=	--enable-wx
.endif
```

此片段可以用于一个启用 wxPython 支持的 port，如果 wxWidgets 已安装或者选中了相关选项，此外还包括版本 `2.8`。

```sh
USE_WX=		2.8
WX_COMPS=	wx
WANT_WX=	2.8

.include <bsd.port.pre.mk>

.if defined(WITH_WXPYTHON) || !empty(PORT_OPTIONS:MWXPYTHON) || !empty(HAVE_WX:Mpython)
WX_COMPS+=		python
CONFIGURE_ARGS+=	--enable-wxpython
.endif
```

### 6.21.5. 已定义的变量

这些变量可以在 port 中使用（在定义了 [选择 wxWidgets 版本的变量](https://docs.freebsd.org/en/books/porters-handbook/special/#wx-ver-sel-table) 后）。

**表 35. 为使用 wxWidgets 的 Ports 定义的变量**

| 名称           | 说明                                     |
| ------------ | -------------------------------------- |
| `WX_CONFIG`  | wxWidgets 的 `wx-config` 脚本的路径（具有不同的名称） |
| `WXRC_CMD`   | wxWidgets 的 `wxrc` 程序的路径（具有不同的名称）      |
| `WX_VERSION` | 将使用的 wxWidgets 版本（例如，`2.6`）            |

### 6.21.6. 在 **bsd.port.pre.mk** 中的处理

定义 `WX_PREMK` 以便在包含 **bsd.port.pre.mk** 后可以使用这些变量。

>**重要**
>
>当定义了 `WX_PREMK`，则版本、依赖、组件和定义的变量在修改 wxWidgets port 变量 *后* 包含 **bsd.port.pre.mk** 时不会发生变化。

**示例 30. 在命令中使用 wxWidgets 变量**

此片段展示了如何通过运行 `wx-config` 脚本来获取完整的版本字符串，将其分配给变量并传递给程序，使用 `WX_PREMK`。

```sh
USE_WX=		2.8
WX_PREMK=	yes

.include <bsd.port.pre.mk>

.if exists(${WX_CONFIG})
VER_STR!=	${WX_CONFIG} --release

PLIST_SUB+=	VERSION="${VER_STR}"
.endif
```

>**注意**
>
>当 wxWidgets 变量位于目标中时，无需使用 `WX_PREMK`，可以安全地在命令中使用这些变量。


### 6.21.7. 额外的 `configure` 参数

一些 GNU `configure` 脚本仅通过设置 `WX_CONFIG` 环境变量无法找到 wxWidgets，可能需要额外的参数。可以使用 `WX_CONF_ARGS` 提供这些参数。

**表 36. `WX_CONF_ARGS` 的合法值**

| 可能的值       | 结果参数                                                     |
| ---------- | -------------------------------------------------------- |
| `absolute` | `--with-wx-config=${WX_CONFIG}`                          |
| `relative` | `--with-wx=${LOCALBASE} --with-wx-config=${WX_CONFIG:T}` |

## 6.22. 使用 Lua

本节介绍了 Lua 库在 Ports 树中的状态及其与 Ports 系统的集成。

### 6.22.1. 引言

Lua 库和对应的解释器有多个版本，它们之间存在冲突（安装文件时会使用相同的名称）。在 Ports 树中，通过使用版本号后缀将每个版本安装为不同的名称来解决这个问题。

显而易见的缺点是，每个应用程序都需要修改以找到期望的版本。但是，可以通过向编译器和链接器添加一些额外的标志来解决此问题。

使用 Lua 的应用程序通常应仅为一个版本构建。然而，针对 Lua 的可加载模块会为其支持的每个 Lua 版本构建单独的版本，并且依赖这些模块的 Port 应该通过在 Port 源中使用 `@${LUA_FLAVOR}` 后缀来指定版本。

### 6.22.2. 版本选择

使用 Lua 的 Port 应该有以下形式的行：

```sh
USES=	lua
```

如果需要特定版本的 Lua 或版本范围，可以按 `XY`（可以多次使用）、`XY+`、`-XY` 或 `XY-ZA` 的格式指定。如果 `DEFAULT_VERSIONS` 设置的默认 Lua 版本位于请求的范围内，则将使用它，否则将使用默认版本最接近请求的版本。例如：

```sh
USES=	lua:52-53
```

请注意，版本选择不会尝试基于已安装的 Lua 版本进行调整。

>**注意**
>
>使用 `XY+` 格式的版本指定应该谨慎，Lua 的 API 在每个版本中都会有所变化，配置工具如 CMake 或 Autoconf 在未来的 Lua 版本上通常无法正常工作，直到它们被更新为支持这些版本。 

### 6.22.3. 配置和编译器标志

使用 Lua 的软件可能已经编写为自动检测正在使用的 Lua 版本。通常，Port 应该覆盖这一假设，强制使用如上所述选择的特定 Lua 版本。根据正在移植的软件，这可能需要以下任何或所有操作：

* 使用 `LUA_VER` 作为参数的一部分，通过 `CONFIGURE_ARGS` 或 `CONFIGURE_ENV`（或其他构建系统的等效项）传递给软件的配置脚本；
* 向 `CFLAGS`、`LDFLAGS` 和 `LIBS` 分别添加 `-I${LUA_INCDIR}`、`-L${LUA_LIBDIR}` 和 `-llua-${LUA_VER}`；
* 修补软件的配置或构建文件，以选择正确的版本。

### 6.22.4. 版本风味

一个安装 Lua 模块的 Port（而不是仅仅使用 Lua 的应用程序）应该为每个支持的 Lua 版本构建一个单独的风味。这可以通过添加 `module` 参数来完成：

```sh
USES=	lua:module
```

也可以指定版本号或版本范围；使用逗号分隔多个参数。

由于每个风味必须有不同的包名，因此提供了变量 `LUA_PKGNAMEPREFIX`，该变量将被设置为适当的值；其预期用法为：

```sh
PKGNAMEPREFIX=	${LUA_PKGNAMEPREFIX}
```

模块 Port 通常只应安装文件到 `LUA_MODLIBDIR`、`LUA_MODSHAREDIR`、`LUA_DOCSDIR` 和 `LUA_EXAMPLESDIR`，所有这些目录都已设置为引用特定版本的子目录。安装任何其他文件必须小心，以避免版本之间的冲突。

一个希望为每个 Lua 版本构建单独包的 Port（而不是 Lua 模块）应该使用 `flavors` 参数：

```sh
USES=	lua:flavors
```

这与上述说明的 `module` 参数的工作方式相同，但不假设包应该作为 Lua 模块来记录（因此默认情况下不定义 `LUA_DOCSDIR` 和 `LUA_EXAMPLESDIR`）。不过，Port 可以选择定义 `LUA_DOCSUBDIR` 作为合适的子目录名称（通常是 Port 的 `PORTNAME`，只要它不与任何模块的 `PORTNAME` 冲突），在这种情况下，框架将定义 `LUA_DOCSDIR` 和 `LUA_EXAMPLESDIR`。

与模块 Port 一样，风味 Port 应避免安装可能在版本之间冲突的文件。通常通过将 `LUA_VER_STR` 作为程序名称的后缀来完成此操作（例如使用 [`uniquefiles`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-uniquefiles)），并在其他文件或子目录中使用 `LUA_VER` 或 `LUA_VER_STR`，这些文件或子目录位于 `LUA_MODLIBDIR` 和 `LUA_MODSHAREDIR` 之外。

### 6.22.5. 定义的变量

在 Port 中可以使用以下变量。

**表 37. 使用 Lua 的 Port 中定义的变量**

| 名称                   | 说明                               |
| -------------------- | -------------------------------- |
| `LUA_VER`            | 将使用的 Lua 版本（例如，`5.4`）            |
| `LUA_VER_STR`        | 不带点的 Lua 版本（例如，`54`）             |
| `LUA_FLAVOR`         | 对应所选 Lua 版本的风味名称，用于指定依赖          |
| `LUA_BASE`           | 用于定位已安装的 Lua（和组件）的前缀             |
| `LUA_PREFIX`         | 该 Port 将安装 Lua（和组件）的位置前缀         |
| `LUA_INCDIR`         | 安装 Lua 头文件的目录                    |
| `LUA_LIBDIR`         | 安装 Lua 库的目录                      |
| `LUA_REFMODLIBDIR`   | 已安装的 Lua 模块库（**.so**）的目录         |
| `LUA_REFMODSHAREDIR` | 已安装的 Lua 模块（**.lua**）的目录         |
| `LUA_MODLIBDIR`      | 该 Port 将安装 Lua 模块库（**.so**）的位置目录 |
| `LUA_MODSHAREDIR`    | 该 Port 将安装 Lua 模块（**.lua**）的位置目录 |
| `LUA_PKGNAMEPREFIX`  | Lua 模块使用的包名称前缀                   |
| `LUA_CMD`            | Lua 解释器的名称（例如，`lua54`）           |
| `LUAC_CMD`           | Lua 编译器的名称（例如，`luac54`）          |

对于指定了 `module` 参数的 Port，还有以下附加变量：

**表 38. Lua 模块 Port 中定义的变量**

| 名称                | 说明             |
| ----------------- | -------------- |
| `LUA_DOCSDIR`     | 模块文档应安装到的目录。   |
| `LUA_EXAMPLESDIR` | 模块示例文件应安装到的目录。 |

### 6.22.6. 示例

**示例 31. 使用 Lua 的应用程序 Makefile**

此示例展示了如何引用运行时所需的 Lua 模块。请注意，引用必须指定一个风味。

```sh
PORTNAME=	sample
DISTVERSION=	1.2.3
CATEGORIES=	whatever

MAINTAINER=	fred.bloggs@example.com
COMMENT=	Sample
WWW=		https://example.com/lua_sample/sample/

RUN_DEPENDS=	${LUA_REFMODLIBDIR}/lpeg.so:devel/lua-lpeg@${LUA_FLAVOR}

USES=		lua

.include <bsd.port.mk>
```

** 32. 简单 Lua 模块的 Makefile**

```sh
PORTNAME=	sample
DISTVERSION=	1.2.3
CATEGORIES=	whatever
PKGNAMEPREFIX=	${LUA_PKGNAMEPREFIX}

MAINTAINER=	fred.bloggs@example.com
COMMENT=	Sample
WWW=		https://example.com/lua_sample/sample/

USES=		lua:module

DOCSDIR=	${LUA_DOCSDIR}

.include <bsd.port.mk>
```

## 6.23. 使用 Guile

本节介绍了 Guile 在 Ports 树中的状态以及它与 Ports 系统的集成。

### 6.23.1. 介绍

Guile 库和相应的解释器有多个版本，它们之间存在冲突（以相同的名称安装文件）。在 Ports 树中，通过使用版本号后缀将每个版本安装为不同的名称来解决这个问题。在大多数情况下，应用程序应该通过提供的配置变量检测正确的版本，并使用 `pkg-config` 来确定名称和相关路径。然而，一些应用程序（尤其是那些使用自己配置规则的应用程序，如 `cmake` 或 `meson`）将始终尝试使用最新版本。在这种情况下，要么修补 Port ，要么声明构建冲突（见下文的 `conflicts` 选项），以确保在非 poudriere 构建时生成正确的依赖项。

使用 Guile 的应用程序通常应该仅为一个版本构建，最好是 `DEFAULT_VERSIONS` 中指定的版本，或者如果不支持该版本，则为它们支持的最新版本。然而，Guile 或 Scheme 库，或 Guile 的扩展模块，会为它们支持的每个 Guile 版本分别构建一个风味，并且这些模块的依赖关系应该使用 `@${GUILE_FLAVOR}` 后缀来指定 Port 来源。

### 6.23.2. 版本选择

使用 Guile 的 Port 应该定义 `USES=guile:<em>arg,arg…</em>`，并根据以下参数选择适当的值：

**表 39. 使用 Guile 的 Ports 定义的参数**

| 名称             | 说明                                                                          |
| -------------- | --------------------------------------------------------------------------- |
| `<em>X.Y</em>` | 声明与 Guile 版本 `X.Y` 的兼容性。目前可用的版本为 `1.8`（已废弃）、`2.2` 和 `3.0`。可以指定多个版本。         |
| `flavors`      | 为每个指定的 Guile 版本创建一个风味。由 `DEFAULT_VERSIONS` 指定的版本将成为默认风味。风味名称的格式为 `guileXY`。 |
| `build`        | 仅将 Guile 解释器添加为构建依赖项，而不是库依赖项。可以同时指定 `build` 和 `run`。                        |
| `run`          | 仅将 Guile 解释器添加为运行时依赖项，而不是库依赖项。可以同时指定 `build` 和 `run`。                       |
| `alias`        | 为解释器和工具添加 `BINARY_ALIAS` 值。                                                 |
| `conflicts`    | 为比选定版本更新的 Guile 版本声明 `CONFLICTS_BUILD`。当 Port 无法配置为使用特定版本的 Guile 时，使用此选项。       |

有一些额外的参数可用于处理特殊情况；详细信息请参见 `Mk/Uses/guile.mk`。

除非指定了 `build` 或 `run`，否则 `LIB_DEPENDS` 会同时接收 `libguile` 库依赖项以及 Guile 版本所需的其他依赖项，例如 `libgc`。通常情况下，Port 不需要与其使用 Guile 相关的其他依赖项。


### 6.23.3. 配置标志

使用 Guile 的软件应使用 `pkg-config` 机制来获取编译器和链接器标志。一些较旧或特殊的 Port 可能仍在使用 `guile-config` 或直接从 `guile` 获取值，这些方法也应该有效（在某些情况下，`alias` 参数可能会有帮助）。

框架会尝试通过以下方式通知 Port 所需的 Guile 版本：

* `GUILE_EFFECTIVE_VERSION` 被添加到 `CONFIGURE_ENV` 中；
* Guile 二进制文件的完整路径被指定在 `CONFIGURE_ENV` 和 `MAKE_ENV` 中的 `GUILE` 变量；
* 如果使用了 `alias` 选项，则所需 Guile 版本的二进制文件是别名文件；
* 如果没有使用 `alias` 选项，则所需 Guile 版本的工具路径（如 `guild`、`guile-config` 等）将作为变量 `GUILD`、`GUILE_CONFIG` 等被添加到 `CONFIGURE_ENV` 和 `MAKE_ENV` 中。

对于某些 Port ，可能需要通过其他方式指定版本，例如通过 `CONFIGURE_ARGS` 或 `MESON_ARGS`，具体取决于 Port 。

如果这些方法都无法在存在其他版本的情况下使 Port 选择指定的 Guile 版本，最好是修补 Port 以实现此目的。如果不可行，则应指定 `conflicts` 选项，以防止在错误版本被检测到的情况下构建 Port 。

### 6.23.4. 版本风味

安装 Guile 扩展或库的 Port ，或为 Guile 预编译的 Scheme 库，应为每个支持的 Guile 版本构建一个单独的风味。通过添加 `flavors` 选项来实现此目的。

由于每个风味必须具有不同的包名称，因此这些 Port 必须设置 `PKGNAMESUFFIX`，通常为：

```sh
PKGNAMESUFFIX=	-${FLAVOR}
```

这些 Port 必须将 Scheme 文件安装到 `GUILE_SITE_DIR`，而不是 `GUILE_GLOBAL_SITE_DIR`，即使这些文件不是版本特定的。这通常需要修补 Port 。

此外，如果此类 Port 安装 `.pc` 文件，则必须将其放置在 `GUILE_PKGCONFIG_PATH` 中，而不是放入全局 `pkgconfig` 目录。这允许依赖的 Port 为正在使用的特定 Guile 版本找到正确的配置。

如果 Guile 扩展 Port 安装 `.so` 文件，则通常必须将其放置在与 Guile 版本相关的 `extensions` 目录中。通常不应使用 `USE_LDCONFIG`。

任何由风味 Port 安装的其他文件也必须位于版本特定的目录中，或使用版本特定的文件名。对于文档和，`GUILE_DOCS_DIR` 和 `GUILE_EXAMPLES_DIR` 指定了适合的位置，在这些位置 Port 应创建子目录，详见下文。

### 6.23.5. 已定义的变量

这些变量在 Port 中可用。

**表 40. 使用 Guile 的 Port 所定义的变量**

| 名称                                  | 值                                        | 说明                                               |
| ----------------------------------- | ------------------------------------------ | ------------------------------------------------ |
| `GUILE_VER`                         | `3.0`                                      | 正在使用的 Guile 版本。                                  |
| `GUILE_SFX`                         | `3`                                        | 用于某些名称的短后缀。仅在小心使用时使用；可能不是唯一的，或将来可能会改变。           |
| `GUILE_FLAVOR`                      | `guile30`                                  | 与所选版本对应的风味名称。                                    |
| `GUILE_PORT`                        | `lang/guile3`                              | 指定的 Guile 版本的 Port 来源。                               |
| `GUILE_PREFIX`                      | `${PREFIX}`                                | 用于安装的目录前缀。                                       |
| `GUILE_CMD`                         | `guile-3.0`                                | 带有版本后缀的 Guile 解释器名称。                             |
| `GUILE_CMDPATH`                     | `${LOCALBASE}/bin/guile-3.0`               | Guile 解释器的完整路径。                                  |
| `GUILD_CMD`                         | `guild-3.0`                                | 带有版本后缀的 Guild 工具名称。                              |
| `GUILD_CMDPATH`                     | `${LOCALBASE}/bin/guild-3.0`               | Guild 工具的完整路径。                                   |
| `GUILE_*_CMD`<br/>`GUILE_*_CMDPATH` |                                            | 类似于 `GUILE_CMD` 和 `GUILE_CMDPATH`，但适用于其他工具二进制文件。 |
| `GUILE_PKGCONFIG_PATH`              | `${LOCALBASE}/libdata/pkgconfig/guile/3.0` | 使用 `flavors` 的 Port 应将 `.pc` 文件安装到此位置。               |
| `GUILE_INFO_PATH`                   | `share/info/guile3`                        | 对于使用 `flavors` 选项的 Port ，`INFO_PATH` 的合适值。           |

### 6.23.6. 路径替代

以下内容定义了作为变量和 `PLIST_SUB` 条目的路径替代。变量形式以 `_DIR` 作为后缀，并且是完整路径（以 `GUILE_PREFIX` 为前缀）。

**表 41. 使用 Guile 的 Port 定义的路径替代**

| 名称                  | 值                         | 说明                          |
| ------------------- | --------------------------- | --------------------------- |
| `GUILE_GLOBAL_SITE` | `share/guile/site`          | 所有 Guile 版本共享的站点目录；通常不建议使用。 |
| `GUILE_SITE`        | `share/guile/3.0/site`      | 选定的 Guile 版本的站点目录。          |
| `GUILE_SITE_CCACHE` | `lib/guile/3.0/site-ccache` | 编译字节码文件的目录。                 |
| `GUILE_DOCS`        | `share/doc/guile30`         | 版本特定文档的父目录。                 |
| `GUILE_EXAMPLES`    | `share/examples/guile30`    | 版本特定的父目录。                 |

## 6.24. 使用 `iconv`

FreeBSD 操作系统自带了本地的 `iconv`。

对于需要 `iconv` 的软件，定义 `USES=iconv`。

当一个 Port 定义了 `USES=iconv` 时，以下变量将可用：

| 变量名                    | 目的                               |  Port  `iconv`（使用 WCHAR\_T 或 //TRANSLIT 扩展时） | 基础 `iconv`         |
| ---------------------- | -------------------------------- | ---------------------------------------- | ------------------ |
| `ICONV_CMD`            | `iconv` 二进制文件所在的目录               | `${LOCALBASE}/bin/iconv`                 | **/usr/bin/iconv** |
| `ICONV_LIB`            | 链接到 **libiconv** 的 `ld` 参数（如果需要） | `-liconv`                                | （空）                |
| `ICONV_PREFIX`         | `iconv` 实现所在的目录（对配置脚本有用）         | `${LOCALBASE}`                           | **/usr**           |
| `ICONV_CONFIGURE_ARG`  | 为配置脚本预构建的配置参数                    | `--with-libiconv-prefix=${LOCALBASE}`    | （空）                |
| `ICONV_CONFIGURE_BASE` | 为配置脚本预构建的配置参数                    | `--with-libiconv=${LOCALBASE}`           | （空）                |

这两个自动为使用 [converters/libiconv](https://cgit.freebsd.org/ports/tree/converters/libiconv/) 或本地 `iconv` 的系统填充正确的变量值：

** 34. 简单的 `iconv` 使用**

```sh
USES=		iconv
LDFLAGS+=	-L${LOCALBASE}/lib ${ICONV_LIB}
```

** 35. 使用 `iconv` 配置**

```sh
USES=		iconv
CONFIGURE_ARGS+=${ICONV_CONFIGURE_ARG}
```

如上所示，当存在本地 `iconv` 时，`ICONV_LIB` 为空。这可以用来检测本地 `iconv` 并作出适当响应。

有时，程序的 **Makefile** 或配置脚本中会硬编码 `ld` 参数或搜索路径。可以通过以下方法解决此问题：

** 36. 修复硬编码的 `-liconv`**

```sh
USES=		iconv

post-patch:
	@${REINPLACE_CMD} -e 's/-liconv/${ICONV_LIB}/' ${WRKSRC}/Makefile
```

在某些情况下，可能需要根据是否存在本地 `iconv` 来设置备用值或执行操作。必须在测试 `ICONV_LIB` 的值之前包含 **bsd.port.pre.mk**：

** 37. 检查本地 `iconv` 可用性**

```sh
USES=		iconv

.include <bsd.port.pre.mk>

post-patch:
.if empty(ICONV_LIB)
	# 检测到本地 iconv
	@${REINPLACE_CMD} -e 's|iconv||' ${WRKSRC}/Config.sh
.endif

.include <bsd.port.post.mk>
```

### 6.25. 使用 Xfce

需要 Xfce 库或应用程序的 Port 应设置 `USES=xfce`。

特定的 Xfce 库和应用程序依赖项通过赋值给 `USE_XFCE` 来设置。它们定义在 **/usr/ports/Mk/Uses/xfce.mk** 文件中。可用的值如下：

**`USE_XFCE` 的值**

|       |                                                                                      |
| ------- | ----------------------------------------------------------------------------------------- |
| garcon  | [sysutils/garcon](https://cgit.freebsd.org/ports/tree/sysutils/garcon/)                   |
| libexo  | [x11/libexo](https://cgit.freebsd.org/ports/tree/x11/libexo/)                             |
| libgui  | [x11-toolkits/libxfce4gui](https://cgit.freebsd.org/ports/tree/x11-toolkits/libxfce4gui/) |
| libmenu | [x11/libxfce4menu](https://cgit.freebsd.org/ports/tree/x11/libxfce4menu/)                 |
| libutil | [x11/libxfce4util](https://cgit.freebsd.org/ports/tree/x11/libxfce4util/)                 |
| panel   | [x11-wm/xfce4-panel](https://cgit.freebsd.org/ports/tree/x11-wm/xfce4-panel/)             |
| thunar  | [x11-fm/thunar](https://cgit.freebsd.org/ports/tree/x11-fm/thunar/)                       |
| xfconf  | [x11/xfce4-conf](https://cgit.freebsd.org/ports/tree/x11/xfce4-conf/)                     |

** 38. `USES=xfce` 示例**

```sh
USES=		xfce
USE_XFCE=	libmenu
```

**示例 39. 使用 Xfce 的 GTK2 小部件**

在此示例中， Port 化的应用程序使用 GTK2 特定的小部件 [x11/libxfce4menu](https://cgit.freebsd.org/ports/tree/x11/libxfce4menu/) 和 [x11/xfce4-conf](https://cgit.freebsd.org/ports/tree/x11/xfce4-conf/)。

```sh
USES=		xfce:gtk2
USE_XFCE=	libmenu xfconf
```

Xfce 组件通过这种方式包含时，会自动包括它们所需的任何依赖项。现在不再需要列出整个依赖列表。如果 Port 仅需要 [x11-wm/xfce4-panel](https://cgit.freebsd.org/ports/tree/x11-wm/xfce4-panel/)，只需使用以下配置：

```sh
USES=		xfce
USE_XFCE=	panel
```

不需要像这样列出 [x11-wm/xfce4-panel](https://cgit.freebsd.org/ports/tree/x11-wm/xfce4-panel/) 所需的组件：

```sh
USES=		xfce
USE_XFCE=	libexo libmenu libutil panel
```

但是，Xfce 组件和 Port 的非 Xfce 依赖项必须显式包含。不要指望某个 Xfce 组件会为主 Port 提供除了它本身之外的子依赖项。

### 6.26. 使用 Budgie

依赖于 Budgie 桌面的应用程序或库应设置 `USES=budgie` 并将 `USE_BUDGIE` 设置为所需组件的列表。


| 名称            | 说明                      |
| ------------- | ----------------------- |
| `libbudgie`   | 桌面核心（库）                 |
| `libmagpie`   | Budgie 的 X11 窗口管理器和合成器库 |
| `raven`       | 用于访问不同应用程序小部件的面板一体化中心   |
| `screensaver` | 桌面特定的屏幕保护程序             |

所有应用程序小部件通过 **org.budgie\_desktop.Raven** 服务进行通信。默认的依赖项是 lib 和运行时，可以通过 `:build` 或 `:run` 来更改。例如：

```sh
USES=		budgie
USE_BUDGIE=	screensaver:build
```

**示例 40. `USE_BUDGIE` 示例**

```sh
USES=		budgie gettext gnome meson pkgconfig
USE_BUDGIE=	libbudgie
```

### 6.27. 使用数据库

使用以下 `USES` 宏之一来添加数据库依赖项，具体参考 [Database `USES` Macros](https://docs.freebsd.org/en/books/porters-handbook/special/#using-databases-uses)。

**数据库 `USES` 宏**

**表 42 数据库宏**

| 数据库                     | `USES` 宏                                                                         |
| ----------------------- | -------------------------------------------------------------------------------- |
| Berkeley DB             | [`bdb`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-bdb)       |
| MariaDB, MySQL, Percona | [`mysql`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-mysql)   |
| PostgreSQL              | [`pgsql`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-pgsql)   |
| SQLite                  | [`sqlite`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-sqlite) |

**示例 41. 使用 Berkeley DB 6**

```sh
USES=	bdb:6
```

详见 [`bdb`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-bdb)。

**示例 42. 使用 MySQL**

当 Port 需要 MySQL 客户端库时，添加：

```sh
USES=	mysql
```

详见 [`mysql`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-mysql)。

**示例 43. 使用 PostgreSQL**

当 Port 需要 PostgreSQL 9.6 或更高版本的服务器时，添加：

```sh
USES=		pgsql:9.6+
WANT_PGSQL=	server
```

详见 [`pgsql`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-pgsql)。

**示例 44. 使用 SQLite 3**

```sh
USES=	sqlite:3
```

详见 [`sqlite`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-sqlite)。

## 6.28. 启动和停止服务（`rc` 脚本）

**rc.d** 脚本用于在系统启动时启动服务，并为管理员提供一种标准的方式来停止、启动和重新启动服务。 Port 集成到系统 **rc.d** 框架中。有关其用法的详细信息，可以参见 [rc.d Handbook 章节](https://docs.freebsd.org/en/books/handbook/#configtuning-rcd)。有关可用命令的详细解释，请参见 [rc(8)](https://man.freebsd.org/cgi/man.cgi?query=rc&sektion=8&format=html) 和 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html)。此外，还有一篇关于 **rc.d** 脚本实践方面的 [文章](https://docs.freebsd.org/en/articles/rc-scripting/)。

假设有一个名为 *doorman* 的虚拟 Port ，需要启动 *doormand* 守护进程。在 **Makefile** 中添加以下内容：

```sh
USE_RC_SUBR=	doormand
```

可以列出多个脚本，它们将被安装。脚本必须放在 **files** 子目录中，并且文件名必须添加 `.in` 后缀。标准的 `SUB_LIST` 展开将会应用到这个文件上。强烈建议使用 `%%PREFIX%%` 和 `%%LOCALBASE%%` 展开。有关 `SUB_LIST` 的更多信息，请参见 [相关章节](https://docs.freebsd.org/en/books/porters-handbook/pkg-files/#using-sub-files)。

自 FreeBSD 6.1-RELEASE 以来，本地的 **rc.d** 脚本（包括 Port 安装的）已包含在基础系统的 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 中。

这是一个简单的 **rc.d** 脚本示例，用于启动 *doormand* 守护进程：

```sh
#!/bin/sh

# PROVIDE: doormand
# REQUIRE: LOGIN
# KEYWORD: shutdown
#
# 添加以下行到 /etc/rc.conf.local 或 /etc/rc.conf 来启用此服务：
#
# doormand_enable (bool): 默认为 NO。 设置为 YES 启用 doormand。
# doormand_config (path): 默认为 %%PREFIX%%/etc/doormand/doormand.cf

. /etc/rc.subr

name=doormand
rcvar=doormand_enable

load_rc_config $name

: ${doormand_enable:="NO"}
: ${doormand_config="%%PREFIX%%/etc/doormand/doormand.cf"}

command=%%PREFIX%%/sbin/${name}
pidfile=/var/run/${name}.pid

command_args="-p $pidfile -f $doormand_config"

run_rc_command "$1"
```

除非有非常充分的理由需要更早启动服务，或者服务以特定用户（非 root）身份运行，所有 Port 脚本必须使用：

```sh
REQUIRE: LOGIN
```

如果启动脚本启动了必须在系统关闭时停止的守护进程，以下设置将触发服务在系统关闭时停止：

```sh
KEYWORD: shutdown
```

如果脚本启动的不是持久化服务，则不需要此设置。

对于可选的配置元素，最好使用 `=` 风格的默认变量赋值，而不是 `:=` 风格，因为前者仅在变量未设置时设置默认值，而后者则会在变量未设置 *或* 为空时设置默认值。例如，用户可能在 **rc.conf.local** 中包含如下内容：

```sh
doormand_flags=""
```

使用 `:=` 的变量替换会不适当地覆盖用户的意图。`_enable` 变量是不可选的，必须使用 `:` 来设置默认值。

>**警告**
>
> Port  *不能* 在安装和卸载时启动和停止其服务。不要滥用 [@preexec 命令、@postexec 命令、@preunexec 命令、@postunexec 命令](https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-keywords-base-exec) 中说明的 **plist** 关键字，通过运行修改当前运行系统的命令，包括启动或停止服务。 


### 6.28.1. 提交前检查清单

在提交一个包含 **rc.d** 脚本的 Port 之前，尤其是提交之前，请查阅以下检查清单，以确保它已经准备好。

[devel/rclint](https://cgit.freebsd.org/ports/tree/devel/rclint/) Port 可以检查其中的大部分内容，但它不能替代适当的代码审查。

1. 如果这是一个新文件，它是否具有 **.sh** 扩展名？如果是的话，必须将其更改为 **file.in**，因为 **rc.d** 文件不能以该扩展名结尾。
2. 文件名（去掉 **.in** 后），`PROVIDE` 行和 `$` *name* 是否匹配？文件名与 `PROVIDE` 匹配可以使调试变得更容易，特别是对于 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 问题。匹配文件名和 `$` *name* 可以更容易地弄清楚哪些变量在 **rc.conf\[.local]** 中是相关的。这也是所有新脚本的政策，包括基础系统中的脚本。
3. `REQUIRE` 行是否设置为 `LOGIN`？这是运行为非 root 用户的脚本的强制要求。如果它以 root 身份运行，是否有充分的理由让它在 `LOGIN` 之前运行？如果没有，它必须在后面运行，以便本地脚本可以大致分组到 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 中大多数基础服务已启动后的位置。
4. 脚本是否启动了一个持久性服务？如果是的话，必须添加 `KEYWORD: shutdown`。
5. 确保没有 `KEYWORD: FreeBSD`。多年来这已经不必要也不可取了。这也是新脚本可能是从旧脚本复制/粘贴的一个迹象，因此审查时必须格外小心。
6. 如果脚本使用像 `perl`、`python` 或 `ruby` 这样的解释性语言，请确保 `command_interpreter` 设置得当，例如，对于 Perl，添加 `PERL=${PERL}` 到 `SUB_LIST` 并使用 `%%PERL%%`。否则，

   ```sh
   # service name stop
   ```

   可能无法正确工作。请参阅 [service(8)](https://man.freebsd.org/cgi/man.cgi?query=service&sektion=8&format=html) 获取更多信息。
7. 所有 **/usr/local** 的出现是否都已被替换为 `%%PREFIX%%`？
8. 默认变量赋值是否出现在 `load_rc_config` 之后？
9. 是否有默认的空字符串赋值？它们应该被删除，但请再次确认该选项是否在文件顶部的注释中有所记录。
10. 脚本中设置的变量是否被实际使用？
11. 默认的 **name**\`\_flags\` 中列出的选项是否确实是必须的？如果是，它们必须在 `command_args` 中。`-d` 是这里的一个警告标志（请原谅这个双关语），因为它通常是“守护进程化”过程的选项，因此实际上是必须的。
12. **\_name\_\_flags** 绝不能出现在 `command_args` 中（反之亦然，尽管这种错误较少见）。
13. 脚本是否无条件执行任何代码？这是不被推荐的。通常这些事情必须通过 `start_precmd` 进行处理。
14. 所有布尔测试必须使用 `checkyesno` 函数。不要手动编写 `[Yy][Ee][Ss]` 等测试。
15. 如果有一个循环（例如，等待某些东西启动），是否有一个计数器来终止该循环？如果出现错误，我们不希望引导过程永远卡住。
16. 脚本是否创建了需要特定权限的文件或目录，例如需要由运行该进程的用户拥有的 **pid**？与传统的 [touch(1)](https://man.freebsd.org/cgi/man.cgi?query=touch&sektion=1&format=html)/[chown(8)](https://man.freebsd.org/cgi/man.cgi?query=chown&sektion=8&format=html)/[chmod(1)](https://man.freebsd.org/cgi/man.cgi?query=chmod&sektion=1&format=html) 例程不同，考虑使用 [install(1)](https://man.freebsd.org/cgi/man.cgi?query=install&sektion=1&format=html) 并提供适当的命令行参数，一步完成整个过程。

## 6.29. 添加用户和组

一些 Ports 需要特定的用户帐户，通常是运行该用户的守护进程。对于这些 Ports，请从 50 到 999 中选择一个 *唯一* 的 UID，并在 **ports/UIDs**（用于用户）和 **ports/GIDs**（用于组）中注册它。用户和组的唯一标识符应当相同。

当需要为 Port 创建新用户或组时，请包括这两个文件的补丁。

然后在 **Makefile** 中使用 `USERS` 和 `GROUPS`，用户将在安装该 Port 时自动创建。

```sh
USERS=	pulse
GROUPS=	pulse pulse-access pulse-rt
```

当前保留的 UID 和 GID 列表可以在 **ports/UIDs** 和 **ports/GIDs** 中找到。

## 6.30. 依赖内核源代码的 Ports

一些 Ports（例如内核加载模块）需要内核源代码文件，以便能够编译。正确的判断用户是否已安装这些文件的方法如下：

```sh
USES=	kmod
```

除了这个检查，`kmod` 特性会处理这些 Ports 需要考虑的大部分事项。

## 6.31. Go 库

Ports 不得打包或安装 Go 库或源代码。Go Ports 必须在正常的获取阶段获取所需的依赖项，并且只应安装用户所需要的程序和内容，而不是 Go 开发人员需要的内容。

Ports 应该（按优先顺序）：

* 使用与包源一起提供的供应依赖项。
* 获取上游指定的依赖版本（如果使用 go.mod、vendor.json 或类似文件）。
* 作为最后的手段（如果没有包含依赖或未明确指定版本）获取上游开发/发布时可用的依赖版本。

## 6.32. Haskell 库

与 Go 语言一样，Ports 不得打包或安装 Haskell 库。Haskell Ports 必须静态链接它们的依赖，并在获取阶段获取所有的分发文件。

## 6.33. Shell 完成文件

许多现代 shell（包括 bash、fish、tcsh 和 zsh）都支持参数和/或选项的 Tab 补全。这种支持通常来自完成文件，这些文件包含如何为某个命令工作进行 Tab 补全的定义。Ports 有时会附带自己的完成文件，或者 Port 维护者可能已经自己创建了它们。

如果有可用的完成文件，应该始终安装它们。没有必要为此创建选项。如果使用了选项，则始终在 `OPTIONS_DEFAULT` 中启用它。

**表 43. 完整的 shell 完成文件名**

| `bash` | **\${PREFIX}/etc/bash\_completion.d** 或 **\${PREFIX}/share/bash-completion/completions** | （这些文件夹中的任何唯一文件名） |
| ------ | ---------------------------------------------------------------------------------------- | ---------------- |
| `fish` | **\${PREFIX}/share/fish/completions/\${PORTNAME}.fish**                                  |                  |
| `zsh`  | **\${PREFIX}/share/zsh/site-functions/\_\${PORTNAME}**                                   |                  |

不要注册对 shell 本身的任何依赖。
