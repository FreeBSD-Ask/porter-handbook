# 第6章 特殊情况

本节解释了创建port时要考虑的最常见事项。

## 分割长文件

有时，port Makefile 可能会非常长。例如，rust ports 可能会有一个非常长的 CARGO_CRATES 列表。在其他情况下，Makefile 可能会有根据架构变化的代码。在这种情况下，将原始 Makefile 拆分为多个文件可能很方便。bsd.port.mk 会自动将某些类型的 Makefile 包含到主 port Makefile 中。

这些是框架自动处理的文件（如果找到的话）:

* Makefile.crates。 在 audio/ebur128 中可以找到一个示例。
* Makefile.inc。 在 net/ntp 中可以找到一个示例。
* Makefile.${ARCH}-${OPSYS}
* Makefile.${OPSYS}. 一个示例可在 net/cvsup-static 中找到。
* Makefile.${ARCH}
* Makefile.local

将port的打包清单拆分为多个文件也是通常做法，如果清单在不同架构之间变化很大或取决于所选的风味。在这种情况下，每个架构的 pkg-plist 文件的命名遵循 pkg-plist.${ARCH}或 pkg-plist.${FLAVOR}的模式。如果存在多个 pkg-plist 文件，框架不会自动创建打包清单。选择适当的 pkg-plist 并将其分配给 PLIST 变量是 porter 的责任。如何处理这种情况的示例可以在 audio/logitechmediaserver 和 deskutils/libportal 中找到。

## 6.2. 分阶段

bsd.port.mk 期望ports与“阶段目录”一起工作。这意味着port不能直接将文件安装到常规目标目录（例如，在 PREFIX 下），而是安装到一个单独的目录，然后从该目录构建软件包。在许多情况下，这不需要 root 权限，使得可以作为非特权用户构建软件包。使用分阶段，port被构建并安装到阶段目录 STAGEDIR 中。从阶段目录创建软件包，然后安装到系统上。Automake 工具将此概念称为 DESTDIR ，但在 FreeBSD 中， DESTDIR 有不同的含义（请参阅 PREFIX 和 DESTDIR ）。

|  | 没有 port 真的需要成为 root。大部分情况下可以通过使用 USES\=uidfix 来避免。如果 port 仍然运行诸如 chown(8)、chgrp(1)之类的命令，或者通过 install(1) 强制所有者或组，则使用 USES\=fakeroot 来模拟这些调用。需要对 port 的 Makefiles 进行一些修补。 |
| -- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

元 ports，或者不安装文件而只依赖其他 ports 的 ports，必须避免不必要地将 mtree(8) 提取到阶段目录。这是软件包的基本目录布局，这些空目录将被视为孤立文件。为了防止 mtree(8) 提取，添加以下行：

```
NO_MTREE=	yes
```

|  | 元端口应该使用 USES\=metaport 。它为那些不获取、构建或安装任何内容的 ports 设置默认值。 |
| -- | -------------------------------------------------------------------------------------------- |

通过在 pre-install ， do-install 和 post-install 目标中使用路径之前添加 STAGEDIR 来启用分段（请参阅本书中的示例）。通常，这包括 PREFIX ， ETCDIR ， DATADIR ， EXAMPLESDIR ， DOCSDIR 等。目录应作为 post-install 目标的一部分创建。尽量避免使用绝对路径。

|  | 安装内核模块的Ports必须在默认情况下将 STAGEDIR 添加到其目的地，即/boot/modules。 |
| -- | ---------------------------------------------------------------------------------- |

### 6.2.1. 处理符号链接

当创建符号链接时，强烈建议使用相对路径。使用 ${RLN} 来创建相对符号链接。它在幕后使用 install(1)自动计算要创建的相对链接。

例子 1. 自动创建相对符号链接

${RLN} 使用 install(1)的相对符号功能，使得端口的计算相对路径得到释放。

```
${RLN} ${STAGEDIR}${PREFIX}/lib/libfoo.so.42 ${STAGEDIR}${PREFIX}/lib/libfoo.so
${RLN} ${STAGEDIR}${PREFIX}/libexec/foo/bar ${STAGEDIR}${PREFIX}/bin/bar
${RLN} ${STAGEDIR}/var/cache/foo ${STAGEDIR}${PREFIX}/share/foo
```

 将生成:

```
% ls -lF ${STAGEDIR}${PREFIX}/lib
lrwxr-xr-x  1 nobody  nobody    181 Aug  3 11:27 libfoo.so@ -> libfoo.so.42
-rwxr-xr-x  1 nobody  nobody     15 Aug  3 11:24 libfoo.so.42*
% ls -lF ${STAGEDIR}${PREFIX}/bin
lrwxr-xr-x  1 nobody  nobody    181 Aug  3 11:27 bar@ -> ../libexec/foo/bar
% ls -lF ${STAGEDIRDIR}${PREFIX}/share
lrwxr-xr-x  1 nobody  nobody    181 Aug  3 11:27 foo@ -> ../../../var/cache/foo
```

## 6.3. 捆绑库

本节解释为什么捆绑依赖项被视为不好，并介绍如何处理它们。

### 6.3.1. 为什么捆绑库是不好的

有些软件要求制作者找到第三方库并将所需的依赖项添加到定制中。其他软件将所有必要的库捆绑到分发文件中。起初，第二种方法似乎更容易，但存在一些严重的缺点：

这个列表基本上是基于 Fedora 和 Gentoo 维基百科，两者都在 CC-BY-SA 3.0 许可下授权。

如果在上游库中发现漏洞并进行修复，可能不会在与 port 捆绑的库中修复。一个原因可能是作者不知道这个问题。这意味着移植者必须修复它们，或者升级到一个非易受攻击的版本，并将补丁发送给作者。所有这些都需要时间，这导致软件比必要的时间更容易受到攻击。这反过来使得更难协调修复，而不必要地泄露有关漏洞的信息。

Bug 这个问题类似于上一段中安全性问题，但一般来说不那么严重。

分叉对于作者来说更容易在捆绑后分叉上游库。虽然乍一看很方便，但这意味着代码与上游不同，使得更难解决软件的安全或其他问题。这样做的原因之一是补丁变得更加困难。

另一个分叉的问题是，由于代码与上游分歧，错误会一遍又一遍地被解决，而不是在一个中心位置解决一次。这背离了开源软件的初衷。

符号冲突当一个库被安装在系统上时，它可能与捆绑版本冲突。这可能导致在编译或链接时立即出现错误。它还会导致运行程序时出现错误，这可能更难追踪。后一问题可能是由于两个库的版本不兼容造成的。

许可证问题当从不同来源捆绑项目时，许可证问题可能更容易出现，特别是当许可证不兼容时。

资源浪费打包的库会在多个层面上浪费资源。构建实际应用程序需要更长的时间，特别是如果这些库已经存在于系统上。在运行时，当系统范围的库已被一个程序加载，而打包的库被另一个程序加载时，它们可能占用不必要的内存。

努力的浪费当一个库需要为 FreeBSD 打补丁时，这些补丁必须在打包的库中再次复制。这会浪费开发人员的时间，因为这些补丁可能无法干净地应用。很难注意到这些补丁是必需的。

### 如何处理打包的库

尽可能使用库的非捆绑版本，通过向 LIB_DEPENDS 添加port来实现。如果尚未存在这样的port，请考虑创建它。

仅当上游在安全方面有良好记录且使用非捆绑版本会导致过于复杂的补丁时才使用捆绑库。

|  | 在一些非常特殊的情况下，例如模拟器，如 Wine，port必须捆绑库，因为它们处于不同的架构中，或者它们已被修改以适应软件的使用。在这种情况下，这些库不应暴露给其他ports以进行链接。向port的 Makefile 添加 BUNDLE_LIBS=yes 。这将告诉 pkg(8) 不计算提供的库。在将此添加到port之前，请始终向Ports管理团队询问。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

## 6.4. 共享库

如果port安装了一个或多个共享库，请定义一个 USE_LDCONFIG 的 make 变量，这将指导 bsd.port.mk 在安装新库的目录（通常为 PREFIX/lib）上运行 ${LDCONFIG} -m ，以便在 post-install 目标期间将其注册到共享库缓存中。当定义了这个变量时，还将有助于在 pkg-plist 中添加适当的 @exec /sbin/ldconfig -m 和 @unexec /sbin/ldconfig -R 对，以便安装了该软件包的用户可以立即开始使用共享库，卸载不会导致系统仍然认为库仍然存在。

```
USE_LDCONFIG=	yes
```

默认目录可以通过将 USE_LDCONFIG 设置为要安装共享库的目录列表来覆盖。例如，如果port将共享库安装到 PREFIX/lib/foo 和 PREFIX/lib/bar，请在 Makefile 中使用此命令：

```
USE_LDCONFIG=	${PREFIX}/lib/foo ${PREFIX}/lib/bar
```

请仔细检查，通常这根本不必要，或者可以通过 -rpath 或在链接过程中设置 LD_RUN_PATH 来避免（请参阅 lang/mosml 中的示例），或者通过一个在调用二进制文件之前设置 LD_LIBRARY_PATH 的shell包装器，就像 www/seamonkey 一样。

在 64 位系统上安装 32 位库时，请使用 USE_LDCONFIG32 。

如果软件使用 autotools，并且特别是 libtool ，请添加 USES=libtool 。

当主要库版本号在更新到新版时递增时，所有其他链接到受影响库的ports必须递增，以强制使用新库版本重新编译。

## 6.5. Ports带有分发限制或法律问题

许可证各不相同，其中一些对应用程序的打包方式、是否可出售以获取利润等方面设置了限制。

|  | 作为一个 porter 的责任是阅读软件的许可条款，并确保 FreeBSD 项目不会因通过 FTP/HTTP 或 CD-ROM 分发源代码或编译后的二进制文件而违反这些条款。如有疑问，请联系 FreeBSD ports 邮件列表。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

在这种情况下，可以设置下一节中描述的变量。

### 6.5.1. `NO_PACKAGE`

此变量表示我们可能不会生成应用程序的二进制软件包。例如，许可证可能禁止二进制重新分发，或者可能禁止从打补丁的源代码创建的软件包的分发。

但是，port的 DISTFILES 可以在 FTP/HTTP 上自由镜像。除非 NO_CDROM 也被设置，否则它们也可以在 CD-ROM（或类似媒体）上分发。

如果二进制软件包没有普遍用途，并且应用程序必须始终从源代码编译，请使用 NO_PACKAGE 。例如，如果应用程序在编译时将站点特定的配置信息硬编码到其中，请设置 NO_PACKAGE 。

将 NO_PACKAGE 设置为描述软件包无法生成原因的字符串。

### 6.5.2. `NO_CDROM`

单独这个变量表明，尽管我们可以生成二进制软件包，但我们既不能将这些软件包也不能将port的 DISTFILES 放入光盘（或类似媒体）进行转售。然而，二进制软件包和port的 DISTFILES 仍然可以通过 FTP/HTTP 获得。

如果设置了这个变量以及 NO_PACKAGE ，那么只有port的 DISTFILES 将可用，并且只能通过 FTP/HTTP 获得。

将 NO_CDROM 设置为描述port无法在光盘上重新分发的原因的字符串。例如，如果port的许可证仅限于“非商业”使用，请使用此选项。

### 6.5.3. `NOFETCHFILES`

在 NOFETCHFILES 中定义的文件不能从任何 MASTER_SITES 获取。此类文件的一个示例是供应商通过 CD-ROM 提供的文件。

检查这些文件在 MASTER_SITES 上的可用性的工具必须忽略这些文件并且不报告它们。

### 6.5.4. `RESTRICTED`

如果应用程序的许可证既不允许镜像应用程序的 DISTFILES 也不允许以任何方式分发二进制包，则单独设置此变量。

不要设置 NO_CDROM 或 NO_PACKAGE 以及 RESTRICTED ，因为后一个变量意味着前面的变量。

将 RESTRICTED 设置为描述为什么port 不能重新分发的字符串。通常，这表示port 包含专有软件，用户需要手动下载 DISTFILES ，可能需要在注册软件或同意接受 EULA 条款后进行。

### 6.5.5. `RESTRICTED_FILES`

当设置 RESTRICTED 或 NO_CDROM 时，此变量默认为 ${DISTFILES} ${PATCHFILES} ，否则为空。如果仅限制一些发布文件，则将此变量设置为列出它们。

### 6.5.6. `LEGAL_TEXT`

如果port存在未在上述变量中解决的法律问题，请将 LEGAL_TEXT 设置为解释该问题的字符串。例如，如果为了 FreeBSD 重新分发二进制文件而获得了特殊许可，则此变量必须指示如此。

### 6.5.7. /usr/ports/LEGAL 和 LEGAL

设置任何上述变量的port也必须添加到/usr/ports/LEGAL。第一列是匹配受限制的分发文件的通配符。第二列是port的来源。第三列是 make -VLEGAL 的输出。

### 6.5.8。 示例

陈述“此 port 的 distfiles 必须手动获取”的首选方式如下：

```
.if !exists(${DISTDIR}/${DISTNAME}${EXTRACT_SUFX})
IGNORE=	may not be redistributed because of licensing reasons. Please visit some-website to accept their license and download ${DISTFILES} into ${DISTDIR}
.endif
```

这既通知用户，又为自动化程序在用户机器上设置正确的元数据。

请注意，此段必须由 bsd.port.pre.mk 的包含形式先导。

## 6.6. 构建机制

### 6.6.1. 并行构建 Ports

FreeBSD ports 框架支持使用多个 make 子进程进行并行构建，这允许 SMP 系统利用所有可用的 CPU 力量，从而使 port 构建更快更有效。

通过在运行供应商代码的 make(1) 上添加 -jX 标志来实现这一目标。这是 ports 的默认构建行为。不幸的是，并非所有 ports 都能很好地处理并行构建，可能需要通过添加 MAKE_JOBS_UNSAFE=yes 变量来显式禁用此功能。当已知某个 port 由于竞争条件导致间歇性构建失败时，就会使用它。

|  | 在设置 MAKE_JOBS_UNSAFE 时，非常重要的是要在 Makefile 中进行注释说明，或者至少在提交消息中解释为什么启用时 port 无法构建。否则，在以后的更新提交时，要么无法修复问题，要么无法测试是否已修复问题，几乎是不可能的。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### 6.6.2. make ， gmake 和 imake

存在几种不同的 make 实现。移植软件通常需要特定的实现，如 GNU make ，在 FreeBSD 中称为 gmake 。

如果port使用 GNU make，请将 gmake 添加到 USES 。

MAKE_CMD 可以用来引用在 port 的 Makefile 中配置的特定命令。 只在应用程序 Makefiles 中使用 MAKE_CMD ，以调用在移植软件中期望的 make 实现。

如果 port 是一个使用 imake 从 Imakefiles 创建 Makefiles 的 X 应用程序，请设置 USES= imake 。 查看《使用 USES 宏》一节以获取更多详细信息。

如果 port 的源 Makefile 中除了 all 以外还有其他主要构建目标，请相应设置 ALL_TARGET 。 对于 install 和 INSTALL_TARGET 也是如此。

### 6.6.3. configure 脚本

如果 port 使用 configure 脚本从 Makefile.in 生成 Makefile，请设置 GNU_CONFIGURE=yes 。要向 configure 脚本传递额外参数（默认参数为 --prefix=${PREFIX} --infodir=${PREFIX}/${INFO_PATH} --mandir=${PREFIX}/man --build=${CONFIGURE_TARGET} ），请在 CONFIGURE_ARGS 中设置这些额外参数。可以使用 CONFIGURE_ENV 传递额外环境变量。

表 1. 使用 configure 的 Ports 变量

| 变量 | 意味着                                                                |
| ------ | ----------------------------------------------------------------------- |
| `GNU_CONFIGURE`     | port 使用 configure 脚本准备构建。                                    |
| `HAS_CONFIGURE`     | 与 GNU_CONFIGURE 相同，只是默认配置目标不添加到 CONFIGURE_ARGS 中。   |
| `CONFIGURE_ARGS`     | 传递给 configure 脚本的附加参数。                                     |
| `CONFIGURE_ENV`     | 用于 configure 脚本运行的附加环境变量。                               |
| `CONFIGURE_TARGET`     | 重写默认配置目标。默认值为 ${MACHINE_ARCH}-portbld-freebsd${OSREL} 。 |

### 6.6.4. 使用 cmake

对于使用 CMake 的 ports，定义 USES= cmake 。

Ports使用的 cmake 变量表 2 变量

| 变量 | 手段                                                                                                 |
| ------ | ------------------------------------------------------------------------------------------------------ |
| `CMAKE_ARGS`     | 要传递给二进制文件 cmake 的Port 特定 CMake 标志。                                                    |
| `CMAKE_ON`     | 对于 CMAKE_ON 中的每个条目，将添加一个启用布尔值到 CMAKE_ARGS 。看 CMAKE\_ON 和 CMAKE\_OFF 。  |
| `CMAKE_OFF`     | 对于 CMAKE_OFF 中的每个条目，将添加一个禁用布尔值到 CMAKE_ARGS 。看 CMAKE\_ON 和 CMAKE\_OFF 。 |
| `CMAKE_BUILD_TYPE`     | 构建类型（CMake 预定义的构建配置文件）。默认为 Release ，如果设置了 WITH_DEBUG ，则为 Debug 。       |
| `CMAKE_SOURCE_PATH`     | 源目录路径。默认为 ${WRKSRC} 。                                                                      |
| `CONFIGURE_ENV`     | 要设置的其他环境变量与 cmake 二进制文件相关。                                                        |

cmake 建造的用户可定义的变量表 3

| 变量 | 意味                                                                      |
| ------ | --------------------------------------------------------------------------- |
| `CMAKE_NOCOLOR`     | 禁用彩色构建输出。默认情况下未设置，除非设置 BATCH 或 PACKAGE_BUILDING 。 |

CMake 支持这些构建配置文件： Debug ， Release ， RelWithDebInfo 和 MinSizeRel 。 Debug 和 Release 配置文件遵循系统 *FLAGS ， RelWithDebInfo 和 MinSizeRel 将分别设置 CFLAGS 为 -O2 -g 和 -Os -DNDEBUG 。 CMAKE_BUILD_TYPE 的小写值被导出到 PLIST_SUB ，如果port根据构建类型安装*.cmake（请参阅 devel/kf5-kcrash 的示例）。请注意，一些项目可能定义自己的构建配置文件和/或通过在 CMakeLists.txt 中设置 CMAKE_BUILD_TYPE 来强制特定的构建类型。为了使这样一个项目的port遵循 CFLAGS 和 WITH_DEBUG ，这些文件中的 CMAKE_BUILD_TYPE 定义必须被移除。

大多数基于 CMake 的项目支持一种离源代码构建的方法。对于port，离源代码构建是默认设置。可以通过使用 :insource 后缀来请求源代码内构建。使用离源代码构建时， CONFIGURE_WRKSRC ， BUILD_WRKSRC 和 INSTALL_WRKSRC 将被设置为 ${WRKDIR}/.build ，并且该目录将用于保存在配置和构建阶段生成的所有文件，保持源代码目录不变。

 示例 2. USES= cmake 示例

此片段演示了使用 CMake 构建 port 的方法。通常情况下不需要 CMAKE_SOURCE_PATH ，但当源代码不位于顶级目录中，或者只打算构建项目的子集时，可以设置 port。

```
USES=			cmake
CMAKE_SOURCE_PATH=	${WRKSRC}/subproject
```

示例 3. CMAKE_ON 和 CMAKE_OFF

当向 CMAKE_ARGS 添加布尔值时，最好使用 CMAKE_ON 和 CMAKE_OFF 变量。这样做会更容易：

```
CMAKE_ON=	VAR1 VAR2
CMAKE_OFF=	VAR3
```

 等同于：

```
CMAKE_ARGS=	-DVAR1:BOOL=TRUE -DVAR2:BOOL=TRUE -DVAR3:BOOL=FALSE
```

|  | 这仅适用于 CMAKE_ARGS 的默认值。 OPT\_CMAKE\_BOOL 和 OPT\_CMAKE\_BOOL\_OFF 中描述的辅助程序使用相同的语义，但适用于可选值。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------- |

### 使用 scons 6.6.5。

如果 port 使用 SCons，请定义 USES=scons 。

要使第三方 SConstruct 尊重通过环境传递给 SCons 的所有内容（即，最重要的是 CC/CXX/CFLAGS/CXXFLAGS ），请修补 SConstruct，以便构建 Environment 如下所示：

```
env = Environment(**ARGUMENTS)
```

它可以然后通过 env.Append 和 env.Replace 进行修改。

### 6.6.6. 用 cargo 构建 Rust 应用程序

对于使用 Cargo 的ports，定义 USES=cargo 。

用户可以为 cargo 构建定义的变量表 4

| 变量 | 默认        | 描述                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------ | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CARGO_CRATES`     |             | 列出port依赖的箱子列表。每个条目都需要像 cratename-semver 这样的格式，例如， libc-0.2.40 。Port的维护者可以使用 make cargo-crates 从 Cargo.lock 生成此列表。手动提升箱子版本是可能的，但要注意传递依赖关系。如果 make cargo-crates 生成的列表很大，可能将其放在顶层port目录中的 Makefile.crates 文件中更方便。如果存在，ports框架会自动源自该文件。这有助于保持主port Makefile 处于可管理的大小范围内。 |
| `CARGO_FEATURES`     |             | 要构建的应用功能列表（以空格分隔）。要禁用所有默认功能，请向 CARGO_FEATURES 添加特殊令牌 --no-default-features 。手动传递给 CARGO_BUILD_ARGS ， CARGO_INSTALL_ARGS 和 CARGO_TEST_ARGS 不是必需的。                                                                                                                                                                                                      |
| `CARGO_CARGOTOML`     | `${WRKSRC}/Cargo.toml`            | 使用的 Cargo.toml 路径。                                                                                                                                                                                                                                                                                                                                                                                |
| `CARGO_CARGOLOCK`     | `${WRKSRC}/Cargo.lock`            | 用于 make cargo-crates 的 Cargo.lock 路径。必要时可以指定多个锁定文件。                                                                                                                                                                                                                                                                                                                                 |
| `CARGO_ENV`     |             | 要传递给 Cargo 类似 MAKE_ENV 的环境变量列表。                                                                                                                                                                                                                                                                                                                                                           |
| `RUSTFLAGS`     |             | 传递给 Rust 编译器的标志。                                                                                                                                                                                                                                                                                                                                                                              |
| `CARGO_CONFIGURE`     | `yes`            | 使用默认的 do-configure 。                                                                                                                                                                                                                                                                                                                                                                              |
| `CARGO_UPDATE_ARGS`     |             | 在配置阶段传递给 Cargo 的额外参数。可以使用 cargo update --help 查找有效参数。                                                                                                                                                                                                                                                                                                                          |
| `CARGO_BUILDDEP`     | `yes`            | 在 lang/rust 上添加构建依赖。                                                                                                                                                                                                                                                                                                                                                                           |
| `CARGO_CARGO_BIN`     | `${LOCALBASE}/bin/cargo`            | cargo 二进制文件的位置。                                                                                                                                                                                                                                                                                                                                                                                |
| `CARGO_BUILD`     | `yes`            | 使用默认 do-build 。                                                                                                                                                                                                                                                                                                                                                                                    |
| `CARGO_BUILD_ARGS`     |             | 在构建阶段传递给 Cargo 的额外参数。可以使用 cargo build --help 查找有效参数。                                                                                                                                                                                                                                                                                                                           |
| `CARGO_INSTALL`     | `yes`            | 使用默认 do-install 。                                                                                                                                                                                                                                                                                                                                                                                  |
| `CARGO_INSTALL_ARGS`     |             | 在安装阶段传递给 Cargo 的额外参数。可以使用 cargo install --help 查找有效参数。                                                                                                                                                                                                                                                                                                                         |
| `CARGO_INSTALL_PATH`     | `.`            | 安装木筏的路径。 这通过它的 --path 参数传递给 cargo install 。 当指定多个路径时， cargo install 会多次运行。                                                                                                                                                                                                                                                                                            |
| `CARGO_TEST`     | `yes`            | 使用默认的 do-test 。                                                                                                                                                                                                                                                                                                                                                                                   |
| `CARGO_TEST_ARGS`     |             | 在测试阶段传递给货物的额外参数。 可以使用 cargo test --help 查找有效参数。                                                                                                                                                                                                                                                                                                                              |
| `CARGO_TARGET_DIR`     | `${WRKDIR}/target`            | 货物输出目录的位置。                                                                                                                                                                                                                                                                                                                                                                                    |
| `CARGO_DIST_SUBDIR`     | rust/crates | 相对于 DISTDIR 的目录，将存储板块分发文件。                                                                                                                                                                                                                                                                                                                                                             |
| `CARGO_VENDOR_DIR`     | `${WRKSRC}/cargo-crates`            | 存放供应商目录的位置，所有的板条箱都将被提取到这里。尽量保持在 PATCH_WRKSRC 以下，这样可以更容易地应用补丁。                                                                                                                                                                                                                                                                                            |
| `CARGO_USE_GITHUB`     | `no`            | 通过 GH_TUPLE 启用对 GitHub 上特定 Git 提交锁定的板条箱的提取。这将尝试对所有 Cargo.toml 进行修补，使其指向离线源而不是在构建过程中从 Git 存储库获取它们。                                                                                                                                                                                                                                              |
| `CARGO_USE_GITLAB`     | `no`            | 与 CARGO_USE_GITHUB 相同，但适用于 GitLab 实例和 GL_TUPLE 。                                                                                                                                                                                                                                                                                                                                            |

创建一个简单的 Rust 应用程序Port的示例 4。

创建基于 Cargo 的port是一个三阶段的过程。首先，我们需要提供一个ports模板，用于获取应用程序分发文件：

```
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

生成初始的 distinfo：

```
% make makesum
=> Aaronepower-tokei-v7.0.2_GH0.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch https://codeload.github.com/Aaronepower/tokei/tar.gz/v7.0.2?dummy=/Aaronepower-tokei-v7.0.2_GH0.tar.gz
fetch: https://codeload.github.com/Aaronepower/tokei/tar.gz/v7.0.2?dummy=/Aaronepower-tokei-v7.0.2_GH0.tar.gz: size of remote file is not known
Aaronepower-tokei-v7.0.2_GH0.tar.gz                     45 kB  239 kBps 00m00s
```

现在分发文件已准备就绪，我们可以继续从捆绑的 Cargo.lock 中提取箱依赖项：

```
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

```
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

需要重新生成 distinfo 以包含所有箱分发文件：

```
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

port现在已准备好进行测试构建和进一步调整，如创建 plist，撰写描述，添加许可信息，选项等等，一切如常。

如果您没有在像 poudriere 这样的干净环境中测试您的port，请记得在任何测试之前运行 make clean 。

示例 5. 启用额外的应用程序功能

一些应用程序在其 Cargo.toml 中定义了额外的功能。可以通过在 port 中设置 CARGO_FEATURES 来进行编译。

在这里，我们启用 Tokei 的 json 和 yaml 功能：

```
CARGO_FEATURES=	json yaml
```

示例 6. 将应用程序功能编码为 Port 选项。

Cargo.toml 中的一个示例 [features] 部分可能如下所示：

```
[features]
pulseaudio_backend = ["librespot-playback/pulseaudio-backend"]
portaudio_backend = ["librespot-playback/portaudio-backend"]
default = ["pulseaudio_backend"]
```

pulseaudio_backend 是一个默认功能。除非我们通过向 CARGO_FEATURES 添加 --no-default-features 来明确关闭默认功能，否则它始终启用。在这里，我们将 portaudio_backend 和 pulseaudio_backend 功能转换为 port 选项：

```
CARGO_FEATURES=	--no-default-features

OPTIONS_DEFINE=	PORTAUDIO PULSEAUDIO

PORTAUDIO_VARS=		CARGO_FEATURES+=portaudio_backend
PULSEAUDIO_VARS=	CARGO_FEATURES+=pulseaudio_backend
```

示例 7. 列出 Crate 许可证

包有它们自己的许可证。在向 LICENSE 块添加到 port 时，了解它们是很重要的（请参阅许可证）。助手目标 cargo-crates-licenses 将尝试列出在 CARGO_CRATES 中定义的所有包的所有许可证。

```
% make cargo-crates-licenses
aho-corasick-0.6.4  Unlicense/MIT
ansi_term-0.11.0    MIT
arrayvec-0.4.7      MIT/Apache-2.0
atty-0.2.9          MIT
bitflags-1.0.1      MIT/Apache-2.0
byteorder-1.2.2     Unlicense/MIT
[...]
```

|  | 许可证名称 make cargo-crates-licenses 输出是 SPDX 2.1 许可证表达式，与 ports 框架中定义的许可证名称不匹配。它们需要被翻译为预定义许可证列表中的名称。 |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |

### 使用 meson 6.6.7。

对于使用 Meson 的 ports，定义 USES=meson 。

表 5. 使用 meson 的 Ports 的变量

| 变量 | 描述                                               |
| ------ | ---------------------------------------------------- |
| `MESON_ARGS`     | 要传递给 meson 二进制文件的 Port 特定 Meson 标志。 |
| `MESON_BUILD_DIR`     | 相对于 WRKSRC 的构建目录路径。默认为 _build 。     |

 示例 8. USES=meson 示例

此片段演示了使用 Meson 进行 port 的用法。

```
USES=		meson
MESON_ARGS=	-Dfoo=enabled
```

### 6.6.8. 构建 Go 应用程序

对于使用 Go 的ports，定义 USES=go 。有关可以设置以控制构建过程的变量列表，请参阅 go 。

示例 9. 为基于 Go 模块的应用程序创建Port。

在大多数情况下，将 GO_MODULE 变量设置为 go.mod 中 module 指令指定的值就足够了。

```
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

如果“简单”方法不够或需要更多对依赖项的控制，则下面描述了完整的移植过程。

创建基于 Go 的 port 是一个五阶段的过程。首先，我们需要提供一个 ports 模板，用于获取应用程序分发文件：

```
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

生成初始的 distinfo：

```
% make makesum
===>  License MIT accepted by the user
=> motemen-ghq-v0.12.5_GH0.tar.gz doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch https://codeload.github.com/motemen/ghq/tar.gz/v0.12.5?dummy=/motemen-ghq-v0.12.5_GH0.tar.gz
fetch: https://codeload.github.com/motemen/ghq/tar.gz/v0.12.5?dummy=/motemen-ghq-v0.12.5_GH0.tar.gz: size of remote file is not known
motemen-ghq-v0.12.5_GH0.tar.gz                          32 kB  177 kBps    00s
```

现在分发文件已准备就绪，我们可以提取所需的 Go 模块依赖项。此步骤需要安装ports-mgmt/modules2tuple：

```
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

此命令的输出需要直接粘贴到 Makefile 中：

```
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

需要重新生成 distinfo 以包含所有分发文件：

```
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

port现在已准备好进行测试构建和进一步调整，如创建 plist，编写描述，添加许可信息，选项等等，与正常操作一样。

如果您没有在像 poudriere 这样的干净环境中测试您的port，请记得在任何测试之前运行 make clean 。

示例 10. 设置输出二进制名称或安装路径

一些 ports 需要将结果二进制文件安装到不同名称或默认路径以外的路径。这可以通过使用 GO_TARGET 元组语法来实现，例如：

```
GO_TARGET=  ./cmd/ipfs:ipfs-go
```

将 ipfs 二进制文件安装为 ${PREFIX}/bin/ipfs-go

```
GO_TARGET=  ./dnscrypt-proxy:${PREFIX}/sbin/dnscrypt-proxy
```

将 dnscrypt-proxy 安装到 ${PREFIX}/sbin

### 使用 cabal 来构建 Haskell 应用程序的 6.6.9

对于使用 Cabal 的ports，构建系统定义了 USES=cabal 。请参考 cabal 以查看可设置以控制构建过程的变量列表。

示例 11. 为一个 Hackage 托管的 Haskell 应用程序创建Port

在准备 Haskell Cabal port 时，需要安装 devel/hs-cabal-install 和 ports-mgmt/hs-cabal2tuple 程序，因此请确保事先安装它们。首先，我们需要定义通用的 ports 变量，以便 cabal-install 可以获取软件包分发文件：

```
PORTNAME=	ShellCheck
DISTVERSION=	0.6.0
CATEGORIES=	devel

MAINTAINER=	haskell@FreeBSD.org
COMMENT=	Shell script analysis tool
WWW=		https://www.shellcheck.net/

USES=		cabal

.include <bsd.port.mk>
```

这个最小的 Makefile 使用 cabal-extract 辅助目标来获取分发文件：

```
% make cabal-extract
[...]
Downloading the latest package list from hackage.haskell.org
cabal get ShellCheck-0.6.0
Downloading  ShellCheck-0.6.0
Downloaded   ShellCheck-0.6.0
Unpacking to ShellCheck-0.6.0/
```

现在我们在 ${WRKSRC} 下有 ShellCheck.cabal 软件包描述文件，我们可以使用 cabal-configure 来生成构建计划：

```
% make cabal-configure
[...]
Resolving dependencies...
Build profile: -w ghc-8.10.7 -O1
In order, the following would be built (use -v for more details):
 - Diff-0.4.1 (lib) (requires download & build)
 - OneTuple-0.3.1 (lib) (requires download & build)
[...]
```

一旦完成，可以生成所需依赖项列表：

```
% make make-use-cabal
USE_CABAL=	QuickCheck-2.12.6.1 \
		hashable-1.3.0.0 \
		integer-logarithms-1.0.3 \
[...]
```

Haskell 软件包可能包含修订版本，就像 FreeBSD ports 一样。修订版本只会影响 .cabal 文件。请注意 _ 符号后面的附加版本号。请将新生成的 USE_CABAL 列表放在旧列表的位置。

最后，需要重新生成 distinfo 以包含所有分发文件：

```
% make makesum
=> ShellCheck-0.6.0.tar.gz doesn't seem to exist in /usr/local/poudriere/ports/git/distfiles/cabal.
=> Attempting to fetch https://hackage.haskell.org/package/ShellCheck-0.6.0/ShellCheck-0.6.0.tar.gz
ShellCheck-0.6.0.tar.gz                                136 kB  642 kBps    00s
=> QuickCheck-2.12.6.1/QuickCheck-2.12.6.1.tar.gz doesn't seem to exist in /usr/local/poudriere/ports/git/distfiles/cabal.
=> Attempting to fetch https://hackage.haskell.org/package/QuickCheck-2.12.6.1/QuickCheck-2.12.6.1.tar.gz
QuickCheck-2.12.6.1/QuickCheck-2.12.6.1.tar.gz          65 kB  361 kBps    00s
[...]
```

port现在已准备好进行测试构建和进一步调整，如创建 plist，编写描述，添加许可信息，选项等等，一切如常。

如果您没有在像 poudriere 这样的干净环境中测试您的port，请记得在任何测试之前运行 make clean 。

一些 Haskell ports 在 share/${PORTNAME} 下安装各种数据文件。对于这种情况，需要在port端进行特殊处理。port应定义 CABAL_WRAPPER_SCRIPTS 旋钮，列出每个将使用数据文件的可执行文件。此外，在罕见情况下，被移植程序使用其他 Haskell 软件包的数据文件，这时 FOO_DATADIR_VARS 就派上用场了。

示例 12. 在 Haskell 中处理数据文件 Port

devel/hs-profiteur 是一个 Haskell 应用程序，用于生成带有一些内容的单页面 HTML。

```
PORTNAME=	profiteur

[...]

USES=		cabal

USE_CABAL=	OneTuple-0.3.1_2 \
		QuickCheck-2.14.2 \
		[...]

.include <bsd.port.mk>
```

它在 share/profiteur 下安装 HTML 模板，因此我们需要添加 CABAL_WRAPPER_SCRIPTS 旋钮：

```
[...]

USE_CABAL=	OneTuple-0.3.1_2 \
		QuickCheck-2.14.2 \
		[...]


CABAL_WRAPPER_SCRIPTS=		${CABAL_EXECUTABLES}

.include <bsd.port.mk>
```

该程序还尝试访问 jquery.js 文件，该文件是 js-jquery-3.3.1 Haskell 软件包的一部分。为了找到该文件，我们需要使包装脚本也查找 share/profiteur 中的 js-jquery 数据文件。我们用 profiteur_DATADIR_VARS 来实现这一点。

```
[...]

CABAL_WRAPPER_SCRIPTS=		${CABAL_EXECUTABLES}
profiteur_DATADIR_VARS=		js-jquery

.include <bsd.port.mk>
```

现在port将实际的二进制文件安装到 libexec/cabal/profiteur 中，并将脚本安装到 bin/profiteur 中。

除了运行该程序并检查一切正常之外，没有更容易找到 FOO_DATADIR_VARS 旋钮的正确值的方法。幸运的是，使用 FOO_DATADIR_VARS 的需求非常罕见。

在移植复杂的 Haskell 程序时，另一个边缘情况是 cabal.project 文件中存在 VCS 依赖项。

示例 13. 使用 VCS 依赖项移植 Haskell 应用程序

net-p2p/cardano-node 是一款极其复杂的软件。在其 cabal.project 中有很多类似这样的块：

```
[...]
source-repository-package
  type: git
  location: https://github.com/input-output-hk/cardano-crypto
  tag: f73079303f663e028288f9f4a9e08bcca39a923e
[...]
```

类型 source-repository-package 的依赖项在构建过程中会被 cabal 自动拉取。不幸的是，这会在 fetch 阶段之后使用网络。这是 ports 框架不允许的。这些来源需要在 port 的 Makefile 中列出。 make-use-cabal 辅助目标可以使托管在 GitHub 上的软件包变得更容易。在通常的 cabal-extract 和 cabal-configure 之后运行此目标将不仅生成 USE_CABAL 开关，还会生成 GH_TUPLE ：

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

从 make-use-cabal 获取的 GH_TUPLE 项目与其他项目分开可能很有用，以便更容易更新 port：

```
GH_TUPLE=	input-output-hk:cardano-base:0f3a867493059e650cda69e20a5cbf1ace289a57:cardano_base/dist-newstyle/src/cardano-b_-c8db9876882556ed \
		input-output-hk:cardano-crypto:f73079303f663e028288f9f4a9e08bcca39a923e:cardano_crypto/dist-newstyle/src/cardano-c_-253fd88117badd8f \
		[...]

GH_TUPLE+=	bitcoin-core:secp256k1:ac83be33d0956faf6b7f61a60ab524ef7d6a473a:secp
```

Haskell ports 与 VCS 依赖项还需要暂时使用以下技巧：

```
BINARY_ALIAS=	git=true
```

## 6.7. 使用 GNU Autotools

如果 port 需要任何 GNU Autotools 软件，请添加 USES=autoreconf 。查看 autoreconf 获取更多信息。

## 6.8. 使用 GNU gettext

### 6.8.1. 基本用法

如果 port 需要 gettext ，设置 USES= gettext ，port 将从 devel/gettext 继承对 libintl.so 的依赖。 gettext 的其他值在 USES=gettext 中列出。

一个相当常见的情况是 port 使用 gettext 和 configure 。通常，GNU configure 应该能够自动定位 gettext 。

```
USES=	gettext
GNU_CONFIGURE=	yes
```

如果它无法成功，可以通过以下方式传递 gettext 的位置提示：在 CPPFLAGS 和 LDFLAGS 中使用 localbase 。

```
USES=	gettext localbase:ldflags
GNU_CONFIGURE=	yes
```

### 6.8.2. 可选用法

一些软件产品允许禁用 NLS。例如，通过将 --disable-nls 传递给 configure 。在这种情况下，port必须有条件地使用 gettext ，具体取决于 NLS 选项的状态。对于低到中等复杂度的ports，请使用这种习惯用语：

```
GNU_CONFIGURE=		yes

OPTIONS_DEFINE=		NLS
OPTIONS_SUB=		yes

NLS_USES=		gettext
NLS_CONFIGURE_ENABLE=	nls

.include <bsd.port.mk>
```

或者使用使用选项的较旧方法：

```
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

待办事项清单上的下一项是要安排消息目录文件在打包清单中有条件地包含。 此任务的 Makefile 部分已由该习惯提供。 在高级 pkg-plist 实践部分有解释。 简而言之，pkg-plist 中的每个 %%NLS%% 出现将在 pkg-plist 中消息目录文件的每个路径之前用" @comment " if NLS is disabled, or by a null string if NLS is enabled. Consequently, the lines prefixed by %%NLS%%will become mere comments in the final packing list if NLS is off; otherwise the prefix will be just left out. Then insert%%NLS%% 替换。 例如：

```
%%NLS%%share/locale/fr/LC_MESSAGES/foobar.mo
%%NLS%%share/locale/no/LC_MESSAGES/foobar.mo
```

在复杂情况下，可能需要更先进的技术，如动态打包清单生成。

### 6.8.3. 处理消息目录目录

安装消息目录文件有一个需要注意的地方。它们的目标目录，位于 LOCALBASE/share/locale 下，不得由port创建和移除。最流行的语言有它们各自的目录列在 PORTSDIR/Templates/BSD.local.dist 中。许多其他语言的目录受 devel/gettext port管理。查看其 pkg-plist，并查看port是否将为某种独特语言安装消息目录文件。

## 6.9. 使用 Perl

如果 MASTER_SITES 设置为 CPAN ，通常会自动选择正确的子目录。如果默认子目录错误，可以使用 CPAN/Module 进行更改。 MASTER_SITES 也可以设置为旧的 MASTER_SITE_PERL_CPAN ，然后 MASTER_SITE_SUBDIR 的首选值是顶层层次结构名称。例如， p5-Module-Name 的推荐值是 Module 。可以在 cpan.org 上查看顶层层次结构。这样可以在模块作者更改时保持 port 的正常工作。

例外情况是相关目录不存在或者在该目录中不存在 distfile 时。在这种情况下，允许使用作者的 id 作为 MASTER_SITE_SUBDIR 。可以使用 CPAN:AUTHOR 宏，它将被转换为散列作者目录。例如， CPAN:AUTHOR 将被转换为 authors/id/A/AU/AUTHOR 。

当 port 需要 Perl 支持时，必须使用 perl5 USES 描述中描述的可选 USE_PERL5 设置 USES=perl5 。

表 6. 使用 Perl 的 Ports 只读变量

| 只读变量 | 方法                                                                                                                                                                 |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PERL`         | Perl 5 解释器的完整路径，可以是系统中的或者从软件包中安装的，但不包括版本号。当软件需要 Perl 解释器的路径时使用此路径。要替换脚本中的 " #! "行，请使用 shebangfix 。 |
| `PERL_VERSION`         | 已安装的 Perl 完整版本（例如， 5.8.9 ）。                                                                                                                            |
| `PERL_LEVEL`         | 已安装的 Perl 版本为形式为 MNNNPP 的整数（例如， 500809 ）。                                                                                                         |
| `PERL_ARCH`         | 存储架构相关库的位置。默认为 ${ARCH}-freebsd .                                                                                                                       |
| `PERL_PORT`         | 安装的 Perl port的名称（例如， perl5 ）。                                                                                                                            |
| `SITE_PERL`         | 站点特定 Perl 软件包所在的目录名。此值将添加到 PLIST_SUB 中。                                                                                                        |

|  | Perl 模块中没有官方网站的Ports必须在 Makefile 的 WWW 行中链接到 cpan.org 。首选的 URL 形式是 https://search.cpan.org/dist/Module-Name/ （包括尾部斜杠）。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |

|  | 不要在依赖声明中使用 ${SITE_PERL} 。这样做假定已经包含了 perl5.mk，而这并不总是正确的。如果此port的文件在升级后移动，依赖于此port的Ports将具有不正确的依赖关系。声明 Perl 模块依赖关系的正确方法如下例所示。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

示例 14. Perl 依赖示例

```
p5-IO-Tee>=0.64:devel/p5-IO-Tee
```

对于安装手册页的 Perl ports，在 pkg-plist 中可以使用宏 PERL5_MAN3 和 PERL5_MAN1 。例如，

```
lib/perl5/5.14/man/man1/event.1.gz
lib/perl5/5.14/man/man3/AnyEvent::I3.3.gz
```

可以替换为

```
%%PERL5_MAN1%%/event.1.gz
%%PERL5_MAN3%%/AnyEvent::I3.3.gz
```

|  | 对于其他部分（x 在 2 和 4 到 9 中）没有 PERL5_MAN_x_ 宏，因为这些会安装在常规目录中。 |
| -- | --------------------------------------------------------------------------------------- |

例 15. 一个 Port 只需要 Perl 来构建

由于默认的 USE_PERL5 值是构建和运行，将其设置为：

```
USES=		perl5
USE_PERL5=	build
```

例 16. 一个 Port 也需要 Perl 来修补

有时候，仅仅使用 sed(1)进行补丁不够。当使用 perl(1)更容易时，请使用：

```
USES=		perl5
USE_PERL5=	patch build run
```

例子 17. 一个需要 ExtUtils::MakeMaker 构建的 Perl 模块

大多数 Perl 模块附带 Makefile.PL 配置脚本。在这种情况下，请设置：

```
USES=		perl5
USE_PERL5=	configure
```

示例 18. 一个需要 Module::Build 构建的 Perl 模块

当一个 Perl 模块带有一个 Build.PL 配置脚本时，它可能需要 Module::Build，在这种情况下，设置

```
USES=		perl5
USE_PERL5=	modbuild
```

如果它需要 Module::Build::Tiny，设置

```
USES=		perl5
USE_PERL5=	modbuildtiny
```

## 6.10. 使用 X11

### 6.10.1. X.Org 组件

The X11 实现在 The Ports Collection 中提供的是 X.Org。如果应用程序依赖于 X 组件，请添加 USES= xorg 并将 USE_XORG 设置为所需组件列表中的一部分。完整列表可以在 xorg 中找到。

Mesa 项目是提供免费的 OpenGL 实现的努力。要指定对该项目各组件的依赖关系，请使用 USES= gl 和 USE_GL 。请查看 gl 以获取可用组件的完整列表。为了向后兼容， yes 的值映射到 glu 。

例 19. USE_XORG 示例

```
USES=		gl xorg
USE_GL=		glu
USE_XORG=	xrender xft xkbfile xt xaw
```

表 7. 使用 X 的 Ports 变量

| `USES= imake` | port 使用 imake 。                                          |
| -- | ------------------------------------------------------------- |
| `XMKMF` | 如果不在 PATH 中，则设置为 xmkmf 的路径。默认为 xmkmf -a 。 |

示例 20. 使用 X11 相关变量

```
# Use some X11 libraries
USES=		xorg
USE_XORG=	x11 xpm
```

### 6.10.2. Ports 需要 Motif 的

如果 port 需要 Motif 库，请在 Makefile 中定义 USES= motif 。默认的 Motif 实现是 x11-toolkits/open-motif。用户可以通过在他们的 make.conf 中设置 WANT_LESSTIF 来选择 x11-toolkits/lesstif。类似地，x11-toolkits/open-motif-devel 可以通过在 make.conf 中设置 WANT_OPEN_MOTIF_DEVEL 来选择。

MOTIFLIB 将被 motif.mk 设置为引用适当的 Motif 库。请对 port 的源代码进行补丁，以便在原始 Makefile 或 Imakefile 中引用 Motif 库的地方使用 ${MOTIFLIB} 。

有两种常见情况：

* 如果port在其 Makefile 或 Imakefile 中将 Motif 库称为 -lXm ，请用 ${MOTIFLIB} 替换它。
* 如果port在其 Imakefile 中使用 XmClientLibs ，请将其更改为 ${MOTIFLIB} ${XTOOLLIB} ${XLIB} 。

注意， MOTIFLIB （通常）会扩展到 -L/usr/local/lib -lXm -lXp 或 /usr/local/lib/libXm.a ，因此无需在前面添加 -L 或 -l 。

### 6.10.3. X11 字体

如果 port 安装 X Window 系统的字体，请将它们放在 LOCALBASE/lib/X11/fonts/local 中。

### 6.10.4. 使用 Xvfb 获取虚假 DISPLAY

一些应用程序需要一个正常工作的 X11 显示来确保编译成功。这对于无头操作的机器构成了问题。当使用这个变量时，构建基础设施将启动虚拟帧缓冲 X 服务器。然后将工作的 DISPLAY 传递给构建。查看 USES=display 以获取可能的参数。

```
USES=	display
```

### 6.10.5. 桌面条目

桌面条目（Freedesktop 标准）提供一种在安装新程序时自动调整桌面功能的方式，无需用户干预。例如，新安装的程序会自动出现在兼容桌面环境的应用程序菜单中。桌面条目起源于 GNOME 桌面环境，但现在是一个标准，并且也适用于 KDE 和 Xfce。这种自动化提供了真正的用户福利，鼓励将桌面条目用于可以在桌面环境中使用的应用程序。

#### 6.10.5.1. 使用预定义的 .desktop 文件

Ports 包括预定义的 *.desktop 文件的软件包必须在 pkg-plist 中包含这些文件，并将它们安装在 $LOCALBASE/share/applications 目录中。 INSTALL_DATA 宏对于安装这些文件很有用。

#### 6.10.5.2. 更新桌面数据库

如果port在其 portname.desktop 中有 MimeType 条目，则在安装和卸载后必须更新桌面数据库。要做到这一点，请定义 USES = desktop-file-utils。

#### 6.10.5.3. 使用 DESKTOP_ENTRIES 创建桌面条目

使用 DESKTOP_ENTRIES 可以轻松地为应用程序创建桌面条目。将创建一个名为 name.desktop 的文件，该文件将自动安装并添加到 pkg-plist 中。语法为：

```
DESKTOP_ENTRIES=	"NAME" "COMMENT" "ICON" "COMMAND" "CATEGORY" StartupNotify
```

可在 Freedesktop 网站上找到可能的类别列表。 StartupNotify 指示应用程序是否与启动通知兼容。这些通常是图形指示器，比如一个在鼠标指针、菜单或面板上出现的时钟，用于向用户指示程序何时正在启动。与启动通知兼容的程序在启动后会清除指示器。不兼容启动通知的程序永远不会清除指示器（可能会使用户感到困惑和愤怒），必须将 StartupNotify 设为 false 以便根本不显示指示器。

 示例:

```
DESKTOP_ENTRIES=	"ToME" "Roguelike game based on JRR Tolkien's work" \
			"${DATADIR}/xtra/graf/tome-128.png" \
			"tome -v -g" "Application;Game;RolePlaying;" \
			false
```

DESKTOP_ENTRIES 被安装在变量指向的目录中。 DESKTOPDIR 默认为${PREFIX}/share/applications

## 6.11. 使用 GNOME

### 6.11.1. 介绍

本章节解释了 ports 使用的 GNOME 框架。该框架可以大致分为基本组件、GNOME 桌面组件以及一些简化 port 维护者工作的特殊宏。

### 6.11.2. 使用 USE_GNOME

将此变量添加到 port 允许使用在 bsd.gnome.mk 中定义的宏和组件。bsd.gnome.mk 中的代码添加了所需的构建时、运行时或库依赖项，或者处理特殊文件。FreeBSD 下的 GNOME 应用程序使用 USE_GNOME 基础设施。将所有所需组件作为以空格分隔的列表包含。这些 USE_GNOME 组件被分为基本组件、GNOME 3 组件和遗留组件这些虚拟列表。如果 port 只需要 GTK3 库，这是定义的最简单方式：

```
USE_GNOME=	gtk30
```

USE_GNOME 组件会自动添加它们所需的依赖关系。请查看 GNOME 组件，了解所有 USE_GNOME 组件及它们隐含的其他组件以及它们的依赖关系的详尽列表。

这是一个示例 Makefile，用于一个使用了本文档中概述的许多技术的 GNOME port。请将其用作创建新 ports 的指南。

```
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

|  | 不带任何参数的 USE_GNOME 宏不会向 port 添加任何依赖关系。 USE_GNOME 不能在 bsd.port.pre.mk 之后设置。 |
| -- | ------------------------------------------------------------------------------------------------------- |

### 6.11.3。变量

本节解释了可用的宏及其如何使用。就像它们在上面的例子中使用的那样。 GNOME 组件有更深入的解释。 必须为这些宏设置 USE_GNOME 才能发挥作用。

GLIB_SCHEMAS 所有 glib 模式文件的列表port 安装。该宏将添加这些文件到{{1002} plist，并处理这些文件在安装和卸载中的注册。

glib 模式文件是用 XML 编写的，并以 gschema.xml 扩展名结尾。它们安装在 share/glib-2.0/schemas/目录中。这些模式文件包含所有应用程序配置值及其默认设置。应用程序实际使用的数据库是由 glib-compile-schema 构建的，该程序由 GLIB_SCHEMAS 宏运行。

```
GLIB_SCHEMAS=foo.gschema.xml
```

|  | 不要将 glib 模式添加到 pkg-plist 中。如果它们在 pkg-plist 中列出，它们将不会被注册，应用程序可能无法正常工作。 |
| -- | ---------------------------------------------------------------------------------------------------------------- |

GCONF_SCHEMAS 列出所有 gconf 模式文件。该宏将向port plist 添加模式文件，并在安装和卸载时处理其注册。

GConf 是基于 XML 的数据库，几乎所有的 GNOME 应用程序都用它来存储它们的设置。这些文件被安装到 etc/gconf/schemas 目录中。这个数据库是由安装的模式文件定义的，这些文件用于生成%gconf.xml 关键文件。对于port安装的每个模式文件，Makefile 中必须有一个条目：

```
GCONF_SCHEMAS=my_app.schemas my_app2.schemas my_app3.schemas
```

|  | Gconf 模式列在 GCONF_SCHEMAS 宏中，而不是 pkg-plist 中。如果它们在 pkg-plist 中列出，它们将不会被注册，应用程序可能无法正常工作。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------- |

INSTALLS_OMF 开放源码元数据框架（OMF）文件通常被 GNOME 2 应用程序使用。这些文件包含应用程序帮助文件信息，并需要通过 ScrollKeeper/rarian 进行特殊处理。在从软件包安装 GNOME 应用程序时正确注册 OMF 文件，请确保 omf 文件在 pkg-plist 中列出，并且port的 Makefile 中定义了 INSTALLS_OMF ：

```
INSTALLS_OMF=yes
```

当设置了 bsd.gnome.mk 后，自动扫描 pkg-plist，并为每个 .omf 添加适当的 @exec 和 @unexec 指令，以便在 OMF 注册数据库中跟踪。

## 6.12. GNOME 组件

若要进一步了解关于 GNOME port 的帮助，请查看一些现有的 ports 以获取示例。如果需要更多帮助，FreeBSD GNOME 页面上有联系信息。这些组件分为当前正在使用的 GNOME 组件和旧版组件。如果组件支持参数，则在描述中将它们列在括号内。第一个是默认设置。“Both” 是指组件默认添加到构建和运行依赖项中。

GNOME 组件表 8

| 组件 | 相关程序                                              | 描述                                                                                                                          |
| ------ | ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `atk`     | accessibility/atk                                     | 无障碍工具包 (ATK)                                                                                                            |
| `atkmm`     | 辅助功能/atkmm                                        | atk 的 C++绑定                                                                                                                |
| `cairo`     | 图形/cairo                                            | 具有跨设备输出支持的矢量图形库                                                                                                |
| `cairomm`     | graphics/cairomm                                      | 用于 cairo 的 C++绑定                                                                                                         |
| `dconf`     | 开发/dconf                                            | 配置数据库系统（构建、运行）                                                                                                  |
| `evolutiondataserver3`     | 数据库/evolution-data-server                          | 用于 Evolution 集成邮件/PIM 套件的数据后端                                                                                    |
| `gdkpixbuf2`     | 图形/gdk-pixbuf2                                      | 用于 GTK+ 的图形库                                                                                                            |
| `glib20`     | 开发/glib20                                           | GNOME 核心库 glib20                                                                                                           |
| `glibmm`     | 开发/glibmm                                           | 为 glib20 提供的 C++ 绑定                                                                                                     |
| `gnomecontrolcenter3`     | sysutils/gnome-control-center                         | GNOME 3 控制中心                                                                                                              |
| `gnomedesktop3`     | x11/gnome-desktop                                     | GNOME 3 桌面 UI 库                                                                                                            |
| `gsound`     | audio/gsound                                          | 用于播放系统声音的 GObject 库（构建和运行）                                                                                   |
| `gtk-update-icon-cache`     | 图形/gtk-update-icon-cache                            | 来自 Gtk+ 工具包的 Gtk-update-icon-cache 实用程序                                                                             |
| `gtk20`     | x11-toolkits/gtk20                                    | Gtk+ 2 工具包                                                                                                                 |
| `gtk30`     | x11-toolkits/gtk30                                    | Gtk+ 3 工具包                                                                                                                 |
| `gtkmm20`     | x11-toolkits/gtkmm20                                  | c++ 为 gtk20 工具包提供的绑定 2.0                                                                                             |
| `gtkmm24`     | x11-toolkits/gtkmm24                                  | 为 gtk20 工具包提供的 c++绑定 2.4                                                                                             |
| `gtkmm30`     | x11-toolkits/gtkmm30                                  | C++绑定 3.0 用于 gtk30 工具包                                                                                                 |
| `gtksourceview2`     | x11 工具包/gtksourceview2                             | 向 GtkTextView 添加语法高亮显示的小部件                                                                                       |
| `gtksourceview3`     | GtkTextView 小部件，为 GtkTextView 小部件添加语法高亮 | 在 GtkTextView 小部件添加语法高亮的文本小部件                                                                                 |
| `gtksourceviewmm3`     | GtkTextView 小部件，为 GtkTextView 小部件添加语法高亮 | C++绑定用于 gtksourceview3 库                                                                                                 |
| `gvfs`     | devel/gvfs                                            | GNOME 虚拟文件系统                                                                                                            |
| `intltool`     | 国际化工具（另请参阅 intlhack）                       | 国际化工具（另请参阅 intlhack）                                                                                               |
| `introspection`     | gobject-introspection 的开发                          | 基本的内省绑定和生成内省绑定工具。大多数情况下：构建足够了，:both/:run 只需要用于使用内省绑定的应用程序。（both, build, run） |
| `libgda5`     | 数据库/libgda5                                        | 提供对不同类型数据源的统一访问                                                                                                |
| `libgda5-ui`     | 数据库/libgda5-ui                                     | 来自 libgda5 库的 UI 库                                                                                                       |
| `libgdamm5`     | 数据库/libgdamm5                                      | 用于 libgda5 库的 c++ 绑定                                                                                                    |
| `libgsf`     | devel/libgsf                                          | 用于处理结构化文件格式的可扩展 I/O 抽象                                                                                       |
| `librsvg2`     | 图形/ librsvg2                                        | 用于解析和渲染 SVG 矢量图形文件的库                                                                                           |
| `libsigc++20`     | devel/libsigc++20                                     | C++的回调框架                                                                                                                 |
| `libxml++26`     | textproc/libxml++26                                   | libxml2 库的 C++绑定                                                                                                          |
| `libxml2`     | XML 解析器库（构建和运行）                            | XML 解析器库（构建和运行）                                                                                                    |
| `libxslt`     | 文本处理/libxslt                                      | XSLT C 库（构建、运行）                                                                                                       |
| `metacity`     | x11-wm/metacity                                       | 来自 GNOME 的窗口管理器                                                                                                       |
| `nautilus3`     | 文件管理器                                            | GNOME 文件管理器                                                                                                              |
| `pango`     | Pango 工具包                                          | 用于布局和呈现 i18n 文本的开源框架                                                                                            |
| `pangomm`     | x11 工具包/pangomm                                    | pango 库的 c++绑定                                                                                                            |
| `py3gobject3`     | Python 3，GObject 3.0 绑定                            | Python 3，GObject 3.0 绑定                                                                                                    |
| `pygobject3`     | Python 3，GObject 3.0 绑定                            | Python 2，GObject 3.0 绑定                                                                                                    |
| `vte3`     | x11-toolkits/vte3                                     | 具有改进的辅助功能和 I18N 支持的终端部件                                                                                      |

GNOME 宏组件表 9

| 组件 | 描述                                                                             |
| ------ | ---------------------------------------------------------------------------------- |
| `gnomeprefix`     | 为 configure 提供一些默认位置。                                                  |
| `intlhack`     | 与 intltool 相同，但补丁确保使用 share/locale/。请仅在 intltool 单独不足时使用。 |
| `referencehack`     | 此宏用于帮助将 API 或参考文档拆分为其自己的 port。                               |

桌面 10. GNOME 传统组件

| 组件 | 相关程序                          | 描述                                      |
| ------ | ----------------------------------- | ------------------------------------------- |
| `atspi`     | 辅助功能/at-spi                   | 辅助技术服务提供者接口                    |
| `esound`     | 音频/esound                       | 启蒙音效包                                |
| `gal2`     | x11 工具包/gal2                   | 从 GNOME 2 gnumeric 中提取的小部件集合    |
| `gconf2`     | devel/gconf2                      | 用于 GNOME 2 的配置数据库系统             |
| `gconfmm26`     | 开发/gconfmm26                    | 用于 gconf2 的 C++绑定                    |
| `gdkpixbuf`     | 图形/gdk-pixbuf                   | GTK+的图形库                              |
| `glib12`     | devel/glib12                      | glib 1.2 核心库                           |
| `gnomedocutils`     | 文本处理/ GNOME 文档实用程序      | GNOME 文档实用程序                        |
| `gnomemimedata`     | 杂项/ GNOME MIME 数据             | GNOME 2 的 MIME 和应用程序数据库          |
| `gnomesharp20`     | x11-toolkits/gnome-sharp20        | 用于.NET 运行时的 GNOME 2 界面            |
| `gnomespeech`     | 辅助功能/ GNOME 语音              | GNOME 2 文字转语音 API                    |
| `gnomevfs2`     | devel/gnome-vfs                   | GNOME 2 虚拟文件系统                      |
| `gtk12`     | x11-toolkits/gtk12                | Gtk+ 1.2 工具包                           |
| `gtkhtml3`     | 轻量级 HTML 渲染/打印/编辑引擎    | 轻量级 HTML 渲染/打印/编辑引擎            |
| `gtkhtml4`     | www/gtkhtml4                      | 輕巧型 HTML 渲染/打印/編輯引擎            |
| `gtksharp20`     | x11-toolkits/gtk-sharp20          | .NET 執行階段的 GTK+ 和 GNOME 2 介面      |
| `gtksourceview`     | 包含语法高亮显示的小部件          | 在 GtkTextView 中添加语法高亮显示的小部件 |
| `libartgpl2`     | 图形库 art_lgpl                   | 高性能 2D 图形库                          |
| `libbonobo`     | devel/libbonobo                   | 用于 GNOME 2 的组件和复合文档系统         |
| `libbonoboui`     | x11-toolkits/libbonoboui          | GNOME 2 的 libbonobo 组件的 GUI 前端      |
| `libgda4`     | databases/libgda4                 | 提供对不同类型数据源的统一访问            |
| `libglade2`     | devel/libglade2                   | GNOME 2 glade 库                          |
| `libgnome`     | x11/libgnome                      | 用于 GNOME 2 的库，一个 GNU 桌面环境      |
| `libgnomecanvas`     | graphics/libgnomecanvas           | GNOME 2 图形库                            |
| `libgnomekbd`     | x11/libgnomekbd                   | GNOME 2 键盘共享库                        |
| `libgnomeprint`     | 打印/libgnomeprint                | Gnome 2 打印支持库                        |
| `libgnomeprintui`     | x11 工具包/libgnomeprintui        | Gnome 2 打印支持库                        |
| `libgnomeui`     | x11-toolkits/libgnomeui           | 用于 GNOME 2 GUI 的库，一个 GNU 桌面环境  |
| `libgtkhtml`     | 轻量级 HTML 渲染/打印/编辑引擎    | 轻量级 HTML 渲染/打印/编辑引擎            |
| `libgtksourceviewmm`     | x11-toolkits/libgtksourceviewmm   | GtkSourceView 的 C++绑定                  |
| `libidl`     | devel/libIDL                      | 用于创建 CORBA IDL 文件树的库             |
| `libsigc++12`     | C++回调框架                       | C++回调框架                               |
| `libwnck`     | x11-toolkits/libwnck              | 用于编写分页和任务列表的库                |
| `libwnck3`     | x11 工具包/libwnck3               | 用于编写分页和任务列表的库                |
| `orbit2`     | 高级性能的 CORBA ORB，支持 C 语言 | 高性能 CORBA ORB，支持 C 语言             |
| `pygnome2`     | x11 工具包/py-gnome2              | GNOME 2 的 Python 绑定                    |
| `pygobject`     | devel/py-gobject                  | Python 2，GObject 2.0 绑定                |
| `pygtk2`     | x11-toolkits/py-gtk2              | 用于 GTK+的 Python 绑定集                 |
| `pygtksourceview`     | x11-toolkits/py-gtksourceview     | Python 绑定用于 GtkSourceView 2           |
| `vte`     | x11 工具包/vte                    | 具有改进辅助功能和 I18N 支持的终端部件    |

弃用组件表 11：请勿使用

| 组件 | 描述                                                |
| ------ | ----------------------------------------------------- |
| `pangox-compat`     | pangox-compat 已被弃用并从 pango 软件包中拆分出来。 |

## 6.13. 使用 Qt

|  | 对于 Qt 本身的ports，请参见 qt-dist 。 |
| -- | ---------------------------------------- |

### 6.13.1. Ports 需要 Qt

The Ports 集合为 Qt 5 和 Qt 6 提供支持，分别使用 USES+=qt:5 和 USES+=qt:6 。将 USE_QT 设置为所需的 Qt 组件列表（库、工具、插件）。

Qt 框架导出了一些变量，ports可以使用其中一些，以下是其中一些列出的变量：

表 12. 提供给Ports使用 Qt 的变量

| `QMAKE` | qmake 二进制文件的完整路径。 |
| -- | ------------------------------ |
| `LRELEASE` | lrelease 工具的完整路径。    |
| `MOC` | 完整路径到 moc 。            |
| `RCC` | 完整路径到 rcc 。            |
| `UIC` | 完整路径到 uic 。            |
| `QT_INCDIR` | Qt 包含目录。                |
| `QT_LIBDIR` | Qt 库路径。                  |
| `QT_PLUGINDIR` | Qt 插件路径。                |

### 6.13.2. 组件选择

每个 Qt 工具和库依赖项必须在 USE_QT 中指定。每个组件都可以加上 _build 或 _run 后缀，后缀表示对组件的依赖是在构建时还是运行时。如果没有后缀，该组件将在构建时和运行时都有依赖。通常，库组件被指定为无后缀，工具组件大多使用 _build 后缀，插件组件使用 _run 后缀。下面列出了最常用的组件（所有可用组件都列在 _USE_QT_ALL 中，该列表是从 _USE_QT_COMMON 和 _USE_QT[56]_ONLY 生成的，位于/usr/ports/Mk/Uses/qt.mk 中）：

表 13. 可用的 Qt 库组件

| 名称 | 描述                                                |
| ------ | ----------------------------------------------------- |
| `3d`     | Qt3D 模块                                           |
| `5compat`     | Qt 6 的 Qt 5 兼容模块                               |
| `assistant`     | Qt 5 文档浏览器                                     |
| `base`     | Qt 6 基础模块                                       |
| `canvas3d`     | Qt 画布 3D 模块                                     |
| `charts`     | Qt 5 图表模块                                       |
| `concurrent`     | Qt 多线程模块                                       |
| `connectivity`     | Qt 连接性（蓝牙/NFC）模块                           |
| `core`     | Qt 核心非图形模块                                   |
| `datavis3d`     | Qt 5 3D 数据可视化模块                              |
| `dbus`     | Qt D-Bus 进程间通信模块                             |
| `declarative`     | Qt 声明性框架用于动态用户界面                       |
| `designer`     | Qt 5 图形用户界面设计师                             |
| `diag`     | 用于报告有关 Qt 及其环境的诊断信息的工具            |
| `doc`     | Qt 5 文档                                           |
| `examples`     | Qt 5 示例源代码                                     |
| `gamepad`     | Qt 5 游戏手柄模块                                   |
| `graphicaleffects`     | Qt Quick 图形效果                                   |
| `gui`     | Qt 图形用户界面模块                                 |
| `help`     | Qt 在线帮助集成模块                                 |
| `l10n`     | Qt 本地化消息                                       |
| `languageserver`     | Qt 6 语言服务器协议实现                             |
| `linguist`     | Qt 5 翻译工具                                       |
| `location`     | Qt 位置模块                                         |
| `lottie`     | Qt 6 QML API 用于渲染图形和动画                     |
| `multimedia`     | Qt 音频、视频、收音机和相机支持模块                 |
| `network`     | Qt 网络模块                                         |
| `networkauth`     | Qt 网络认证模块                                     |
| `opengl`     | Qt 5 兼容的 OpenGL 支持模块                         |
| `paths`     | QStandardPaths 的命令行客户端                       |
| `phonon4`     | KDE 多媒体框架                                      |
| `pixeltool`     | Qt 5 屏幕放大镜                                     |
| `plugininfo`     | Qt 5 插件元数据转储程序                             |
| `positioning`     | 从卫星、wifi 或文本文件等来源获取的 Qt 6 定位 API。 |
| `printsupport`     | Qt 打印支持模块                                     |
| `qdbus`     | Qt 命令行接口到 D-Bus                               |
| `qdbusviewer`     | Qt 5 图形界面到 D-Bus                               |
| `qdoc`     | Qt 文档生成器                                       |
| `qdoc-data`     | QDoc 配置文件                                       |
| `qev`     | Qt QWidget 事件内省工具                             |
| `qmake`     | Qt Makefile 生成器                                  |
| `quickcontrols`     | 用于在 Qt Quick 中构建完整界面的控件集              |
| `quickcontrols2`     | 用于在 Qt Quick 中构建完整界面的控件集              |
| `remoteobjects`     | Qt 5 SXCML 模块                                     |
| `script`     | Qt 4 兼容脚本模块                                   |
| `scripttools`     | Qt 脚本附加组件                                     |
| `scxml`     | Qt 5 SXCML 模块                                     |
| `sensors`     | Qt 传感器模块                                       |
| `serialbus`     | Qt 访问工业总线系统的功能                           |
| `serialport`     | 用于访问串行ports的 Qt 函数                         |
| `shadertools`     | 用于跨平台 Qt 着色器流水线的 Qt 6 工具              |
| `speech`     | Qt5 的无障碍功能                                    |
| `sql`     | Qt SQL 数据库集成模块                               |
| `sql-ibase`     | Qt InterBase/Firebird 数据库插件                    |
| `sql-mysql`     | Qt MySQL 数据库插件                                 |
| `sql-odbc`     | Qt 开放数据库连接插件                               |
| `sql-pgsql`     | Qt PostgreSQL 数据库插件                            |
| `sql-sqlite2`     | Qt SQLite 2 数据库插件                              |
| `sql-sqlite3`     | Qt SQLite 3 数据库插件                              |
| `sql-tds`     | Qt TDS 数据库连接数据库插件                         |
| `svg`     | Qt SVG 支持模块                                     |
| `testlib`     | Qt 单元测试模块                                     |
| `tools`     | Qt 6 各种工具                                       |
| `translations`     | Qt 6 翻译模块                                       |
| `uiplugin`     | 定制 Qt 小部件插件接口用于 Qt Designer              |
| `uitools`     | Qt Designer UI 表单支持模块                         |
| `virtualkeyboard`     | Qt 5 虚拟键盘模块                                   |
| `wayland`     | 用于 Wayland 的 Qt 5 封装器                         |
| `webchannel`     | 用于将 C++/QML 与 HTML/js 客户端集成的 Qt 5 库      |
| `webengine`     | 用于渲染网页内容的 Qt 5 库                          |
| `webkit`     | 具有更现代的 WebKit 代码库的 QtWebKit               |
| `websockets`     | WebSocket 协议的 Qt 实现                            |
| `websockets-qml`     | WebSocket 协议的 Qt 实现（QML 绑定）                |
| `webview`     | 用于显示网络内容的 Qt 组件                          |
| `widgets`     | Qt C++小部件模块                                    |
| `x11extras`     | 面向基于 X11 系统的 Qt 平台特定功能                 |
| `xml`     | Qt SAX 和 DOM 实现                                  |
| `xmlpatterns`     | Qt 对 XPath、XQuery、XSLT 和 XML Schema 的支持      |

要确定应用程序依赖的库，请在成功编译后的主可执行文件上运行 ldd 。

可用的 Qt 工具组件表 14。

| 名称 | 描述                                                 |
| ------ | ------------------------------------------------------ |
| `buildtools`     | 构建工具 ( moc , rcc ), 几乎每个 Qt 应用程序都需要。 |
| `linguisttools`     | 本地化工具: lrelease , lupdate                       |
| `qmake`     | Makefile 生成器/构建实用程序                         |

可用的 Qt 插件组件表 15

| 名称 | 描述                                 |
| ------ | -------------------------------------- |
| `imageformats`     | 用于 TGA、TIFF 和 MNG 图像格式的插件 |

示例 21. 选择 Qt 5 组件

在这个例子中，移植的应用程序使用了 Qt 5 图形用户界面库、Qt 5 核心库、所有 Qt 5 代码生成工具和 Qt 5 的 Makefile 生成器。由于 gui 库意味着对核心库的依赖，因此不需要指定 core 。Qt 5 代码生成工具 moc 、 uic 和 rcc ，以及 Makefile 生成器 qmake 仅在构建时需要，因此它们带有 _build 后缀指定：

```
USES=	qt:5
USE_QT=	gui buildtools_build qmake_build
```

### 6.13.3. 使用 qmake

如果应用程序提供一个 qmake 项目文件 (*.pro)，则应同时定义 USES= qmake 和 USE_QT 。 USES= qmake 已经暗示了对 qmake 的构建依赖，因此可以省略 qmake 组件。类似于 CMake，qmake 支持在源码目录之外进行构建，可以通过指定 outsource 参数来启用 (请参见 USES= qmake 示例)。还请参阅 USES= qmake 的可能参数。

表 16. USES= qmake 的可能参数

| 变量 | 描述                                                                                                                                                   |
| ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `no_configure`     | 不要添加配置目标。这是由 HAS_CONFIGURE=yes 和 GNU_CONFIGURE=yes 隐含的。当构建仅需要来自 USES= qmake 的环境设置时，它是必需的，否则会独立运行 qmake 。 |
| `no_env`     | 抑制对配置和构建环境的修改。仅在使用 qmake 配置软件且构建无法理解 USES= qmake 设置的环境时才需要。                                                     |
| `norecursive`     | 不要将 -recursive 参数传递给 qmake 。                                                                                                                  |
| `outsource`     | 执行一个外部源码构建。                                                                                                                                 |

表格 17. Ports 使用的变量

| 变量 | 描述                                                                              |
| ------ | ----------------------------------------------------------------------------------- |
| `QMAKE_ARGS`     | 要传递给 qmake 二进制文件的 Port 特定的 qmake 标志。                              |
| `QMAKE_ENV`     | 要为 qmake 二进制文件设置的环境变量。默认值为 ${CONFIGURE_ENV} 。                 |
| `QMAKE_SOURCE_PATH`     | qmake 项目文件 (.pro) 的路径。如果请求了外部构建，则默认为 ${WRKSRC} ，否则为空。 |

在使用 USES= qmake 时，这些设置会被部署：

```
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

一些配置脚本不支持上述参数。要抑制 CONFIGURE_ENV 和 CONFIGURE_ARGS 的修改，设置 USES= qmake:no_env 。

示例 22. USES= qmake 示例

这段代码演示了如何在 Qt 5 中使用 qmake port：

```
USES=	qmake:outsource qt:5
USE_QT=	buildtools_build
```

Qt 应用程序通常是跨平台编写的，通常不是在 X11/Unix 平台上开发，这导致某些问题，比如：

* 缺少额外的包含路径。许多应用程序支持系统托盘图标，但忽略在 X11 目录中查找包含和/或库。要通过命令行将目录添加到`qmake`的包含和库搜索路径中，请使用：

  ```
  QMAKE_ARGS+=	INCLUDEPATH+=${LOCALBASE}/include \
  		LIBS+=-L${LOCALBASE}/lib
  ```
* 虚假的安装路径。有时，诸如图标或 .desktop 文件之类的数据默认安装到 XDG 兼容应用程序未扫描的目录中。editors/texmaker 就是一个例子 - 查看 port 的文件目录中的 patch-texmaker.pro 模板，了解如何在 qmake 项目文件中直接解决此问题。

## 6.14. 使用 KDE

### 6.14.1. KDE 变量定义

如果应用程序依赖于 KDE，请将 USES+=kde:5 和 USE_KDE 设置为所需组件列表。可以使用 _build 和 _run 后缀来强制组件依赖类型（例如， baseapps_run ）。如果未设置后缀，则将使用默认的依赖类型。要强制使用两种类型，请将组件两次添加，每次都使用后缀（例如， ecm_build ecm_run ）。可用组件列在下面（最新组件也列在/usr/ports/Mk/Uses/kde.mk 中）：

表 18. 可用的 KDE 组件

| 名称 | 描述                                                  |
| ------ | ------------------------------------------------------- |
| `activities`     | 用于在单独的活动中组织工作的 KF5 运行时和库           |
| `activities-stats`     | 用于活动的 KF5 统计                                   |
| `activitymanagerd`     | 系统服务，用于管理用户活动，跟踪使用模式              |
| `akonadi`     | 用于 KDE-Pim 的存储服务器                             |
| `akonadicalendar`     | Akonadi 日历集成                                      |
| `akonadiconsole`     | Akonadi 管理和调试控制台                              |
| `akonadicontacts`     | 用于在 Akonadi 中实现联系人管理的库和守护程序         |
| `akonadiimportwizard`     | 从其他邮件客户端导入数据到 KMail                      |
| `akonadimime`     | 实现基本电子邮件处理的库和守护程序                    |
| `akonadinotes`     | 用于访问 MBox 格式邮件存储的 KDE 库                   |
| `akonadisearch`     | 在 Akonadi 中实现搜索的库和守护程序                   |
| `akregator`     | 由 KDE 提供的订阅阅读器                               |
| `alarmcalendar`     | 用于 KAlarm 闹钟的 KDE API                            |
| `apidox`     | KF5 API 文档工具                                      |
| `archive`     | 处理存档格式的类的 KF5 库                             |
| `attica`     | 开放协作服务 API 库 KDE5 版本                         |
| `attica5`     | 开放协作服务 API 库 KDE5 版本                         |
| `auth`     | KF5 抽象到系统策略和认证功能                          |
| `baloo`     | KF5 框架用于搜索和管理用户元数据                      |
| `baloo-widgets`     | BalooWidgets 库                                       |
| `baloo5`     | 用于搜索和管理用户元数据的 KF5 框架                   |
| `blog`     | 用于网络日志访问的 KDE API                            |
| `bookmarks`     | 用于书签和 XBEL 格式的 KF5 库                         |
| `breeze`     | Breeze 可视样式的 Plasma5 艺术品、风格和素材          |
| `breeze-gtk`     | Gtk 的 Plasma5 Breeze 可视样式                        |
| `breeze-icons`     | 用于 KDE 的 Breeze 图标主题                           |
| `calendarcore`     | KDE 日历访问库                                        |
| `calendarsupport`     | 用于 KDEPim 的日历支持库                              |
| `calendarutils`     | 用于访问日历的 KDE 实用程序和用户界面函数             |
| `codecs`     | 用于字符串操作的 KF5 库                               |
| `completion`     | KF5 文本完成助手和小部件                              |
| `config`     | 用于配置对话框的 KF5 小部件                           |
| `configwidgets`     | 配置对话框的 KF5 小部件                               |
| `contacts`     | 管理联系信息的 KDE API                                |
| `coreaddons`     | 用于 QtCore 的 KF5 附加组件                           |
| `crash`     | KF5 库用于处理应用程序崩溃分析和错误报告              |
| `dbusaddons`     | KF5 附加组件到 QtDBus                                 |
| `decoration`     | Plasma5 库用于创建窗口装饰                            |
| `designerplugin`     | 在 Qt Designer/Creator 中集成 Frameworks 小部件的 KF5 |
| `discover`     | Plasma5 软件包管理工具                                |
| `dnssd`     | KF5 对系统 DNSSD 功能的抽象                           |
| `doctools`     | 从 docbook 生成 KF5 文档                              |
| `drkonqi`     | Plasma5 崩溃处理程序                                  |
| `ecm`     | 用于 CMake 的额外模块和脚本                           |
| `emoticons`     | 用于转换表情符号的 KF5 库                             |
| `eventviews`     | 用于 KDEPim 的事件视图库                              |
| `filemetadata`     | 用于提取文件元数据的 KF5 库                           |
| `frameworkintegration`     | KF5 工作区和跨框架集成插件                            |
| `gapi`     | 基于 KDE 的库，用于访问谷歌服务                       |
| `globalaccel`     | KF5 库，用于增加全局工作区快捷键支持                  |
| `grantlee-editor`     | Grantlee 主题编辑器                                   |
| `grantleetheme`     | KDE PIM grantleetheme                                 |
| `gravatar`     | 为 gravatar 支持提供库                                |
| `guiaddons`     | QtGui 的 KF5 插件                                     |
| `holidays`     | 日历节假日的 KDE 库                                   |
| `hotkeys`     | 热键的 Plasma5 库                                     |
| `i18n`     | KF5 高级国际化框架                                    |
| `iconthemes`     | 用于处理应用程序中图标的 KF5 库                       |
| `identitymanagement`     | KDE pim 身份                                          |
| `idletime`     | 用于监控用户活动的 KF5 库                             |
| `imap`     | 用于 IMAP 支持的 KDE API                              |
| `incidenceeditor`     | 用于 KDEPim 的事件编辑库                              |
| `infocenter`     | 提供系统信息的 Plasma5 实用程序                       |
| `init`     | 用于加速启动 KDE 应用程序的 KF5 进程启动器            |
| `itemmodels`     | 用于 Qt Model/View 系统的 KF5 模型                    |
| `itemviews`     | 为 Qt Model/View 提供的 KF5 小部件附加组件            |
| `jobwidgets`     | 用于跟踪 KJob 实例的 KF5 小部件                       |
| `js`     | 提供 ECMAScript 解释器的 KF5 库                       |
| `jsembed`     | 用于将 JavaScript 对象绑定到 QObjects 的 KF5 库       |
| `kaddressbook`     | KDE 联系人管理器                                      |
| `kalarm`     | 个人闹钟调度程序                                      |
| `kalarm`     | 个人报警计划程序                                      |
| `kate`     | KDE 系统的基础编辑器框架                              |
| `kcmutils`     | 用于处理 KCModules 的 KF5 工具                        |
| `kde-cli-tools`     | Plasma5 非交互式系统工具                              |
| `kde-gtk-config`     | Plasma5 GTK2 和 GTK3 配置器                           |
| `kdeclarative`     | KF5 库，提供 QML 和 KDE 框架集成                      |
| `kded`     | 为提供系统级服务的 KF5 可扩展守护程序                 |
| `kdelibs4support`     | 从 KDELibs4 移植辅助工具的 KF5 版本                   |
| `kdepim-addons`     | KDE PIM 插件                                          |
| `kdepim-apps-libs`     | KDE PIM 邮件相关库                                    |
| `kdepim-runtime5`     | KDE PIM 工具和服务                                    |
| `kdeplasma-addons`     | 用于改善 Plasma 体验的 Plasma5 附加组件               |
| `kdesu`     | KF5 与 su 进行提升权限集成                            |
| `kdewebkit`     | KF5 库提供 QtWebKit 集成                              |
| `kgamma5`     | Plasma5 监视器的伽马设置                              |
| `khtml`     | KF5 的 KTHML 渲染引擎                                 |
| `kimageformats`     | KF5 库提供支持额外图像格式                            |
| `kio`     | KF5 的资源和网络访问抽象化                            |
| `kirigami2`     | 基于 QtQuick 的组件集                                 |
| `kitinerary`     | 用于旅行预订信息的数据模型和提取系统                  |
| `kmail`     | KDE 邮件客户端                                        |
| `kmail`     | KDE 邮件客户端                                        |
| `kmail-account-wizard`     | KDE 邮件帐户向导                                      |
| `kmenuedit`     | Plasma5 菜单编辑器                                    |
| `knotes`     | 弹出式注释                                            |
| `kontact`     | KDE 个人信息管理器                                    |
| `kontact`     | KDE 个人信息管理器                                    |
| `kontactinterface`     | 嵌入到 Kontact 中的 KParts 的 KDE 粘合剂              |
| `korganizer`     | 日历和日程安排程序                                    |
| `kpimdav`     | 带有 KJobs 的 DAV 协议实现                            |
| `kpkpass`     | 处理苹果钱包通行文件的库                              |
| `kross`     | KF5 多语言应用程序脚本                                |
| `kscreen`     | Plasma5 屏幕管理库                                    |
| `kscreenlocker`     | Plasma5 安全锁屏架构                                  |
| `ksmtp`     | 通过 SMTP 服务器发送电子邮件的基于工作的库            |
| `ksshaskpass`     | Plasma5 ssh-add 前端                                  |
| `ksysguard`     | 用于跟踪和控制运行中进程的 Plasma5 实用程序           |
| `kwallet-pam`     | Plasma5 KWallet PAM 集成                              |
| `kwayland-integration`     | 用于基于 Wayland 的桌面的集成插件                     |
| `kwin`     | Plasma5 窗口管理器                                    |
| `kwrited`     | Plasma5 守护程序监听墙和写消息                        |
| `ldap`     | 用于 KDE 的 LDAP 访问 API                             |
| `libkcddb`     | KDE CDDB 库                                           |
| `libkcompactdisc`     | 用于与音频 CD 交互的 KDE 库                           |
| `libkdcraw`     | 用于 KDE 的 LibRaw 接口                               |
| `libkdegames`     | 由 KDE 游戏使用的库                                   |
| `libkdepim`     | KDE PIM 库                                            |
| `libkeduvocdocument`     | 用于读写词汇文件的库                                  |
| `libkexiv2`     | 用于 KDE 的 Exiv2 库接口                              |
| `libkipi`     | KDE 镜像插件接口                                      |
| `libkleo`     | 用于 KDE 的证书管理器                                 |
| `libksane`     | KDE 的 SANE 库接口                                    |
| `libkscreen`     | Plasma5 屏幕管理库                                    |
| `libksieve`     | KDEPim 的 Sieve 库                                    |
| `libksysguard`     | 用于跟踪和控制运行进程的 Plasma5 库                   |
| `mailcommon`     | 用于 KDEPim 的常用库                                  |
| `mailimporter`     | 将 mbox 文件导入 KMail                                |
| `mailtransport`     | 管理邮件传输的 KDE 库                                 |
| `marble`     | 用于 KDE 的虚拟地球和世界地图                         |
| `mbox`     | 用于访问 MBox 格式邮件存储的 KDE 库                   |
| `mbox-importer`     | 将 mbox 文件导入 KMail                                |
| `mediaplayer`     | 用于媒体播放器功能的 KF5 插件接口                     |
| `messagelib`     | 处理消息的库                                          |
| `milou`     | 搜索的 Plasma5 小部件                                 |
| `mime`     | 处理 MIME 数据的库                                    |
| `newstuff`     | 用于从网络下载应用程序资产的 KF5 库                   |
| `notifications`     | 用于系统通知的 KF5 抽象                               |
| `notifyconfig`     | 用于 KNotify 的 KF5 配置系统                          |
| `okular`     | KDE 通用文档查看器                                    |
| `oxygen`     | 气浪 5 氧气风格                                       |
| `oxygen-icons5`     | 用于 KDE 的氧气图标主题                               |
| `package`     | KF5 库加载和安装软件包                                |
| `parts`     | KF5 文档中心插件系统                                  |
| `people`     | KF5 库提供访问联系人的功能                            |
| `pim-data-exporter`     | 导入和导出 KDE PIM 设置                               |
| `pimcommon`     | 用于 KDEPim 的常见库                                  |
| `pimtextedit`     | 用于 PIM 特定文本编辑实用程序的 KDE 库                |
| `plasma-browser-integration`     | 将浏览器集成到桌面中的 Plasma5 组件                   |
| `plasma-desktop`     | Plasma5 桌面                                          |
| `plasma-framework`     | KF5 基于插件的 UI 运行时用于编写用户界面              |
| `plasma-integration`     | 用于 Plasma 工作空间的 Qt 平台主题集成插件            |
| `plasma-pa`     | Plasma5 Plasma 脉冲音频混音器                         |
| `plasma-sdk`     | Plasma5 应用程序，有助于 Plasma 开发                  |
| `plasma-workspace`     | Plasma5 Plasma 工作空间                               |
| `plasma-workspace-wallpapers`     | Plasma5 壁纸                                          |
| `plotting`     | KF5 轻量级绘图框架                                    |
| `polkit-kde-agent-1`     | Plasma5 守护程序，提供 polkit 认证界面                |
| `powerdevil`     | 管理电力消耗设置的 Plasma5 工具                       |
| `prison`     | 生成条形码的 API                                      |
| `pty`     | KF5 Pty 抽象                                          |
| `purpose`     | 为特定目的提供可用操作                                |
| `qqc2-desktop-style`     | 用于 KDE 的 Qt QuickControl2 样式                     |
| `runner`     | KF5 并行查询系统                                      |
| `service`     | KF5 高级插件和服务内省                                |
| `solid`     | KF5 硬件集成和检测                                    |
| `sonnet`     | KF5 基于插件的拼写检查库                              |
| `syndication`     | KDE RSS feed 处理库                                   |
| `syntaxhighlighting`     | 用于结构化文本和代码的 KF5 语法高亮引擎               |
| `systemsettings`     | Plasma5 系统设置                                      |
| `texteditor`     | KF5 高级可嵌入文本编辑器                              |
| `textwidgets`     | KF5 高级文本编辑窗口小部件                            |
| `threadweaver`     | KF5 对 QtDBus 的附加组件                              |
| `tnef`     | 处理 TNEF 数据的 KDE API                              |
| `unitconversion`     | 用于单位转换的 KF5 库                                 |
| `user-manager`     | Plasma5 用户管理器                                    |
| `wallet`     | 为用户密码提供 KF5 安全且统一的容器                   |
| `wayland`     | 用于 Wayland 库的 KF5 客户端和服务器库包装器          |
| `widgetsaddons`     | KF5 附加组件到 QtWidgets                              |
| `windowsystem`     | 访问窗口系统的 KF5 库                                 |
| `xmlgui`     | KF5 用户可配置的主窗口                                |
| `xmlrpcclient`     | 与 XMLRPC 服务交互的 KF5                              |

示例 23. USE_KDE 示例

这是一个简单的示例，用于 KDE port。 USES= cmake 指导 port 使用 CMake，这是 KDE 项目广泛使用的配置工具（有关详细使用说明，请参阅《使用 cmake 》）。 USE_KDE 需要依赖 KDE 库。 通过配置日志可以确定所需的 KDE 组件和其他依赖项。 USE_KDE 不意味着 USE_QT 。 如果一个 port 需要一些 Qt 组件，请在 USE_QT 中指定它们。

```
USES=		cmake kde:5 qt:5
USE_KDE=	ecm
USE_QT=		core buildtools_build qmake_build
```

## 6.15. 使用 LXQt

依赖于 LXQt 的应用程序应设置 USES+= lxqt 并将 USE_LXQT 设置为下表中所需组件的列表

表 19. 可用的 LXQt 组件

| 名称 | 描述                               |
| ------ | ------------------------------------ |
| `buildtools`     | 用于额外 CMake 模块的助手          |
| `libfmqt`     | Libfm Qt 绑定                      |
| `lxqt`     | LXQt 核心库                        |
| `qtxdg`     | freedesktop.org XDG 规范的 Qt 实现 |

示例 24. USE_LXQT 示例

这是一个简单的例子， USE_LXQT 会添加对 LXQt 库的依赖。可以从配置日志中确定所需的 LXQt 组件和其他依赖关系。

```
USES=	cmake lxqt qt:5 tar:xz
USE_QT=		core dbus widgets buildtools_build qmake_build
USE_LXQT=	buildtools libfmqt
```

## 6.16. 使用 Java

### 6.16.1. 变量定义

如果port需要 Java™开发工具包（JDK™）来构建、运行甚至提取分发文件，则定义 USE_JAVA 。

在ports集合中有几个来自不同供应商、多个版本的 JDK。如果port必须使用特定版本，请使用 JAVA_VERSION 变量指定。最新版本为 java/openjdk18，还提供 java/openjdk17、java/openjdk16、java/openjdk15、java/openjdk14、java/openjdk13、java/openjdk12、java/openjdk11、java/openjdk8 和 java/openjdk7。

表 20。由Ports使用 Java 的可以设置的变量

| 变量 | 意味着                                                                                                                                        |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `USE_JAVA`     | 定义为其余变量产生任何效果。                                                                                                                  |
| `JAVA_VERSION`     | 适用于 port 的一系列适当的 Java 版本的以空格分隔的列表。一个可选的 + 允许指定版本范围（允许值： 8[+] 11[+] 17[+] 18[+] 19[+] 20[+] 21[+] ）。 |
| `JAVA_OS`     | 适用于 port 的一系列适当的 JDK port 操作系统的以空格分隔的列表（允许值： native linux ）。                                                    |
| `JAVA_VENDOR`     | 适用于 port 的一系列适当的 JDK port 供应商的以空格分隔的列表（允许值： openjdk oracle ）。                                                    |
| `JAVA_BUILD`     | 当设置时，将所选的 JDK port 添加到构建依赖项中。                                                                                              |
| `JAVA_RUN`     | 当设置时，将所选的 JDK port 添加到运行依赖项中。                                                                                              |
| `JAVA_EXTRACT`     | 当设置时，将所选的 JDK port 添加到提取依赖项中。                                                                                              |

以下是设置 USE_JAVA 后 port 将收到的所有设置列表：

表 21. 使用 Java 的 Ports 提供的变量

| 变量 | 值                                                                                                                                  |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| `JAVA_PORT`     | JDK 的名称 port（例如， java/openjdk6 ）。                                                                                          |
| `JAVA_PORT_VERSION`     | JDK 的完整版本 port（例如， 1.6.0 ）。只需要这个版本号的前两位数字，使用 ${JAVA_PORT_VERSION:C/^([0-9])\.([0-9])(.*)$/\1.\2/} 。 |
| `JAVA_PORT_OS`     | JDK port 使用的操作系统 'native' (例如，'native')。                                                                                 |
| `JAVA_PORT_VENDOR`     | JDK port 的供应商 (例如， 'openjdk' )。                                                                                             |
| `JAVA_PORT_OS_DESCRIPTION`     | JDK port 使用的操作系统描述 (例如， 'Native' )。                                                                                    |
| `JAVA_PORT_VENDOR_DESCRIPTION`     | JDK 供应商的描述port（例如， 'OpenJDK BSD Porting Team' ）。                                                                        |
| `JAVA_HOME`     | JDK 安装目录的路径（例如，'/usr/local/openjdk6'）。                                                                                 |
| `JAVAC`     | 要使用的 Java 编译器路径（例如，'/usr/local/openjdk6/bin/javac'）。                                                                 |
| `JAR`     | 使用 jar 工具的路径（例如，'/usr/local/openjdk6/bin/jar'或'/usr/local/bin/fastjar'）。                                              |
| `APPLETVIEWER`     | 使用 appletviewer 实用程序的路径（例如，'/usr/local/openjdk6/bin/appletviewer'）。                                                  |
| `JAVA`     | 使用 java 可执行文件的路径。用于执行 Java 程序（例如，'/usr/local/openjdk6/bin/java'）。                                            |
| `JAVADOC`     | javadoc 实用程序的路径。                                                                                                            |
| `JAVAH`     | javah 程序的路径。                                                                                                                  |
| `JAVAP`     | javap 程序的路径。                                                                                                                  |
| `JAVA_KEYTOOL`     | 到 keytool 实用程序的路径。                                                                                                         |
| `JAVA_N2A`     | 到 native2ascii 工具的路径。                                                                                                        |
| `JAVA_POLICYTOOL`     | 到 policytool 程序的路径。                                                                                                          |
| `JAVA_SERIALVER`     | 到 serialver 实用程序的路径。                                                                                                       |
| `RMIC`     | 到 RMI 存根/骨架生成器的路径， rmic 。                                                                                              |
| `RMIREGISTRY`     | 到 RMI 注册表程序的路径， rmiregistry 。                                                                                            |
| `RMID`     | 到 RMI 守护程序的路径 rmid 。                                                                                                       |
| `JAVA_CLASSES`     | 包含 JDK 类文件的存档路径，${JAVA_HOME}/jre/lib/rt.jar。                                                                            |

使用 java-debug make 目标获取用于调试 port 的信息。它将显示许多先前列出的变量的值。

另外，这些常量被定义为所有 Java ports 可以以一致的方式安装：

表 22. 为使用 Java 的 Ports 定义的常量

| 常量 | 值                                                                              |
| ------ | --------------------------------------------------------------------------------- |
| `JAVASHAREDIR`     | 与 Java 相关的所有内容的基本目录。默认值：${PREFIX}/share/java。                |
| `JAVAJARDIR`     | 安装 JAR 文件的目录。默认值：${JAVASHAREDIR}/classes.                           |
| `JAVALIBDIR`     | 由其他ports安装的 JAR 文件所在的目录。默认值：${LOCALBASE}/share/java/classes。 |

相关条目在 PLIST_SUB （在更改 pkg-plist 基于 Make 变量中记录）和 SUB_LIST 中定义。

### 6.16.2. 使用 Ant 构建

当使用 Apache Ant 构建port时，必须定义 USE_ANT 。因此，Ant 被视为子构建命令。当port未定义 do-build 目标时，默认目标将被设置，根据 MAKE_ENV 、 MAKE_ARGS 和 ALL_TARGET 运行 Ant。这类似于 USES= gmake 机制，其在构建机制中有文档记录。

### 6.16.3. 最佳实践

在移植 Java 库时，port必须将 JAR 文件安装在${JAVAJARDIR}中，其他所有内容安装在${JAVASHAREDIR}/${PORTNAME}下（除了文档，详见下文）。为了减小打包文件大小，在 Makefile 中直接引用 JAR 文件。使用此语句（其中 myport.jar 是作为port的一部分安装的 JAR 文件的名称）：

```
PLIST_FILES+=	${JAVAJARDIR}/myport.jar
```

在移植 Java 应用程序时，port通常会将所有内容安装在单个目录下（包括其 JAR 依赖项）。在这方面强烈建议使用${JAVASHAREDIR}/${PORTNAME}。由移植者决定是否将额外的 JAR 依赖项安装在此目录下，或者使用已安装的依赖项（来自${JAVAJARDIR}）。

在移植需要应用服务器（如 www/tomcat7）来运行服务的 Java™应用程序时，供应商通常会分发一个 .war 文件。.war 是 Web 应用程序归档文件，当应用程序调用时会被解压缩。避免将 .war 添加到 pkg-plist 中。这不被视为最佳实践。应用服务器将展开 war 归档文件，但如果移除 port，则不会正确清理它。处理此文件的更理想方式是先解压缩归档文件，然后安装文件，最后将这些文件添加到 pkg-plist 中。

```
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

无论port的类型是库还是应用程序，附加文档都安装在与任何其他port相同的位置。Javadoc 工具据说会根据所使用的 JDK 版本生成不同的文件集。对于不强制使用特定 JDK 的ports，因此指定打包清单（pkg-plist）是一项复杂的任务。这也是为什么强烈建议移植者使用 PORTDOCS 的原因之一。此外，即使可以预测 javadoc 将生成的文件集，生成的 pkg-plist 的大小也倡导使用 PORTDOCS 。

DATADIR 的默认值为${PREFIX}/share/${PORTNAME}。将 DATADIR 覆盖为${JAVASHAREDIR}/${PORTNAME}是一个好主意，特别是对于 Java ports。实际上， DATADIR 会自动添加到 PLIST_SUB 中（在根据 Make 变量更改 pkg-plist 中有记录），因此在 pkg-plist 中直接使用 %%DATADIR%% 。

至于是从源代码构建 Java ports还是直接从二进制发行版安装它们的选择，在撰写本文时尚无明确定的政策。然而，FreeBSD Java 项目的成员鼓励端口维护者在任务简单时从源代码构建他们的ports。

本节中介绍的所有功能都在 bsd.java.mk 中实现。如果port需要更复杂的 Java 支持，请首先查看 bsd.java.mk Git 日志，因为通常需要一些时间来记录最新功能。然后，如果缺少的所需支持对许多其他 Java ports有益，请随时在 freebsd-java 上讨论。

尽管有一个 java 类别适用于 PRs，但它是指来自 FreeBSD Java 项目的 JDK 移植工作。因此，像其他port一样，将 Javaport提交到 ports 类别中，除非问题与 JDK 实现或 bsd.java.mk 有关。

同样，关于 Javaport的 CATEGORIES 有一个明确定义的政策，详细内容请参阅分类。

## 6.17. 网络应用程序，Apache 和 PHP

### 6.17.1. Apache

表 23. 变量为使用 Apache 的Ports

| `USE_APACHE` | port需要 Apache。可能的值: yes （获取任何版本）， 22 ， 24 ， 22-24 ， 22+ 等。默认 APACHE 版本为 22 。更多细节请参见ports/Mk/bsd.apache.mk 和 wiki.freebsd.org/Apache/. |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `APXS` | 完整路径到 apxs 二进制文件。可以在 port 中被覆盖。                                                                                                                       |
| `HTTPD` | 完整路径到 httpd 二进制文件。可以在 port 中被覆盖。                                                                                                                      |
| `APACHE_VERSION` | 现有 Apache 安装的版本（只读变量）。此变量仅在包含 bsd.port.pre.mk 后才可用。可能的值: 22 , 24 .                                                                         |
| `APACHEMODDIR` | Apache 模块的目录。此变量将自动在 pkg-plist 中展开。                                                                                                                     |
| `APACHEINCLUDEDIR` | Apache 头文件的目录。此变量将自动在 pkg-plist 中展开。                                                                                                                   |
| `APACHEETCDIR` | Apache 配置文件的目录。此变量将自动在 pkg-plist 中展开。                                                                                                                 |

将 Apache 模块移植的有用变量表 24

| `MODULENAME` | 模块的名称。默认值为 PORTNAME 。示例： mod_hello               |
| -- | ---------------------------------------------------------------- |
| `SHORTMODNAME` | 模块的简称。自动从 MODULENAME 派生，但可以被覆盖。示例： hello |
| `AP_FAST_BUILD` | 使用 apxs 来编译和安装模块。                                   |
| `AP_GENPLIST` | 还会自动创建 pkg-plist。                                       |
| `AP_INC` | 在编译过程中将一个目录添加到头文件搜索路径中。                 |
| `AP_LIB` | 在编译过程中将目录添加到库搜索路径。                           |
| `AP_EXTRAS` | 传递给 apxs 的附加标志。                                       |

### 6.17.2. 网络应用程序

Web 应用程序必须安装到 PREFIX/www/appname。此路径在 Makefile 和 pkg-plist 中都可用，分别表示为 WWWDIR ，在 Makefile 中相对于 PREFIX 的路径表示为 WWWDIR_REL 。

Web 服务器进程的用户和组分别表示为 WWWOWN 和 WWWGRP ，如果需要更改某些文件的所有权，则默认值分别为 www 。如果port需要不同的值，请使用 WWWOWN?= myuser 和 WWWGRP?= mygroup 。这使用户可以轻松覆盖它们。

|  | 谨慎使用 WWWOWN 和 WWWGRP 。请记住，Web 服务器可以写入的每个文件都是潜在的安全风险。 |
| -- | -------------------------------------------------------------------------------------- |

不要依赖 Apache，除非 Web 应用明确需要 Apache。尊重用户可能希望在除 Apache 之外的 Web 服务器上运行 Web 应用程序。

### 6.17.3. PHP

PHP Web 应用程序使用 USES=php 声明它们对它的依赖。有关更多信息，请参阅 php 。

### 6.17.4. PEAR 模块

移植 PEAR 模块是一个非常简单的过程。

将 USES=pear 添加到 port 的 Makefile。框架将在正确的位置安装相关文件，并在安装时自动生成 plist。

示例 25. PEAR 类的示例 Makefile

```
PORTNAME=       Date
DISTVERSION=	1.4.3
CATEGORIES=	devel www pear

MAINTAINER=	someone@example.org
COMMENT=	PEAR Date and Time Zone Classes
WWW=		https://pear.php.net/package/Date/

USES=	pear

.include <bsd.port.mk>
```

|  | 使用 PHP 风格对 PEAR 模块进行自动 flavor 化。 |
| -- | ----------------------------------------------- |

|  | 如果使用非默认 PEAR_CHANNEL ，将自动添加构建和运行时依赖项。 |
| -- | -------------------------------------------------------------- |

|  | PEAR 模块不需要定义 PKGNAMESUFFIX ，它会自动使用 PEAR_PKGNAMEPREFIX 填充。如果需要向 PKGNAMEPREFIX 添加 port ，还必须使用 PEAR_PKGNAMEPREFIX 来区分不同的风味。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |

#### 6.17.4.1. Horde 模块

同样，移植 Horde 模块是一个简单的过程。

将 USES=horde 添加到port的 Makefile 中。该框架将把相关文件安装在正确的位置，并在安装时自动生成 plist。

USE_HORDE_BUILD 和 USE_HORDE_RUN 变量可用于在其他 Horde 模块上添加构建时间和运行时依赖关系。请查看 Mk/Uses/horde.mk，了解所有可用模块的完整列表。

Horde 模块的示例 26. Horde 模块的示例 Makefile。

```
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

|  | 由于 Horde 模块也是 PEAR 模块，它们也将自动使用 PHP flavors 进行调味。 |
| -- | ------------------------------------------------------------------------ |

## 6.18. 使用 Python

Ports 集合支持多个 Python 版本的并行安装。Ports 必须使用正确的 python 解释器，根据用户可设置的 PYTHON_VERSION 。最重要的是，这意味着在脚本中用 PYTHON_CMD 的值替换 python 可执行文件的路径。

安装文件在 PYTHON_SITELIBDIR 下的Ports必须使用 pyXY- 软件包名称前缀，以便其软件包名称嵌入安装的 Python 版本。

```
PKGNAMEPREFIX=	${PYTHON_PKGNAMEPREFIX}
```

表 25. 使用 Python 的Ports最有用的变量

| `USES=python` | port需要 Python。最低要求版本可以用值如 3.10+ 指定。版本范围也可以通过用短横线分隔两个版本号来指定： USES=python:3.8-3.9 。请注意 USES=python 不包括 Python 2.7，需要使用 USES=python:2.7+ 显式请求。                                     |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `USE_PYTHON=distutils` | 使用 Python distutils 进行配置，编译和安装。当port带有 setup.py 时，这是必需的。这将覆盖 do-build 和 do-install 目标，并且如果未定义 GNU_CONFIGURE ，还可能覆盖 do-configure 。此外，它意味着 USE_PYTHON=flavors 。                       |
| `USE_PYTHON=autoplist` | 自动创建打包列表。这还要求设置 USE_PYTHON=distutils 。                                                                                                                                                                                    |
| `USE_PYTHON=concurrent` | port将使用唯一前缀，通常为 PYTHON_PKGNAMEPREFIX ，用于某些目录，例如 EXAMPLESDIR 和 DOCSDIR ，并且还将附加后缀，即来自 PYTHON_VER 的 Python 版本，以安装二进制文件和脚本。这允许同时为不同的 Python 版本安装ports，否则会安装冲突的文件。 |
| `USE_PYTHON=flavors` | port 不使用 distutils，但仍支持多个 Python 版本。 FLAVORS 将设置为支持的 Python 版本。有关更多信息，请参阅 =python 和 Flavors。                                                                                                           |
| `USE_PYTHON=optsuffix` | 如果当前 Python 版本不是默认版本，则port 将获得 PKGNAMESUFFIX=${PYTHON_PKGNAMESUFFIX} 。仅在使用 flavors 时有用。                                                                                                                         |
| `PYTHON_PKGNAMEPREFIX` | 用作区分不同 Python 版本包的 PKGNAMEPREFIX 。示例： py27-                                                                                                                                                                                 |
| `PYTHON_SITELIBDIR` | 站点包树的位置，包含 Python 的安装路径（通常为 LOCALBASE ）。在安装 Python 模块时， PYTHON_SITELIBDIR 非常有用。                                                                                                                          |
| `PYTHONPREFIX_SITELIBDIR` | PYTHON_SITELIBDIR 的 PREFIX-clean 变体。尽可能在 pkg-plist 中使用 %%PYTHON_SITELIBDIR%% 。 %%PYTHON_SITELIBDIR%% 的默认值为 lib/python%%PYTHON_VERSION%%/site-packages                                                                    |
| `PYTHON_CMD` | Python 解释器命令行，包括版本号。                                                                                                                                                                                                         |

Python 模块依赖辅助程序表 26。

| `PYNUMERIC` | 数值扩展的依赖行。                                                           |
| -- | ------------------------------------------------------------------------------ |
| `PYNUMPY` | 新数值扩展 numpy 的依赖行（PYNUMERIC 已被上游供应商弃用）。                  |
| `PYXML` | XML 扩展的依赖行（对于 Python 2.0 及更高版本不需要，因为它也在基本分发中）。 |
| `PY_ENUM34` | 根据 Python 版本对 devel/py-enum34 进行条件依赖。                            |
| `PY_ENUM_COMPAT` | 根据 Python 版本对 devel/py-enum-compat 进行条件依赖。                       |
| `PY_PATHLIB` | 根据 Python 版本对 devel/py-pathlib 进行条件依赖。                           |
| `PY_IPADDRESS` | 根据 Python 版本对 net/py-ipaddress 进行条件依赖。                           |
| `PY_FUTURES` | 根据 Python 版本对 devel/py-futures 进行条件依赖。                           |

可在 /usr/ports/Mk/Uses/python.mk 中找到所有可用变量的完整列表。

|  | 所有依赖于 Python ports 的内容都必须使用 Python flavors（使用 USE_PYTHON=distutils 或 USE_PYTHON=flavors ）并在其来源处附加 Python flavor。请查看一个简单的 Python 模块的 Makefile。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

示例 27. 一个简单的 Python 模块的 Makefile。

```
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

一些 Python 应用程序声称具有 DESTDIR 支持（这对于分阶段是必需的），但是它是有问题的（例如，Mailman 版本为 2.1.16）。可以通过重新编译脚本来解决这个问题。例如，可以在 post-build 目标中完成这个操作。假设 Python 脚本应该在安装后驻留在 PYTHONPREFIX_SITELIBDIR 中，可以应用这个解决方案：

```
(cd ${STAGEDIR}${PREFIX} \
  && ${PYTHON_CMD} ${PYTHON_LIBDIR}/compileall.py \
   -d ${PREFIX} -f ${PYTHONPREFIX_SITELIBDIR:S;${PREFIX}/;;})
```

这将使用相对于阶段目录的路径重新编译源代码，并通过 -d 将值添加到字节编译输出文件中记录的文件名之前。需要 -f 强制重新编译，并且 :S;${PREFIX}/;; 会从 PYTHONPREFIX_SITELIBDIR 的值中剥离前缀，使其相对于 PREFIX 。

## 6.19. 使用 Tcl/Tk

Ports 集合支持多个 Tcl/Tk 版本的并行安装。 Ports 应该尝试至少支持默认的 Tcl/Tk 版本及更高版本，使用 USES=tcl 。可以通过附加 :_xx_ 来指定所需的 tcl 版本，例如， USES=tcl:85 。

表 27. 使用 Tcl/Tk 的 Ports 最有用的只读变量

| `TCL_VER` | 选择了 Tcl 的主要.次要版本 |
| -- | ---------------------------- |
| `TCLSH` | Tcl 解释器的完整路径       |
| `TCL_LIBDIR` | Tcl 库的路径               |
| `TCL_INCLUDEDIR` | Tcl C 头文件的路径         |
| `TK_VER` | 选择了 Tk 的主要次要版本   |
| `WISH` | Tk 解释器的完整路径        |
| `TK_LIBDIR` | Tk 库的路径                |
| `TK_INCLUDEDIR` | Tk C 头文件的路径          |

请参阅使用 USES=tcl 和 USES=tk 宏 的完整描述。这些变量的完整列表位于 /usr/ports/Mk/Uses/tcl.mk 中。

## 6.20. 使用 SDL

USE_SDL 用于自动配置依赖项以支持使用像 devel/sdl12 和 graphics/sdl_image 这样的 SDL 库的 ports。

这些 SDL 1.2 版本的库已被识别：

* sdl: [devel/sdl12](https://cgit.freebsd.org/ports/tree/devel/sdl12/)
* 控制台: devel/sdl_console
* 图形: graphics/sdl_gfx
* 图像: graphics/sdl_image
* 混音器：音频/ sdl_mixer
* mm：devel / sdlmm
* 网络：net / sdl_net
* pango：x11-toolkits/sdl_pango
* 声音：audio/sdl_sound
* ttf：graphics/sdl_ttf

这些版本为 2.0 的 SDL 库被认可：

* sdl: [devel/sdl20](https://cgit.freebsd.org/ports/tree/devel/sdl20/)
* gfx: [graphics/sdl2_gfx](https://cgit.freebsd.org/ports/tree/graphics/sdl2_gfx/)
* 图像: 图形/SDL2 图像
* 混音器: 音频/SDL2 混音器
* 网络: 网络/SDL2 网络
* ttf：图形/ sdl2_ttf

因此，如果port依赖于 net/sdl_net 和 audio/sdl_mixer，则语法将是：

```
USE_SDL=	net mixer
```

所需的依赖项 devel/sdl12，也会自动添加到 net/sdl_net 和 audio/sdl_mixer。

使用 USE_SDL 与 SDL 1.2 条目，它将自动：

* 为 BUILD_DEPENDS 添加对 sdl12-config 的依赖项
* 将变量 SDL_CONFIG 添加到 CONFIGURE_ENV
* 将所选库的依赖项添加到 LIB_DEPENDS

使用 USE_SDL ，带有 SDL 2.0 条目，它将自动：

* 将 sdl2-config 的依赖项添加到 BUILD_DEPENDS
* 将变量 SDL2_CONFIG 添加到 CONFIGURE_ENV
* 将所选库的依赖项添加到 LIB_DEPENDS

## 6.21. 使用 wxWidgets

本节描述了 wxWidgets 库在 ports 树中的状态以及与 ports 系统的集成。

### 6.21.1. 介绍

wxWidgets 库有许多版本之间存在冲突（在相同名称下安装文件）。在 ports 树中，通过使用版本号后缀将每个版本安装在不同的名称下来解决了这个问题。

这样做的明显缺点是，每个应用程序都必须修改以查找预期版本。幸运的是，大多数应用程序调用 wx-config 脚本来确定必要的编译器和链接器标志。该脚本对每个可用版本的命名不同。大多数应用程序支持环境变量，或者接受一个配置参数，以指定调用哪个 wx-config 脚本。否则它们必须被打补丁。

### 6.21.2. 版本选择

要使port使用特定版本的 wxWidgets，有两个可用于定义的变量（如果只定义一个，另一个将设置为默认值）:

选择 wxWidgets 版本的变量|变量

| 描述 | 默认值                  |              |
| ------ | ------------------------- | -------------- |
| `USE_WX`     | port 可以使用的版本列表 | 所有可用版本 |
| `USE_WX_NOT`     | port 不能使用的版本列表 | 无           |

树中可用的 wxWidgets 版本及相应的 ports 如下：

表 29. 可用的 wxWidgets 版本

| 版本 | Port |
| ------ | ------ |
| `2.8`     | [x11-toolkits/wxgtk28](https://cgit.freebsd.org/ports/tree/x11-toolkits/wxgtk28/)     |
| `3.0`     | [x11-toolkits/wxgtk30](https://cgit.freebsd.org/ports/tree/x11-toolkits/wxgtk30/)     |

变量在选择 wxWidgets 版本时可以设置为这些组合之一或多个，用空格分隔：

表 30. wxWidgets 版本规格

| 描述                 | 例子 |
| ---------------------- | ------ |
| 单个版本             | `2.8`     |
| 升序范围             | `2.8+`     |
| 降序范围             | `3.0-`     |
| 全范围（必须是升序） | `2.8-3.0`     |

还有一些变量可用于从可用版本中选择首选版本。它们可以设置为版本列表，前面的版本优先级更高。

表 31. 选择首选 wxWidgets 版本的变量

| 名称 | 为设计而制定 |
| ------ | -------------- |
| `WANT_WX_VER`     | port的       |
| `WITH_WX_VER`     | 用户         |

### 6.21.3. 组件选择

有其他应用程序，虽然不是 wxWidgets 库，但与它们相关。这些应用程序可以在 WX_COMPS 中指定。这些组件可用：

表 32. 可用的 wxWidgets 组件

| 名称 | 描述                    | 版本限制 |
| ------ | ------------------------- | ---------- |
| `wx`     | 主要图书馆              | 无       |
| `contrib`     | 贡献的图书馆            | `none`         |
| `python`     | wxPython（Python 绑定） | `2.8-3.0`         |

通过添加以分号分隔的后缀来为每个组件选择依赖类型。如果不存在，则将使用默认类型（请参阅默认的 wxWidgets 依赖类型）。这些类型可用：

表 33.可用的 wxWidgets 依赖类型

| 名称 | 描述                                     |
| ------ | ------------------------------------------ |
| `build`     | 组件是构建所需，相当于 BUILD_DEPENDS     |
| `run`     | 运行所需的组件，相当于 RUN_DEPENDS       |
| `lib`     | 构建和运行所需的组件，相当于 LIB_DEPENDS |

组件的默认值详见本表：

默认 wxWidgets 依赖类型表 34

| 组件 | 依赖类型 |
| ------ | ---------- |
| `wx`     | `lib`         |
| `contrib`     | `lib`         |
| `python`     | `run`         |
| `mozilla`     | `lib`         |
| `svg`     | `lib`         |

例 28. 选择 wxWidgets 组件

该片段对应于使用 wxWidgets 版本 0 及其贡献库的port。

```
USE_WX=		2.8
WX_COMPS=	wx contrib
```

### 6.21.4. 检测已安装的版本

要检测已安装的版本，请定义 WANT_WX 。如果未设置为特定版本，则组件将具有版本后缀。 HAVE_WX 将在检测后填充。

示例 29. 检测已安装的 wxWidgets 版本和组件

此片段可用于使用已安装的 wxWidgets 的port，或者选择了一个选项。

```
WANT_WX=	yes

.include <bsd.port.pre.mk>

.if defined(WITH_WX) || !empty(PORT_OPTIONS:MWX) || !empty(HAVE_WX:Mwx-2.8)
USE_WX=			2.8
CONFIGURE_ARGS+=	--enable-wx
.endif
```

如果已安装 wxPython 支持，或选择了某个选项，在启用 wxPython 支持的port中，将同时安装版本 2.8 的 wxWidgets。

```
USE_WX=		2.8
WX_COMPS=	wx
WANT_WX=	2.8

.include <bsd.port.pre.mk>

.if defined(WITH_WXPYTHON) || !empty(PORT_OPTIONS:MWXPYTHON) || !empty(HAVE_WX:Mpython)
WX_COMPS+=		python
CONFIGURE_ARGS+=	--enable-wxpython
.endif
```

### 6.21.5. 已定义变量

在从变量中选择 wxWidgets 版本后，这些变量将在port中可用。

对于使用 wxWidgets 的 Ports 定义的变量表 35

| 名字 | 描述                                             |
| ------ | -------------------------------------------------- |
| `WX_CONFIG`     | wxWidgets wx-config 脚本的路径（具有不同的名称） |
| `WXRC_CMD`     | wxWidgets wxrc 程序的路径（具有不同的名称）      |
| `WX_VERSION`     | 将要使用的 wxWidgets 版本（例如， 2.6 ）         |

### 6.21.6. 在 bsd.port.pre.mk 中处理

定义 WX_PREMK 以便在包含 bsd.port.pre.mk 后立即使用变量

|  | 当定义 WX_PREMK 时，如果在包含 bsd.port.pre.mk 后修改 wxWidgets port 变量，则版本、依赖项、组件和定义的变量将不会更改 |
| -- | ----------------------------------------------------------------------------------------------------------------------- |

例 30。在命令中使用 wxWidgets 变量

该片段通过运行 wx-config 脚本来演示 WX_PREMK 的使用，以获取完整的版本字符串，将其分配给一个变量并将其传递给程序。

```
USE_WX=		2.8
WX_PREMK=	yes

.include <bsd.port.pre.mk>

.if exists(${WX_CONFIG})
VER_STR!=	${WX_CONFIG} --release

PLIST_SUB+=	VERSION="${VER_STR}"
.endif
```

|  | 当 wxWidgets 变量位于目标内部时，无需 WX_PREMK 即可安全地在命令中使用它们。 |
| -- | ----------------------------------------------------------------------------- |

### 6.21.7. 额外 configure 参数

一些 GNU configure 脚本仅通过设置 WX_CONFIG 环境变量无法找到 wxWidgets，需要额外的参数。可以使用 WX_CONF_ARGS 来提供它们。

表 36. WX_CONF_ARGS 的合法值

| 可能的价值 | 结果参数 |
| ------------ | ---------- |
| `absolute`           | `--with-wx-config=${WX_CONFIG}`         |
| `relative`           | `--with-wx=${LOCALBASE} --with-wx-config=${WX_CONFIG:T}`         |

## 6.22. 使用 Lua

这一节描述了 Lua 库在ports树中的状态以及它与ports系统的集成。

### 6.22.1. 介绍

Lua 库和相应解释器有许多版本，它们之间存在冲突（在相同名称下安装文件）。在ports树中，通过使用版本号后缀将每个版本安装在不同的名称下来解决了这个问题。

这样做的明显缺点是，每个应用程序都必须进行修改以找到预期的版本。但可以通过向编译器和链接器添加一些额外的标志来解决这个问题。

使用 Lua 的应用程序通常应为一个版本构建。然而，为 Lua 构建的可加载模块是针对它们支持的每个 Lua 版本单独构建的，对这些模块的依赖关系应该使用port原点上的 @${LUA_FLAVOR} 后缀指定这种风味。

### 6.22.2. 版本选择

一个使用 Lua 的 port 应该具有以下形式的行：

```
USES=	lua
```

如果需要特定版本的 Lua，或者需要范围内的版本，可以将其指定为 XY （可以多次使用）， XY+ ， -XY 或 XY-ZA 的形式作为参数。如果所需范围内的 Lua 的默认版本通过 DEFAULT_VERSIONS 设置，那么将使用它；否则将使用最接近默认版本的请求版本。例如：

```
USES=	lua:52-53
```

请注意，不会根据已安装的 Lua 版本的存在来调整版本选择。

|  | 版本规范的 XY+ 形式在不加仔细考虑的情况下不应该使用； Lua API 在每个版本都会有一定程度的更改，并且诸如 CMake 或 Autoconf 之类的配置工具在未更新情况下通常无法在将来的 Lua 版本上正常工作。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### 6.22.3. 配置和编译器标志

使用 Lua 的软件可能已经编写成自动检测正在使用的 Lua 版本。通常情况下ports应该覆盖这种假设，并强制使用如上所述选择的特定 Lua 版本。根据正在移植的软件，这可能需要以下任何或所有操作：

* 使用 LUA_VER 作为软件配置脚本的参数之一，通过 CONFIGURE_ARGS 或 CONFIGURE_ENV （或其他构建系统的等效方式）;
* 分别将 -I${LUA_INCDIR} ， -L${LUA_LIBDIR} 和 -llua-${LUA_VER} 添加到 CFLAGS ， LDFLAGS ， LIBS 中，视情况而定;
* 对软件的配置或构建文件进行补丁，以选择正确的版本。

### 6.22.4. 版本风味

对于安装 Lua 模块（而不仅仅是使用 Lua 的应用程序）的 Custom，应为每个支持的 Lua 版本构建单独的风味。这是通过添加 Custom 参数来实现的：

```
USES=	lua:module
```

也可以指定版本号或版本范围；使用逗号分隔参数。

由于每种版本必须具有不同的软件包名称，提供了变量 LUA_PKGNAMEPREFIX ，该变量将设置为适当的值；预期的用法是：

```
PKGNAMEPREFIX=	${LUA_PKGNAMEPREFIX}
```

模块ports通常只应安装文件到 LUA_MODLIBDIR ， LUA_MODSHAREDIR ， LUA_DOCSDIR 和 LUA_EXAMPLESDIR ，所有这些都设置为指向特定版本子目录。安装任何其他文件必须小心进行，以避免版本之间的冲突。

希望为每个 Lua 版本构建单独软件包的port（不是 Lua 模块）应使用 flavors 参数：

```
USES=	lua:flavors
```

这与上述 module 参数描述的方式相同，但没有默认应将包文档化为 Lua 模块的假设（因此 LUA_DOCSDIR 和 LUA_EXAMPLESDIR 未定义）。然而，port可能会选择将 LUA_DOCSUBDIR 定义为适当的子目录名称（通常为port的 PORTNAME 只要不与任何模块的 PORTNAME 冲突），在这种情况下，框架将同时定义 LUA_DOCSDIR 和 LUA_EXAMPLESDIR 。

与模块ports一样，一种带有风味的port应避免安装可能在版本之间产生冲突的文件。通常通过向程序名称添加 LUA_VER_STR 作为后缀来完成此操作（例如，使用 uniquefiles ），以及在 LUA_MODLIBDIR 和 LUA_MODSHAREDIR 之外使用的任何其他文件或子目录中使用 LUA_VER 或 LUA_VER_STR 。

### 6.22.5. 定义的变量

这些变量在port中可用。

表 37。定义了使用 Lua 的Ports变量。

| 名称 | 描述                                                |
| ------ | ----------------------------------------------------- |
| `LUA_VER`     | 将要使用的 Lua 版本（例如， 5.4 ）                  |
| `LUA_VER_STR`     | 没有点的 Lua 版本（例如， 54 ）                     |
| `LUA_FLAVOR`     | 对应于所选 Lua 版本的 flavor 名称，用于指定依赖关系 |
| `LUA_BASE`     | 应该用来定位已经安装的 Lua（和组件）的前缀          |
| `LUA_PREFIX`     | 此 port 安装 Lua（和组件）的前缀                    |
| `LUA_INCDIR`     | Lua 头文件安装目录                                  |
| `LUA_LIBDIR`     | Lua 库安装目录                                      |
| `LUA_REFMODLIBDIR`     | 已安装 Lua 模块库（.so）的目录                      |
| `LUA_REFMODSHAREDIR`     | 已安装 Lua 模块（.lua 文件）的目录                  |
| `LUA_MODLIBDIR`     | Lua 模块库（.so 文件）将被安装到的目录              |
| `LUA_MODSHAREDIR`     | 将要安装 Lua 模块（.lua 文件）的目录                |
| `LUA_PKGNAMEPREFIX`     | Lua 模块使用的包名前缀                              |
| `LUA_CMD`     | Lua 解释器的名称（例如 lua54 ）                     |
| `LUAC_CMD`     | Lua 编译器的名称（例如 luac54 ）                    |

为ports指定 module 参数可用的其他变量

Lua 模块Ports定义的变量表 38

| 名称 | 描述                         |
| ------ | ------------------------------ |
| `LUA_DOCSDIR`     | 模块文档应安装到的目录。     |
| `LUA_EXAMPLESDIR`     | 模块示例文件应安装到的目录。 |

### 6.22.6。例子

例子 31。使用 Lua 的应用程序 Makefile

此示例显示如何引用运行时所需的 Lua 模块。请注意，引用必须指定一个特定的风味。

```
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

例 32。一个简单 Lua 模块的 Makefile

```
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

## 使用 Guile 6.23。

本节描述了ports树中 Guile 的状态及其与ports系统的集成。

### 6.23.1. 介绍

Guile 库和相应的解释器有多个版本，它们之间存在冲突（在相同名称下安装文件）。在 ports 树中，通过使用版本号后缀将每个版本安装在不同的名称下来解决了这个问题。在大多数情况下，应用程序应该从提供的配置变量中检测正确的版本，并使用 pkg-config 来确定名称和相关路径。然而，一些应用程序（特别是那些使用自己的配置规则的应用程序 cmake 或 meson ）将始终尝试使用最新可用的版本。在这种情况下，要么修补 port，要么声明一个构建冲突（参见下面的 conflicts 选项），以确保在 poudriere 外部构建时生成正确的依赖关系。

使用 Guile 的应用程序通常应该只为一个版本构建，最好是在 DEFAULT_VERSIONS 中指定的版本，或者是它们支持的最新版本。然而，为每个支持的 Guile 版本构建 Guile 或 Scheme 库，或者为 Guile 构建扩展模块的应用程序，应该为每个 Guile 版本构建一个单独的 flavor，并且对这样的 ports 的依赖关系应该使用 @${GUILE_FLAVOR} 后缀在 port 源上指定 flavor。

### 6.23.2. 版本选择

使用 Guile 的 port 应该定义适当参数的 USES=guile:<em>arg,arg…</em> 如下：

表 39. 使用 Guile 的 Ports 定义的参数

| 名稱 | 描述                                                                                                             |
| ------ | ------------------------------------------------------------------------------------------------------------------ |
| `<em>X.Y</em>`     | 聲明與 Guile 版本 X.Y 兼容。目前可用的版本包括 1.8 （已過時）、 2.2 和 3.0 。可以指定多個版本。                  |
| `flavors`     | 为每个指定的 Guile 版本创建一个口味。由 DEFAULT_VERSIONS 指定的版本将成为默认的口味。口味名称的形式为 guileXY 。 |
| `build`     | 将 Guile 解释器作为构建依赖项，而不是库依赖项。 build 和 run 都可以指定。                                        |
| `run`     | 将 Guile 解释器仅作为运行时依赖项，而不是库依赖项。 build 和 run 都可以指定。                                    |
| `alias`     | 为解释器和工具添加 BINARY_ALIAS 值。                                                                             |
| `conflicts`     | 为新于所选版本的 Guile 版本声明 CONFLICTS_BUILD 。当port无法配置为使用特定的 Guile 版本时，请使用此选项。        |

一些额外的参数用于处理异常情况；有关详细信息，请参阅 Mk/Uses/guile.mk 。

除非明确指定 build 或者 run ，否则 LIB_DEPENDS 将同时接受 libguile 库依赖以及 Guile 版本所需的任何其他依赖，例如 libgc 。通常情况下， port 不应该需要与其对 Guile 的使用相关的任何其他依赖项。

### 6.23.3. 配置标志

使用 Guile 的软件应该使用 pkg-config 机制来获取编译器和链接器标志。一些较旧或者特殊的 ports 可能正在使用 guile-config 或直接从 guile 获取值，这也应该能够工作（在其中一些情况下， alias 参数可能会有用）。

框架尝试使用以下方法通知所需的 Guile 版本的 port：

* GUILE_EFFECTIVE_VERSION 被添加到 CONFIGURE_ENV ；
* Guile 二进制文件的完整路径在 GUILE 变量中指定为 CONFIGURE_ENV 和 MAKE_ENV ；
* 如果使用 alias 选项，则所需的 Guile 版本的二进制文件将被别名为;
* 如果不使用 alias 选项，则将所需的 Guile 版本工具的路径（ guild ， guile-config 等）添加到 CONFIGURE_ENV 和 MAKE_ENV 作为变量 GUILD ， GUILE_CONFIG 等。

对于某些ports，可能需要以其他方式指定版本，例如通过 CONFIGURE_ARGS 或 MESON_ARGS ，具体取决于port。

如果这些方法都无法在其他版本存在的情况下使 port 选择指定的 Guile 版本，那么最好修补它以这样做。如果不可行，指定 conflicts 选项以防止在检测到错误版本的情况下构建 port。

### 6.23.4. 版本类型

安装 Guile 扩展或库的 port，或为 Guile 预编译的 Scheme 库，应为每个支持的 Guile 版本构建单独的类型。这是通过添加 flavors 选项来完成的。

由于每种定制必须具有不同的软件包名称，因此必须设置 PKGNAMESUFFIX ，通常为：

```
PKGNAMESUFFIX=	-${FLAVOR}
```

即使文件不是特定于版本的，此类ports必须将 Scheme 文件安装到 GUILE_SITE_DIR 而不是 GUILE_GLOBAL_SITE_DIR 。这通常需要修补port。

另外，如果这样的port安装了一个 .pc 文件，则必须将其放置在 GUILE_PKGCONFIG_PATH 而不是全局 pkgconfig 目录中。这样可以使依赖的ports找到用于正在使用的特定 Guile 版本的正确配置。

如果一个 Guile 扩展 port 安装了一个 .so 文件，则通常应该放在特定于 Guile 版本的 extensions 目录中。通常不应该使用 USE_LDCONFIG 。

由有风味的 port 安装的任何其他文件也必须放在特定版本的目录中，或者使用特定版本的文件名。对于文档和示例， GUILE_DOCS_DIR 和 GUILE_EXAMPLES_DIR 指定了适合的位置，port 应该在其中创建一个子目录，见下文。

### 6.23.5. 定义变量

这些变量在port中可用。

表 40。使用 Guile 的Ports定义的变量

| 名称 | 範例數值 | 描述                                                             |
| ------ | ---------- | ------------------------------------------------------------------ |
| `GUILE_VER`     | `3.0`         | 使用的 Guile 版本。                                              |
| `GUILE_SFX`     | `3`         | 一些名称上使用的简短后缀。请谨慎使用；可能不唯一或将来可能更改。 |
| `GUILE_FLAVOR`     | `guile30`         | 与所选版本对应的口味名称。                                       |
| `GUILE_PORT`     | `lang/guile3`         | Port 指定 Guile 版本的来源。                                     |
| `GUILE_PREFIX`     | `${PREFIX}`         | 用于安装的目录前缀。                                             |
| `GUILE_CMD`     | `guile-3.0`         | Guile 解释器的名称，带版本后缀。                                 |
| `GUILE_CMDPATH`     | `${LOCALBASE}/bin/guile-3.0`         | Guile 解释器的完整路径。                                         |
| `GUILD_CMD`     | `guild-3.0`         | 工会工具的名称，带版本后缀。                                     |
| `GUILD_CMDPATH`     | `${LOCALBASE}/bin/guild-3.0`         | 工会工具的完整路径。                                             |
| `GUILE_*_CMD`<br />`GUILE_*_CMDPATH`   |          | 像 GUILE_CMD 和 GUILE_CMDPATH 一样，但用于其他工具二进制文件。   |
| `GUILE_PKGCONFIG_PATH`     | `${LOCALBASE}/libdata/pkgconfig/guile/3.0`         | 在安装文件时应该使用 flavors 的软件包安装 .pc 文件。             |
| `GUILE_INFO_PATH`     | `share/info/guile3`         | 使用 flavors 选项为ports提供合适的 INFO_PATH 值。                |

下列内容被定义为变量和 PLIST_SUB 条目。 变量形式后缀为 _DIR ，并且是一个完整路径（前缀为 GUILE_PREFIX ）。

为使用 Guile 定义的 Ports 路径替换表 41

| 名称 | 示例值 | 描述                                                  |
| ------ | -------- | ------------------------------------------------------- |
| `GUILE_GLOBAL_SITE`     | `share/guile/site`       | 所有 guile 版本共享的站点目录；通常不应该使用此目录。 |
| `GUILE_SITE`     | `share/guile/3.0/site`       | 所选 Guile 版本的站点目录。                           |
| `GUILE_SITE_CCACHE`     | `lib/guile/3.0/site-ccache`       | 编译字节码文件的目录。                                |
| `GUILE_DOCS`     | `share/doc/guile30`       | 版本特定文档的父目录。                                |
| `GUILE_EXAMPLES`     | `share/examples/guile30`       | 版本特定示例的父目录。                                |

### 6.23.6. 示例

示例 33. 使用 Guile 的应用程序的 Makefile

此示例显示如何引用构建和运行时所需的 Guile 库。请注意，引用必须指定一个 flavor。此示例假定应用程序正在使用 pkg-config 来定位依赖项。

```
PORTNAME=	sample
DISTVERSION=	1.2.3
CATEGORIES=	whatever

MAINTAINER=	fred.bloggs@example.com
COMMENT=	Sample
WWW=		https://example.com/guile_sample/sample/

BUILD_DEPENDS=	guile-lib-${GUILE_FLAVOR}>=0.2.5:devel/guile-lib@${GUILE_FLAVOR}
RUN_DEPENDS=	guile-lib-${GUILE_FLAVOR}>=0.2.5:devel/guile-lib@${GUILE_FLAVOR}

USES=		guile:2.2,3.0 pkgconfig

.include <bsd.port.mk>
```

## 6.24. 使用 iconv

FreeBSD 在操作系统中有一个本地 iconv 。

对于需要 iconv 的软件，定义 USES=iconv 。

当port定义 USES=iconv 时，这些变量将可用：

| 变量名称 | 用途                                       | 使用 WCHAR_T 或 //TRANSLIT 扩展时的 Port iconv | 基本的 iconv   |
| ---------- | -------------------------------------------- | ------------------------------------------------ | ---------------- |
| `ICONV_CMD`         | 存放二进制文件 iconv 的目录                | `${LOCALBASE}/bin/iconv`                                               | /usr/bin/iconv |
| `ICONV_LIB`         | ld 参数用于链接到 libiconv（如果需要）     | `-liconv`                                               | （空）         |
| `ICONV_PREFIX`         | iconv 实现所在的目录（对于配置脚本很有用） | `${LOCALBASE}`                                               | /usr           |
| `ICONV_CONFIGURE_ARG`         | 配置脚本的预先构建配置参数                 | `--with-libiconv-prefix=${LOCALBASE}`                                               | (empty)        |
| `ICONV_CONFIGURE_BASE`         | 配置脚本的预先构建配置参数                 | `--with-libiconv=${LOCALBASE}`                                               | （空白）       |

这两个示例会自动为使用转换器/ libiconv 或本机 iconv 的系统填写正确的变量。

示例 34。简单的 iconv 使用

```
USES=		iconv
LDFLAGS+=	-L${LOCALBASE}/lib ${ICONV_LIB}
```

示例 35。 iconv 与 configure 的使用

```
USES=		iconv
CONFIGURE_ARGS+=${ICONV_CONFIGURE_ARG}
```

如上所示，当存在本地 iconv 时， ICONV_LIB 为空。这可以用于检测本地 iconv 并做出相应的反应。

有时程序中的一个 ld 参数或搜索路径在 Makefile 或配置脚本中是硬编码的。可以使用这种方法来解决这个问题：

示例 36. 修复硬编码 -liconv

```
USES=		iconv

post-patch:
	@${REINPLACE_CMD} -e 's/-liconv/${ICONV_LIB}/' ${WRKSRC}/Makefile
```

在某些情况下，有必要设置替代值或根据是否存在本机 iconv 执行操作。在测试 ICONV_LIB 的值之前，必须包含 bsd.port.pre.mk：

示例 37. 检查本机 iconv 的可用性

```
USES=		iconv

.include <bsd.port.pre.mk>

post-patch:
.if empty(ICONV_LIB)
	# native iconv detected
	@${REINPLACE_CMD} -e 's|iconv||' ${WRKSRC}/Config.sh
.endif

.include <bsd.port.post.mk>
```

## 6.25。使用 Xfce

需要 Xfce 库或应用程序设置 USES=xfce

特定的 Xfce 库和应用程序依赖项使用分配给 USE_XFCE 的值进行设置。它们在 /usr/ports/Mk/Uses/xfce.mk 中定义。可能的值有：

  USE_XFCE 的值

garcon[sysutils/garcon](https://cgit.freebsd.org/ports/tree/sysutils/garcon/)

libexo[x11/libexo](https://cgit.freebsd.org/ports/tree/x11/libexo/)

libgui[x11-toolkits/libxfce4gui](https://cgit.freebsd.org/ports/tree/x11-toolkits/libxfce4gui/)

libmenu[x11/libxfce4menu](https://cgit.freebsd.org/ports/tree/x11/libxfce4menu/)

libutil[x11/libxfce4util](https://cgit.freebsd.org/ports/tree/x11/libxfce4util/)

 面板 x11-wm/xfce4-panel

 文件管理器 x11-fm/thunar

 配置管理器 x11/xfce4-conf

示例 38。 USES=xfce 示例

```
USES=		xfce
USE_XFCE=	libmenu
```

示例 39。使用 Xfce 自己的 GTK2 小部件

在这个例子中，移植的应用程序使用了 GTK2 特定的小部件 x11/libxfce4menu 和 x11/xfce4-conf。

```
USES=		xfce:gtk2
USE_XFCE=	libmenu xfconf
```

```
USES=		xfce
USE_XFCE=	panel
```

需要列出 x11-wm/xfce4-panel 需要的组件本身，就像这样：

```
USES=		xfce
USE_XFCE=	libexo libmenu libutil panel
```

但是，必须明确包含 port 的 Xfce 组件和非 Xfce 依赖项。不要指望 Xfce 组件为主 port 提供除本身以外的子依赖项。

## 使用 Budgie

应用程序或依赖于 Budgie 桌面的库应将 USES= budgie 设置为所需组件的列表，并将 USE_BUDGIE 设置为所需组件的列表。

| 名称 | 描述                                         |
| ------ | ---------------------------------------------- |
| `libbudgie`     | 桌面核心 (库)                                |
| `libmagpie`     | Budgie 的 X11 窗口管理器和合成器库           |
| `raven`     | 面板中的全功能中心，用于访问不同的应用小部件 |
| `screensaver`     | 桌面专用屏保程序                             |

```
USES=		budgie
USE_BUDGIE=	screensaver:build
```

例 40。 USE_BUDGIE 例

```
USES=		budgie gettext gnome meson pkgconfig
USE_BUDGIE=	libbudgie
```

## 6.27。使用数据库

使用 Database USES 宏之一来添加对数据库的依赖关系。

表 42. 数据库 USES 宏

| 数据库                  | 使用宏 |
| ------------------------- | -------- |
| Berkeley DB             | [`bdb`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-bdb)       |
| MariaDB，MySQL，Percona | [`mysql`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-mysql)       |
| PostgreSQL 的           | [`pgsql`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-pgsql)       |
| SQLite 的               | [`sqlite`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-sqlite)       |

示例 41。使用 Berkeley DB 6

```
USES=	bdb:6
```

查看 bdb 了解更多信息。

例子 42。使用 MySQL

当port需要 MySQL 客户端库添加

```
USES=	mysql
```

查看 mysql 以获取更多信息。

示例 43。使用 PostgreSQL

当 port 需要 PostgreSQL 服务器版本 9.6 或更高版本时添加

```
USES=		pgsql:9.6+
WANT_PGSQL=	server
```

查看 pgsql 以获取更多信息。

示例 44。使用 SQLite 3

```
USES=	sqlite:3
```

查看 sqlite 以获取更多信息。

## 6.28. 启动和停止服务（ rc 脚本）

rc.d 脚本用于在系统启动时启动服务，并为管理员提供一种标准的停止、启动和重新启动服务的方法。 Ports 集成到系统 rc.d 框架中。有关其使用的详细信息可以在 rc.d 手册章节中找到。有关可用命令的详细说明，请参阅 rc(8) 和 rc.subr(8)。最后，还有一篇关于 rc.d 脚本的实际方面的文章。

带有一个名为门卫（doorman）的神话故事，需要启动一个 doormand 守护程序。将以下内容添加到 Makefile 中：

```
USE_RC_SUBR=	doormand
```

可以列出多个脚本并进行安装。脚本必须放置在 files 子目录中，并在文件名后添加 .in 后缀。将针对该文件运行标准 SUB_LIST 扩展。强烈建议使用 %%PREFIX%% 和 %%LOCALBASE%% 扩展。有关 SUB_LIST 的更多信息，请参见相关部分。

从 FreeBSD 6.1-RELEASE 开始，本地 rc.d 脚本（包括由ports安装的脚本）包含在基础系统的整体 rcorder(8)中。

一个简单的 rc.d 脚本示例，用于启动 doormand 守护进程：

```
#!/bin/sh

# PROVIDE: doormand
# REQUIRE: LOGIN
# KEYWORD: shutdown
#
# Add these lines to /etc/rc.conf.local or /etc/rc.conf
# to enable this service:
#
# doormand_enable (bool):	Set to NO by default.
#				Set it to YES to enable doormand.
# doormand_config (path):	Set to %%PREFIX%%/etc/doormand/doormand.cf
#				by default.

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

除非有非常好的理由提前启动服务，或者它以特定用户身份（而不是 root）运行，所有ports脚本必须使用：

```
REQUIRE: LOGIN
```

如果启动脚本启动必须关闭的守护进程，在系统关闭时将触发服务停止：

```
KEYWORD: shutdown
```

如果脚本未启动持久服务，则无需执行此操作。

对于可选配置元素，此处更喜欢使用“=”风格的默认变量赋值，而不是“:=”风格，因为前者仅在变量未设置时设置默认值，而后者在变量未设置或为空时设置默认值。用户很可能会在其 rc.conf.local 中包含类似以下内容：

```
doormand_flags=""
```

并且使用“:=”的变量替换会不当地覆盖用户的意图。 _enable 变量是非可选的，并且必须使用“:”进行默认设置。

|  | Ports 在安装和卸载时不能启动和停止其服务。不要滥用@preexec 命令，@postexec 命令，@preunexec 命令，@postunexec 命令部分中描述的 plist 关键字运行修改当前运行系统的命令，包括启动或停止服务。 |
| -- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### 6.28.1。先决清单

在贡献带有 rc.d 脚本的 port 之前，更重要的是，在提交之前，请查阅此清单以确保准备就绪。

devel/rclint port 可以检查大部分内容，但它不能替代适当的审查。

1. 如果这是一个新文件，它是否具有 .sh 扩展名？如果是，那么必须将其更改为只是 file.in ，因为 rc.d 文件不得以该扩展名结尾。
2. 文件的名称（减去 .in）， PROVIDE 行和 $ 名称是否都匹配？文件名匹配 PROVIDE 有助于更容易调试，特别是对于 rcorder(8) 问题。匹配文件名和 $ 名称有助于更容易地确定 rc.conf[.local] 中哪些变量是相关的。对于所有新脚本，包括基本系统中的脚本，这也是一个政策。
3. REQUIRE 行是否设置为 LOGIN ？这对于作为非根用户运行的脚本是强制的。如果它以 root 运行，那么必须有一个良好的理由使其在 LOGIN 之前运行吗？如果没有，它必须在之后运行，以便将本地脚本松散地分组到 rcorder(8) 中的某个点，在这个点之后，基本系统中的大多数内容已经在运行。
4. 脚本是否启动了持久服务？如果是，必须有 KEYWORD: shutdown 。
5. 确保没有 KEYWORD: FreeBSD 存在。多年来这已经不再必要也不希望存在。这也表明新脚本是从旧脚本复制/粘贴而来，因此在审查时必须格外小心。
6. 如果脚本使用解释性语言如 perl 、 python 或 ruby ，请确保设置适当，例如对于 Perl，通过添加 PERL=${PERL} 到 SUB_LIST 并使用 %%PERL%% 来设置 command_interpreter 。否则，

    ```
    # service name stop
    ```

    可能不会正常工作。有关更多信息，请参阅 service(8)。
7. 所有的/usr/local 出现都被替换为 %%PREFIX%% 了吗？
8. 默认变量赋值是在 load_rc_config 之后吗？
9. 默认情况下，空字符串会分配到哪里？它们应该被移除，但务必检查文件顶部的注释中是否记录了该选项。
10. 变量中设置的内容是否真正被脚本使用？
11. 默认名称列表中列出的选项是否都是必需的？如果是，它们必须在 command_args 中。这里 -d 是一个警示信号（请原谅双关语），因为它通常是"daemonize"进程的选项，因此实际上是必需的。
12. _name__flags 绝不能包含在 command_args 中（反之亦然，尽管这种错误较少见）。
13. 脚本是否无条件执行任何代码？这是不被赞同的。通常这些事情必须通过 start_precmd 来处理。
14. 所有布尔测试必须使用 checkyesno 函数。不能手工编写 [Yy][Ee][Ss] 等测试。
15. 如果有循环（例如，等待某物启动），是否有计数器来终止循环？如果出现错误，我们不希望引导永远停在那里。
16. 脚本是否创建需要特定权限的文件或目录，例如，需要由运行进程的用户拥有的 pid？与传统的 touch(1)/ chown(8)/ chmod(1)例程不同，请考虑使用 install(1)并使用适当的命令行参数来一步完成整个过程。

## 6.29. 添加用户和组

有些 ports 需要存在特定的用户帐户，通常是为了作为该用户运行的守护进程。对于这些 ports，请选择一个从 50 到 999 的唯一 UID，并在 ports/UIDs（用于用户）和 ports/GIDs（用于组）中注册该 UID。用户和组的唯一标识应该是相同的。

当需要为 port 创建新用户或组时，请在这两个文件中包含补丁。

然后在 Makefile 中使用 USERS 和 GROUPS ，在安装 port 时用户将被自动创建。

```
USERS=	pulse
GROUPS=	pulse pulse-access pulse-rt
```

当前保留的 UIDs 和 GIDs 列表可以在ports/UIDs 和ports/GIDs 中找到。

## 6.30. 依赖于内核源代码的Ports

一些ports（如内核可加载模块）需要内核源文件，以便port进行编译。以下是确定用户是否已安装它们的正确方法：

```
USES=	kmod
```

除了这个检查之外， kmod 功能会处理大多数这些ports需要考虑的项目。

## 6.31. Go 库

Ports不得打包或安装 Go 库或源代码。Goports必须在正常获取时间获取所需的依赖项，并且只应安装用户需要的程序和内容，而不是 Go 开发人员需要的内容。

Ports应该（按优先顺序）：

* 使用包源代码中包含的 vendored 依赖项。
* 获取上游指定的依赖项版本（在 go.mod、vendor.json 或类似情况下）。
* 作为最后的手段（依赖项不包括，版本也没有明确指定），获取上游开发/发布时可用的依赖项版本。

## 6.32. 哈斯克尔库

就像 Go 语言一样，Ports 不得打包或安装 Haskell 库。Haskellports 必须与其依赖项静态链接，并在提取阶段获取所有分发文件。

## 6.33. Shell 完成文件

许多现代 shells （包括 bash、fish、tcsh 和 zsh）支持参数和/或选项的制表完成。这种支持通常来自完成文件，其中包含了特定命令的制表完成工作定义。Ports 有时会随附其自己的完成文件，或者移植者可能已经自己创建了它们。

可用时，应始终安装完成文件。没有必要对此进行选项。不过，如果使用选项，在 OPTIONS_DEFAULT 中始终启用它。

表 43. 完整shell完成文件名

| `bash` | ${PREFIX}/etc/bash_completion.d 或${PREFIX}/share/bash-completion/completions | （这些文件夹中的任何唯一文件名） |
| -- | ------------------------------------------------------------------------------- | ---------------------------------- |
| `fish` | \${PREFIX}/share/fish/completions/\${PORTNAME}.fish                     |                                  |
| `zsh` | \${PREFIX}/share/zsh/site-functions/\_\${PORTNAME}                   |                                  |

不要在 shell 本身上注册任何依赖项。
