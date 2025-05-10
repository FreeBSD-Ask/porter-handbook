# 第13章 该做什么和不该做什么

## 13.1. 介绍

以下是移植过程中常见的一些注意事项和禁忌。请对照本清单检查 Port，同时也请查看其他人提交到 [PR 数据库](https://bugs.freebsd.org/search/) 中的 Ports。根据 [Bug Reports and General Commentary](https://docs.freebsd.org/en/articles/contributing/#CONTRIB-GENERAL) 中的说明提交对 Ports 的任何意见。检查 PR 数据库中的 Ports 不仅可以加快我们提交的速度，还可以证明你熟悉相关操作。

## 13.2. `WRKDIR`

不要向 `WRKDIR` 之外的文件写入任何内容。`WRKDIR` 是构建 Port 过程中唯一保证可写的地方（参见 [从光盘安装 Ports](https://docs.freebsd.org/en/books/handbook/#PORTS-CD) 中只读树构建 Ports 的示例）。**pkg-\*** 文件可以通过 [重新定义变量](https://docs.freebsd.org/en/books/porters-handbook/pkg-files/#pkg-names) 的方式修改，而不是直接覆盖文件。

## 13.3. `WRKDIRPREFIX`

确保 Port 支持 `WRKDIRPREFIX`。大多数 Ports 不需要关心这一点。尤其是在引用其他 Port 的 `WRKDIR` 时，应注意正确的位置是 **\${WRKDIRPREFIX}\${PORTSDIR}/subdir/name/work**，而不是 **\${PORTSDIR}/subdir/name/work** 或 **\${.CURDIR}/../../subdir/name/work** 或类似路径。

## 13.4. 区分操作系统及其版本

某些代码需要根据所运行的 FreeBSD Unix 版本进行修改或条件编译。区分 FreeBSD 版本的推荐方式是使用定义在 [sys/param.h](https://cgit.freebsd.org/src/tree/sys/sys/param.h) 中的 `__FreeBSD_version` 和 `__FreeBSD__` 宏。如果未包含该文件，请在对应的 **.c** 文件中添加如下代码：

```c
#include <sys/param.h>
```

`__FreeBSD__` 在所有版本的 FreeBSD 中都定义为其主版本号。例如，在 FreeBSD 9.x 中，`__FreeBSD__` 被定义为 `9`。

```c
#if __FreeBSD__ >= 9
#  if __FreeBSD_version >= 901000
	 /* 9.1+ 版本特有的代码 */
#  endif
#endif
```

`__FreeBSD_version` 的完整值列表可参考 [\_\_FreeBSD\_version 值](https://docs.freebsd.org/en/books/porters-handbook/versions/#versions)。

## 13.5. 在 bsd.port.mk 之后写内容

不要在 `.include <bsd.port.mk>` 这一行之后写任何内容。通常可以通过在 **Makefile** 中部引入 **bsd.port.pre.mk**，在末尾引入 **bsd.port.post.mk** 来避免这种情况。

>**重要**
>
>要么只使用 **bsd.port.mk**，要么使用 **bsd.port.pre.mk**/**bsd.port.post.mk** 配对；请不要混用这两种方式。 


**bsd.port.pre.mk** 只定义了一些变量，可以在 **Makefile** 中进行判断使用，而 **bsd.port.post.mk** 定义其余变量。

下面是一些 **bsd.port.pre.mk** 定义的重要变量（不是完整列表，完整内容请参考 **bsd.port.mk**）：

| 变量名         | 描述                                                                                                             |
| ----------- | -------------------------------------------------------------------------------------------------------------- |
| `ARCH`      | 架构类型，`uname -m` 返回的结果（例如，`i386`）                                                                               |
| `OPSYS`     | 操作系统类型，`uname -s` 返回的结果（例如，`FreeBSD`）                                                                          |
| `OSREL`     | 操作系统的发布版本（例如，`2.1.5` 或 `2.2.7`）                                                                                |
| `OSVERSION` | 操作系统的数字版本；与 [`__FreeBSD_version`](https://docs.freebsd.org/en/books/porters-handbook/versions/#versions) 相同    |
| `LOCALBASE` | “本地”目录的根目录（例如，`/usr/local`）                                                                                    |
| `PREFIX`    | Port 安装的位置（详见 [关于 `PREFIX` 的更多信息](https://docs.freebsd.org/en/books/porters-handbook/testing/#porting-prefix)） |

>**注意**
>
> 如果需要使用 `MASTERDIR`，务必在包含 **bsd.port.pre.mk** 之前定义它。 

以下是一些可以在 **bsd.port.pre.mk** 之后添加的示例：

```c
# 如果系统中已有 perl5，就无需编译 lang/perl5
.if ${OSVERSION} > 300003
BROKEN=	perl is in system
.endif
```

`BROKEN=` 后请始终使用 tab 而不是空格。

## 13.6. 在封装脚本中使用 `exec` 语句

如果 Port 安装了一个 shell 脚本，而该脚本的目的是启动另一个程序，并且启动该程序是脚本执行的最后一步，那么请务必使用 `exec` 语句来启动该程序，例如：

```sh
#!/bin/sh
exec %%LOCALBASE%%/bin/java -jar %%DATADIR%%/foo.jar "$@"
```

`exec` 语句会将当前 shell 进程替换为指定程序。如果省略 `exec`，则 shell 进程在程序运行期间仍会驻留在内存中，造成系统资源的无谓浪费。

## 13.7. 理性地处理问题

**Makefile** 应该以简单合理的方式完成任务。使其更简洁或更易读总是更好的做法。示例包括使用 make 的 `.if` 结构代替 shell 的 `if` 结构；如果只需重定义 `EXTRACT*` 就足够了，就不要重定义 `do-extract`；使用 `GNU_CONFIGURE` 而不是 `CONFIGURE_ARGS += --prefix=${PREFIX}`。

如果为了实现某个功能需要添加大量新代码，很可能在 **bsd.port.mk** 中已经有相关实现。虽然 **bsd.port.mk** 难以阅读，但它为许多看似复杂的问题提供了简洁的解决方案。

## 13.8. 遵守 `CC` 和 `CXX`

Port 必须遵守 `CC` 和 `CXX`。这意味着不能直接覆盖这两个变量的值，而应在已有值的基础上追加所需内容。这样做可以确保影响所有 Ports 的构建选项能被全局设置。

如果 Port 没有遵守这些变量，请在 **Makefile** 中添加 `NO_PACKAGE=ignores either cc or cxx`。

以下是一个遵守 `CC` 和 `CXX` 的 **Makefile** 示例。注意 `?=` 的使用：

```makefile
CC?= gcc
```

```makefile
CXX?= g++
```

以下是不遵守的示例：

```makefile
CC= gcc
```

```makefile
CXX= g++
```

在 FreeBSD 系统中，`CC` 和 `CXX` 可以在 **/etc/make.conf** 中定义。第一个示例仅在变量未被预先定义时赋值，保留了系统范围的定义；而第二个示例则会覆盖已有定义。

## 13.9. 遵守 `CFLAGS`

Port 必须遵守 `CFLAGS`。这意味着不能直接覆盖该变量的值，而应在已有值的基础上追加所需内容。这样可以使影响所有 Ports 的构建选项可以全局设置。

如果未遵守，请在 **Makefile** 中添加 `NO_PACKAGE=ignores cflags`。

以下是一个遵守 `CFLAGS` 的 **Makefile** 示例。注意 `+=` 的使用：

```makefile
CFLAGS+= -Wall -Werror
```

以下是不遵守的示例：

```makefile
CFLAGS= -Wall -Werror
```

`CFLAGS` 在 FreeBSD 系统中定义于 **/etc/make.conf**。第一个示例将额外参数附加到已有的 `CFLAGS` 上，保留系统级定义；而第二个示例会覆盖原有定义。

请从第三方 **Makefile** 中移除优化标志。系统的 `CFLAGS` 已包含系统范围的优化选项。以下是未修改的 **Makefile** 中的示例：

```makefile
CFLAGS= -O3 -funroll-loops -DHAVE_SOUND
```

使用系统优化标志后的 **Makefile** 应类似如下：

```makefile
CFLAGS+= -DHAVE_SOUND
```

## 13.10. 显示详细构建日志

请确保 Port 的构建系统在构建阶段显示所有执行的命令。完整的构建日志对于调试 Port 问题至关重要。

不具可读性的构建日志示例（错误）：

```sh
CC     source1.o
  CC     source2.o
  CCLD   someprogram
```

详细的构建日志示例（正确）：

```sh
cc -O2 -pipe -I/usr/local/include -c -o source1.o source1.c
cc -O2 -pipe -I/usr/local/include -c -o source2.o source2.c
cc -o someprogram source1.o source2.o -L/usr/local/lib -lsomelib
```

某些构建系统（如 CMake、ninja 和 GNU configure）在 Ports 框架中已经配置为显示详细日志。其他情况下，Port 可能需要单独调整。

## 13.11. 反馈

请将相关的更改和补丁提交给上游维护者，以便在下一版本中包含这些更改。这样有助于简化后续版本的更新过程。

## 13.12. README.html

**README.html** 并不是 Port 的一部分，而是通过执行 `make readme` 生成的。请勿将此文件包含在补丁或提交中。

>**注意**
>
>如果 `make readme` 失败，请确认 Port 未修改 `ECHO_MSG` 的默认值。 
  

## 13.13. 使用 `BROKEN`、`FORBIDDEN` 或 `IGNORE` 标记不可安装的 Port

在某些情况下，必须阻止用户安装某个 Port。可以在 Port 的 **Makefile** 中使用几个变量来告诉用户该 Port 无法安装。这些 make 变量的值将作为 Port 拒绝安装的原因显示给用户。请使用正确的 make 变量。每个变量在语义上有明显不同，不仅对用户如此，对依赖 **Makefile** 的自动化系统（如 [Ports 构建集群](https://docs.freebsd.org/en/books/porters-handbook/keeping-up/#build-cluster) 和 [FreshPorts](https://docs.freebsd.org/en/books/porters-handbook/keeping-up/#freshports)）也是如此。

### 13.13.1. 各变量说明

* `BROKEN`：用于目前无法编译、安装、卸载或正常运行的 Port。适用于预计问题是暂时性的情形。
  如果设置，构建集群在收到指示的情况下仍可能尝试构建以查看问题是否已经解决。（但通常构建集群不会这样做。）

  例如，当一个 Port：

  * 无法编译；
  * 配置或安装过程失败；
  * 将文件安装到 **\${PREFIX}** 之外；
  * 卸载时未能清除所有文件（不过如果留下的是用户已修改的文件，可能是可以接受甚至是期望的）；
  * 在本应能正常运行的系统上运行失败；

  应使用 `BROKEN`。

* `FORBIDDEN`：用于含有安全漏洞，或对 FreeBSD 系统安全构成严重隐患的 Port（例如：臭名昭著的不安全程序，或提供易被攻击的服务的程序）。一旦发现某个软件存在漏洞且无可用升级版本，应立即将相关 Port 标记为 `FORBIDDEN`。理想情况下，发现安全问题时应尽快升级 Port，以减少 FreeBSD 主机的漏洞数量（我们以安全著称）。除安全问题外，请勿使用 `FORBIDDEN`。

* `IGNORE`：用于因其他原因绝不能构建的 Port。适用于认为问题具有结构性特征的情形。构建集群在任何情况下都不会构建被标记为 `IGNORE` 的 Port。例如，在以下情况下应使用 `IGNORE`：

  * Port 不支持当前安装的 FreeBSD 版本；
  * distfile 因许可证限制不能自动获取；
  * 与当前已安装的其他 Port 不兼容（例如：该 Port 依赖 [www/drupal7](https://cgit.freebsd.org/ports/tree/www/drupal7/)，而系统已安装的是 [www/drupal8](https://cgit.freebsd.org/ports/tree/www/drupal8/)）；

  >**注意**
  >
  >如果 Port 与当前安装的其他 Port 冲突（例如，它们安装了具有不同功能但同名的文件），[请使用 `CONFLICTS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#conflicts)。`CONFLICTS` 会自动设置 `IGNORE`。 


### 13.13.2. 实现注意事项

不要给 `BROKEN`、`IGNORE` 及相关变量的值加引号。由于这些信息最终会显示给用户，每个变量所使用的消息措辞是不同的：

```makefile
BROKEN=	fails to link with base -lcrypto
```

```makefile
IGNORE=	unsupported on recent versions
```

这样，`make describe` 的输出如下所示：

```makefile
===>  foobar-0.1 is marked as broken: fails to link with base -lcrypto.
```

```makefile
===>  foobar-0.1 is unsupported on recent versions.
```

## 13.14. 架构相关注意事项

### 13.14.1. 架构通用说明

FreeBSD 支持的处理器架构远不止常见的 x86 架构。一些 Port 仅适用于某些特定架构，或者在某些架构下会有问题。

要查看支持的架构列表，请执行以下命令：

```makefile
cd ${SRCDIR}; make targets
```

返回的值为 `TARGET`/`TARGET_ARCH` 形式。只读 make 变量 `ARCH` 的值基于 `TARGET_ARCH` 设置。Port 的 **Makefile** 应当测试该变量的值。

### 13.14.2. 标记为架构无关的 Port

不包含任何依赖特定架构文件或要求的 Port 应设置 `NO_ARCH=yes`。

这类 Port 构建出的软件包，其架构字符串将以 `:*`（通配架构）结尾，例如不是 `freebsd:13:x86:64`（即 amd64 架构）。

>**注意**
>
>设置 `NO_ARCH` 的目的在于表明无需为所有支持架构单独构建包。这样可以减少构建和分发软件包所消耗的资源，例如网络带宽、镜像服务器和分发介质的磁盘空间。但当前的软件包基础设施（例如软件包管理器、镜像、构建系统）尚未完全利用 `NO_ARCH` 带来的优势。 

### 13.14.3. 仅在特定架构上标记为不可构建的 Port

* 若希望仅在某些架构上将 Port 标记为 `IGNORE`，可使用 `ONLY_FOR_ARCHS` 和 `NOT_FOR_ARCHS` 这两个便利变量，它们会自动设置 `IGNORE`。示例：

  ```makefile
  ONLY_FOR_ARCHS=	i386 amd64
  ```

  ```makefile
  NOT_FOR_ARCHS=	ia64 sparc64
  ```

  可使用 `ONLY_FOR_ARCHS_REASON` 和 `NOT_FOR_ARCHS_REASON` 设置自定义的 `IGNORE` 消息。也可以针对特定架构使用 `ONLY_FOR_ARCHS_REASON_ARCH` 和 `NOT_FOR_ARCHS_REASON_ARCH` 设定说明。

* 若 Port 下载并安装的是 i386 的二进制文件，应设置 `IA32_BINARY_PORT`。若设置了该变量，则系统必须存在 **/usr/lib32** 以提供 IA32 库，同时内核必须支持 IA32 兼容性。如果这两个条件之一未满足，则将自动设置 `IGNORE`。

### 13.14.4. 与构建集群相关的注意事项

* 某些 Port 会尝试通过向编译器传递 `-march=native` 参数来针对当前构建机器做出优化。这种做法应避免：要么作为默认关闭的选项列出，要么彻底删除该参数。
  否则，由构建集群构建的默认软件包可能无法在该架构下的所有机器上运行。

## 13.15. 使用 `DEPRECATED` 和 `EXPIRATION_DATE` 标记待移除的 Port

请记住，`BROKEN` 和 `FORBIDDEN` 应仅作为暂时手段。如果一个 Port 永久不可用，它将会被从 Ports 树中移除。

在适当的时候，可以使用 `DEPRECATED` 和 `EXPIRATION_DATE` 提前提醒用户 Port 即将被移除。其中，`DEPRECATED` 是说明被弃用原因的字符串，`EXPIRATION_DATE` 是 ISO 8601 格式的日期字符串（YYYY-MM-DD）。两者都会显示给用户。

可以仅设置 `DEPRECATED` 而不设置 `EXPIRATION_DATE`（例如建议使用新版 Port），但反过来是不允许的。

>**注意**
>
>标记 Port 为 `DEPRECATED` 时，若有替代 Port 可用，建议在提交说明中提及替代项。

并没有明确规定应提前多久通知用户。当前的惯例是：安全相关问题提前一个月通知，构建相关问题提前两个月通知。这也为感兴趣的提交者留出时间修复问题。

## 13.16. 避免使用 `.error` 结构

**Makefile** 中如果需要因为某些外部因素（例如用户指定了不合法的构建选项组合）而阻止 Port 安装，正确的做法是给 `IGNORE` 赋一个非空的值。该值会在执行 `make install` 时被格式化并展示给用户。

一个常见的错误是为此目的使用 `.error`。这样做的问题在于，许多与 Ports 树交互的自动化工具在遇到这种情况时会失败。最常见的例子是在尝试构建 **/usr/ports/INDEX** 时失败（参见 [运行 `make describe`](https://docs.freebsd.org/en/books/porters-handbook/testing/#make-describe)）。但即使是非常简单的命令，例如 `make maintainer`，也会因此失败。这种行为是不可接受的。

**示例 1：如何避免使用 `.error`**

下面两个 **Makefile** 片段中，第一个会导致 `make index` 失败，而第二个不会：

```makefile
.error "option is not supported"
```

```makefile
IGNORE=option is not supported
```

## 13.17. 使用 sysctl

除非用于目标（targets）中，通常不应在 Makefile 中使用 **sysctl**。这是因为在处理 `makevar`（例如运行 `make index` 时）时会执行该命令，从而进一步拖慢处理速度。

仅应通过 `SYSCTL` 使用 [sysctl(8)](https://man.freebsd.org/cgi/man.cgi?query=sysctl&sektion=8&format=html)，因为它包含完整路径，并且在必要时可以被覆盖。

## 13.18. 重新打包 distfile

有时软件作者会在不更改文件名的情况下，修改已发布的 distfile 内容。应验证这些更改是官方行为，且确实由作者执行。过去曾有 distfile 在下载服务器上被暗地里篡改，意图造成损害或危及终端用户的安全。

请将旧的 distfile 备份，下载新的文件，解压后使用 [diff(1)](https://man.freebsd.org/cgi/man.cgi?query=diff&sektion=1&format=html) 比较内容。如果确认没有可疑之处，再更新 **distinfo**。

>**重要**
>
> 请务必在 PR 和提交日志中简要说明差异，这样其他人可以了解没有发生恶意行为。


联系软件作者并向他们确认这些更改。

## 13.19. 遵循 POSIX 标准

FreeBSD Ports 通常期望软件遵循 POSIX 标准。有些软件和构建系统依赖某些特定操作系统或环境的行为，这会在作为 Port 使用时引发问题。

如果有其他方式获取信息，不要使用 **/proc**。例如，可以在 `main()` 中使用 `setprogname(argv[0])`，随后调用 [getprogname(3)](https://man.freebsd.org/cgi/man.cgi?query=getprogname&sektion=3&format=html) 来获取可执行文件名称。

不要依赖 POSIX 没有文档化的行为。

如果应用程序在没有时间戳的情况下也能正常工作，就不要在关键路径中记录时间戳。获取时间戳可能较慢，具体取决于操作系统的时间戳精度。如果确实需要时间戳，应确定所需精度，并使用明确能提供该精度的 API。

一些简单的系统调用（例如 [gettimeofday(2)](https://man.freebsd.org/cgi/man.cgi?query=gettimeofday&sektion=2&format=html)、[getpid(2)](https://man.freebsd.org/cgi/man.cgi?query=getpid&sektion=2&format=html)）在 Linux® 上因为缓存和 vsyscall 优化而非常快，但在其他操作系统上就没有这些优势。在性能敏感的程序中，不要假设这些调用是廉价的。总的来说，应尽可能避免使用系统调用。

不要依赖 Linux® 特有的 socket 行为。特别是默认的 socket 缓冲区大小不同（应通过调用 [setsockopt(2)](https://man.freebsd.org/cgi/man.cgi?query=setsockopt&sektion=2&format=html) 设置 `SO_SNDBUF` 和 `SO_RCVBUF`），并且在 socket 缓冲区已满时，Linux® 的 [send(2)](https://man.freebsd.org/cgi/man.cgi?query=send&sektion=2&format=html) 会阻塞，而 FreeBSD 会返回失败并设置 errno 为 `ENOBUFS`。

如果确实需要依赖非标准行为，请将其正确封装为通用 API，并在配置阶段检测该行为，若不满足则中止。

请通过查阅 [man 页面](https://man.freebsd.org/cgi/man.cgi) 的 “STANDARDS” 部分来确认所使用的函数是否为 POSIX 接口。

不要假设 **/bin/sh** 是 bash。应确保传递给 [system(3)](https://man.freebsd.org/cgi/man.cgi?query=system&sektion=3&format=html) 的命令行可以在 POSIX 兼容的 shell 下正常执行。

常见 bash 主义列表可见于 [此处](https://wiki.ubuntu.com/DashAsBinSh)。

请检查头文件是否按照 POSIX 或 man 页面推荐的方式包含。例如，经常会忘记包含 **sys/types.h**，这在 Linux® 上问题不大，但在 FreeBSD 上就不行。

## 13.20. 其他注意事项

请务必反复检查 **pkg-descr** 和 **pkg-plist**。如果在审查 Port 时发现可以改进其措辞，请进行修正。

请务必注意任何法律问题！不要让我们非法分发软件！

你是否在维护 Ports 时遇到过具体的非 POSIX 行为或仅 Linux 依赖？

