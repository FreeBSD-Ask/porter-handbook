# 第17章 使用 USES 宏

## 17.1. USES 简介

USES 宏使得为 port 声明需求和设置变得容易。它们可以添加依赖关系，改变构建行为，为软件包添加元数据等等，所有这些都通过选择简单的预设值来实现。

本章的每个部分描述了 USES 的一个可能值，以及它可能的参数。参数会在值后面加上冒号 ( : )，多个参数之间用逗号分隔 ( , )。

示例 1. 使用多个值

```
USES=	bison perl
```

示例 2. 添加一个参数

```
USES=	tar:xz
```

示例 3. 添加多个参数

```
USES=	drupal:7,theme
```

示例 4. 混合所有内容

```
USES=	pgsql:9.3+ cpe python:2.7,build
```

## 17.2. `7z`

可能的参数: (无), p7zip , partial

使用 7z(1) 提取而非 bsdtar(1)，并设置 EXTRACT_SUFX=.7z 。使用 p7zip 选项会强制依赖于来自 archivers/p7zip 的 7z ，如果基本系统的工具无法提取文件。如果使用了 partial 选项， EXTRACT_SUFX 不会改变，这在主分发文件没有 .7z 扩展名时可以使用。

## 17.3. `ada`

可能的参数: (无), 6 , 12 , (run)

依赖于支持 Ada 的编译器，并相应地设置 CC 。默认使用 gcc6-aux 来自 ports。

## 17.4. `autoreconf`

可能的参数： （无）， build

运行 autoreconf 。它封装了 aclocal 、 autoconf 、 autoheader 、 automake 、 autopoint 和 libtoolize 命令。每个命令应用于 ${AUTORECONF_WRKSRC}/configure.ac 或其旧名称 ${AUTORECONF_WRKSRC}/configure.in。如果 configure.ac 定义了具有自己 configure.ac 的子目录，使用 AC_CONFIG_SUBDIRS ， autoreconf 将递归更新这些目录。 :build 参数仅在这些工具上添加构建时依赖关系，但不运行 autoreconf 。如果 WRKSRC 不包含 configure.ac 的路径，则 port 可以设置 AUTORECONF_WRKSRC 。

## 17.5. `blaslapack`

可能的参数：（无）, atlas , netlib （默认）, gotoblas , openblas

添加对 Blas / Lapack 库的依赖。

## 17.6. `bdb`

可能的参数：（无）, 48 , 5 （默认）, 6

添加对伯克利数据库库的依赖。默认为 databases/db5。当使用 :48 参数时也可以依赖 databases/db48，或者使用 :6 依赖 databases/db6。可以声明可接受的版本范围， :48+ 可以找到已安装的最高版本，并在未安装其他版本时回退到 4.8。 INVALID_BDB_VER 可用于指定与此 port 不兼容的版本。框架向 port 暴露以下变量：

BDB_LIB_NAME 伯克利数据库库的名称。例如，使用 databases/db5 时，它包含 db-5.3 。

BDB_LIB_CXX_NAME 伯克利 DBC++ 库的名称。例如，使用 databases/db5 时，它包含 db_cxx-5.3 。

BDB_INCLUDE_DIR Berkeley DB 包含目录的位置。例如，当使用 databases/db5 时，它将包含 ${LOCALBASE}/include/db5 。

BDB_LIB_DIR Berkeley DB 库目录的位置。例如，当使用 databases/db5 时，它包含 ${LOCALBASE}/lib 。

BDB_VER 检测到的 Berkeley DB 版本。例如，如果使用 USES=bdb:48+ 并安装了 Berkeley DB 5，它包含 5 。

|  | 数据库/db48 已经被弃用且不再支持。任何port都不应使用它。 |
| -- | ---------------------------------------------------------- |

## 17.7. `bison`

可能的参数：(无), build , run , both

默认使用 devel/bison。不带参数或使用 build 参数时，意味着 bison 是构建时依赖， run 意味着运行时依赖，而 both 则既是运行时依赖也是构建时依赖。

## 17.8. `budgie`

可能的参数：(无)

为 Budgie 桌面环境提供支持。 使用 USE_BUDGIE 来选择所需的组件。 有关更多信息，请参阅使用 Budgie。

## 17.9. `cabal`

|  | 不应为 Haskell 库创建Ports，请查看 Haskell 库以获取更多信息。 |
| -- | --------------------------------------------------------------- |

可能的参数：(无), hpack , nodefault

设置用于使用 Cabal 构建 Haskell 软件的默认值和目标。添加对 Haskell 编译器 port ( lang/ghc) 的构建依赖。如果已在 BUILD_DEPENDS 变量中列出了其他版本的 GHC（例如，lang/ghc810），则将使用它。如果给出 hpack 参数，则添加对 devel/hs-hpack 的构建依赖，并在配置步骤中调用 hpack 以生成 .cabal 文件。如果给出 nodefault 参数，该框架将不尝试从 Hackage 获取主要发布文件。如果存在 USE_GITHUB 或 USE_GITLAB ，则隐含添加此参数。

该框架提供以下变量：

CABAL_REVISION 存储在 Hackage 上的 Haskell 软件包可能有修订版本。将此旋钮设置为整数以拉取已修订的软件包描述。

USE_CABAL 如果软件使用 Haskell 依赖项，请在此变量中列出它们。每个项目应该在 Hackage 上并且列在 packagename-0.1.2 形式中。依赖项也可以有修订版本，修订版本在 _ 符号之后指定。支持依赖项列表的自动生成，请参阅使用 cabal 构建 Haskell 应用程序。

CABAL_FLAGS 传递到配置和构建阶段中 cabal-install 的标志列表。这些标志完全按原样传递。通常使用此变量来启用或禁用在.cabal 文件中声明的标志。传递 foo 以启用 foo 标志，并传递 -foo 以禁用它。

CABAL_EXECUTABLES 列出由 port 安装的可执行文件列表。默认值： ${PORTNAME} 。请查看要移植项目的 .cabal 文件，以获取此变量可能值的列表。每个值对应 .cabal 文件中的一个 executable 部分。此列表中的项目会自动添加到 pkg-plist 中。

SKIP_CABAL_PLIST 如果定义，不要将 ${CABAL_EXECUTABLES} 中的项目添加到 pkg-plist。

opt_USE_CABAL 根据 opt 选项向 ${USE_CABAL} 添加项目。

opt_CABAL_EXECUTABLES 根据 ${CABAL_EXECUTABLES} 选项将项目添加到 opt 中。

opt_CABAL_FLAGS 如果 opt 启用，则将值附加到 ${CABAL_FLAGS} 。否则，将 -value 附加以禁用标志。请注意，这种行为与普通的 CABAL_FLAGS 略有不同，因为它不接受以 - 开头的值。

CABAL_WRAPPER_SCRIPTS 一个包含 Haskell 程序的 ${CABAL_EXECUTABLES} 子集，这些程序将被包装成 shell 脚本，在运行程序之前设置 *_datadir 环境变量。这也导致实际的 Haskell 二进制文件安装在 libexec/cabal/ 目录下。这个选项是为了那些在 share/ 目录下安装数据文件的 Haskell 程序所需的。

FOO_DATADIR_VARS 列出额外的 Haskell 包，这些包的数据文件应可被名为 FOO 的可执行文件访问。该可执行文件应是 ${CABAL_WRAPPER_SCRIPTS} 的一部分。列在那里的 Haskell 包不应具有版本后缀。

CABAL_PROJECT 一些 Haskell 项目可能已经有一个 cabal.project 文件，该文件也是由ports框架生成的。如果是这种情况，请使用此变量指定如何处理原始 cabal.project 。将此变量设置为 remove 将导致删除原始文件。将此变量设置为 append 将：

1. 在 extract 阶段期间将原始文件移动到 cabal.project.${PORTNAME} 。
2. 将原始 cabal.project.${PORTNAME} 和生成的 cabal.project 在 patch 阶段之后合并到单个文件中。使用 append 可以在合并之前对原始文件进行修补。

## 17.10. `cargo`

可能的参数：（无）

使用 Cargo 进行配置、构建和测试。它可以用于 port 使用 Cargo 构建系统的 Rust 应用程序。有关更多信息，请参阅 使用 cargo 构建 Rust 应用程序。

## 17.11. `charsetfix`

可能的参数：（无）

阻止 port 安装 charset.alias。这必须仅由 converters/libiconv 安装。如果 charset.alias 未由 ${WRKSRC}/Makefile.in 安装， CHARSETFIX_MAKEFILEIN 可设置为相对于 WRKSRC 的路径。

## 17.12. `cmake`

可能的参数：（无）， insource ， noninja ， run ， testing

使用 CMake 配置port并生成一个构建系统。

默认情况下，执行无源代码构建，使源码 WRKSRC 中免受构建工件影响。使用 insource 参数，将执行源代码构建。这个参数应该是一种例外，仅在常规的无源代码构建无法工作时使用。

默认情况下，使用 Ninja (devel/ninja)进行构建。在某些情况下，这种方法不起作用。使用 noninja 参数，构建将使用常规 make 进行构建。该参数仅在 Ninja 构建不起作用时使用。

使用 run 参数时，除了构建依赖外，还注册了运行依赖。

使用 testing 参数时，将添加一个使用 CTest 的测试目标。在运行测试时，port 将被重新配置以进行测试和重新构建。

欲了解更多信息，请参阅使用 cmake 。

## 17.13. `compiler`

可能的参数: (无), env (默认, 隐式), C++17-lang , C++14-lang , C++11-lang , gcc-C++11-lib , C++11-lib , C++0x , c11 , nestedfct , features

根据任何给定的需求确定使用哪个编译器。如果 port 需要支持 C++17 的编译器，请使用 C++17-lang ；如果 port 需要支持 C++14 的编译器，请使用 C++14-lang ；如果 port 需要支持 C++11 的编译器，请使用 C++11-lang ；如果 port 需要带有 C++11 库的 g++ 编译器，请使用 gcc-C++11-lib ；如果 port 需要准备好支持 C++11 标准库的编译器，请使用 C++11-lib 。如果 port 需要理解 C++0X、C11 或嵌套函数的编译器，则应使用相应的参数。

使用 features 请求由默认编译器支持的功能列表。在包含 bsd.port.pre.mk 后，port 可以使用这些变量检查结果。

* COMPILER_TYPE ：系统上的默认编译器，可以是 gcc 或 clang
* ALT_COMPILER_TYPE ：系统上的备用编译器，可以是 gcc 或 clang。仅在基本系统中存在两个编译器时设置。
* COMPILER_VERSION ：默认编译器版本的前两位数字。
* ALT_COMPILER_VERSION : 替代编译器版本的前两个数字（如果存在的话）。
* CHOSEN_COMPILER_TYPE : 选择的编译器，可能是 gcc 或 clang
* COMPILER_FEATURES : 默认编译器支持的功能。目前列出了 C++ 库。

## 17.14. `cpe`

可能的参数: (无)

在软件包清单中包含公共平台枚举（CPE）信息，作为一个 CPE 2.3 格式的字符串。查看 CPE 规范获取详细信息。要向 port 添加 CPE 信息，请按照以下步骤进行:

1. 通过使用 NVD 的 CPE 搜索引擎或在官方 CPE 词典中搜索软件产品的官方 CPE 条目。永远不要虚构 CPE 数据。
2. 将 cpe 添加到 USES 并将 make -V CPE_STR 的结果与 CPE 字典条目进行比较。一步一步进行，直到 make -V CPE_STR 正确为止。
3. 如果产品名称（第二字段，默认为 PORTNAME ）不正确，请定义 CPE_PRODUCT 。
4. 如果供应商名称（第一个字段，默认为 CPE_PRODUCT ）不正确，请定义 CPE_VENDOR 。
5. 如果版本字段(第三字段，默认为 PORTVERSION )不正确，请定义 CPE_VERSION 。
6. 如果更新字段(第四字段，默认为空)不正确，请定义 CPE_UPDATE 。
7. 如果仍然不正确，请查看 Mk/Uses/cpe.mk 以获取更多详细信息，或联系Ports安全团队 <ports-secteam@FreeBSD.org>。
8. 尽量从现有变量（如 PORTNAME 和 PORTVERSION ）中汲取 CPE 名称的大部分内容。使用变量修饰符从这些变量中提取相关部分，而不是硬编码名称。
9. 在提交任何更改 PORTNAME 或 PORTVERSION 或用于派生 CPE_STR 的任何其他变量之前，始终运行 make -V CPE_STR 并检查输出。

## 17.15. `cran`

可能的参数：(无)， auto-plist ， compiles

使用综合 R 存档网络。 指定 auto-plist 以自动生成 pkg-plist。 如果 port 有需要编译的代码，则指定 compiles 。

## 17.16. `desktop-file-utils`

可能的参数:（无）

使用 devel/desktop-file-utils 中的 update-desktop-database。 将运行额外的安装后步骤，而不会干扰已在 port Makefile 中的任何安装后步骤。 将在 plist 中添加一个包含 @desktop-file-utils 的行。 只有在 port 提供包含 MimeType 条目的 .desktop 文件时才使用此宏。

## 17.17. `desthack`

可能的参数：（无）

更改 GNU 配置的行为，以正确支持 DESTDIR ，以防原始软件不支持。

## 17.18. `display`

可能的参数：（无），ARGS

设置虚拟显示环境。如果环境变量 DISPLAY 未设置，则将 Xvfb 添加为构建依赖项，并将 CONFIGURE_ENV 扩展到当前运行的 Xvfb 实例的 port 号。ARGS 参数默认为 install ，控制启动和停止虚拟显示的阶段。

## 17.19. `dos2unix`

可能的参数：（无）

port 有带 DOS 格式换行符的文件需要转换。可以设置多个变量来控制要转换的文件。默认是转换所有文件，包括二进制文件。请参见简单自动替换的示例。

* DOS2UNIX_REGEX ：根据正则表达式匹配文件名。
* DOS2UNIX_FILES ：匹配字面文件名。
* DOS2UNIX_GLOB ：根据通配符模式匹配文件名。
* DOS2UNIX_WRKSRC ：开始转换的目录。默认为 ${WRKSRC} 。

## 17.20. `drupal`

可能的参数： 7 ， module ， theme

自动安装适用于 Drupal 主题或模块的port。与port期望的 Drupal 版本一起使用。例如， USES=drupal:7,module 表示此port创建一个 Drupal 7 模块。Drupal 7 主题可以用 USES=drupal:7,theme 指定。

## 17.21. `ebur128`

可能的参数: (无), build , lib , run , test

添加对 audio/ebur128 的依赖。通过在 make.conf 中使用 DEFAULT_VERSIONS ，可以透明地依赖于 rust 或 legacy 变体。例如，要使用旧版，请使用 DEFAULT_VERSIONS+=ebur128=legacy

当不使用任何参数时，行为与提供 lib 参数相同。其余的参数提供相应的依赖类别。

## 17.22. `eigen`

可能的参数：2, 3, 构建（默认），运行

添加依赖于 math/eigen。

## 17.23. `elfctl`

可能的参数：（无）

通过设置 ELF_FEATURES 更改 ELF 二进制文件的特征控制注释。

 示例 5。使用 elfctl

```
USES=           elfctl
ELF_FEATURES=	featurelist:path/to/file1 \
		featurelist:path/to/file1 \
		featurelist:path/to/file2
```

featurelist 的格式在 elfctl(1) 中描述。文件路径相对于 ${BUILD_WRKSRC}。

## 17.24. `erlang`

可能的参数: (无), enc , rebar , rebar3

添加了对 lang/erlang 的构建和运行时依赖。根据参数不同，可能会增加额外的构建依赖。 enc 添加了对 devel/erlang-native-compiler 的依赖， rebar 添加了对 devel/rebar 的依赖， rebar3 添加了对 devel/rebar3 的依赖。

此外，以下变量对 port 可用：

* ERL_APP_NAME ：Erlang 应用程序名称，安装在 Erlang 的 lib 目录中（减去版本）
* ERL_APP_ROOT ：此 Erlang 应用程序的根目录
* REBAR_CMD ："rebar"命令的路径
* REBAR3_CMD ：到 "rebar3" 命令的路径
* REBAR_PROFILE ：Rebar 配置文件
* REBAR_TARGETS ：Rebar 目标列表（通常是编译，也可能是生成可执行文件）
* ERL_BUILD_NAME ：rebar3 构建名称
* ERL_BUILD_DEPS ：按类别/ Port 名称格式列出的 BUILD_DEPENDS 列表
* ERL_RUN_DEPS ：按类别/ Port 名称格式列出的 RUN_DEPENDS 列表
* ERL_DOCS ：文档文件和目录列表

## 17.25. `fakeroot`

可能的参数：（无）

更改一些构建系统的默认行为，以允许以用户身份安装。有关 fakeroot 的更多信息，请参见 https://wiki.debian.org/FakeRoot。

## 17.26. `fam`

可能的参数：（无）， fam ， gamin

使用文件更改监视器作为库依赖项，可以是 devel/fam 或 devel/gamin。最终用户可以设置 WITH_FAM_SYSTEM 以指定他们的首选项。

## 17.27. `firebird`

可能的参数：（无）， 25

添加到 Firebird 数据库客户端库的依赖项。

## 17.28. `fonts`

可能的参数：(无)， fc ， fontsdir （默认）， none

添加到需要注册字体的工具的运行时依赖项。根据参数添加 <a href="https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-keywords-fc">@fc</a><span> </span>${FONTSDIR} 行、 <a href="https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-keywords-fontsdir">@fontsdir</a><span> </span>${FONTSDIR} 行，如果参数为 none 则不添加行到 plist。 FONTSDIR 默认为 ${PREFIX}/share/fonts/${FONTNAME}， FONTNAME 为 ${PORTNAME} 。添加 FONTSDIR 到 PLIST_SUB 和 SUB_LIST 。

## 17.29. `fortran`

可能的参数： gcc （默认）

使用 GNU Fortran 编译器。

## 17.30. `fuse`

可能的参数： 2 （默认）， 3

根据 FreeBSD 的版本，port 将取决于 FUSE 库并处理内核模块的依赖项。

## 17.31. `gem`

可能的参数：（无）， noautoplist

处理使用 RubyGems 构建。如果使用 noautoplist ，则不会自动生成包装清单。

 这意味着 USES=ruby 。

## 17.32. `gettext`

可能的参数: (无)

已弃用。将包括 gettext-runtime 和 gettext-tools 。

## 17.33. `gettext-runtime`

可能的参数: (无), lib (默认), build , run

使用 devel/gettext-runtime。默认情况下，没有参数或使用 lib 参数，意味着对 libintl.so 的库依赖。 build 和 run 分别意味着构建时和运行时对 gettext 的依赖。

## 17.34. `gettext-tools`

可能的参数: (无), build (默认), run

使用 devel/gettext-tools。默认情况下，没有参数，或者使用 build 参数时，将注册 msgfmt 的构建时间依赖项。使用 run 参数时，将注册运行时依赖项。

## 17.35. `ghostscript`

可能的参数: X, build , run , nox11

可以使用特定版本 X。可用版本为 7 , 8 , 9 , 和 agpl （默认）。 nox11 表示需要port的 -nox11 版本。 build 和 run 增加对 Ghostscript 的构建和运行时依赖项。默认情况下是构建和运行时依赖项。

## 17.36. `gl`

可能的参数：(无)

提供了一种依赖于 GL 组件的简便方法。这些组件应该列在 USE_GL 中。可用的组件有：

egl 在 graphics/libglvnd 中添加对 libEGL.so 的库依赖

gbm 从 graphics/mesa-libs 添加对 libgbm.so 的库依赖

gl 从 graphics/libglvnd 添加对 libGL.so 的库依赖

glesv2 从 graphics/libglvnd 添加对 libGLESv2.so 的库依赖

glew 在 graphics/glew 中添加对 libGLEW.so 的库依赖

glu 在 graphics/libGLU 中添加对 libGLU.so 的库依赖

glut 在 graphics/freeglut 中添加对 libglut.so 的库依赖

opengl 添加一个库依赖于 graphics/libglvnd 上的 libOpenGL.so

## 17.37. `gmake`

可能的参数: (无)

使用 devel/gmake 作为构建时依赖，并设置环境以使用 gmake 作为构建的默认 make 。

## 17.38. `gnome`

可能的参数：（无）

提供一种依赖于 GNOME 组件的简单方式。组件应列在 USE_GNOME 中。可用的组件包括：

* `atk`
* `atkmm`
* `cairo`
* `cairomm`
* `dconf`
* `esound`
* `evolutiondataserver3`
* `gconf2`
* `gconfmm26`
* `gdkpixbuf`
* `gdkpixbuf2`
* `glib12`
* `glib20`
* `glibmm`
* `gnomecontrolcenter3`
* `gnomedesktop3`
* `gnomedocutils`
* `gnomemenus3`
* `gnomemimedata`
* `gnomeprefix`
* `gnomesharp20`
* `gnomevfs2`
* `gsound`
* `gtk-update-icon-cache`
* `gtk12`
* `gtk20`
* `gtk30`
* `gtkhtml3`
* `gtkhtml4`
* `gtkmm20`
* `gtkmm24`
* `gtkmm30`
* `gtksharp20`
* `gtksourceview`
* `gtksourceview2`
* `gtksourceview3`
* `gtksourceviewmm3`
* `gvfs`
* `intlhack`
* `intltool`
* `introspection`
* `libartlgpl2`
* `libbonobo`
* `libbonoboui`
* `libgda5`
* `libgda5-ui`
* `libgdamm5`
* `libglade2`
* `libgnome`
* `libgnomecanvas`
* `libgnomekbd`
* `libgnomeprint`
* `libgnomeprintui`
* `libgnomeui`
* `libgsf`
* `libgtkhtml`
* `libgtksourceviewmm`
* `libidl`
* `librsvg2`
* `libsigc++12`
* `libsigc++20`
* `libwnck`
* `libwnck3`
* `libxml++26`
* `libxml2`
* `libxslt`
* `metacity`
* `nautilus3`
* `orbit2`
* `pango`
* `pangomm`
* `pangox-compat`
* `py3gobject3`
* `pygnome2`
* `pygobject`
* `pygobject3`
* `pygtk2`
* `pygtksourceview`
* `referencehack`
* `vte`
* `vte3`

默认的依赖性是构建和运行时的，可以使用 :build 或 :run 进行更改。例如：

```
USES=		gnome
USE_GNOME=	gnomemenus3:build intlhack
```

查看有关使用 GNOME 的更多信息。

## 17.39. `go`

|  | 不应为 Go 库创建 Ports，请参阅关于 Go 库的更多信息。 |
| -- | ------------------------------------------------------ |

可能的参数: (none), N.NN , N.NN-devel , modules , no_targets , run

设置默认值和用于构建 Go 软件的目标。添加对 Go 编译器port的构建依赖，port维护者可以设置所需的版本。默认情况下，使用 GOPATH 模式进行构建。如果 Go 软件使用模块，可以使用 modules 参数切换到支持模块的模式。 no_targets 将设置构建环境，类似于 GO_ENV ， GO_BUILDFLAGS ，但跳过创建提取和构建目标。 run 还会添加对 Go 编译器port的运行依赖。

构建过程由多个变量控制：

GO_MODULE 应用程序模块的名称，由 module 指令在 go.mod 中指定。在大多数情况下，这是ports使用 Go 模块的唯一必需变量。

GO_PKGNAME 构建时在 GOPATH 模式下的 Go 包名称。这是在 ${GOPATH}/src 中创建的目录。如果未显式设置且存在 GH_SUBDIR 或 GL_SUBDIR ，将从中推断出 GO_PKGNAME 。在模块感知模式下构建时不需要它。

GO_TARGET 要构建的软件包。默认值为 ${GO_PKGNAME} 。 GO_TARGET 也可以是形如 package:path 的元组，其中路径可以是简单的文件名，也可以是以 ${PREFIX} 开头的完整路径。

GO_TESTTARGET 要测试的软件包。默认值为 ./… （当前包及所有子包）。

CGO_CFLAGS 附加 CFLAGS 值，通过 go 传递给 C 编译器。

CGO_LDFLAGS 附加 LDFLAGS 值，通过 go 传递给 C 编译器。

GO_BUILDFLAGS 附加构建参数，传递给 go build 。

GO_TESTFLAGS 要传递给 go test 的附加构建参数。

查看构建 Go 应用程序以获取使用示例。

## 17.40. `gperf`

可能的参数：（无）

如果基础系统中不存在 gperf ，则在构建时增加对 devel/gperf 的依赖。

## 17.41. `grantlee`

可能的参数: 5 , selfbuild

处理对 Grantlee 的依赖。指定 5 来依赖基于 Qt5 的版本 devel/grantlee5。 selfbuild 在 devel/grantlee5 内部用于获取它们的版本号。

## 17.42. `groff`

可能的参数： build ， run ， both

如果基础系统中不存在，注册对 textproc/groff 的依赖。

## 17.43. `gssapi`

可能的参数：（无）， base （默认）， heimdal ， mit ， flags ， bootstrap

为 GSS-API 的使用者处理所需的依赖关系。仅提供提供 Kerberos 机制的库。默认情况下，或设置为 base ，使用基本系统中的 GSS-API 库。也可以设置为 heimdal 使用 security/heimdal，或 mit 使用 security/krb5。

当本地 Kerberos 安装不在 LOCALBASE 时，请设置 HEIMDAL_HOME （对于 heimdal ）或 KRB5_HOME （对于 krb5 ）为 Kerberos 安装的位置。

这些变量被导出供 ports 使用：

* `GSSAPIBASEDIR`
* `GSSAPICPPFLAGS`
* `GSSAPIINCDIR`
* `GSSAPILDFLAGS`
* `GSSAPILIBDIR`
* `GSSAPILIBS`
* `GSSAPI_CONFIGURE_ARGS`

可以将 flags 选项与 base 、 heimdal 或 mit 一起使用，以自动将 GSSAPICPPFLAGS 、 GSSAPILDFLAGS 和 GSSAPILIBS 分别添加到 CFLAGS 、 LDFLAGS 和 LDADD 。例如，使用 base,flags 。

bootstrap 选项是一个特殊前缀，仅供 security/krb5 和 security/heimdal 使用。例如，使用 bootstrap,mit 。

示例 6. 典型用法

```
OPTIONS_SINGLE=	GSSAPI
OPTIONS_SINGLE_GSSAPI=	GSSAPI_BASE GSSAPI_HEIMDAL GSSAPI_MIT GSSAPI_NONE

GSSAPI_BASE_USES=	gssapi
GSSAPI_BASE_CONFIGURE_ON=	--with-gssapi=${GSSAPIBASEDIR} ${GSSAPI_CONFIGURE_ARGS}
GSSAPI_HEIMDAL_USES=	gssapi:heimdal
GSSAPI_HEIMDAL_CONFIGURE_ON=	--with-gssapi=${GSSAPIBASEDIR} ${GSSAPI_CONFIGURE_ARGS}
GSSAPI_MIT_USES=	gssapi:mit
GSSAPI_MIT_CONFIGURE_ON=	--with-gssapi=${GSSAPIBASEDIR} ${GSSAPI_CONFIGURE_ARGS}
GSSAPI_NONE_CONFIGURE_ON=	--without-gssapi
```

## 17.44. `gstreamer`

可能的参数：（无）

提供一种依赖于 GStreamer 组件的简单方式。组件应列在 USE_GSTREAMER 中。可用的组件包括：

* `a52dec`
* `aalib`
* `amrnb`
* `amrwbdec`
* `aom`
* `assrender`
* `bad`
* `bs2b`
* `cairo`
* `cdio`
* `cdparanoia`
* `chromaprint`
* `curl`
* `dash`
* `dtls`
* `dts`
* `dv`
* `dvd`
* `dvdread`
* `editing-services`
* `faac`
* `faad`
* `flac`
* `flite`
* `gdkpixbuf`
* `gl`
* `gme`
* `gnonlin`
* `good`
* `gsm`
* `gtk4`
* `gtk`
* `hal`
* `hls`
* `jack`
* `jpeg`
* `kate`
* `kms`
* `ladspa`
* `lame`
* `libav`
* `libcaca`
* `libde265`
* `libmms`
* `libvisual`
* `lv2`
* `mm`
* `modplug`
* `mpeg2dec`
* `mpeg2enc`
* `mpg123`
* `mplex`
* `musepack`
* `neon`
* `ogg`
* `opencv`
* `openexr`
* `openh264`
* `openjpeg`
* `openmpt`
* `opus`
* `pango`
* `png`
* `pulse`
* `qt`
* `resindvd`
* `rsvg`
* `rtmp`
* `shout2`
* `sidplay`
* `smoothstreaming`
* `sndfile`
* `sndio`
* `soundtouch`
* `soup`
* `spandsp`
* `speex`
* `srtp`
* `taglib`
* `theora`
* `ttml`
* `twolame`
* `ugly`
* `v4l2`
* `vorbis`
* `vpx`
* `vulkan`
* `wavpack`
* `webp`
* `webrtcdsp`
* `x264`
* `x265`
* `x`
* `ximagesrc`
* `zbar`

## 17.45. `guile`

可能的参数：（无）， X.Y ， flavors ， build ， run ， alias ， conflicts

增加对 Guile 的依赖。默认情况下，这是对适当 libguile*.so 的库依赖，除非被 build 和/或 run 选项覆盖。 alias 选项适当地配置 BINARY_ALIAS （请参阅使用  BINARY_ALIAS ）。

默认版本由通常 DEFAULT_VERSIONS 机制设置；如果默认版本不是所列版本之一，则使用最新可用的所列版本。

使用 Guile 的应用程序通常仅构建为单个 Guile 版本。但是，扩展或库模块应使用 flavors 选项以与多个 flavors 构建。

获取更多信息，请参阅使用 Guile。

## 17.46. `horde`

可能的参数：（无）

添加构建时间和运行时依赖项至 devel/pear-channel-horde。其他 Horde 依赖项可以通过 USE_HORDE_BUILD 和 USE_HORDE_RUN 添加。有关更多信息，请参阅 Horde 模块。

## 17.47. `iconv`

可能的参数：(无)， lib ， build ， patch ， translit ， wchar_t

使用 iconv 函数，要么来自port转换器/libiconv 作为构建时和运行时依赖，要么来自基本系统。默认情况下，没有参数或使用 lib 参数，意味着使用构建时和运行时依赖的 iconv 。 build 意味着构建时依赖， patch 意味着补丁时间依赖。如果port 使用 WCHAR_T 或 //TRANSLIT 的 iconv 扩展，添加相关参数以使用正确的 iconv。有关更多信息，请参阅 使用 iconv 。

## 17.48. `imake`

可能的参数：(无)， env ， notall ， noman

将 devel/imake 添加为构建时依赖项，并在 configure 阶段运行 xmkmf -a 。如果给定了 env 参数，则不设置 configure 目标。如果 -a 标志对port造成问题，请添加 notall 参数。如果 xmkmf 未生成 install.man 目标，请添加 noman 参数。

## 17.49. `kde`

可能的参数： 5

添加对 KDE 组件的依赖。有关更多信息，请参阅使用 KDE。

## 17.50. `kmod`

可能的参数：（无）， debug

填充内核模块ports的样板，目前：

* 将 kld 添加到 CATEGORIES 。
* 設置 SSP_UNSAFE 。
* 如果在 SRC_BASE 中找不到內核源碼，則設置 IGNORE 。
* 默認將 KMODDIR 定義為 /boot/modules，將其添加到 PLIST_SUB 和 MAKE_ENV 中，並在安裝時創建它。 如果 KMODDIR 設置為 /boot/kernel，則將其重寫為 /boot/modules。 這可以防止在升級內核過程中由於 /boot/kernel 被重命名為 /boot/kernel.old 而破壞包。
* 使用 @kld 处理内核模块在安装和卸载时的交叉引用。
* 如果提供了 debug 参数，port 可以将模块的调试版本安装到 KERN_DEBUGDIR/KMODDIR 中。默认情况下， KERN_DEBUGDIR 从 DEBUGDIR 复制并设置为 /usr/lib/debug。框架将负责创建和删除任何所需的目录。

## 17.51. `ldap`

可能的参数：（无），，客户端，服务器

注册对 net/openldap 的依赖。如果设置了特定的 <version> （不使用点符号），则使用该版本。否则，尝试查找当前安装的版本。如有必要，将回退到 bsd.default-versions.mk 中找到的默认版本。 client 指定对客户端库的运行时依赖。这也是默认设置。 server 指定对服务器的运行时依赖。

可以由 port 访问以下变量：

`IGNORE_WITH_OPENLDAP`

如果 ports 不支持一个或多个版本的 OpenLDAP，则可以定义此变量。

`WITH_OPENLDAP_VER`

用户定义的变量以设置 OpenLDAP 版本。

`OPENLDAP_VER`

检测到的 OpenLDAP 版本。

## 17.52. `lha`

可能的参数：（无）

 将 EXTRACT_SUFX 设置为 .lzh

## 17.53. `libarchive`

可能的参数：（无）

注册对 archivers/libarchive 的依赖。任何依赖于 libarchive 的 ports 都必须包含 USES=libarchive 。

## 17.54. `libedit`

可能的参数：（无）

注册对 devel/libedit 的依赖。任何依赖于 libedit 的 ports 必须包含 USES=libedit 。

## 17.55. `libtool`

可能的参数：（无）， keepla ， build

补丁 libtool 脚本。这必须添加到所有使用 libtool 的 ports 中。可以使用 keepla 参数保留 .la 文件。一些 ports 不包含自己的 libtool 副本，需要在构建时依赖 devel/libtool，请使用 :build 参数添加此类依赖关系。

## 17.56. `linux`

可能的参数： c6 ， c7

Ports Linux 兼容框架。指定 c6 以依赖于 CentOS 6 包。指定 c7 以依赖于 CentOS 7 包。可用的软件包有：

* `allegro`
* `alsa-plugins-oss`
* `alsa-plugins-pulseaudio`
* `alsalib`
* `atk`
* `avahi-libs`
* `base`
* `cairo`
* `cups-libs`
* `curl`
* `cyrus-sasl2`
* `dbusglib`
* `dbuslibs`
* `devtools`
* `dri`
* `expat`
* `flac`
* `fontconfig`
* `gdkpixbuf2`
* `gnutls`
* `graphite2`
* `gtk2`
* `harfbuzz`
* `jasper`
* `jbigkit`
* `jpeg`
* `libasyncns`
* `libaudiofile`
* `libelf`
* `libgcrypt`
* `libgfortran`
* `libgpg-error`
* `libmng`
* `libogg`
* `libpciaccess`
* `libsndfile`
* `libsoup`
* `libssh2`
* `libtasn1`
* `libthai`
* `libtheora`
* `libv4l`
* `libvorbis`
* `libxml2`
* `mikmod`
* `naslibs`
* `ncurses-base`
* `nspr`
* `nss`
* `openal`
* `openal-soft`
* `openldap`
* `openmotif`
* `openssl`
* `pango`
* `pixman`
* `png`
* `pulseaudio-libs`
* `qt`
* `qt-x11`
* `qtwebkit`
* `scimlibs`
* `sdl12`
* `sdlimage`
* `sdlmixer`
* `sqlite3`
* `tcl85`
* `tcp_wrappers-libs`
* `tiff`
* `tk85`
* `ucl`
* `xorglibs`

## 17.57. `llvm`

可能的参数：(无)， XY ，最小值= XY ，最大值= XY ，构建，运行，库

添加对 LLVM 的依赖。默认情况下，这是构建依赖，除非被 run 或 lib 选项覆盖。默认版本是设置在 LLVM_DEFAULT 中的版本。也可以指定特定版本。最小和最大版本可以分别用 min 和 max 参数指定。ports框架将以下变量导出到port：

`LLVM_VERSION`

从 llvm.mk 的参数中选择的版本

`LLVM_PORT`

 已选 llvm port

`LLVM_CONFIG`

已选 port 的 llvm-config

`LLVM_LIBLLVM`

已选 port 的 libLLVM.so

`LLVM_PREFIX`

选定port的安装前缀

## 17.58. `localbase`

可能的参数：(无), ldflags

确保在 LOCALBASE 中使用依赖项中的库，而不是基本系统中的库。指定 ldflags 将 -L${LOCALBASE}/lib 添加到 LDFLAGS 而不是 LIBS 。应使用在基本系统中也存在库的Ports。一些其他 USES 也在内部使用它。

## 17.59. `lua`

可能的参数有：(无)， XY ， XY+ ， -XY ， XY-ZA ， module ， flavors ， build ， run ， env

添加对 Lua 的依赖。默认情况下这是一个库依赖，除非通过 build 和/或 run 选项覆盖。 env 选项阻止添加任何依赖，同时仍然定义所有通常的变量。

默认版本由通常的 DEFAULT_VERSIONS 机制设置，除非指定了版本或版本范围作为参数，例如， 51 或 51-54 。

使用 Lua 的应用通常仅构建为一个 Lua 版本。然而，预期由 Lua 代码加载的库模块应使用 module 选项构建多个 flavors。

更多信息请参阅 Lua 使用。

## 17.60. `luajit`

可能的参数：(无), X

添加对 luajit 运行时的依赖。可以使用特定版本 X。可能的版本是 luajit ， luajit-devel ， luajit-openresty

在包含 bsd.port.options.mk 或 bsd.port.pre.mk 之后，port可以检查这些变量：

LUAJIT_VER 所选的 luajit 版本

LUAJIT_INCDIR 查找 luajit 的头文件路径

LUAJIT_LUAVER 选择的 luajit 规范版本（luajit 为 2.0，否则为 2.1）

查看更多信息，请参阅 Lua 使用。

## 17.61. `lxqt`

可能的参数：（无）

处理 LXQt 桌面环境的依赖关系。使用 USE_LXQT 选择所需的组件port。更多信息，请参阅使用 LXQt。

## 17.62. `magick`

可能的参数：（无）， X ， build ， nox11 ， run ， test

添加对 ImageMagick 的库依赖。可以使用特定版本 X。可能的版本为 6 和 7 （默认）。 nox11 表示需要 port 的 -nox11 版本。 build 、 run 和 test 添加到 ImageMagick 的构建、运行时和测试依赖。

## 17.63. `makeinfo`

可能的参数：（无）

如果基础系统中不存在 makeinfo ，则添加到构建时依赖。

## 17.64. `makeself`

可能的参数：（无）

表示分发文件为 makeself 存档，并设置适当的依赖关系。

## 17.65. `mate`

可能的参数：（无）

提供一种依赖于 MATE 组件的简单方法。组件应列在 USE_MATE 中。可用的组件是：

* `autogen`
* `caja`
* `common`
* `controlcenter`
* `desktop`
* `dialogs`
* `docutils`
* `icontheme`
* `intlhack`
* `intltool`
* `libmatekbd`
* `libmateweather`
* `marco`
* `menus`
* `notificationdaemon`
* `panel`
* `pluma`
* `polkit`
* `session`
* `settingsdaemon`

默认依赖是构建和运行时的，可以通过 :build 或 :run 更改。例如：

```
USES=		mate
USE_MATE=	menus:build intlhack
```

## 17.66. `meson`

可能的参数：（无）

支持基于 Meson 的项目。有关更多信息，请参见使用 meson 。

## 17.67. `metaport`

可能的参数：(无)

设置以下变量，以便更轻松地创建元 Port ： MASTER_SITES ， DISTFILES ， EXTRACT_ONLY ， NO_BUILD ， NO_INSTALL ， NO_MTREE ， NO_ARCH 。

## 17.68. `minizip`

可能的参数：（无）， ng

添加对 archivers/minizip 或 archivers/minizip-ng 的库依赖。

## 17.69. `mysql`

可能的参数：（无）， version ， client （默认）， server ， embedded

如果没有给出版本号，则尝试查找当前安装的版本。回退到默认版本，MySQL-5.6。可能的版本是 55 ， 55m ， 55p ， 56 ， 56p ， 56w ， 57 ， 57p ， 80 ， 100m ， 101m 和 102m 。 m 和 p 后缀用于 MySQL 的 MariaDB 和 Percona 变体。 server 和 embedded 添加了对 MySQL 服务器的构建和运行时依赖。使用 server 或 embedded 时，还添加 client 以添加对 libmysqlclient.so 的依赖。port可以设置 IGNORE_WITH_MYSQL ，如果某些版本不受支持。

框架将 MYSQL_VER 设置为检测到的 MySQL 版本。

## 17.70. `mono`

可能的参数：（无）， nuget

通过设置适当的依赖关系，向 Mono（目前仅支持 C#）框架添加依赖。

当port使用 nuget 软件包时，请指定 nuget 。 NUGET_DEPENDS 需要以 name=version 格式设置 nuget 软件包的名称和版本。可以使用 name=version:_origin_ 添加可选的软件包来源。

助手目标 buildnuget 将根据提供的 packages.config 输出 NUGET_DEPENDS 的内容。

## 17.71. `motif`

可能的参数: (无)

使用 x11-toolkits/open-motif 作为库依赖。最终用户可以在 make.conf 中设置 WANT_LESSTIF 使用 x11-toolkits/lesstif 作为依赖，而不是 x11-toolkits/open-motif。类似地，设置 WANT_OPEN_MOTIF_DEVEL 在 make.conf 中将添加依赖 x11-toolkits/open-motif-devel。

## 17.72. `ncurses`

可能的参数: (无), base , port

使用 ncurses，并设置一些有用的变量。

## 17.73. `nextcloud`

可能的参数：(无)

通过在运行时依赖于 www/nextcloud 来增加对 Nextcloud 应用程序的支持。

## 17.74. `ninja`

可能的参数：(无)， build ， make （默认）， run

如果指定了 build 或 run 参数，则分别添加构建或运行时依赖于 devel/ninja。如果提供了 make 或没有参数，则使用 ninja 来构建port而不是 make。 make 意味着 build 。如果变量 NINJA_DEFAULT 设置为 samurai ，则依赖项设置为 devel/samurai。

## 17.75. `nodejs`

可能的参数：(无)， build ， run ， current ， lts ， 10 ， 14 ， 16 ， 17 。

使用 nodejs。 在 www/node* 上添加依赖项。如果指定了受支持的版本，则还必须指定 run 和/或 build 。

## 17.76. `objc`

可能的参数：（无）

如果基础系统不支持，添加 Objective C 依赖项（编译器，运行时库）。

## 17.77. `octave`

可能的参数：（无）, env

使用 math/octave。 env 仅加载一个 OCTAVE_VERSION 环境变量。

## 17.78. `openal`

可能的参数： al , soft （默认）, si , alut

使用 OpenAL 。 可以指定后端，软件实现为默认设置。 用户可以使用 WANT_OPENAL 指定首选后端。 此旋钮的有效值为 soft （默认）和 si 。

## 17.79. `pathfix`

可能的参数：（无）

查找 Makefile.in 和 configure 在 PATHFIX_WRKSRC （默认为 WRKSRC ） 中，并修复常见路径，以确保它们符合 FreeBSD 层次结构。 例如，它自动修复 pkgconfig’s .pc files to ${PREFIX}/libdata/pkgconfig. If the port uses  USES=autoreconf , Makefile.am will be added to  PATHFIX_MAKEFILEIN`的安装目录。

如果port USES=cmake ，它会查找 PATHFIX_WRKSRC 中的 CMakeLists.txt。如有需要，可以使用 PATHFIX_CMAKELISTSTXT 来更改默认文件名。

## 17.80. `pear`

可能的参数: env

添加对 devel/pear 的依赖。它将为使用 PHP 扩展和应用程序存储库的软件设置默认行为。仅仅使用 env 参数设置 PEAR 环境变量。有关更多信息，请参见 PEAR 模块。

## 17.81. `perl5`

可能的参数：（无）

依赖于 Perl。配置使用 USE_PERL5 完成。

USE_PERL5 可以包含使用 Perl 的阶段，可以是 extract ， patch ， build ， run 或 test 。

USE_PERL5 在需要 Makefile.PL、Build.PL 或 Module::Build::Tiny 的 Build.PL 的时候，也可以包含 configure 、 modbuild 或 modbuildtiny 。

USE_PERL5 默认为 build run 。在使用 configure 、 modbuild 或 modbuildtiny 时，意味着 build 和 run 。

查看有关更多信息，请参阅《使用 Perl》。

## 17.82. `pgsql`

可能的参数：(无)， X.Y ， X.Y+ ， X.Y- ， X.Y-Z.A

提供对 PostgreSQL 的支持。Port 维护者可以设置所需的版本。可以指定最小和最大版本或范围；例如， 9.0- ， 8.4+ ， 8.4-9.2.

默认情况下，添加的依赖将是客户端，但如果 port 需要额外的组件，则可以使用 WANT_PGSQL=component[:target] ；例如， WANT_PGSQL=server:configure pltcl plperl 。可用的组件包括：

* `client`
* `contrib`
* `docs`
* `pgtcl`
* `plperl`
* `plpython`
* `pltcl`
* `server`

## 17.83. `php`

可能的参数：(无)， phpize ， ext ， zend ， build ， cli ， cgi ， mod ， web ， embed ， pecl ， flavors ， noflavors

支持 PHP。添加对默认 PHP 版本 lang/php81 的运行时依赖。

`phpize`

用于构建 PHP 扩展。启用 flavors。

`ext`

用于构建、安装和注册 PHP 扩展。启用 flavors。

`zend`

用于构建、安装和注册 Zend 扩展。启用 flavors。

`build`

将 PHP 也设置为构建时依赖项。

`cli`

需要 PHP 的 CLI 版本。

`cgi`

需要 PHP 的 CGI 版本。

`mod`

需要 PHP 的 Apache 模块。

`web`

需要 Apache 模块或 PHP 的 CGI 版本。

`embed`

需要 PHP 的嵌入式库版本。

`pecl`

为从 PECL 存储库获取 PHP 扩展提供默认值。启用flavors。

`flavors`

启用自动生成 PHP flavors。将为除了 IGNORE_WITH_PHP 中已有的所有 PHP 版本生成 Flavors 。

`noflavors`

禁用自动生成 PHP flavors。仅适用于 PHP 自身提供的扩展。

变量用于指定所需的 PHP 模块以及支持的 PHP 版本。

`USE_PHP`

在运行时所需的 PHP 扩展列表。将 :build 添加到扩展名中以添加构建时依赖项。例如： pcre xml:build gettext

`IGNORE_WITH_PHP`

port与给定版本的 PHP 不兼容。有关可能的值，请查看 Mk/Uses/php.mk 中 _ALL_PHP_VERSIONS 的内容。

使用 :ext 或 :zend 构建 PHP 或 Zend 扩展时，可以设置这些变量：

`PHP_MODNAME`

PHP 或 Zend 扩展的名称。默认值是 ${PORTNAME} 。

`PHP_HEADER_DIRS`

从中安装头文件的子目录列表。框架将始终安装与扩展位于同一目录中的头文件。

`PHP_MOD_PRIO`

加载扩展的优先级。它是 00 和 99 之间的一个数字。

对于不依赖任何扩展的扩展，优先级会自动设置为 20 ，对于依赖另一个扩展的扩展，优先级会自动设置为 30 。有些扩展可能需要在所有其他扩展之前加载，例如 www/php56-opcache。有些可能需要在具有 30 优先级的扩展之后加载。在这种情况下，在port的 Makefile 中添加 PHP_MOD_PRIO=XX 。例如：

```
USES=		php:ext
USE_PHP=	wddx
PHP_MOD_PRIO=	40
```

这些变量可用于 PKGNAMEPREFIX 或 PKGNAMESUFFIX ：

`PHP_PKGNAMEPREFIX`

包含 php_XY_- ，其中 XY 是当前flavor的 PHP 版本。与 PHP 扩展和模块一起使用。

`PHP_PKGNAMESUFFIX`

包含 -php_XY_ ，其中 XY 是当前 flavor 的 PHP 版本。与 PHP 应用程序一起使用。

`PECL_PKGNAMEPREFIX`

包含 php_XY_-pecl- ，其中 XY 是当前 flavor 的 PHP 版本。与 PECL 模块一起使用。

|  | 使用 flavors，所有 PHP 扩展、PECL 扩展、PEAR 模块必须有不同的包名称，因此它们必须在其 PKGNAMEPREFIX 或 PKGNAMESUFFIX 中使用这三个变量之一。 |
| -- | --------------------------------------------------------------------------------------------------------------------------------------------- |

## 17.84. `pkgconfig`

可能的参数: (无), build (默认), run , both

使用 devel/pkgconf。没有参数或使用 build 参数，意味着 pkg-config 作为构建时依赖。 run 意味着运行时依赖， both 意味着同时是运行时和构建时依赖。

## 17.85. `pure`

可能的参数: (无), ffi

使用 lang/pure。主要用于构建相关的纯 ports。使用 ffi 参数时，意味着 devel/pure-ffi 作为运行时依赖。

## 17.86. `pyqt`

可能的参数: (无), 4 , 5

使用 PyQt。如果 port 是 PyQt 本身的一部分，请设置 PYQT_DIST 。使用 USE_PYQT 选择 port 需要的组件。可用的组件有:

* `core`
* `dbus`
* `dbussupport`
* `demo`
* `designer`
* `designerplugin`
* `doc`
* `gui`
* `multimedia`
* `network`
* `opengl`
* `qscintilla2`
* `sip`
* `sql`
* `svg`
* `test`
* `webkit`
* `xml`
* `xmlpatterns`

这些组件只能在 PyQT4 中使用：

* `assistant`
* `declarative`
* `help`
* `phonon`
* `script`
* `scripttools`

这些组件只能在 PyQT5 中使用：

* `multimediawidgets`
* `printsupport`
* `qml`
* `serialport`
* `webkitwidgets`
* `widgets`

每个组件的默认依赖是构建时和运行时的，要只选择构建或运行时，请在组件名称后添加 _build 或 _run 。例如：

```
USES=		pyqt
USE_PYQT=	core doc_build designer_run
```

## 17.87. `pytest`

可能的参数：（无），4

引入了新的依赖关系 devel/pytest。它定义了一个 do-test 目标，将正确运行测试。使用参数依赖于特定的 devel/pytest 版本。对于ports使用 devel/pytest 的人考虑使用这个而不是特定的 do-test 目标。该框架向port公开以下变量：

PYTEST_ARGS pytest 的额外参数（默认为空）。

PYTEST_IGNORED_TESTS 个测试忽略模式列表 pytest -k （默认为空）。适用于不会通过的测试，比如需要数据库访问的测试。

PYTEST_BROKEN_TESTS 个测试忽略模式列表 pytest -k （默认为空）。适用于需要修复的错误测试。

另外，用户可以设置以下变量：

PYTEST_ENABLE_IGNORED_TESTS 启用通常被 PYTEST_IGNORED_TESTS 忽略的测试。

PYTEST_ENABLE_BROKEN_TESTS 启用通常被 PYTEST_BROKEN_TESTS 忽略的测试。

PYTEST_ENABLE_ALL_TESTS 启用通常被 PYTEST_IGNORED_TESTS 和 PYTEST_BROKEN_TESTS 忽略的测试。

## 17.88. `python`

可能的参数: (无), X.Y , X.Y+ , -X.Y , X.Y-Z.A , patch , build , run , test

使用 Python。可以指定支持的版本或版本范围。如果 Python 仅在构建时、运行时或用于测试时需要，可以将其设置为构建、运行或测试依赖项，使用 build , run , 或 test 。如果在修补阶段也需要 Python，请使用 patch 。有关详细信息，请参阅使用 Python。

当需要框架导出的变量但不需要依赖 Python 时，可以使用 USES=python:env 。这可能发生在使用 USES=shebangfix 时，目标只是修复 shebangs 而不添加对 Python 的依赖。

## 17.89. `qmail`

可能的参数：(无), build , run , both , vars

使用 mail/qmail。使用 build 参数时，意味着 qmail 是一个构建时依赖。 run 意味着是运行时依赖。不使用参数或使用 both 参数意味着同时是运行时和构建时依赖。 vars 只会设置 QMAIL 变量供 port 使用。

## 17.90. `qmake`

可能的参数：(无), norecursive , outsource , no_env , no_configure

使用 QMake 进行配置。有关详细信息，请参阅 使用 qmake 。

## 17.91. `qt`

可能的参数： 5 ， 6 ， no_env

添加依赖于 Qt 组件。 no_env 直接传递给 USES= qmake 。有关详细信息，请参阅 使用 Qt。

## 17.92. `qt-dist`

可能的参数：（无）或 5 和（无）或 6 和（无）或其中之一 3d ， 5compat ， base ， charts ， connectivity ， datavis3d ， declarative ， doc languageserver ， gamepad ， graphicaleffects ， imageformats ， locat ion ， lottie ， multimedia ， networkauth ， positioning ， quick3d ， quickcontrols2 ， quickcontrols ， quicktimeline ， remoteobjects ， script ， scxml  ， sensors ， serialbus ， serialPort， shadertools ， speech ， svg ， tools ， translations ， virtualkeyboard ， wayland ， webchannel ， webengine ， webglplugin ， websockets ， webview ， x11extras ， xmlpatterns 。

提供构建 Qt 5 和 Qt 6 组件的支持。 它会处理配置适当环境的设置port构建。

示例 7. 构建 Qt 5 组件

port是 Qt 5 的 networkauth 组件，是 networkauth 分发文件的一部分。

```
PORTNAME=	networkauth
DISTVERSION=	${QT5_VERSION}

USES=		qt-dist:5
```

示例 8. 构建 Qt 6 组件

port是 Qt 6 的 websockets 组件，是 websockets 分发文件的一部分。

```
PORTNAME=       websockets
PORTVERSION=    ${QT6_VERSION}

USES=           qt-dist:6
```

如果 PORTNAME 与组件名称不匹配，则可以将其作为参数传递给 qt-dist 。

例 9。使用不同名称构建 Qt 5 组件

port是 Qt 5 的 gui 组件，是 base 分发文件的一部分。

```
PORTNAME=	gui
DISTVERSION=	${QT5_VERSION}

USES=		qt-dist:5,base
```

## 17.93. `readline`

可能的参数：(无)， port

使用 readline 作为库依赖，并根据需要设置 CPPFLAGS 和 LDFLAGS 。如果使用了 port 参数或基本系统中没有安装 readline，则添加一个 devel/readline 的依赖项。

## 17.94. `ruby`

可能的参数：(无)， build ， extconf ， run ， setup

提供对与 Ruby 相关的 ports 的支持。 (none) 没有参数会在运行时依赖于 lang/ruby。 build 在构建时会依赖于 lang/ruby。 extconf 表示 port 使用 extconf.rb 进行配置。 run 在运行时会依赖于 lang/ruby，这也是默认设置。 setup 表示 port 使用 setup.rb 进行配置和构建。

用户可以定义以下变量：

RUBY_VER 表示 ruby 的替代短版本，格式为 `x.y'。

RUBY_DEFAULT_VER 设置为 (例如) 2.7 以使用 ruby27 作为默认版本。

RUBY_ARCH 设置架构名称 (例如 i386-freebsd7)。

导出以下变量供 port 使用:

RUBY 设为 Ruby 的完整路径。如果设置，以下变量的值将自动从 Ruby 可执行文件中获取： RUBY_ARCH ， RUBY_ARCHLIBDIR ， RUBY_LIBDIR ， RUBY_SITEARCHLIBDIR ， RUBY_SITELIBDIR ， RUBY_VER and RUBY_VERSION

RUBY_VER 设为 Ruby 的替代短版本，格式为 `x.y'。

RUBY_EXTCONF 设为 extconf.rb 的替代名称（默认值：extconf.rb）。

RUBY_EXTCONF_SUBDIRS 设置为子目录列表，如果包含多个模块。

RUBY_SETUP 设置为 setup.rb 的替代名称（默认为 setup.rb）。

## 17.95. `samba`

可能的参数： build , env , lib , run

处理对 Samba 的依赖。 env 不会添加任何依赖，只会设置变量。 build 和 run 会在 smbd 上添加构建时和运行时依赖。 lib 将添加一个对 libsmbclient.so 的依赖。被导出的变量是：

SAMBAPORT 默认 Samba port 的来源。

SAMBAINCLUDES Samba 头文件的位置。

SAMBALIBS Samba 共享库所在的目录。

## 17.96. `scons`

可能的参数：（无）

为 devel/scons 的使用提供支持。有关更多信息，请参阅 使用 scons 。

## 17.97. `shared-mime-info`

可能的参数：(无)

使用 misc/shared-mime-info 中的 update-mime-database。这将自动添加一个 post-install 步骤，使 port 本身仍然可以根据需要指定自己的 post-install 步骤。它还会在 plist 中添加一个 @shared-mime-info 条目。

## 17.98. `shebangfix`

可能的参数：(无)

许多软件在脚本解释器的位置上使用不正确，最明显的是 /usr/bin/perl 和 /bin/bash。shebangfix 宏修复列在 SHEBANG_REGEX 、 SHEBANG_GLOB 或 SHEBANG_FILES 中的脚本的 shebang 行。

`SHEBANG_REGEX`

包含一个扩展正则表达式，并与 find(1) 的 -iregex 参数一起使用。参见 USES=shebangfix 与 SHEBANG_REGEX 。

`SHEBANG_GLOB`

包含与 find(1) 的 -name 参数一起使用的模式列表。参见 USES=shebangfix 与 SHEBANG_GLOB 。

`SHEBANG_FILES`

包含文件列表或 sh(1)通配符。 shebangfix 宏从 ${WRKSRC} 执行，因此 SHEBANG_FILES 可以包含相对于 ${WRKSRC} 的路径。如果 ${WRKSRC} 外的文件需要打补丁，它也可以处理绝对路径。参见 USES=shebangfix 与 SHEBANG_FILES 。

目前默认支持 Bash、Java、Ksh、Lua、Perl、PHP、Python、Ruby、Tcl 和 Tk。

有三个配置变量：

`SHEBANG_LANG`

受支持的解释器列表。

`_interp__CMD`

FreeBSD 上命令解释器的路径。默认值为 ${LOCALBASE}/bin/interp 。

`_interp__OLD_CMD`

解释器的错误调用列表。这些通常是过时的路径，或者是在其他操作系统上使用的在 FreeBSD 上不正确的路径。它们将在 _interp__CMD 中被正确的路径替换。

|  | 这些将永远是 interp__OLD_CMD 的一部分 "/usr/bin/env _interp  " /bin/interp<span> </span>/usr/bin/interp<span> </span>/usr/local/bin/interp 。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

|  | _interp__OLD_CMD 包含多个值。任何带有空格的条目必须用引号括起来。添加解释器到 USES\=shebangfix 时，要指定所有路径。 |
| -- | ------------------------------------------------------------------------------------------------------------------------ |

|  | 修复 shebang 是在 patch 阶段完成的。如果在 build 阶段创建带有不正确 shebang 的脚本，则必须修补构建过程（例如，配置脚本或 Makefiles）或给予正确的路径（例如 CONFIGURE_ENV 、 CONFIGURE_ARGS 、 MAKE_ENV 、 或 MAKE_ARGS ）以生成正确的 shebang。<br /><br />在 _interp__CMD 中，支持的解释器的正确路径可用。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

|  | 当与 USES\=python 一起使用，并且只想修复 shebangs，而不想依赖 Python 本身时，请使用 USES=python:env 。 |
| -- | ----------------------------------------------------------------------------------------------------------- |

示例 10. 添加另一个解释器到 USES=shebangfix

要添加另一个解释器，请设置 SHEBANG_LANG 。例如：

```
SHEBANG_LANG=	lua
```

示例 11. 在添加解释器到 USES=shebangfix 时指定所有路径

如果它尚未定义，并且对于 _interpOLD_CMD 和 _interp 没有默认值，Ksh 条目可以定义为：

```
SHEBANG_LANG=	ksh
ksh_OLD_CMD=	"/usr/bin/env ksh" /bin/ksh /usr/bin/ksh
ksh_CMD=	${LOCALBASE}/bin/ksh
```

示例 12. 为解释器添加一个奇怪的位置

一些软件使用奇怪的位置来放置解释器。例如，一个应用程序可能期望 Python 位于 /opt/bin/python2.7。可以在 port Makefile 中声明要替换的奇怪路径：

```
python_OLD_CMD=	/opt/bin/python2.7
```

示例 13. USES=shebangfix 与 SHEBANG_REGEX

为了修复所有以 .pl、.sh 或 .cgi 结尾的文件，请执行：

```
USES=	shebangfix
SHEBANG_REGEX=	./scripts/.*\.(sh|pl|cgi)
```

|  | SHEBANG_REGEX 是通过运行 find -E 使用的，它使用现代正则表达式，也称为扩展正则表达式。有关更多信息，请参阅 reformat(7)。 |
| -- | ------------------------------------------------------------------------------------------------------------------------- |

示例 14。 USES=shebangfix 和 SHEBANG_GLOB

修复所有以 .pl 或 .sh 结尾的文件 ${WRKSRC} ，请执行：

```
USES=	shebangfix
SHEBANG_GLOB=	*.sh *.pl
```

示例 15. USES=shebangfix 和 SHEBANG_FILES

要修复 ${WRKSRC} 中的文件 script/foobar.pl 和 script/*.sh，请执行：

```
USES=	shebangfix
SHEBANG_FILES=	scripts/foobar.pl scripts/*.sh
```

## 17.99. `sqlite`

可能的参数：（无）, 2 , 3

添加对 SQLite 的依赖。默认使用的版本是 3，但也可以使用 :2 修改器来使用版本 2。

## 17.100. `ssl`

可能的参数：（无）, build , run

提供对 OpenSSL 的支持。可以使用 build 或 run 来指定仅限构建或运行时的依赖关系。这些变量供port使用，也添加到 MAKE_ENV 中：

`OPENSSLBASE`

OpenSSL 安装基础路径。

`OPENSSLDIR`

OpenSSL 配置文件路径。

`OPENSSLLIB`

OpenSSL 库的路径。

`OPENSSLINC`

OpenSSL 包含文件的路径。

`OPENSSLRPATH`

如果定义了，链接器需要使用的 OpenSSL 库的路径。

```
BROKEN_SSL=	libressl
BROKEN_SSL_REASON_libressl=	needs features only available in OpenSSL
```

## 17.101. `tar`

可能的参数: (无), Z , bz2 , bzip2 , lzma , tbz , tbz2 , tgz , txz , xz , zst , zstd

将 EXTRACT_SUFX 设置为 .tar , .tar.Z , .tar.bz2 , .tar.bz2 , .tar.lzma , .tbz , .tbz2 , .tgz , .txz , .tar.xz , .tar.zst 或 .tar.zstd 之一。

## 17.102. `tcl`

可能的参数: 版本, wrapper , build , run , tea

添加对 Tcl 的依赖。可以使用版本号进行特定版本的请求。版本号可以为空，一个或多个精确的版本号（当前 84 ， 85 或 86 ），或一个最低版本号（当前 84+ ， 85+ 或 86+ ）。若只需请求一个非特定版本的 Wrapper，请使用 wrapper 。可以使用 build 或 run 指定构建时或运行时的依赖项。要使用 Tcl 扩展架构构建 port，请使用 tea 。在包含 bsd.port.pre.mk 后，port 可以使用这些变量检查结果：

* TCL_VER ：所选的 Tcl 主版本.次版本号
* TCLSH ：Tcl 解释器的完整路径
* TCL_LIBDIR ：Tcl 库的路径
* TCL_INCLUDEDIR ：Tcl C 头文件的路径
* TK_VER ：选择的 Tk 主版本号.次版本号
* WISH ：Tk 解释器的完整路径
* TK_LIBDIR ：Tk 库的路径
* TK_INCLUDEDIR ：Tk C 头文件的路径

## 17.103. `terminfo`

可能的参数：(无)

将 @terminfo 添加到 plist 中。在 ${PREFIX}/share/misc 中安装 *.terminfo 文件时使用。

## 17.104. `tex`

可能的参数：(无)

提供对 tex 的支持。加载所有与 TEX 相关的默认变量 ports，并且不增加任何 ports 的依赖。

变量用于指定所需的 TEX 模块。

USE_TEX 运行时所需的 TEX 扩展列表。添加 :build 到扩展名以添加构建时依赖项，添加 :run 以添加运行时依赖项，添加 :test 以添加测试时依赖项，添加 :extract 以添加提取时依赖项。例如： base texmf:build source:run

目前可能的参数如下：

* `base`
* `texmf`
* `source`
* `docs`
* `web2c`
* `kpathsea`
* `ptexenc`
* `basic`
* `tlmgr`
* `texlua`
* `texluajit`
* `synctex`
* `xpdfopen`
* `dvipsk`
* `dvipdfmx`
* `xdvik`
* `gbklatex`
* `formats`
* `tex`
* `latex`
* `pdftex`
* `jadetex`
* `luatex`
* `ptex`
* `xetex`
* `xmltex`
* `texhash`
* `updmap`
* `fmtutil`

## 17.105. `tk`

与 tcl 的参数相同

当同时使用 Tcl 和 Tk 时的小包装。当使用 Tcl 时，返回的变量与之相同。

## 17.106. `uidfix`

可能的参数：（无）

更改构建系统的某些默认行为（主要是变量），以允许将此作为普通用户安装。在使用 USES=fakeroot 或打补丁之前，请在 port 中尝试这种方法。

## 17.107. `uniquefiles`

可能的参数：（无）， dirs

通过添加前缀或后缀使文件或目录 'unique'。如果使用 dirs 参数，则基于 UNIQUE_PREFIX 对标准目录 DOCSDIR 、 EXAMPLESDIR 、 DATADIR 、 WWWDIR 、 ETCDIR 进行前缀添加（仅前缀）。这些变量适用于 ports：

* UNIQUE_PREFIX ：要用于目录和文件的前缀。默认： ${PKGNAMEPREFIX} 。
* UNIQUE_PREFIX_FILES ：需要添加前缀的文件列表。默认：空。
* UNIQUE_SUFFIX ：用于文件的后缀。默认： ${PKGNAMESUFFIX} 。
* UNIQUE_SUFFIX_FILES ：需要添加后缀的文件列表。默认：空。

## 17.108. `vala`

可能的参数： build ， lib ， no_depend

添加构建或库依赖于 lang/vala。 no_depend 参数保留给 lang/vala 本身。

## 17.109. `varnish`

可能的参数: 4 （默认）, 6 , 7

处理对 Varnish Cache 的依赖。添加对 www/varnish*的依赖。

## 17.110. `webplugin`

可能的参数: (无), ARGS

自动为支持 webplugin 框架的每个应用程序创建和删除符号链接。 ARGS 可以是以下之一:

* gecko ：支持基于 Gecko 的插件
* native ：支持 Gecko、Opera 和 WebKit-GTK 插件
* linux ：支持 Linux 插件
* all （默认，隐式）：支持所有插件类型
* (个体条目): 仅支持所列的浏览器

这些变量可以调整：

* WEBPLUGIN_FILES ：无默认值，必须手动设置。安装插件文件。
* WEBPLUGIN_DIR ：安装插件文件的目录，默认为 PREFIX/lib/browser_plugins/WEBPLUGIN_NAME。如果 port 将插件文件安装在默认目录之外，请设置此项以防止损坏的符号链接。
* WEBPLUGIN_NAME ：安装插件文件的最终目录，默认为 PKGBASE 。

## 17.111. `xfce`

可能的参数：(无)， gtk2

提供对与 Xfce 相关的 ports 的支持。详见使用 Xfce 以获取详情。

gtk2 参数指定 port 需要 GTK2 支持。它添加了一些核心组件提供的额外功能，例如 x11/libxfce4menu 和 x11-wm/xfce4-panel。

## 17.112. `xorg`

可能的参数：（无）

提供了依赖于 X.org 组件的简便方法。这些组件应该在 USE_XORG 中列出。可用的组件为：

表 1. 可用的 X.Org 组件

| 名称 | 描述                            |
| ------ | --------------------------------- |
| `dmx`     | DMX 扩展库                      |
| `fontenc`     | fontenc 库                      |
| `fontutil`     | 在目录中创建 X 字体文件的索引   |
| `ice`     | X11 的 Inter Client Exchange 库 |
| `libfs`     | FS 库                           |
| `pciaccess`     | 通用 PCI 访问库                 |
| `pixman`     | 低级像素操作库                  |
| `sm`     | X11 会话管理库                  |
| `x11`     | X11 库                          |
| `xau`     | X11 的认证协议库                |
| `xaw`     | X Athena 小部件库               |
| `xaw6`     | X 雅典娜小部件库                |
| `xaw7`     | X 雅典娜小部件库                |
| `xbitmaps`     | X.Org 位图数据                  |
| `xcb`     | X 协议 C 语言绑定 (XCB) 库      |
| `xcomposite`     | X Composite 扩展库              |
| `xcursor`     | X 客户端光标加载库              |
| `xdamage`     | X 损坏扩展库                    |
| `xdmcp`     | X 显示管理器控制协议库          |
| `xext`     | X11 扩展库                      |
| `xfixes`     | X 修复扩展库                    |
| `xfont`     | X 字体库                        |
| `xfont2`     | X 字体库                        |
| `xft`     | X 应用程序的客户端字体 API      |
| `xi`     | X 输入扩展库                    |
| `xinerama`     | X11 Xinerama 库                 |
| `xkbfile`     | XKB 文件库                      |
| `xmu`     | X 杂项实用程序库                |
| `xmuu`     | X 杂项实用程序库                |
| `xorg-macros`     | X.Org 开发 aclocal 宏           |
| `xorg-server`     | X.Org X 服务器及相关程序        |
| `xorgproto`     | xorg 协议头文件                 |
| `xpm`     | X 镜像库                        |
| `xpresent`     | X 现在扩展库                    |
| `xrandr`     | X 调整和旋转扩展库              |
| `xrender`     | X 渲染扩展库                    |
| `xres`     | X 资源使用库                    |
| `xscrnsaver`     | XScrnSaver 库                   |
| `xshmfence`     | 共享内存'SyncFence'同步原语     |
| `xt`     | X 工具包库                      |
| `xtrans`     | X 的抽象网络代码                |
| `xtst`     | X 测试扩展                      |
| `xv`     | X 视频扩展库                    |
| `xvmc`     | X 视频扩展运动补偿库            |
| `xxf86dga`     | X DGA 扩展                      |
| `xxf86vm`     | X Vidmode 扩展                  |

## 17.113. `xorg-cat`

可能的参数： app ， data ， doc ， driver ， font ， lib ， proto ， util ， xserver 和（无）或一个关闭 autotools （默认）， meson

提供支持构建 Xorg 组件。它负责设置所需的常见依赖项和适当的配置环境。此功能仅用于 Xorg 组件。

类别必须与上游类别匹配。

第二个参数是要使用的构建系统。autotools 是默认选项，但也支持 meson。

## 17.114. `zip`

可能的参数: (无), infozip

表示分发文件使用 ZIP 压缩算法。对于使用 InfoZip 算法的文件，必须传递 infozip 参数以设置适当的依赖项。
