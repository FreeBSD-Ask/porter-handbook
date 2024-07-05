# 第13章 该做什么和不该做什么

## 13.1. 介绍

这里列出了一些常见的注意事项和禁忌，这些在移植过程中经常遇到。将port与此列表进行对比，但也请检查 PR 数据库中他人提交的ports。按照“Bug Reports and General Commentary”中的描述提交任何关于ports的评论。检查 PR 数据库中的ports，这将使我们更快地提交它们，并证明你知道自己在做什么。

## 13.2. `WRKDIR`

不要向 WRKDIR 外的文件写入任何内容。 WRKDIR 是port构建过程中唯一保证可写的地方（参见从 CDROM 安装ports来了解从只读树构建ports的示例）。pkg-*文件可以通过重新定义变量而不是覆盖文件来修改。

## 13.3. `WRKDIRPREFIX`

确保port遵循 WRKDIRPREFIX 。大多数ports不必担心这一点。特别是，当引用另一个port的 WRKDIR 时，注意正确的位置是${WRKDIRPREFIX}${PORTSDIR}/subdir/name/work，而不是${PORTSDIR}/subdir/name/work 或${.CURDIR}/../../subdir/name/work 或其他类似的地方。

## 13.4. 区分操作系统和操作系统版本

一些代码需要根据运行在哪个版本的 FreeBSD Unix 进行修改或条件编译。 区分 FreeBSD 版本的首选方法是在 sys/param.h 中定义的 __FreeBSD_version 和 **FreeBSD** 宏。如果没有包含此文件，请在.c 文件的适当位置添加代码，

```
#include <sys/param.h>
```

到适当位置在.c 文件中。

**FreeBSD** 定义在所有版本的 FreeBSD 中作为它们的主要版本号。例如，在 FreeBSD 9.x 中， **FreeBSD** 被定义为 9 。

```
#if __FreeBSD__ >= 9
#  if __FreeBSD_version >= 901000
	 /* 9.1+ release specific code here */
#  endif
#endif
```

__FreeBSD_version 值的完整列表可在 _ _FreeBSD_version Values 中获得。

## 13.5. 写在 bsd.port.mk 之后

不要在 .include <bsd.port.mk> 行之后写任何内容。通常可以通过在 Makefile 的中间某处包含 bsd.port.pre.mk 来避免这种情况，然后在最后加上 bsd.port.post.mk。

|  | 只包含 bsd.port.pre.mk/bsd.port.post.mk 对或者只包含 bsd.port.mk；不要混合使用这两种方式。 |
| -- | -------------------------------------------------------------------------------------------- |

bsd.port.pre.mk 只定义了一些变量，可以在 Makefile 中的测试中使用，bsd.port.post.mk 定义了其余部分。

这里是在 bsd.port.pre.mk 中定义的一些重要变量（这不是完整列表，请阅读 bsd.port.mk 获取完整列表）。

| 变量 | 描述                                                        |
| ------ | ------------------------------------------------------------- |
| `ARCH`     | 由 uname -m 返回的架构（例如， i386 ）                      |
| `OPSYS`     | 操作系统类型，由 uname -s 返回（例如， FreeBSD ）           |
| `OSREL`     | 操作系统的发行版本（例如， 2.1.5 或 2.2.7 ）                |
| `OSVERSION`     | 操作系统的数字版本；与 \_\_FreeBSD\_version 相同。 |
| `LOCALBASE`     | “本地”树的基础（例如， /usr/local ）                      |
| `PREFIX`     | port 安装自身的位置（请参阅更多关于 PREFIX 的信息）。       |

|  | 当需要 MASTERDIR 时，始终在包含 bsd.port.pre.mk 之前定义它。 |
| -- | -------------------------------------------------------------- |

这里是一些可以在 bsd.port.pre.mk 之后添加的示例：

```
# no need to compile lang/perl5 if perl5 is already in system
.if ${OSVERSION} > 300003
BROKEN=	perl is in system
.endif
```

在 BROKEN= 之后始终使用制表符而不是空格。

## 13.6. 在包装脚本中使用 exec 语句

如果port安装了一个shell脚本，其目的是启动另一个程序，并且如果启动该程序是脚本执行的最后一个操作，请确保使用 exec 语句启动该程序，例如：

```
#!/bin/sh
exec %%LOCALBASE%%/bin/java -jar %%DATADIR%%/foo.jar "$@"
```

exec 语句将shell进程替换为指定的程序。如果省略 exec ，则在程序执行时shell进程仍保留在内存中，并且会不必要地消耗系统资源。

## 13.7. 合理地做事

Makefile 应该以简单和合理的方式做事。让它变得更短或更易读总是更好的。例如，使用 make .if 结构代替 shell if 结构，如果重新定义 EXTRACT* 足够的话不重新定义 do-extract ，并使用 GNU_CONFIGURE 而不是 CONFIGURE_ARGS += --prefix=${PREFIX} 。

如果需要大量新代码来做某事，可能已经在 bsd.port.mk 中有其实现。虽然难以阅读，但似乎有许多看似困难的问题，而 bsd.port.mk 已经提供了一个简便的解决方案。

## 13.8. 尊重 CC 和 CXX

port必须尊重 CC 和 CXX 。我们的意思是port不能绝对设置这些变量的值，覆盖现有的值；相反，它可以向现有值添加所需的值。这样可以全局设置影响所有ports的构建选项。

如果port不尊重这些变量，请将 NO_PACKAGE=ignores either cc or cxx 添加到 Makefile 中。

这是一个尊重 CC 和 CXX 的 Makefile 示例。请注意 ?= ：

```
CC?= gcc
```

```
CXX?= g++
```

这是一个既不尊重 CC 也不尊重 CXX 的示例：

```
CC= gcc
```

```
CXX= g++
```

在 FreeBSD 系统上，可以在 /etc/make.conf 中定义 CC 和 CXX 。第一个示例在未在 /etc/make.conf 中先前设置值时定义一个值，保留任何系统范围的定义。第二个示例会覆盖之前定义的任何内容。

## 赞赏 CFLAGS 安全

必须尊重 CFLAGS 。我们的意思是，port 不得绝对设置此变量的值，覆盖现有值。相反，它可以向现有值追加所需的任何值。这样可以全局设置影响所有 ports 的构建选项。

如果没有，请将 NO_PACKAGE=ignores cflags 添加到 Makefile。

这里是尊重 CFLAGS 的一个示例的 Makefile。请注意 += ：

```
CFLAGS+= -Wall -Werror
```

这是一个不尊重 CFLAGS 的示例：

```
CFLAGS= -Wall -Werror
```

CFLAGS 在 FreeBSD 系统中在 /etc/make.conf 中被定义。第一个示例将附加额外的标志到 CFLAGS ，保留任何系统范围的定义。第二个示例将覆盖先前定义的任何内容。

从第三方 Makefile 中删除优化标志。该系统 CFLAGS 包含全局系统优化标志。未修改的 Makefile 示例：

```
CFLAGS= -O3 -funroll-loops -DHAVE_SOUND
```

使用系统优化标志，Makefile 将类似于此示例：

```
CFLAGS+= -DHAVE_SOUND
```

## 13.10. 详细构建日志

使 port 构建系统在构建阶段显示所有执行的命令。完整的构建日志对调试 port 问题至关重要。

非信息性构建日志示例（不好）：

```
  CC     source1.o
  CC     source2.o
  CCLD   someprogram
```

冗长的构建日志示例（好）：

```
cc -O2 -pipe -I/usr/local/include -c -o source1.o source1.c
cc -O2 -pipe -I/usr/local/include -c -o source2.o source2.c
cc -o someprogram source1.o source2.o -L/usr/local/lib -lsomelib
```

一些构建系统，如 CMake、ninja 和 GNU configure，已经为ports框架设置了详细日志记录。在其他情况下，ports可能需要单独的调整。

## 13.11. 反馈

请向上游维护者发送适用的更改和补丁，以便包含在下一个版本的代码发布中。这样做将使更新到下一个版本变得更加容易。

## 13.12. README.html

README.html 不属于port的一部分，而是由 make readme 生成的。在补丁或提交中不要包括此文件。

|  | 如果 make readme 失败，请确保port未修改 ECHO_MSG 的默认值。 |
| -- | ------------------------------------------------------------- |

## 13.13. 用 Port， BROKEN 或 FORBIDDEN 或 IGNORE 标记Port 为不可安装

在某些情况下，必须阻止用户安装port。 可以在port的 Makefile 中使用几个变量，告诉用户port无法安装。 这些 make 变量的值将是向用户显示port拒绝安装自身的原因。 请使用正确的 make 变量。 每个变量都传达着截然不同的含义，既对用户，也对依赖于 Makefiles 的自动化系统，如ports构建集群和 FreshPorts。

### 13.13.1. 变量

* BROKEN 保留给目前无法正确编译、安装、卸载或运行的 ports。用于被认为是暂时性问题的 ports。如果指示，构建集群仍将尝试构建它们，以查看潜在问题是否已解决。（但是，通常情况下，集群运行时不会这样做。）
  例如，当 port:

  * 无法编译时使用 {{0}}
  * 在配置或安装过程中失败
  * 在${PREFIX}之外安装文件
  * 在卸载时未完全清理所有文件（但是，允许且希望port保留用户修改的文件）
  * 在应该正常运行的系统上存在运行时问题。
* FORBIDDEN 用于包含安全漏洞或引起对 FreeBSD 系统安全性产生严重关注的ports（例如，一个声誉不佳的不安全程序或提供易受攻击服务的程序）。一旦某个软件存在漏洞且没有发布升级版本，立即将ports 标记为 FORBIDDEN 。理想情况下，在发现安全漏洞时尽快升级ports，以减少易受攻击的 FreeBSD 主机数量（我们希望以安全著称），但有时漏洞披露和受影响软件更新发布之间会有明显的时间差。不要因其他原因而将port 标记为 FORBIDDEN ，只能基于安全性原因标记。
* IGNORE 保留给因某些其他原因而不得构建的ports。在认为问题是结构性的ports中使用它。构建集群绝对不会构建标记为 IGNORE 的ports。例如，当port时使用 IGNORE ：

  * 定制安装版本的 FreeBSD 上不起作用
  * 由于许可限制，可能无法自动获取 distfile
  * 与当前安装的某些其他port不兼容（例如，port依赖于 www/drupal7，但安装了 www/drupal8）||如果port与当前安装的port发生冲突（例如，如果它们在执行不同功能的相同位置安装文件），请改用 CONFLICTS 。 CONFLICTS 将自行设置 IGNORE 。|
    | --| ----------------------------------------------------------------------------------------------------------------------------------------------|

### 13.13.2. 实施说明

不要引用 BROKEN ， IGNORE 和相关变量的值。由于向用户显示信息的方式，每个变量的消息措辞不同：

```
BROKEN=	fails to link with base -lcrypto
```

```
IGNORE=	unsupported on recent versions
```

从 make describe 导致此输出：

```
===>  foobar-0.1 is marked as broken: fails to link with base -lcrypto.
```

```
===>  foobar-0.1 is unsupported on recent versions.
```

## 13.14. 建筑考虑

### 13.14.1. 关于架构的一般说明

FreeBSD 运行在许多不仅仅是众所周知的基于 x86 的处理架构上。某些ports有特定于其中一个或多个这些架构的约束条件。

要查看支持的架构列表，请运行：

```
cd ${SRCDIR}; make targets
```

值显示为 TARGET / TARGET_ARCH 。ports只读的 Makevar ARCH 是基于 TARGET_ARCH 的值设定的。Port的 Makefile 应该测试这个 Makevar 的值。

### 将 Port 标记为体系结构中性

没有任何架构相关文件或要求的 Ports 通过设置 NO_ARCH=yes 进行识别。

从这类 ports 构建的软件包，其架构字符串以 :* 结尾（通配符架构），而不是例如 freebsd:13:x86:64 （amd64 架构）。

|  | NO_ARCH 旨在表明无需为每个受支持的架构构建软件包。目标是减少构建和分发软件包所需的资源，如网络带宽、镜像空间和分发媒体上的磁盘空间。然而，目前我们的软件包基础设施（例如软件包管理器、镜像和软件包构建器）尚未设置完全从中受益。 |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### 13.14.3. 将 Port 标记为仅在某些架构上被忽略

* 要将 port 标记为仅在某些架构上被 IGNORE ，有两个其他方便的变量将自动设置 IGNORE ： ONLY_FOR_ARCHS 和 NOT_FOR_ARCHS 。示例：

  ```
  ONLY_FOR_ARCHS=	i386 amd64
  ```

  ```
  NOT_FOR_ARCHS=	ia64 sparc64
  ```

  自定义 IGNORE 消息可使用 ONLY_FOR_ARCHS_REASON 和 NOT_FOR_ARCHS_REASON 进行设置。使用 ONLY_FOR_ARCHS_REASON_ARCH 和 NOT_FOR_ARCHS_REASON_ARCH 可以实现按体系结构的条目。
* 如果port获取 i386 二进制文件并安装它们，则设置 IA32_BINARY_PORT 。如果设置了此变量，则 IA32 版本的库必须存在/usr/lib32，并且内核必须支持 IA32 兼容性。如果这两个依赖关系中的一个未满足，则 IGNORE 将被自动设置。

### 13.14.4. 集群特定考虑事项

* 有些ports会尝试通过向编译器指定 -march=native 来调整到正在构建的准确机器。这应该避免：要么将其列在关闭默认选项下，要么完全删除它。否则，构建集群生成的默认软件包可能无法在该 ARCH 的每台机器上运行。

## 13.15. 使用 DEPRECATED 或 EXPIRATION_DATE 标记一个Port以进行移除

请记住， BROKEN 和 FORBIDDEN 应作为暂时的资源，如果一个port无法正常工作。永久损坏的ports将被完全从树中移除。

当有必要时，可以使用 DEPRECATED 和 EXPIRATION_DATE 提醒用户即将port。前者是一个字符串，解释为什么计划移除port；后者是一个符合 ISO 8601 格式（YYYY-MM-DD）的字符串。这两者将展示给用户。

可以设置 DEPRECATED 而不必有 EXPIRATION_DATE （例如，推荐port的更新版本），但反过来却毫无意义。

|  | 当将port标记为 DEPRECATED 时，如果有任何可用作被废弃替代品的替代ports，在提交消息中提到它们会很方便。 |
| -- | ------------------------------------------------------------------------------------------------------- |

没有关于给予多少通知的固定政策。 目前的做法似乎是对与安全相关问题给予一个月的通知期，对于构建问题则为两个月。 这也给任何感兴趣的贡献者一些时间来修复问题。

## 13.16. 避免使用 .error 结构

Makefile 向用户表明由于某些外部因素（例如，用户已经指定了一组非法的构建选项组合）无法安装 port 的正确方式是将一个非空值设置为 IGNORE 。 这个值将由 make install 格式化并显示给用户。

使用 .error 于此目的是一个常见错误。这样做的问题在于许多与 ports 树相关的自动化工具在这种情况下将失败。最常见的情况是在尝试构建 /usr/ports/INDEX 时发生(请查看运行 make describe )。然而，即使是更琐碎的命令例如 make maintainer 在这种情况下也会失败。这是不可接受的。

示例 1. 如何避免使用 .error

下面两个 Makefile 片段中的第一个将导致 make index 失败，而第二个将不会：

```
.error "option is not supported"
```

```
IGNORE=option is not supported
```

## 13.17. 使用 sysctl

除了在目标中使用外，不推荐使用 sysctl。这是因为像在 make index 中使用时，任何 makevar 命令的评估都必须运行该命令，进一步减慢该过程。

只能通过 SYSCTL 使用 sysctl(8) ，因为它包含完全合格的路径，并且可以被覆盖，如果有特殊需要的话。

## 13.18. 重新生成 Distfiles

有时软件作者会更改已发布的 distfiles 的内容，而不更改文件的名称。验证更改是否是官方的，并已由作者执行。过去曾经发生过，在下载服务器上悄悄更改了 distfile，意图是造成损害或 comprom 动最终用户的安全性。

将旧的 distfile 放在一边，下载新的 distfile，解压它们并使用 diff(1) 比较内容。如果没有可疑之处，就更新 distinfo。

|  | 确保总结 PR 和提交日志中的差异，以便其他人知道没有发生什么不好的事情。 |
| -- | ------------------------------------------------------------------------ |

联系软件的作者，并与他们确认更改。

## 13.19. 使用 POSIX 标准

FreeBSD ports 通常期望符合 POSIX。一些软件和构建系统基于特定操作系统或环境做出假设，可能在 port 中使用时会导致问题。

如果有其他获取信息的方式，请不要使用 /proc。例如，在 main() 中使用 setprogname(argv[0]) ，然后使用 getprogname(3) 来获取可执行文件名。

不要依赖 POSIX 未记录的行为。

不记录应用程序关键路径中的时间戳，如果应用程序没有时间戳也能正常工作。获取时间戳可能会很慢，这取决于操作系统中时间戳的准确性。如果确实需要时间戳，确定它们必须精确到什么程度，并使用一个文档记录为提供所需精度的 API。

在 Linux® 上，许多简单的系统调用（例如 gettimeofday(2)，getpid(2)）比其他任何操作系统都快得多，这是由于缓存和 vsyscall 性能优化。在性能关键的应用程序中，不要指望它们的性能低廉。一般来说，尽量避免系统调用，如果可能的话。

不要依赖于 Linux® 特定的套接字行为。特别是，默认的套接字缓冲区大小是不同的（使用 setsockopt(2)调用 SO_SNDBUF 和 SO_RCVBUF ），并且当套接字缓冲区已满时，Linux® 的 send(2) 会阻塞，而 FreeBSD 的则会失败并在 errno 中设置 ENOBUFS 。

如果需要依赖非标准行为，请将其正确封装到一个通用的 API 中，在配置阶段检查该行为，如果缺失则停止。

检查 man 手册，查看使用的函数是否是 POSIX 接口（在 man 手册的 "STANDARDS" 部分）。

不要假设 /bin/sh 就是 bash。确保传递给 system(3) 的命令行与 POSIX 兼容。

一个常见的 bashism 列表在这里。

检查头部是否以 POSIX 或 man 页推荐的方式包含。例如，sys/types.h 经常被遗忘，这对 Linux®来说不是那么大的问题，但对 FreeBSD 来说是一个问题。

## 13.20. 杂项

要始终仔细检查 pkg-descr 和 pkg-plist。如果正在审核一个port并且可以达到更好的措辞，请这样做。

请注意任何法律问题！不要让我们非法分发软件！
