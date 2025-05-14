# 第 17 章 使用 USES 宏

## 17.1. `USES` 的介绍

`USES` 宏简化了声明 Ports 依赖和设置的过程。它们可以添加依赖关系、修改构建行为、向包添加元数据等，所有这些都通过选择简单的预设值来完成。

本章的每个部分都描述了 `USES` 的一个可能值及其可能的参数。参数在值后通过冒号（`:`）附加。多个参数之间用逗号（`,`）分隔。

**示例 1. 使用多个值**

```makefile
USES=	bison perl
```

**示例 2. 添加一个参数**

```makefile
USES=	tar:xz
```

**示例 3. 添加多个参数**

```makefile
USES=	drupal:7,theme
```

**示例 4. 综合使用**

```makefile
USES=	pgsql:9.3+ cpe python:2.7,build
```

## 17.2. `7z`

可能的参数：（无），`p7zip`，`partial`

使用 [7z(1)](https://man.freebsd.org/cgi/man.cgi?query=7z&sektion=1&format=html) 代替 [bsdtar(1)](https://man.freebsd.org/cgi/man.cgi?query=bsdtar&sektion=1&format=html) 提取文件，并将 `EXTRACT_SUFX` 设置为 `.7z`。如果基本系统中的 `7z` 无法提取文件，`p7zip` 选项会强制依赖于来自 [archivers/p7zip](https://cgit.freebsd.org/ports/tree/archivers/p7zip/) 的 `7z`。如果使用 `partial` 选项，则 `EXTRACT_SUFX` 不会改变，这对于主分发文件没有 **.7z** 扩展名时非常有用。

## 17.3. `ada`

可能的参数：（无），`6`，`12`，`(run)`

依赖一个支持 Ada 的编译器，并相应地设置 `CC`。默认使用来自 Ports 的 `gcc6-aux`。

## 17.4. `angr`

可能的参数：`binaries`，`nose`

为需要 [angrinary 分析平台](https://github.com/angr/angr) 的 Ports 提供支持。

如果存在 `binaries` 参数，则该 Port 需要特殊的 `angr` 二进制文件进行测试。

如果存在 `nose` 参数，则该 Port 使用 `nosetests` 作为测试目标。该参数隐式地意味着 `USES=python:test`。

该框架提供以下变量供 Port 设置：

`ANGR_VERSION`：`angr` 项目程序的版本。

`ANGR_BINARIES_TAGNAME`：`angr` 二进制文件的标签名。

`ANGR_NOSETESTS`：`nosetests` 程序的路径。

## 17.5. `ansible`

可能的参数：`env`，`module`，`plugin`

为依赖 [sysutils/ansible](https://cgit.freebsd.org/ports/tree/sysutils/ansible/) 的 Ports 提供支持。

如果存在 `env` 参数，则该 Port 不依赖 [sysutils/ansible](https://cgit.freebsd.org/ports/tree/sysutils/ansible/)，但需要设置一些 Ansible 变量。

如果存在 `module` 参数，则该 Port 是一个 Ansible 模块。

如果存在 `plugin` 参数，则该 Port 是一个 Ansible 插件。

该框架向 Port 暴露以下变量：

`ANSIBLE_CMD`：ansible 程序的路径。

`ANSIBLE_DOC_CMD`：ansible-doc 程序的路径。

`ANSIBLE_RUN_DEPENDS`：与 Ansible 相关的 RUN\_DEPENDS。

`ANSIBLE_DATADIR`：Ansible 模块和插件所在的目录结构的根路径。

`ANSIBLE_ETCDIR`：Ansible 配置文件目录的路径。

`ANSIBLE_PLUGINS_PREFIX`：`${ANSIBLE_DATADIR}` 中 "plugins" 目录的路径。

`ANSIBLE_MODULESDIR`：本地 Ansible 模块的目录路径。

`ANSIBLE_PLUGINSDIR`：本地 Ansible 插件的目录路径。

`ANSIBLE_PLUGIN_TYPE`：Ansible 插件类型（例如，“connection”，“inventory”或“vars”）。

## 17.6. `apache`

可能的参数：（无），`2.4`，`build`，`run`，`server`

为依赖 Apache Web 服务器的 Ports 提供支持。

版本参数可用于要求特定版本的 Apache httpd。可以设置特定版本（`USES=apache:2.4`），最低版本（`USES=apache:2.4+`）或最高版本（`USES=apache:-2.4`）。

如果提供 `build` 参数，则会将构建依赖项添加到 Port。

如果提供 `run` 参数，则会将运行时依赖项添加到 Port。

如果提供 `server` 参数，则表示该 Port 是一个服务器 Port。

该框架提供以下变量供 Port 设置：

`AP_FAST_BUILD`：自动模块构建。

`AP_GENPLIST`：自动生成 `PLIST`，并将禁用的模块添加到 **httpd.conf** 中（仅在没有 `pkg-plist` 文件时）。

`MODULENAME`：Apache 模块的名称。默认值：`${PORTNAME}`。

`SHORTMODNAME`：Apache 模块的简短名称。默认值：`${MODULENAME:S/mod_//}`。

`SRC_FILE`：Apache 模块的源文件。默认值：`${MODULENAME}.c`。

以下变量可以被 Port 访问：

`APACHE_VERSION`：所选 Apache 服务器的主版本号和次版本号，例如 2.4。

`APACHEETCDIR`：Apache 配置目录的位置。默认值：**\${LOCALBASE}/etc/apache24**。

`APACHEINCLUDEDIR`：Apache 包含文件的位置。默认值：**\${LOCALBASE}/include/apache24**。

`APACHEMODDIR`：Apache 模块的位置。默认值：**\${LOCALBASE}/libxexec/apache24**。

`APACHE_DEFAULT`：默认的 Apache 版本。

## 17.7. `autoreconf`

可能的参数：（无），`build`

运行 `autoreconf`。它封装了 `aclocal`、`autoconf`、`autoheader`、`automake`、`autopoint` 和 `libtoolize` 命令。每个命令都应用于 **\${AUTORECONF\_WRKSRC}/configure.ac** 或其旧名称 **\${AUTORECONF\_WRKSRC}/configure.in**。如果 **configure.ac** 使用 `AC_CONFIG_SUBDIRS` 定义了子目录并有自己的 **configure.ac**，则 `autoreconf` 将递归更新这些子目录。`:build` 参数仅添加对这些工具的构建时依赖项，但不运行 `autoreconf`。如果 `WRKSRC` 中不包含 **configure.ac** 的路径，可以通过设置 `AUTORECONF_WRKSRC` 来指定路径。

## 17.8. `azurepy`

可能的参数：（无）

为 `py-azure*` Ports 提供支持。移除 `azure` 命名空间并清理常见文件。

## 17.9. `blaslapack`

可能的参数：（无），`atlas`，`netlib`（默认），`gotoblas`，`openblas`

添加对 Blas / Lapack 库的依赖。

## 17.10. `bdb`

可能的参数：（无），`5`（默认），`18`

添加对 Berkeley DB 库的依赖。默认为 [databases/db5](https://cgit.freebsd.org/ports/tree/databases/db5/)。在使用 `:18` 参数时，也可以依赖于 [databases/db18](https://cgit.freebsd.org/ports/tree/databases/db18/)。可以声明一个接受值的范围，`：5+` 会查找安装的最高版本，如果没有其他版本则回退到 5。`INVALID_BDB_VER` 可用于指定与此 Port 不兼容的版本。该框架向 Port 暴露以下变量：

`BDB_LIB_NAME`：Berkeley DB 库的名称。例如，使用 [databases/db5](https://cgit.freebsd.org/ports/tree/databases/db5/) 时，它包含 `db-5.3`。

`BDB_LIB_CXX_NAME`：Berkeley DBC++ 库的名称。例如，使用 [databases/db5](https://cgit.freebsd.org/ports/tree/databases/db5/) 时，它包含 `db_cxx-5.3`。

`BDB_INCLUDE_DIR`：Berkeley DB 包含目录的位置。例如，使用 [databases/db5](https://cgit.freebsd.org/ports/tree/databases/db5/) 时，它包含 `${LOCALBASE}/include/db5`。

`BDB_LIB_DIR`：Berkeley DB 库目录的位置。例如，使用 [databases/db5](https://cgit.freebsd.org/ports/tree/databases/db5/) 时，它包含 `${LOCALBASE}/lib`。

`BDB_VER`：检测到的 Berkeley DB 版本。例如，如果使用 `USES=bdb:5+` 并安装了 Berkeley DB 18，则它包含 `18`。

>**重要**
>
>[databases/db48](https://cgit.freebsd.org/ports/tree/databases/db48/) 已弃用并且不再支持。任何 Port 都不得使用它


## 17.11. `bison`

可能的参数：（无），`build`，`run`，`both`

使用 [devel/bison](https://cgit.freebsd.org/ports/tree/devel/bison/)。默认情况下，未指定参数或指定 `build` 参数时，表示 `bison` 是构建时依赖；`run` 表示运行时依赖；`both` 表示同时是构建时和运行时依赖。

## 17.12. `budgie`

可能的参数：（无）

为 Budgie 桌面环境提供支持。使用 `USE_BUDGIE` 选择 Port 所需的组件。有关更多信息，请参见 [使用 Budgie](https://docs.freebsd.org/en/books/porters-handbook/special/#using-budgie)。

## 17.13. `cabal`

>**重要**
>
>不应为 Haskell 库创建 Port，详情见 [Haskell Libraries](https://docs.freebsd.org/en/books/porters-handbook/special/#haskell-libs)。


可能的参数：（无），`hpack`，`nodefault`

设置用于构建 Haskell 软件的默认值和目标，使用 Cabal。添加对 Haskell 编译器 Port（[lang/ghc](https://cgit.freebsd.org/ports/tree/lang/ghc/)）的构建依赖。如果 `BUILD_DEPENDS` 变量中已经列出了其他版本的 GHC（例如 [lang/ghc810](https://cgit.freebsd.org/ports/tree/lang/ghc810/)），则使用该版本。如果提供了 `hpack` 参数，则添加对 [devel/hs-hpack](https://cgit.freebsd.org/ports/tree/devel/hs-hpack/) 的构建依赖，并在配置步骤中调用 `hpack` 生成 .cabal 文件。如果提供了 `nodefault` 参数，框架将不会尝试从 Hackage 拉取主分发文件。如果存在 `USE_GITHUB` 或 `USE_GITLAB`，则此参数会隐式添加。

框架提供以下变量：

`CABAL_REVISION`：Haskell 包在 Hackage 上可能有修订版本。将此选项设置为整数以拉取修订后的包描述。

`USE_CABAL`：如果软件使用 Haskell 依赖项，则在此变量中列出它们。每个项目应以 `packagename-0.1.2` 的形式列出。依赖项也可以有修订版本，修订版本在 `_` 符号后指定。支持自动生成依赖项列表，详情见 [使用 Cabal 构建 Haskell 应用程序](https://docs.freebsd.org/en/books/porters-handbook/special/#using-cabal)。

`CABAL_FLAGS`：在配置和构建阶段传递给 `cabal-install` 的标志列表。标志按原样传递。此变量通常用于启用或禁用 .cabal 文件中声明的标志。使用 `foo` 启用 `foo` 标志，使用 `-foo` 禁用它。

`CABAL_EXECUTABLES`：Port 安装的可执行文件列表。默认值：`${PORTNAME}`。请参阅项目的 .cabal 文件，获取此变量可能的值。每个值对应 .cabal 文件中的 `executable` 项。此列表中的项目会自动添加到 pkg-plist。

`SKIP_CABAL_PLIST`：如果定义了此变量，则不会将 `${CABAL_EXECUTABLES}` 中的项目添加到 pkg-plist。

`opt_USE_CABAL`：根据 `opt` 选项向 `${USE_CABAL}` 中添加项目。

`opt_CABAL_EXECUTABLES`：根据 `opt` 选项向 `${CABAL_EXECUTABLES}` 中添加项目。

`opt_CABAL_FLAGS`：如果启用了 `opt`，则将该值追加到 `${CABAL_FLAGS}` 中。否则，追加 `-value` 以禁用该标志。请注意，此行为与普通的 `CABAL_FLAGS` 略有不同，因为它不接受以 `-` 开头的值。

`CABAL_WRAPPER_SCRIPTS`：`${CABAL_EXECUTABLES}` 的子集，包含要包装到 shell 脚本中的 Haskell 程序，该脚本在运行程序之前设置 `*_datadir` 环境变量。这还会导致实际的 Haskell 二进制文件安装到 `libexec/cabal/` 目录。对于将数据文件安装到 `share/` 目录中的 Haskell 程序，必须使用此选项。

`FOO_DATADIR_VARS`：其他 Haskell 包的列表，其数据文件应由名为 `FOO` 的可执行文件访问。该可执行文件应是 `${CABAL_WRAPPER_SCRIPTS}` 的一部分。列出的 Haskell 包不应有版本后缀。

`CABAL_PROJECT`：某些 Haskell 项目可能已经有一个 `cabal.project` 文件，该文件也由 Ports 框架生成。如果是这种情况，请使用此变量指定如何处理原始的 `cabal.project`。将此变量设置为 `remove` 将导致原始文件在提取阶段被删除。将此变量设置为 `append` 将：

1. 在提取阶段将原始文件移动到 `cabal.project.${PORTNAME}`。
2. 在补丁阶段将原始的 `cabal.project.${PORTNAME}` 和生成的 `cabal.project` 合并成一个文件。使用 `append` 可以在合并之前对原始文件进行补丁处理。

## 17.14. `cargo`

可能的参数：（无）

使用 Cargo 进行配置、构建和测试。它可以用于 Port 化使用 Cargo 构建系统的 Rust 应用程序。更多信息请参见 [使用 `cargo` 构建 Rust 应用程序](https://docs.freebsd.org/en/books/porters-handbook/special/#using-cargo)。

## 17.15. `charsetfix`

可能的参数：（无）

防止 Port 安装 **charset.alias**。该文件必须仅由 [converters/libiconv](https://cgit.freebsd.org/ports/tree/converters/libiconv/) 安装。如果 **charset.alias** 不是由 **\${WRKSRC}/Makefile.in** 安装的，可以通过设置 `CHARSETFIX_MAKEFILEIN` 来指定相对于 `WRKSRC` 的路径。

## 17.16. `cl`

可能的参数：（无）

为 Common Lisp  Port 提供支持。

框架提供以下变量，Port 可以设置：

`ASDF_MODULES`：当设置 `FASL_TARGET`（默认为 `PORTNAME`）时，要构建的 `ASDF` 模块列表。

`FASL_TARGET`：构建 Port 的 fasl 变体（可以是 `ccl`、`clisp` 或 `sbcl`）。

`USE_ASDF`：依赖 [devel/cl-asdf](https://cgit.freebsd.org/ports/tree/devel/cl-asdf/)。

`USE_ASDF_FASL`：依赖 `devel/cl-asdf-<FASL_TARGET>`。

`USE_CCL`：依赖 [lang/ccl](https://cgit.freebsd.org/ports/tree/lang/ccl/)；当 `FASL_TARGET=ccl` 时，隐式依赖。

`USE_CLISP`：依赖 [lang/clisp](https://cgit.freebsd.org/ports/tree/lang/clisp/)；当 `FASL_TARGET=clisp` 时，隐式依赖。

`USE_SBCL`：依赖 [lang/sbcl](https://cgit.freebsd.org/ports/tree/lang/sbcl/)；当 `FASL_TARGET=SBCL` 时，隐式依赖。

框架提供以下变量，Port 可以读取：

`ASDF_PATHNAME`：CL 源代码路径。

`ASDF_REGISTRY`：包含 asd 文件的 CL 注册表路径。

`CCL`：Clozure Common Lisp 编译器的路径。

`CLISP`：GNU Common Lisp 编译器的路径。

`CL_LIBDIR_REL`：相对于 `LOCALBASE` 或 `PREFIX` 的 CL 库目录。

`FASL_DIR_REL`：编译后的 fasl 文件的相对路径，取决于 `FASL_TARGET`。

`FASL_PATHNAME`：CL fasl 文件的路径。

`LISP_EXTRA_ARG`：构建 fasl 时使用的额外参数。

`SBCL`：Steel Bank Common Lisp 编译器的路径。

## 17.17. `cmake`

可能的参数：（无），`insource`，`noninja`，`run`，`testing`

使用 CMake 配置 Port 并生成构建系统。

默认情况下，执行的是源代码外构建，将源代码中的构建产物与源代码分离。使用 `insource` 参数时，将执行源代码内构建。该参数应仅作为例外，在常规的源代码外构建无法工作时使用。

默认情况下，使用 Ninja（[devel/ninja](https://cgit.freebsd.org/ports/tree/devel/ninja/)）进行构建。在某些情况下，这可能无法正确工作。使用 `noninja` 参数时，构建将使用常规的 `make`。只有当基于 Ninja 的构建不工作时，才应使用此参数。

使用 `run` 参数时，除了构建依赖外，还会注册一个运行时依赖。

使用 `testing` 参数时，将添加一个测试目标，该目标使用 CTest。当运行测试时，Port 将为测试重新配置并重新构建。

有关更多信息，请参见 [使用 `cmake`](https://docs.freebsd.org/en/books/porters-handbook/special/#using-cmake)。

## 17.18. `compiler`

可能的参数：（无），`env`（默认，隐式），`C++17-lang`，`C++14-lang`，`C++11-lang`，`gcc-C++11-lib`，`C++11-lib`，`C++0x`，`c11`，`nestedfct`，`features`

根据给定的需求确定要使用的编译器。如果 Port 需要支持 C++17 的编译器，请使用 `C++17-lang`；如果需要支持 C++14 的编译器，请使用 `C++14-lang`；如果需要支持 C++11 的编译器，请使用 `C++11-lang`；如果需要带有 C++11 库的 `g++` 编译器，请使用 `gcc-C++11-lib`；如果需要支持 C++11 标准库的编译器，请使用 `C++11-lib`。如果 Port 需要支持 C++0x、C11 或嵌套函数，应使用相应的参数。

使用 `features` 请求默认编译器支持的特性列表。在包含 **bsd.port.pre.mk** 后，Port 可以使用以下变量检查结果：

* `COMPILER_TYPE`：系统上的默认编译器，可能是 gcc 或 clang。
* `ALT_COMPILER_TYPE`：系统上的替代编译器，可能是 gcc 或 clang，仅在系统中存在两个编译器时设置。
* `COMPILER_VERSION`：默认编译器版本的前两位数字。
* `ALT_COMPILER_VERSION`：替代编译器的前两位版本数字（如果存在）。
* `CHOSEN_COMPILER_TYPE`：选择的编译器，可能是 gcc 或 clang。
* `COMPILER_FEATURES`：默认编译器支持的特性，目前列出 C++ 库。

## 17.19. `cpe`

可能的参数：（无）

在包清单中包含 Common Platform Enumeration (CPE) 信息，以 CPE 2.3 格式的字符串表示。有关详细信息，请参见 [CPE 规范](https://scap.nist.gov/specifications/cpe/)。要将 CPE 信息添加到 Port，请按照以下步骤操作：

1. 通过使用 NVD 的 [CPE 搜索引擎](https://web.nvd.nist.gov/view/cpe/search) 或在 [官方 CPE 字典](https://nvd.nist.gov/feeds/xml/cpe/dictionary/official-cpe-dictionary_v2.3.xml.gz) 中搜索软件产品的官方 CPE 条目（警告，此文件为非常大的 XML 文件）。*切勿自行编造 CPE 数据。*
2. 将 `cpe` 添加到 `USES` 中，并使用 `make -V CPE_STR` 检查结果，确保它与 CPE 字典条目一致。逐步调整直到 `make -V CPE_STR` 正确。
3. 如果产品名称（第二个字段，默认为 `PORTNAME`）不正确，定义 `CPE_PRODUCT`。
4. 如果厂商名称（第一个字段，默认为 `CPE_PRODUCT`）不正确，定义 `CPE_VENDOR`。
5. 如果版本字段（第三个字段，默认为 `PORTVERSION`）不正确，定义 `CPE_VERSION`。
6. 如果更新字段（第四个字段，默认为空）不正确，定义 `CPE_UPDATE`。
7. 如果仍然不正确，请检查 **Mk/Uses/cpe.mk** 以获取更多细节，或联系 Ports 安全团队 \[[ports-secteam@FreeBSD.org](mailto:ports-secteam@FreeBSD.org)]。
8. 尽可能从现有变量（如 `PORTNAME` 和 `PORTVERSION`）中推导出 CPE 名称。使用变量修饰符提取这些变量的相关部分，而不是硬编码名称。
9. *始终*在提交任何更改（如更改 `PORTNAME` 或 `PORTVERSION` 或任何其他用于推导 `CPE_STR` 的变量）之前，运行 `make -V CPE_STR` 并检查输出。

## 17.20. `cran`

可能的参数：（无），`auto-plist`，`compiles`

使用综合 R 存档网络。指定 `auto-plist` 以自动生成 **pkg-plist**。如果 Port 包含需要编译的代码，请指定 `compiles`。

## 17.21. `desktop-file-utils`

可能的参数：（无）

使用来自 [devel/desktop-file-utils](https://cgit.freebsd.org/ports/tree/devel/desktop-file-utils/) 的 update-desktop-database。在不干扰 Port **Makefile** 中任何其他安装后步骤的情况下，将运行额外的安装后步骤。会在 plist 中添加一行 [`@desktop-file-utils`](https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-keywords-desktop-file-utils)。仅在 Port 提供包含 `MimeType` 条目的 `.desktop` 文件时使用此宏。

## 17.22. `desthack`

可能的参数：（无）

更改 GNU configure 的行为，以便在原始软件不支持 `DESTDIR` 时正确处理。

## 17.23. `display`

可能的参数：（无），*ARGS*

设置虚拟显示环境。如果环境变量 `DISPLAY` 未设置，则会将 Xvfb 添加为构建依赖，并将 `CONFIGURE_ENV` 扩展为当前运行的 Xvfb 实例的 Port 号。*ARGS* 参数默认值为 `install`，控制何时启动和停止虚拟显示。

## 17.24. `dos2unix`

可能的参数：（无）

Port 中包含需要转换为 Unix 格式的 DOS 行结束符。可以设置多个变量来控制哪些文件将被转换。默认情况下，将转换 *所有* 文件，包括二进制文件。有关示例，请参见 [简单的自动替换](https://docs.freebsd.org/en/books/porters-handbook/slow-porting/#slow-patch-automatic-replacements)。

* `DOS2UNIX_REGEX`：根据正则表达式匹配文件名。
* `DOS2UNIX_FILES`：匹配字面文件名。
* `DOS2UNIX_GLOB`：根据 glob 模式匹配文件名。
* `DOS2UNIX_WRKSRC`：开始转换的目录，默认为 `${WRKSRC}`。

## 17.25. `drupal`

可能的参数：`7`，`module`，`theme`

自动化安装作为 Drupal 主题或模块的 Port。使用 Port 所期望的 Drupal 版本。例如，`USES=drupal:7,module` 表示该 Port 创建一个 Drupal 7 模块。可以使用 `USES=drupal:7,theme` 指定 Drupal 7 主题。

## 17.26. `ebur128`

可能的参数：（无），`build`，`lib`，`run`，`test`

添加对 [audio/ebur128](https://cgit.freebsd.org/ports/tree/audio/ebur128/) 的依赖。它允许通过使用 **make.conf** 中的 `DEFAULT_VERSIONS` 来透明地依赖 `rust` 或 `legacy` 版本。例如，要使用传统版本，可以在 `DEFAULT_VERSIONS+=ebur128=legacy` 中设置。

当不使用任何参数时，行为与提供 `lib` 参数时相同。其余参数提供相应的依赖类别。

## 17.27. `eigen`

可能的参数：2，3，build（默认），run

添加对 [math/eigen](https://cgit.freebsd.org/ports/tree/math/eigen/) 的依赖。

## 17.28. `electronfix`

可能的参数：`31`，`32`，`33`

为容易移植以二进制形式分发的 Electron 应用程序提供支持。根据使用的参数，添加对 [devel/electron31](https://cgit.freebsd.org/ports/tree/devel/electron31/)，[devel/electron32](https://cgit.freebsd.org/ports/tree/devel/electron32/)，或 [devel/electron33](https://cgit.freebsd.org/ports/tree/devel/electron33/) 的构建和运行时依赖。

该框架提供以下变量，Port 可以设置：

`ELECTRONFIX_SYMLINK_FILES`：要从 Electron 分发版创建符号链接的文件列表。

`ELECTRONFIX_MAIN_EXECUTABLE`：要替换为原始 Electron 二进制文件的主可执行文件名。

## 17.29. `elfctl`

可能的参数：（无），`build`（默认），`stage`

通过设置 `ELF_FEATURES` 设置 ELF 二进制文件功能控制注释。

当没有参数或提供 `build` 参数时，将操作 `BUILD_WRKSRC` 中的二进制文件，`ELF_FEATURES` 中列出的文件相对于 `BUILD_WRKSRC`。当提供 `stage` 参数时，将操作 `STAGEDIR` 中的二进制文件，`ELF_FEATURES` 中列出的文件相对于 `STAGEDIR`。

**示例 5. 使用 `elfctl`**

```makefile
ELF_FEATURES=	featurelist:path/to/file1 \
		featurelist:path/to/file2
```

`featurelist` 的格式描述见 [elfctl(1)](https://man.freebsd.org/cgi/man.cgi?query=elfctl&sektion=1&format=html)。

## 17.30. `elixir`

可能的参数：（无）

为使用 [lang/elixir](https://cgit.freebsd.org/ports/tree/lang/elixir/) 的 Port 提供支持。添加对 [lang/elixir](https://cgit.freebsd.org/ports/tree/lang/elixir/) 的构建和运行时依赖。

框架提供的变量：

`ELIXIR_APP_NAME`：Elixir 应用程序名称，安装在 Elixir 的 lib 目录中。

`ELIXIR_LIB_ROOT`：Elixir 默认库路径。

`ELIXIR_APP_ROOT`：此 Elixir 应用程序的根目录。

`ELIXIR_HIDDEN`：要从代码路径中隐藏的应用程序；通常是 \${PORTNAME}。

`ELIXIR_LOCALE`：构建期间 Elixir 使用的 UTF-8 本地化（任何 UTF-8 本地化都可以）。

`MIX_CMD`：`mix` 命令。

`MIX_COMPILE`：用于编译 Elixir 应用程序的 `mix` 命令。

`MIX_REWRITE`：自动替换 Mix 依赖项与代码路径。

`MIX_BUILD_DEPS`：以类别/portname 格式列出 `BUILD_DEPENDS`（通常称为 Erlang 和 Elixir 中的 "deps"）。

`MIX_RUN_DEPS`：以类别/portname 格式列出 `RUN_DEPENDS`。

`MIX_DOC_DIRS`：要安装到 `DOCSDIR` 的额外文档目录。

`MIX_DOC_FILES`：要安装到 `DOCSDIR` 的额外文档文件（通常是 README.md）。

`MIX_ENV`：Mix 构建的环境（与 `MAKE_ENV` 相同格式）。

`MIX_ENV_NAME`：Mix 构建环境的名称，通常为 "prod"。

`MIX_BUILD_NAME`：输出的构建名称，位于 \_build/ 中，通常是 `${MIX_ENV_NAME}`。

`MIX_TARGET`：Mix 目标的名称，通常为 "compile"。

`MIX_EXTRA_APPS`：要构建的子应用程序列表（如果有）。

`MIX_EXTRA_DIRS`：要安装到 `ELIXIR_APP_ROOT` 的额外目录列表。

`MIX_EXTRA_FILES`：要安装到 `ELIXIR_APP_ROOT` 的额外文件列表。

## 17.31. `emacs`

可能的参数：（无）（默认），`build`，`run`，`noflavors`

为需要 Emacs 的 Port 提供支持。`build` 参数创建对 Emacs 的构建依赖。`run` 参数创建对 Emacs 的运行时依赖。如果 `build` 和 `run` 参数都缺失，则会创建对 Emacs 的构建和运行时依赖。`noflavors` 参数会防止使用 flavors，如果没有对 Emacs 的运行时依赖，则隐含此参数。

可以在 **make.conf** 中定义使用 `USES=emacs` 的 Port 的默认 Emacs flavor。例如，对于 `nox` flavor，可以使用 `DEFAULT_VERSIONS+= emacs=nox`。有效的 flavors 有：`full`，`canna`，`nox`，`wayland`，`devel_full`，`devel_nox`。

可以由 Port 设置的变量：

`EMACS_FLAVORS_EXCLUDE`：不要构建这些 Emacs flavors。如果未定义 `EMACS_FLAVORS_EXCLUDE` 且：

* 有对 Emacs 的运行时依赖
* 未指定 `noflavors` 参数

则假设所有有效的 Emacs flavors。

`EMACS_NO_DEPENDS`：不要添加对 Emacs 的构建或运行时依赖。这将防止使用 flavors，并且不会生成字节码文件作为包的一部分。

Port 可以读取的变量：

`EMACS_CMD`：Emacs 命令的完整路径（例如 **/usr/local/bin/emacs-30.1**）

`EMACS_FLAVOR`：用于依赖的 Emacs flavor（例如 `BUILD_DEPENDS= dash.el${EMACS_PKGNAMESUFFIX}>0:devel/dash@${EMACS_FLAVOR}`）

`EMACS_LIBDIR`：不包括 `${PREFIX}` 的 Emacs 库目录（例如 **share/emacs**）

`EMACS_LIBDIR_WITH_VER`：不包括 `${PREFIX}` 的库目录，包含版本（例如 **share/emacs/30.1**）

`EMACS_MAJOR_VER`：Emacs 主版本（例如 30）

`EMACS_PKGNAMESUFFIX`：`PKGNAMESUFFIX` 用于区分 Emacs flavors

`EMACS_SITE_LISPDIR`：不包括 `${PREFIX}` 的 Emacs site-lisp 目录（例如 **share/emacs/site-lisp**）

`EMACS_VER`：Emacs 版本（例如 30.1）

`EMACS_VERSION_SITE_LISPDIR`：包括版本的目录（例如 **share/emacs/30.1/site-lisp**）

## 17.32. `erlang`

可能的参数：（无），`enc`，`rebar`，`rebar3`

为 [lang/erlang](https://cgit.freebsd.org/ports/tree/lang/erlang/) 添加构建和运行时依赖。根据参数不同，添加额外的构建依赖。`enc` 添加对 [devel/erlang-native-compiler](https://cgit.freebsd.org/ports/tree/devel/erlang-native-compiler/) 的依赖，`rebar` 添加对 [devel/rebar](https://cgit.freebsd.org/ports/tree/devel/rebar/) 的依赖，`rebar3` 添加对 [devel/rebar3](https://cgit.freebsd.org/ports/tree/devel/rebar3/) 的依赖。

此外，Port 可以使用以下变量：

* `ERL_APP_NAME`：Erlang 应用程序名称，安装在 Erlang 的 lib 目录中（去掉版本）
* `ERL_APP_ROOT`：此 Erlang 应用程序的根目录
* `REBAR_CMD`：`rebar` 命令的路径
* `REBAR3_CMD`：`rebar3` 命令的路径
* `REBAR_PROFILE`：Rebar 配置文件
* `REBAR_TARGETS`：Rebar 目标列表（通常是 compile，也可能是 escriptize）
* `ERL_BUILD_NAME`：Rebar3 的构建名称
* `ERL_BUILD_DEPS`：以类别/portname 格式列出的 `BUILD_DEPENDS`
* `ERL_RUN_DEPS`：以类别/portname 格式列出的 `RUN_DEPENDS`
* `ERL_DOCS`：文档文件和目录列表

## 17.33. `fakeroot`

可能的参数：（无）

更改构建系统的一些默认行为，以允许以用户身份安装。有关 `fakeroot` 的更多信息，请参见 [https://wiki.debian.org/FakeRoot](https://wiki.debian.org/FakeRoot)。

## 17.34. `fam`

可能的参数：（无），`fam`，`gamin`

使用文件修改监视器作为库依赖，可以选择使用 [devel/fam](https://cgit.freebsd.org/ports/tree/devel/fam/) 或 [devel/gamin](https://cgit.freebsd.org/ports/tree/devel/gamin/)。最终用户可以设置 `WITH_FAM_SYSTEM` 来指定其首选项。

## 17.35. `firebird`

可能的参数：（无），`25`

添加对 Firebird 数据库客户端库的依赖。

## 17.36. `fonts`

可能的参数：（无），`fc`，`fontsdir`（默认），`none`

添加一个运行时依赖，所需工具用于注册字体。根据不同的参数，添加 `@fc ${FONTSDIR}` 行，`@fontsdir ${FONTSDIR}` 行，或者如果参数是 `none` 则不添加任何行到 plist。`FONTSDIR` 默认为 **\${PREFIX}/share/fonts/\${FONTNAME}**，`FONTNAME` 默认为 `${PORTNAME}`。将 `FONTSDIR` 添加到 `PLIST_SUB` 和 `SUB_LIST`。

## 17.37. `fortran`

可能的参数：`gcc`（默认）

使用 GNU Fortran 编译器。

## 17.38. `fpc`

可能的参数：（无），`run`

为基于 Free Pascal 的 Port 提供支持。它将安装 Free Pascal 编译器和单元。

添加对 [lang/fpc](https://cgit.freebsd.org/ports/tree/lang/fpc/) 的构建依赖。

如果指定了 `run` 参数，则还会添加运行时依赖。

## 17.39. `fuse`

可能的参数：`2`（默认），`3`

该 Port 将依赖 FUSE 库，并根据 FreeBSD 的版本处理对内核模块的依赖。

## 17.40. `gem`

可能的参数：（无），`noautoplist`

处理 RubyGems 的构建。如果使用 `noautoplist`，则不会自动生成 packing list。

这意味着需要使用 `USES=ruby`。

## 17.41. `gettext`

可能的参数：（无）

已弃用。将同时包含 `gettext-runtime` 和 `gettext-tools`。

## 17.42. `gettext-runtime`

可能的参数：（无），`lib`（默认），`build`，`run`

使用 `devel/gettext-runtime`。默认情况下，未指定参数或使用 `lib` 参数时，会添加对 **libintl.so** 的库依赖。`build` 和 `run` 参数分别表示构建时和运行时依赖 **gettext**。

## 17.43. `gettext-tools`

可能的参数：（无），`build`（默认），`run`

使用 `devel/gettext-tools`。默认情况下，未指定参数或使用 `build` 参数时，会注册构建时依赖 **msgfmt**。使用 `run` 参数时，注册运行时依赖。

## 17.44. `ghostscript`

可能的参数：`*X*`，`build`，`run`，`nox11`

可以使用特定版本 *X*。可选版本有 `7`，`8`，`9`，以及 `agpl`（默认）。`nox11` 表示需要使用 `-nox11` 版本的 Port。`build` 和 `run` 分别会添加构建时和运行时的 Ghostscript 依赖。默认情况下，两者都会添加。

## 17.45. `gl`

可能的参数：（无）

提供了一种轻松方式来依赖 GL 组件。组件应该列出在 `USE_GL` 中。可用的组件有：

`egl`：添加对 `graphics/libglvnd` 的 **libEGL.so** 库的依赖。

`gbm`：添加对 `graphics/mesa-libs` 的 **libgbm.so** 库的依赖。

`gl`：添加对 `graphics/libglvnd` 的 **libGL.so** 库的依赖。

`glesv2`：添加对 `graphics/libglvnd` 的 **libGLESv2.so** 库的依赖。

`glew`：添加对 `graphics/glew` 的 **libGLEW\.so** 库的依赖。

`glu`：添加对 `graphics/libGLU` 的 **libGLU.so** 库的依赖。

`glut`：添加对 `graphics/freeglut` 的 **libglut.so** 库的依赖。

`opengl`：添加对 `graphics/libglvnd` 的 **libOpenGL.so** 库的依赖。

## 17.46. `gmake`

可能的参数：（无）

使用 [devel/gmake](https://cgit.freebsd.org/ports/tree/devel/gmake/) 作为构建时依赖，并设置环境以使用 `gmake` 作为构建的默认 `make`。

## 17.47. `gnome`

可能的参数：（无）

提供一种简便的方式来依赖 GNOME 组件。组件应列在 `USE_GNOME` 中。可用的组件有：

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

默认依赖是构建时和运行时依赖，可以通过 `:build` 或 `:run` 来更改。例如：

```makefile
USES=		gnome
USE_GNOME=	gnomemenus3:build intlhack
```

有关更多信息，请参阅 [Using GNOME](https://docs.freebsd.org/en/books/porters-handbook/special/#using-gnome)。

## 17.48. `go`

>**重要**
>
> 不应为 Go 库创建 Ports，更多信息请参阅 [Go Libraries](https://docs.freebsd.org/en/books/porters-handbook/special/#go-libs)。


可能的参数：（无），`N.NN`，`N.NN-devel`，`modules`，`no_targets`，`run`

设置用于构建 Go 软件的默认值和目标。添加对 Go 编译器 Port 的构建依赖，Port 维护者可以设置所需的版本。默认情况下，构建是在 GOPATH 模式下执行的。如果 Go 软件使用模块，可以通过 `modules` 参数启用模块感知模式。`no_targets` 将设置构建环境（如 `GO_ENV`、`GO_BUILDFLAGS`），但跳过创建提取和构建目标。`run` 还将添加对 Go 编译器 Port 的运行时依赖。

构建过程由多个变量控制：

`GO_MODULE` 作为 `go.mod` 中的 `module` 指令指定的应用模块名称。在大多数情况下，这是使用 Go 模块的 Port 所需的唯一变量。

`GO_PKGNAME` 在 GOPATH 模式下构建时的 Go 包名称。这是将在 `${GOPATH}/src` 中创建的目录。如果没有显式设置，并且存在 `GH_SUBDIR` 或 `GL_SUBDIR`，则将从中推断 `GO_PKGNAME`。在模块感知模式下不需要此设置。

`GO_TARGET` 要构建的包。默认值为 `${GO_PKGNAME}`。`GO_TARGET` 也可以是一个元组，形式为 `package:path`，其中 path 可以是一个简单的文件名或从 `${PREFIX}` 开始的完整路径。

`GO_TESTTARGET` 要测试的包。默认值为 `./…`（当前包及所有子包）。

`CGO_CFLAGS` 要传递给 C 编译器的额外 `CFLAGS` 值。

`CGO_LDFLAGS` 要传递给 C 编译器的额外 `LDFLAGS` 值。

`GO_BUILDFLAGS` 要传递给 `go build` 的额外构建参数。

`GO_TESTFLAGS` 要传递给 `go test` 的额外构建参数。

有关使用示例，请参阅 [Building Go Applications](https://docs.freebsd.org/en/books/porters-handbook/special/#using-go)。

## 17.49. `gperf`

可能的参数：（无）

如果 `gperf` 不在基本系统中，添加对 [devel/gperf](https://cgit.freebsd.org/ports/tree/devel/gperf/) 的构建时依赖。

## 17.50. `grantlee`

可能的参数：`5`，`selfbuild`

处理对 Grantlee 的依赖。指定 `5` 以依赖基于 Qt5 的版本，[devel/grantlee5](https://cgit.freebsd.org/ports/tree/devel/grantlee5/)。`selfbuild` 用于 [devel/grantlee5](https://cgit.freebsd.org/ports/tree/devel/grantlee5/) 来获取其版本号。

## 17.51. `groff`

可能的参数：`build`，`run`，`both`

如果基本系统中没有 `groff`，则注册对 [textproc/groff](https://cgit.freebsd.org/ports/tree/textproc/groff/) 的依赖。

## 17.52. `gssapi`

可能的参数：（无），`base`（默认），`heimdal`，`mit`，`flags`，`bootstrap`

处理 GSS-API 消费者所需的依赖项。仅提供 Kerberos 机制的库。默认情况下，或设置为 `base`，使用基本系统中的 GSS-API 库。也可以设置为 `heimdal` 来使用 [security/heimdal](https://cgit.freebsd.org/ports/tree/security/heimdal/)，或设置为 `mit` 来使用 [security/krb5](https://cgit.freebsd.org/ports/tree/security/krb5/)。

当本地 Kerberos 安装不在 `LOCALBASE` 中时，请设置 `HEIMDAL_HOME`（用于 `heimdal`）或 `KRB5_HOME`（用于 `krb5`）到 Kerberos 安装的位置。

这些变量会被导出供 Port 使用：

* `GSSAPIBASEDIR`
* `GSSAPICPPFLAGS`
* `GSSAPIINCDIR`
* `GSSAPILDFLAGS`
* `GSSAPILIBDIR`
* `GSSAPILIBS`
* `GSSAPI_CONFIGURE_ARGS`

`flags` 选项可以与 `base`、`heimdal` 或 `mit` 一起使用，以自动将 `GSSAPICPPFLAGS`、`GSSAPILDFLAGS` 和 `GSSAPILIBS` 添加到 `CFLAGS`、`LDFLAGS` 和 `LDADD` 中。例如，使用 `base,flags`。

`bootstrap` 选项是一个仅供 [security/krb5](https://cgit.freebsd.org/ports/tree/security/krb5/) 和 [security/heimdal](https://cgit.freebsd.org/ports/tree/security/heimdal/) 使用的特殊前缀。例如，使用 `bootstrap,mit`。

**示例 6. 常见用法**

```makefile
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

## 17.53. `gstreamer`

可能的参数：（无）

提供一种简便的方法来依赖 GStreamer 组件。组件应列在 `USE_GSTREAMER` 中。可用的组件有：

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

## 17.54. `guile`

可能的参数：（无），`X.Y`，`flavors`，`build`，`run`，`alias`，`conflicts`

添加对 Guile 的依赖。默认情况下，这是对适当的 `libguile*.so` 的库依赖，除非通过 `build` 和/或 `run` 选项覆盖。`alias` 选项根据需要配置 `BINARY_ALIAS`（请参见 [使用 `BINARY_ALIAS`](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#binary-alias)）。

默认版本通过常规的 `DEFAULT_VERSIONS` 机制设置；如果默认版本不是列表中的版本之一，则使用可用的最新版本。

使用 Guile 的应用程序通常只构建单一版本的 Guile。然而，扩展或库模块应使用 `flavors` 选项，以构建多个版本的 Guile。

更多信息请参见 [使用 Guile](https://docs.freebsd.org/en/books/porters-handbook/special/#using-guile)。

## 17.55. `horde`

可能的参数：（无）

添加对 [devel/pear-channel-horde](https://cgit.freebsd.org/ports/tree/devel/pear-channel-horde/) 的构建时和运行时依赖。如果需要，可以使用 `USE_HORDE_BUILD` 和 `USE_HORDE_RUN` 添加其他 Horde 依赖项。更多信息请参见 [Horde 模块](https://docs.freebsd.org/en/books/porters-handbook/special/#php-horde)。

## 17.56. `iconv`

可能的参数：（无），`lib`，`build`，`patch`，`translit`，`wchar_t`

使用 `iconv` 函数，来自 Port  [converters/libiconv](https://cgit.freebsd.org/ports/tree/converters/libiconv/) 的构建时和运行时依赖，或者来自基本系统的 `iconv`。默认情况下，如果没有指定参数或指定了 `lib` 参数，则表示使用 `iconv` 作为构建时和运行时依赖。`build` 表示仅作为构建时依赖，`patch` 表示仅作为补丁时依赖。如果 Port 使用 `WCHAR_T` 或 `//TRANSLIT` iconv 扩展，请添加相关参数以确保使用正确的 iconv。更多信息请参见 [使用 `iconv`](https://docs.freebsd.org/en/books/porters-handbook/special/#using-iconv)。

## 17.57. `imake`

可能的参数：（无），`env`，`notall`，`noman`

将 [devel/imake](https://cgit.freebsd.org/ports/tree/devel/imake/) 添加为构建时依赖，并在 `configure` 阶段运行 `xmkmf -a`。如果指定了 `env` 参数，则不会设置 `configure` 目标。如果 `-a` 标志对 Port 有问题，请添加 `notall` 参数。如果 `xmkmf` 无法生成 `install.man` 目标，则添加 `noman` 参数。

## 17.58. `java`

可能的参数：（无），`ant`，`build`，`extract`，`run`

如果没有提供参数且未定义 `NO_BUILD`，则默认使用 `USES=java:build,run`。如果定义了 `NO_BUILD`，则使用 `USES=java:run`。如果指定了 `ant` 参数，则 Port 使用 Apache Ant。如果指定了 `build` 参数，则会将 JDK  Port 添加到构建依赖中。如果指定了 `extract` 参数，则会将 JDK  Port 添加到提取依赖中。如果指定了 `run` 参数，则会将 JDK  Port 添加到运行时依赖中。

该框架提供以下变量供 Port 设置：

`JAVA_VERSION` Port 适用的 Java 版本列表，空格分隔。可选的 `+` 可以用来指定版本范围。（允许的值：`8[+]`，`11[+]`，`17[+]`，`18[+]`，`19[+]`，`20[+]`，`21[+]`，`22[+]`）

`JAVA_OS` Port 适用的 JDK 操作系统列表，空格分隔。（允许的值：`native`，`linux`）

`JAVA_VENDOR` Port 适用的 JDK 供应商列表，空格分隔。（允许的值：`openjdk`，`oracle`）

该框架为 Port 提供以下变量以供读取：

`JAVA_PORT` JDK  Port 的名称。（例如 `java/openjdk8`）

`JAVA_PORT_VERSION` JDK  Port 的版本。（例如 `8`）

`JAVA_PORT_OS` JDK  Port 使用的操作系统。（例如 `linux`）

`JAVA_PORT_VENDOR` JDK  Port 的供应商。（例如 `openjdk`）

`JAVA_PORT_OS_DESCRIPTION` JDK  Port 使用的操作系统描述。（例如 `Linux`）

`JAVA_PORT_VENDOR_DESCRIPTION` JDK  Port 供应商的描述。（例如 `OpenJDK BSD Porting Team`）

`JAVA_HOME` JDK 安装目录的路径。（例如 **/usr/local/openjdk8**）

`JAVAC` 使用的 Java 编译器路径。（例如 **/usr/local/openjdk8/bin/javac** 或 **/usr/local/bin/javac**）

`JAR` 使用的 JAR 工具路径。（例如 **/usr/local/openjdk8/bin/jar** 或 **/usr/local/bin/fastjar**）

`APPLETVIEWER` appletviewer 工具的路径。（例如 **/usr/local/linux-jdk1.8.0/bin/appletviewer**）

`JAVA` `java` 可执行文件的路径。用于执行 Java 程序。（例如 **/usr/local/openjdk8/bin/java**）

`JAVADOC` `javadoc` 工具程序的路径。

`JAVAH` `javah` 程序的路径。

`JAVAP` `javap` 程序的路径。

`JAVA_KEYTOOL` `keytool` 工具程序的路径。

`JAVA_N2A` `native2ascii` 工具的路径。

`JAVA_POLICYTOOL` `policytool` 程序的路径。

`JAVA_SERIALVER` `serialver` 工具程序的路径。

`RMIC` RMI stub/skeleton 生成器 `rmic` 的路径。

`RMIREGISTRY` RMI 注册程序的路径，`rmiregistry`。

`RMID` RMI 守护进程程序的路径。

`JAVA_CLASSES` 包含 JDK 类文件的归档路径。在大多数 JDK 中，这是 **\${JAVA\_HOME}/jre/lib/rt.jar**。

`JAVASHAREDIR` 所有共享 Java 资源的基本目录。

`JAVAJARDIR` Port 应安装 JAR 文件的目录。

`JAVALIBDIR` 由其他 Port 安装的 JAR 文件所在的目录。

## 17.59. `jpeg`

可能的参数：`lib` (default, implicit), `build`, `run`

帮助处理对 `jpeg` 的依赖。

如果提供了 `lib` 参数或没有提供参数，则会向 Port 添加库依赖。

如果提供了 `build` 参数，则会向 Port 添加构建时依赖。

如果提供了 `run` 参数，则会向 Port 添加运行时依赖。

如果提供了 `both` 参数，则会向 Port 添加构建时依赖和运行时依赖。

该框架提供了以下可以由 Port 设置的变量：

`JPEG_PORT` 指定要使用的 JPEG 实现。可选值包括：

* [graphics/jpeg-turbo](https://cgit.freebsd.org/ports/tree/graphics/jpeg-turbo/)（默认）
* [graphics/mozjpeg](https://cgit.freebsd.org/ports/tree/graphics/mozjpeg/)

## 17.60. `kde`

可能的参数：`5`

添加对 KDE 组件的依赖。更多信息请参见 [使用 KDE](https://docs.freebsd.org/en/books/porters-handbook/special/#using-kde)。

## 17.61. `kmod`

可能的参数：(none), `debug`

为内核模块 Port 填充模板，当前包括：

* 将 `kld` 添加到 `CATEGORIES`。
* 设置 `SSP_UNSAFE`。
* 如果在 `SRC_BASE` 中找不到内核源代码，则设置 `IGNORE`。
* 默认情况下，将 `KMODDIR` 定义为 **/boot/modules**，并将其添加到 `PLIST_SUB` 和 `MAKE_ENV` 中，安装时创建该目录。如果将 `KMODDIR` 设置为 **/boot/kernel**，则会重写为 **/boot/modules**。这样可以防止在升级内核时破坏包，因为 **/boot/kernel** 会在此过程中重命名为 **/boot/kernel.old**。
* 在安装和卸载时处理内核模块的交叉引用，使用 [`@kld`](https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-keywords-kld)。
* 如果提供了 `debug` 参数，则 Port 可以将调试版本的模块安装到 **KERN\_DEBUGDIR**/**KMODDIR** 中。默认情况下，`KERN_DEBUGDIR` 从 `DEBUGDIR` 复制并设置为 **/usr/lib/debug**。框架将负责创建和删除所需的目录。

## 17.62. `kodi`

可能的参数：(none), `noautoplist`

提供对 [multimedia/kodi](https://cgit.freebsd.org/ports/tree/multimedia/kodi/) 插件的支持。如果提供了 `noautoplist` 参数，则不会自动生成 `plist`。

## 17.63. `lazarus`

可能的参数：(none), `gtk2` (default), `qt5`, `qt6`, `flavors`

为 [editors/lazarus](https://cgit.freebsd.org/ports/tree/editors/lazarus/) 基础的 Port 提供支持。

如果未提供参数或提供了 `gtk2` 参数，则 lazarus-app 将使用 `gtk2` 界面构建，[editors/lazarus](https://cgit.freebsd.org/ports/tree/editors/lazarus/)  Port 将使用 `gtk2` 界面构建。

如果提供了 `qt5` 参数，则 lazarus-app 将使用 `qt5` 界面构建。

如果提供了 `qt6` 参数，则 lazarus-app 将使用 `qt6` 界面构建。

如果提供了 `flavors` 参数，则 lazarus-app 将使用 flavors 特性构建。

如果 Port 不需要自动编译 lazarus 项目文件，则可以定义以下变量：

`NO_LAZBUILD`= `yes`

以下变量可供 Port 使用：

`LAZARUS_PROJECT_FILES`：lpi 文件列表。不能为空。默认值：空

`LAZARUS_DIR`：lazarus 安装目录的路径，默认值：**\${LOCALBASE}/share/lazarus-\${LAZARUS\_VER}**

`LAZBUILD_ARGS`：lazbuild 额外参数。大多数情况下可以是 `-d`。更多信息请参见 [lazbuild(1)](https://man.freebsd.org/cgi/man.cgi?query=lazbuild&sektion=1&format=html)。默认值：空

`LAZARUS_NO_FLAVORS`：不构建这些 lazarus flavors。如果未定义 `LAZARUS_NO_FLAVORS`，则假定构建所有有效的 lazarus flavors。

`WANT_LAZARUS_DEVEL`：如果设置为 `yes`，则使用 [lazarus/devel](https://cgit.freebsd.org/ports/tree/lazarus/devel/) 作为构建依赖。

## 17.64. `ldap`

可能的参数：(none), <版本>, client, server

注册对 [net/openldap](https://cgit.freebsd.org/ports/tree/net/openldap/) 的依赖。如果设置了特定的 `<版本>`（无点号表示法），则使用该版本。否则，它会尝试找到当前安装的版本。如果需要，它会回退到 `bsd.default-versions.mk` 中找到的默认版本。`client` 指定对客户端库的运行时依赖，这是默认行为。`server` 指定对服务器的运行时依赖。

 Port 可以访问以下变量：

`IGNORE_WITH_OPENLDAP`：如果 Port 不支持某个版本的 OpenLDAP，可以定义此变量。

`WITH_OPENLDAP_VER`：用户定义的变量，用于设置 OpenLDAP 版本。

`OPENLDAP_VER`：检测到的 OpenLDAP 版本。

## 17.65. `lha`

可能的参数：(none)

将 `EXTRACT_SUFX` 设置为 `.lzh`

## 17.66. `libarchive`

可能的参数：(none)

注册对 [archivers/libarchive](https://cgit.freebsd.org/ports/tree/archivers/libarchive/) 的依赖。任何依赖 libarchive 的 Port 必须包含 `USES=libarchive`。

## 17.67. `libedit`

可能的参数：(none)

注册对 [devel/libedit](https://cgit.freebsd.org/ports/tree/devel/libedit/) 的依赖。任何依赖 libedit 的 Port 必须包含 `USES=libedit`。

## 17.68. `libtool`

可能的参数：(none), `keepla`, `build`

修补 `libtool` 脚本。所有使用 `libtool` 的 Port 必须添加此项。`keepla` 参数可用于保留 **.la** 文件。一些 Port 没有自己的 libtool 副本，需要对 [devel/libtool](https://cgit.freebsd.org/ports/tree/devel/libtool/) 进行构建时依赖，使用 `:build` 参数添加此依赖。

## 17.69. `linux`

可能的参数：`c6`, `c7`

Linux 兼容性框架 Port。指定 `c6` 以依赖于 CentOS 6 包。指定 `c7` 以依赖于 CentOS 7 包。可用的包有：

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

## 17.70. `llvm`

可能的参数： (无), `XY`, min=`XY`, max=`XY`, build, run, lib

添加对 LLVM 的依赖。默认情况下，这是一个构建依赖，除非使用 `run` 或 `lib` 选项覆盖。默认版本是通过 `LLVM_DEFAULT` 设置的。也可以指定特定版本。最小和最大版本可以分别通过 `min` 和 `max` 参数指定。Ports 框架将以下变量导出到该 Port：

`LLVM_VERSION` 通过 llvm.mk 选择的版本

`LLVM_PORT` 选择的 llvm Port

`LLVM_CONFIG` 选择的 Port 的 llvm-config

`LLVM_LIBLLVM` 选择的 Port 的 libLLVM.so

`LLVM_PREFIX` 选择的 Port 的安装前缀

## 17.71. `localbase`

可能的参数： (无), `ldflags`

确保使用来自 `LOCALBASE` 的库，而不是来自基本系统的库。指定 `ldflags` 可以将 `-L${LOCALBASE}/lib` 添加到 `LDFLAGS`，而不是 `LIBS`。依赖于基本系统中也存在的库的 Ports 应该使用此选项。它也被其他一些 `USES` 内部使用。

## 17.72. `lua`

可能的参数： (无), `XY`, `XY+`, `-XY`, `XY-ZA`, `module`, `flavors`, `build`, `run`, `env`

添加对 Lua 的依赖。默认情况下，这是一个库依赖，除非通过 `build` 和/或 `run` 选项覆盖。`env` 选项防止添加任何依赖，但仍定义所有常规变量。

默认版本由通常的 `DEFAULT_VERSIONS` 机制设置，除非指定了版本或版本范围作为参数，例如 `51` 或 `51-54`。

使用 Lua 的应用程序通常仅为单个 Lua 版本构建。然而，旨在通过 Lua 代码加载的库模块应使用 `module` 选项以构建多个版本。

有关更多信息，请参见 [使用 Lua](https://docs.freebsd.org/en/books/porters-handbook/special/#using-lua)。

## 17.73. `luajit`

可能的参数： (无), `X`

添加对 luajit 运行时的依赖。可以使用特定版本 *X*。可用版本为 `luajit`，`luajit-devel`，`luajit-openresty`。

在包含 **bsd.port.options.mk** 或 **bsd.port.pre.mk** 后，Port 可以检查以下变量：

`LUAJIT_VER` 选择的 luajit 版本

`LUAJIT_INCDIR` luajit 的头文件路径

`LUAJIT_LUAVER` 选择的 luajit 规范版本（`luajit` 为 2.0，其他为 2.1）

有关更多信息，请参见 [使用 Lua](https://docs.freebsd.org/en/books/porters-handbook/special/#using-lua)。

## 17.74. `lxqt`

可能的参数： (无)

处理 LXQt 桌面环境的依赖。使用 `USE_LXQT` 来选择所需的组件。有关更多信息，请参见 [使用 LXQt](https://docs.freebsd.org/en/books/porters-handbook/special/#using-lxqt)。

## 17.75. `magick`

可能的参数： (无), `X`, `build`, `nox11`, `run`, `test`

添加对 `ImageMagick` 的库依赖。可以使用特定版本 *X*。可用版本为 `6` 和 `7`（默认）。`nox11` 表示需要使用 `-nox11` 版本的 Port。`build`，`run` 和 `test` 分别添加对 ImageMagick 的构建、运行时和测试依赖。

## 17.76. `makeinfo`

可能的参数： (无)

如果基本系统中没有 `makeinfo`，则添加一个构建时依赖。

## 17.77. `makeself`

可能的参数： (无)

表示分发文件是 makeself 存档，并设置相应的依赖。

## 17.78. `mate`

可能的参数： (无)

提供一种简单的方法来依赖 MATE 组件。组件应列在 `USE_MATE` 中。可用的组件有：

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

默认的依赖关系是构建和运行时的，可以通过 `:build` 或 `:run` 进行更改。例如：

```makefile
USES=		mate
USE_MATE=	menus:build intlhack
```

## 17.79. `meson`

可能的参数： (无)

为基于 Meson 的项目提供支持。有关更多信息，请参见 [使用 `meson`](https://docs.freebsd.org/en/books/porters-handbook/special/#using-meson)。

## 17.80. `metaport`

可能的参数： (无)

设置以下变量以简化创建 metaport： `MASTER_SITES`，`DISTFILES`，`EXTRACT_ONLY`，`NO_BUILD`，`NO_INSTALL`，`NO_MTREE`，`NO_ARCH`。

## 17.81. `minizip`

可能的参数： (无), `ng`

分别添加对 [archivers/minizip](https://cgit.freebsd.org/ports/tree/archivers/minizip/) 或 [archivers/minizip-ng](https://cgit.freebsd.org/ports/tree/archivers/minizip-ng/) 的库依赖。

## 17.82. `mlt`

可能的参数： `7`, `nodepend`

为依赖 [multimedia/mlt7](https://cgit.freebsd.org/ports/tree/multimedia/mlt7/) 的 Ports 提供支持。

如果提供了 `nodepend` 参数，则不生成库依赖。此参数仅对 multimedia/mlt7\* Ports 有意义。

## 17.83. `mysql`

可能的参数： (无), `version`, `client`（默认）, `server`, `embedded`

为 MySQL 提供支持。如果未指定版本，则尝试查找当前安装的版本。回退到默认版本 MySQL-5.6。可用的版本为 `55`，`55m`，`55p`，`56`，`56p`，`56w`，`57`，`57p`，`80`，`100m`，`101m`，`102m`。其中 `m` 和 `p` 后缀分别表示 MariaDB 和 Percona 变种的 MySQL。`server` 和 `embedded` 会添加对 MySQL 服务器的构建和运行时依赖。在使用 `server` 或 `embedded` 时，添加 `client` 以同时添加对 **libmysqlclient.so** 的依赖。如果某些版本不受支持，Port 可以设置 `IGNORE_WITH_MYSQL`。

该框架将 `MYSQL_VER` 设置为检测到的 MySQL 版本。

## 17.84. `mono`

可能的参数： (无), `nuget`

通过设置适当的依赖，添加对 Mono（当前仅支持 C#）框架的依赖。

当 Port 使用 nuget 包时，指定 `nuget`。需要设置 `NUGET_DEPENDS`，其格式为 `name=version`，包含 nuget 包的名称和版本。可以使用 `name=version:_origin_` 添加可选的包源。

辅助目标 `buildnuget` 将基于提供的 **packages.config** 输出 `NUGET_DEPENDS` 的内容。

## 17.85. `motif`

可能的参数： (无)

使用 [x11-toolkits/open-motif](https://cgit.freebsd.org/ports/tree/x11-toolkits/open-motif/) 作为库依赖。最终用户可以在 **make.conf** 中设置 `WANT_LESSTIF` 来使用 [x11-toolkits/lesstif](https://cgit.freebsd.org/ports/tree/x11-toolkits/lesstif/) 作为依赖，而不是 [x11-toolkits/open-motif](https://cgit.freebsd.org/ports/tree/x11-toolkits/open-motif/)。同样，在 **make.conf** 中设置 `WANT_OPEN_MOTIF_DEVEL` 将添加对 [x11-toolkits/open-motif-devel](https://cgit.freebsd.org/ports/tree/x11-toolkits/open-motif-devel/) 的依赖。

## 17.86. `mpi`

可能的参数： `mpich`（默认）, `openmpi`

为依赖 `MPI` 的 Ports 提供支持。

如果提供 `mpich` 参数，则添加对 [net/mpich](https://cgit.freebsd.org/ports/tree/net/mpich/) 的依赖。

如果提供 `openmpi` 参数，则添加对 [net/openmpi](https://cgit.freebsd.org/ports/tree/net/openmpi/) 的依赖。

Ports 框架提供以下变量，Port 可以读取：

`MPI_LIBS` 用于链接使用 `MPI` 的程序所需的库。

`MPI_CFLAGS` 编译器标志，用于构建使用 `MPI` 的程序。

`MPICC` `mpicc` 可执行文件的位置。默认： **\${MPI\_HOME}/bin/mpicc**。

`MPICXX` `mpicxx` 可执行文件的位置。默认： **\${MPI\_HOME}/bin/mpicxx**。

`MPIF90` `mpif90` 可执行文件的位置。默认： **\${MPI\_HOME}/bin/mpif90**。

`MPIFC` 与上述相同。

`MPI_HOME` `MPI` 的安装目录。对于 `MPICH` 默认值为 `${LOCALBASE}`。

`MPIEXEC` `mpiexec` 可执行文件的位置。默认： **\${MPI\_HOME}/bin/mpiexec**。

`MPIRUN` `mpirun` 可执行文件的位置。默认： **\${MPI\_HOME}/bin/mpirun**。

## 17.87. `ncurses`

可能的参数： (无), `base`, `port`

使用 ncurses，并设置一些有用的变量。

## 17.88. `nextcloud`

可能的参数： (无)

通过添加对 [www/nextcloud](https://cgit.freebsd.org/ports/tree/www/nextcloud/) 的运行时依赖，添加对 Nextcloud 应用程序的支持。

## 17.89. `ninja`

可能的参数： (无), `build`, `make`（默认）, `run`

如果指定了 `build` 或 `run` 参数，则分别添加对 [devel/ninja](https://cgit.freebsd.org/ports/tree/devel/ninja/) 的构建或运行时依赖。如果提供 `make` 或没有参数，则使用 ninja 来构建该 Port，而不是 make。`make` 隐含了 `build`。如果 `NINJA_DEFAULT` 变量设置为 `samurai`，则将依赖关系设置为 [devel/samurai](https://cgit.freebsd.org/ports/tree/devel/samurai/)。

## 17.90. `nodejs`

可能的参数： (无), `build`, `run`, `current`, `lts`, `10`, `14`, `16`, `17`

使用 nodejs。添加对 [www/node\*](https://cgit.freebsd.org/ports/tree/www/node*/) 的依赖。如果指定了受支持的版本，则必须同时指定 `run` 和/或 `build`。

## 17.91. `objc`

可能的参数： (无)

如果基本系统不支持，则添加 Objective C 依赖（编译器、运行时库）。

## 17.92. `ocaml`

可能的参数： (无)、`build`、`camlp4`、`dune`、`findlib`、`findplist`、`ldconfig`、`run`、`tk`、`tkbuild`、`tkrun`、`wash`

提供对 OCaml 的支持。

如果没有提供任何参数，默认使用 `build` 和 `run`。

如果提供 `build` 参数，则会将 [lang/ocamlc](https://cgit.freebsd.org/ports/tree/lang/ocamlc/) 添加到 `BUILD_DEPENDS`、`EXTRACT` 和 `PATCH_DEPENDS`。

如果提供 `camlp4` 参数，则使用 [devel/ocamlp4](https://cgit.freebsd.org/ports/tree/devel/ocamlp4/) 来构建。

如果提供 `dune` 参数，则使用 [devel/ocaml-dune](https://cgit.freebsd.org/ports/tree/devel/ocaml-dune/) 作为构建系统。

如果提供 `findlib` 参数，则使用 `ocamlfind` 来安装包。包目录会自动删除。

如果提供 `findplist` 参数，则会自动添加 `findlib` 目标目录的内容。

如果提供 `ldconfig` 参数，则会自动处理 OCaml 的 **ld.conf** 文件。当使用 `dune` 时，Dune 可能会将 stublibs 安装到 site-lib 包目录或在 `DUNE_LIBDIR` site-lib 目录下的单一目录中。如果你的 port 安装了共享库到 OCaml 目录，请设置此项。

如果提供 `run` 参数，则将 `ocamlc` 添加到 `RUN_DEPENDS`。

如果提供 `tk` 参数，则会添加 [x11-toolkits/ocaml-labltk](https://cgit.freebsd.org/ports/tree/x11-toolkits/ocaml-labltk/) 的构建和运行时依赖。此参数隐式包含 `tkbuild` 和 `tkrun`。

如果提供 `tkbuild` 参数，则会将 [x11-toolkits/ocaml-labltk](https://cgit.freebsd.org/ports/tree/x11-toolkits/ocaml-labltk/) 添加到 `BUILD_DEPENDS`、`EXTRACT` 和 `PATCH_DEPENDS`。

如果提供 `tkrun` 参数，则会将 [x11-toolkits/ocaml-labltk](https://cgit.freebsd.org/ports/tree/x11-toolkits/ocaml-labltk/) 添加到 `RUN_DEPENDS`。

如果提供 `wash` 参数，则会在卸载时清理 Ocaml 的共享目录。当安装到非标准 `PREFIX` 时，使用此选项。

可以由 port 设置以下变量：

`OCAML_PKGDIRS`：如果指定了 `findlib` 参数，将处理 site-lib 下的目录。默认值：`${PORTNAME}`

`OCAML_LDLIBS`：会自动添加/删除从 **ld.conf** 中的目录。默认值：`${OCAML_SITELIBDIR}/${PORTNAME}`

`OCAML_PACKAGES`：要构建和安装的包列表。默认值：`${PORTNAME}`

## 17.93. `octave`

可能的参数： (无)、`env`

使用 [math/octave](https://cgit.freebsd.org/ports/tree/math/octave/)。`env` 仅加载一个 `OCTAVE_VERSION` 环境变量。

## 17.94. `openal`

可能的参数：`al`、`soft`（默认）、`si`、`alut`

使用 OpenAL。可以指定后端，默认使用软件实现。用户可以通过设置 `WANT_OPENAL` 来指定首选后端。有效的值包括 `soft`（默认）和 `si`。

## 17.95. `pathfix`

可能的参数： (无)

查找 `PATHFIX_WRKSRC` 中的 **Makefile.in** 和 **configure** 文件（默认为 `WRKSRC`），并修复常见路径，以确保它们遵循 FreeBSD 的层次结构。例如，它会修复 `pkgconfig` 的 `.pc` 文件的安装目录为 `${PREFIX}/libdata/pkgconfig`。如果 port 使用了 `USES=autoreconf`，则会自动将 **Makefile.am** 添加到 `PATHFIX_MAKEFILEIN` 中。

如果 port 使用了 [USES=cmake](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-cmake)，则会查找 `PATHFIX_WRKSRC` 中的 **CMakeLists.txt**。如有需要，可以通过 `PATHFIX_CMAKELISTSTXT` 更改默认文件名。

## 17.96. `pear`

可能的参数：`env`

添加对 [devel/pear](https://cgit.freebsd.org/ports/tree/devel/pear/) 的依赖。它将为使用 PHP 扩展和应用程序仓库的软件设置默认行为。使用 `env` 参数仅设置 PEAR 环境变量。有关更多信息，请参见 [PEAR 模块](https://docs.freebsd.org/en/books/porters-handbook/special/#php-pear)。

## 17.97. `perl5`

可能的参数： (无)

依赖于 Perl。配置通过 `USE_PERL5` 进行。

`USE_PERL5` 可以包含使用 Perl 的阶段，包括 `extract`、`patch`、`build`、`run` 或 `test`。

`USE_PERL5` 还可以包含 `configure`、`modbuild` 或 `modbuildtiny`，当需要 **Makefile.PL**、**Build.PL** 或 Module::Build::Tiny 的版本时。

`USE_PERL5` 默认为 `build run`。当使用 `configure`、`modbuild` 或 `modbuildtiny` 时，`build` 和 `run` 是隐式的。

有关更多信息，请参见 [使用 Perl](https://docs.freebsd.org/en/books/porters-handbook/special/#using-perl)。

## 17.98. `pgsql`

可能的参数： (无)、`X.Y`、`X.Y+`、`X.Y-`、`X.Y-Z.A`

提供对 PostgreSQL 的支持。Port 维护者可以设置所需的版本。可以指定最小版本、最大版本或版本范围；例如，`9.0-`、`8.4+`、`8.4-9.2.`

默认情况下，添加的依赖项将是客户端，但如果 port 需要其他组件，可以通过 `WANT_PGSQL=component[:target]` 来指定。例如，`WANT_PGSQL=server:configure pltcl plperl`。可用组件包括：

* `client`
* `contrib`
* `docs`
* `pgtcl`
* `plperl`
* `plpython`
* `pltcl`
* `server`

## 17.99. `php`

可能的参数： (无)、`phpize`、`ext`、`zend`、`build`、`cli`、`cgi`、`mod`、`web`、`embed`、`pecl`、`flavors`、`noflavors`

提供对 PHP 的支持。为默认的 PHP 版本 [lang/php81](https://cgit.freebsd.org/ports/tree/lang/php81/) 添加运行时依赖。

`phpize`：用于构建 PHP 扩展。启用 flavors。

`ext`：用于构建、安装和注册 PHP 扩展。启用 flavors。

`zend`：用于构建、安装和注册 Zend 扩展。启用 flavors。

`build`：将 PHP 也作为构建时的依赖项。

`cli`：需要 PHP 的 CLI 版本。

`cgi`：需要 PHP 的 CGI 版本。

`mod`：需要 PHP 的 Apache 模块。

`web`：需要 PHP 的 Apache 模块或 CGI 版本。

`embed`：需要 PHP 的嵌入式库版本。

`pecl`：提供从 PECL 仓库获取 PHP 扩展的默认配置。启用 flavors。

`flavors`：启用自动生成 PHP flavors。除非出现在 [`IGNORE_WITH_PHP`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-php-ignore) 中的 PHP 版本，否则将为所有 PHP 版本生成 flavors。

`noflavors`：禁用自动生成 PHP flavors。*仅*与 PHP 本身提供的扩展一起使用。

使用变量来指定所需的 PHP 模块，以及支持的 PHP 版本。

`USE_PHP`：运行时所需的 PHP 扩展列表。可以通过在扩展名称后添加 `:build` 来添加构建时依赖项。例如：`pcre xml:build gettext`。

`IGNORE_WITH_PHP`：该 port 不与给定版本的 PHP 一起使用。有关可能的值，请查看 **Mk/Uses/php.mk** 中的 `_ALL_PHP_VERSIONS` 内容。

在使用 `:ext` 或 `:zend` 构建 PHP 或 Zend 扩展时，可以设置以下变量：

`PHP_MODNAME`：PHP 或 Zend 扩展的名称。默认值是 `${PORTNAME}`。

`PHP_HEADER_DIRS`：安装头文件的子目录列表。框架将始终安装与扩展位于同一目录中的头文件。

`PHP_MOD_PRIO`：加载扩展的优先级。它是一个介于 `00` 和 `99` 之间的数字。

对于不依赖其他扩展的扩展，优先级自动设置为 `20`，对于依赖其他扩展的扩展，优先级自动设置为 `30`。一些扩展可能需要在其他扩展之前加载，例如 [www/php56-opcache](https://cgit.freebsd.org/ports/tree/www/php56-opcache/)。一些可能需要在优先级为 `30` 的扩展之后加载。在这种情况下，可以在 port 的 Makefile 中添加 `PHP_MOD_PRIO=XX`。例如：

```makefile
USES=		php:ext
USE_PHP=	wddx
PHP_MOD_PRIO=	40
```

这些变量可以用于 `PKGNAMEPREFIX` 或 `PKGNAMESUFFIX`：

`PHP_PKGNAMEPREFIX`：包含 `php_XY_-`，其中 *XY* 是当前 flavor 的 PHP 版本。用于 PHP 扩展和模块。

`PHP_PKGNAMESUFFIX`：包含 `-php_XY_`，其中 *XY* 是当前 flavor 的 PHP 版本。用于 PHP 应用程序。

`PECL_PKGNAMEPREFIX`：包含 `php_XY_-pecl-`，其中 *XY* 是当前 flavor 的 PHP 版本。用于 PECL 模块。

>**重要**
>
>使用 flavors 时，所有 PHP 扩展、PECL 扩展、PEAR 模块 *必须*具有不同的包名称，因此它们必须在 `PKGNAMEPREFIX` 或 `PKGNAMESUFFIX` 中使用这三个变量之一。

## 17.100. `pkgconfig`

可能的参数： (无)、`build`（默认）、`run`、`both`

使用 [devel/pkgconf](https://cgit.freebsd.org/ports/tree/devel/pkgconf/)。如果没有参数或使用 `build` 参数，则意味着将 `pkg-config` 作为构建时的依赖项。`run` 表示运行时依赖项，`both` 表示构建时和运行时的依赖项。

## 17.101. `pure`

可能的参数： (无)、`ffi`

使用 [lang/pure](https://cgit.freebsd.org/ports/tree/lang/pure/)。主要用于构建相关的 pure port。使用 `ffi` 参数时，它意味着将 [devel/pure-ffi](https://cgit.freebsd.org/ports/tree/devel/pure-ffi/) 作为运行时依赖项。

## 17.102. `pyqt`

可能的参数： (无)、`4`、`5`

使用 PyQt。如果 port 是 PyQt 的一部分，请设置 `PYQT_DIST`。使用 `USE_PYQT` 来选择 port 需要的组件。可用的组件包括：

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

这些组件仅在 PyQt4 中可用：

* `assistant`
* `declarative`
* `help`
* `phonon`
* `script`
* `scripttools`

这些组件仅在 PyQt5 中可用：

* `multimediawidgets`
* `printsupport`
* `qml`
* `serialport`
* `webkitwidgets`
* `widgets`

每个组件的默认依赖项是构建时和运行时的依赖项，要选择仅构建或仅运行，请在组件名称后添加 `_build` 或 `_run`。例如：

```makefile
USES=		pyqt
USE_PYQT=	core doc_build designer_run
```

## 17.103. `pytest`

可能的参数： (无)、4

引入了对 [devel/pytest](https://cgit.freebsd.org/ports/tree/devel/pytest/) 的新依赖。它定义了一个 `do-test` 目标，用于正确运行测试。使用该参数依赖于特定版本的 [devel/pytest](https://cgit.freebsd.org/ports/tree/devel/pytest/)。对于使用 [devel/pytest](https://cgit.freebsd.org/ports/tree/devel/pytest/) 的 port，建议使用此方式，而不是使用特定的 `do-test` 目标。框架向 port 暴露了以下变量：

`PYTEST_ARGS`：传递给 pytest 的额外参数（默认为空）。

`PYTEST_IGNORED_TESTS`：列出需要忽略的 `pytest -k` 测试模式（默认为空）。适用于那些预期不通过的测试，比如需要数据库访问的测试。

`PYTEST_BROKEN_TESTS`：列出需要忽略的 `pytest -k` 损坏测试模式（默认为空）。适用于那些有问题的测试，需要修复。

此外，用户可以设置以下变量：

`PYTEST_ENABLE_IGNORED_TESTS`：启用那些通常会被 `PYTEST_IGNORED_TESTS` 忽略的测试。

`PYTEST_ENABLE_BROKEN_TESTS`：启用那些通常会被 `PYTEST_BROKEN_TESTS` 忽略的测试。

`PYTEST_ENABLE_ALL_TESTS`：启用所有那些通常会被 `PYTEST_IGNORED_TESTS` 和 `PYTEST_BROKEN_TESTS` 忽略的测试。


## 17.104. `python`

可能的参数： (无)、`X.Y`、`X.Y+`、`-X.Y`、`X.Y-Z.A`、`patch`、`build`、`run`、`test`

使用 Python。可以指定支持的版本或版本范围。如果 Python 只在构建时、运行时或测试时需要，可以将其设置为构建、运行或测试依赖项，使用 `build`、`run` 或 `test`。如果 Python 在补丁阶段也需要，则使用 `patch`。更多信息请参见 [使用 Python](https://docs.freebsd.org/en/books/porters-handbook/special/#using-python)。

`USES=python:env` 可以在需要框架导出的变量但不依赖于 Python 时使用。通常用于与 [`USES=shebangfix`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-shebangfix) 配合使用，目标是仅修复 shebang，而不是添加对 Python 的依赖。

## 17.105. `qmail`

可能的参数： (无)、`build`、`run`、`both`、`vars`

使用 [mail/qmail](https://cgit.freebsd.org/ports/tree/mail/qmail/)。使用 `build` 参数时，意味着将 `qmail` 作为构建时的依赖项。`run` 表示运行时依赖项。如果不带参数或使用 `both` 参数，则表示构建时和运行时的依赖项。`vars` 将仅设置 QMAIL 变量供 port 使用。

## 17.106. `qmake`

可能的参数： (无)、`norecursive`、`outsource`、`no_env`、`no_configure`

使用 QMake 进行配置。更多信息请参见 [使用 `qmake`](https://docs.freebsd.org/en/books/porters-handbook/special/#using-qmake)。

## 17.107. `qt`

可能的参数： `5`、`6`、`no_env`

添加对 Qt 组件的依赖。`no_env` 参数直接传递给 `USES= qmake`。更多信息请参见 [使用 Qt](https://docs.freebsd.org/en/books/porters-handbook/special/#using-qt)。

## 17.108. `qt-dist`

可能的参数： (无) 或 `5` 和 (无) 或 `6` 和 (无) 或下列之一： `3d`、`5compat`、`base`、`charts`、`connectivity`、`datavis3d`、`declarative`、`doc`、`languageserver`、`gamepad`、`graphicaleffects`、`imageformats`、`location`、`lottie`、`multimedia`、`networkauth`、`positioning`、`quick3d`、`quickcontrols2`、`quickcontrols`、`quicktimeline`、`remoteobjects`、`script`、`scxml`、`sensors`、`serialbus`、`serialport`、`shadertools`、`speech`、`svg`、`tools`、`translations`、`virtualkeyboard`、`wayland`、`webchannel`、`webengine`、`webglplugin`、`websockets`、`webview`、`x11extras`、`xmlpatterns`。

提供构建 Qt 5 和 Qt 6 组件的支持。它负责为 port 构建设置适当的配置环境。

**示例 7. 构建 Qt 5 组件**

该 port 是 Qt 5 的 `networkauth` 组件，属于 `networkauth` 分发文件。

```makefile
PORTNAME=	networkauth
DISTVERSION=	${QT5_VERSION}

USES=		qt-dist:5
```

**示例 8. 构建 Qt 6 组件**

该 port 是 Qt 6 的 `websockets` 组件，属于 `websockets` 分发文件。

```makefile
PORTNAME=       websockets
PORTVERSION=    ${QT6_VERSION}

USES=           qt-dist:6
```

如果 `PORTNAME` 与组件名称不匹配，则可以将其作为参数传递给 `qt-dist`。

**示例 9. 使用不同名称构建 Qt 5 组件**

该 port 是 Qt 5 的 `gui` 组件，属于 `base` 分发文件。

```makefile
PORTNAME=	gui
DISTVERSION=	${QT5_VERSION}

USES=		qt-dist:5,base
```

## 17.109. `readline`

可能的参数： (无)、`port`

将 readline 作为库依赖项，并根据需要设置 `CPPFLAGS` 和 `LDFLAGS`。如果使用 `port` 参数，或如果基本系统中没有 readline，则会添加对 [devel/readline](https://cgit.freebsd.org/ports/tree/devel/readline/) 的依赖。

## 17.110. `ruby`

可能的参数： (无)、`build`、`extconf`、`run`、`setup`

为 Ruby 相关的 Port 提供支持。`(无)` 参数时，会添加对 [lang/ruby](https://cgit.freebsd.org/ports/tree/lang/ruby/) 的运行时依赖。`build` 添加对 [lang/ruby](https://cgit.freebsd.org/ports/tree/lang/ruby/) 的构建时依赖。`extconf` 表示该 port 使用 `extconf.rb` 来配置。`run` 添加对 [lang/ruby](https://cgit.freebsd.org/ports/tree/lang/ruby/) 的运行时依赖，这是默认设置。`setup` 表示该 port 使用 `setup.rb` 来配置和构建。

用户可以定义以下变量：

`RUBY_VER`Ruby 的替代短版本，以 `x.y` 形式表示。

`RUBY_DEFAULT_VER` 设置为（例如）`2.7`，则默认使用 `ruby27`。

`RUBY_ARCH` 设置架构名称（例如 `i386-freebsd7`）。

以下变量会被导出供 Port 使用：

`RUBY` 设置为 ruby 的完整路径。如果设置了此变量，则以下变量的值将自动从 ruby 执行文件中获得：`RUBY_ARCH`、`RUBY_ARCHLIBDIR`、`RUBY_LIBDIR`、`RUBY_SITEARCHLIBDIR`、`RUBY_SITELIBDIR`、`RUBY_VER` 和 `RUBY_VERSION`。

`RUBY_VER` 设置为 Ruby 的替代短版本，以 `x.y` 形式表示。

`RUBY_EXTCONF` 设置为 `extconf.rb` 的替代名称（默认：`extconf.rb`）。

`RUBY_EXTCONF_SUBDIRS` 设置为子目录列表，如果包含多个模块。

`RUBY_SETUP` 设置为 `setup.rb` 的替代名称（默认：`setup.rb`）。

## 17.111. `samba`

可能的参数： `build`、`env`、`lib`、`run`

处理对 Samba 的依赖。`env` 不会添加任何依赖项，仅设置变量。`build` 和 `run` 分别会添加构建时和运行时对 **smbd** 的依赖。`lib` 会添加对 **libsmbclient.so** 的依赖。导出的变量包括：

`SAMBA_PORT` Samba 默认 port 的来源。

`SAMBA_INCLUDEDIR` Samba 头文件的位置。

`SAMBA_LIBS` Samba 共享库所在目录。

`SAMBA_LDB_PORT` 选定 Samba 版本使用的 ldb port 的来源（例如， [databases/ldb28](https://cgit.freebsd.org/ports/tree/databases/ldb28/)）。如果一个 port 需要依赖于与选定 Samba 版本相同的 ldb 版本，应该使用此变量。

`SAMBA_TALLOC_PORT` 选定 Samba 版本使用的 talloc port 的来源。如果一个 port 需要依赖于与选定 Samba 版本相同的 talloc 版本，应该使用此变量。

`SAMBA_TDB_PORT` 选定 Samba 版本使用的 TDB port 的来源。如果一个 port 需要依赖于与选定 Samba 版本相同的 TDB 版本，应该使用此变量。

`SAMBA_TEVENT_PORT` 选定 Samba 版本使用的 tevent port 的来源。如果一个 port 需要依赖于与选定 Samba 版本相同的 tevent 版本，应该使用此变量。

## 17.112. `scons`

可能的参数： (无)

为使用 [devel/scons](https://cgit.freebsd.org/ports/tree/devel/scons/) 提供支持。更多信息请参见 [使用 `scons`](https://docs.freebsd.org/en/books/porters-handbook/special/#using-scons)。

## 17.113. `sdl`

可能的参数： `sdl`

为使用 `SDL` 软件包提供支持。变量 `USE_SDL` 是强制性的，并指定要添加为依赖的组件。

当前支持的 `SDL1.2` 模块有：

* sdl
* console
* gfx
* image
* mixer
* mm
* net
* pango
* sound
* ttf

当前支持的 `SDL2` 模块有：

* sdl2
* gfx2
* image2
* mixer2
* net2
* sound2
* ttf2

当前支持的 `SDL3` 模块有：

* sdl3
* image3
* ttf3

## 17.114. `shared-mime-info`

可能的参数： (无)

使用来自 [misc/shared-mime-info](https://cgit.freebsd.org/ports/tree/misc/shared-mime-info/) 的 `update-mime-database`。此宏会自动在安装后步骤中添加，以确保该 port 在需要时仍能指定自己的安装后步骤。同时，它还会将 [`@shared-mime-info`](https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-keywords-shared-mime-info) 条目添加到 plist 中。

## 17.115. `shebangfix`

可能的参数： (无)

许多软件使用错误的脚本解释器位置，特别是 **/usr/bin/perl** 和 **/bin/bash**。`shebangfix` 宏修复脚本中的 shebang 行，这些脚本包含在 `SHEBANG_REGEX`、`SHEBANG_GLOB` 或 `SHEBANG_FILES` 中。

`SHEBANG_REGEX` 包含一个扩展正则表达式，并与 [find(1)](https://man.freebsd.org/cgi/man.cgi?query=find&sektion=1&format=html) 的 `-iregex` 参数一起使用。参见 [`USESshebangfix` 与 `SHEBANG_REGEX`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-shebangfix-ex-regex)。

`SHEBANG_GLOB` 包含一个使用 [find(1)](https://man.freebsd.org/cgi/man.cgi?query=find&sektion=1&format=html) 的 `-name` 参数的模式列表。参见 [`USESshebangfix` 与 `SHEBANG_GLOB`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-shebangfix-ex-glob)。

`SHEBANG_FILES` 包含文件列表或 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 模式。`shebangfix` 宏在 `${WRKSRC}` 中运行，因此 `SHEBANG_FILES` 可以包含相对于 `${WRKSRC}` 的路径。如果需要修补 `${WRKSRC}` 之外的文件，也可以处理绝对路径。参见 [`USESshebangfix` 与 `SHEBANG_FILES`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-shebangfix-ex-files)。

当前默认支持的解释器包括：Bash、Java、Ksh、Lua、Perl、PHP、Python、Ruby、Tcl 和 Tk。

有三个配置变量：

`SHEBANG_LANG` 支持的解释器列表。

`_interp__CMD` 在 FreeBSD 上的命令解释器路径。默认值是 `${LOCALBASE}/bin/<interp>`。

`_interp__OLD_CMD` 错误的解释器调用列表。这些通常是过时的路径，或在 FreeBSD 上不正确的路径，它们将被 `_interp__CMD` 中的正确路径所替换。

>**注意**
>
> 这些路径*始终*是 `_interp__OLD_CMD` 的一部分：`"/usr/bin/env _interp"` /bin/interp `/usr/bin/interp` `/usr/local/bin/interp`。


>**技巧**
>
> `_interp__OLD_CMD` 包含多个值。任何包含空格的条目必须加引号。参见 [指定为 `USESshebangfix` 添加解释器时的所有路径](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-shebangfix-ex-ksh)。


>**重要**
>
> shebang 修复在 `patch` 阶段完成。如果在 `build` 阶段创建了错误的 shebang（例如，`configure` 脚本或 `Makefiles`），则必须修补或给定正确的路径（例如，通过 `CONFIGURE_ENV`、`CONFIGURE_ARGS`、`MAKE_ENV` 或 `MAKE_ARGS`）来生成正确的 shebang。
>
>支持的解释器的正确路径可以在 `_interp__CMD` 中找到。

>**技巧**
>
> 当与 [`USES=python`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-python) 一起使用时，如果仅仅是修复 shebang，但不希望依赖于 Python 本身，可以使用 `USES=python:env`。

**示例 10. 向 `USES=shebangfix` 添加另一个解释器**

要添加另一个解释器，设置 `SHEBANG_LANG`。例如：

```makefile
SHEBANG_LANG=	lua
```

**示例 11. 在向 `USES=shebangfix` 添加解释器时指定所有路径**

如果未定义，并且 `_interpOLD_CMD` 和 `_interp`\*`CMD` 没有默认值，则 Ksh 条目可以定义如下：

```makefile
SHEBANG_LANG=	ksh
ksh_OLD_CMD=	"/usr/bin/env ksh" /bin/ksh /usr/bin/ksh
ksh_CMD=	${LOCALBASE}/bin/ksh
```

**示例 12. 添加一个解释器的奇怪位置**

有些软件使用解释器的奇怪位置。例如，应用程序可能期望 Python 位于 **/opt/bin/python2.7**。可以在 port **Makefile** 中声明需要替换的奇怪路径：

```makefile
python_OLD_CMD=	/opt/bin/python2.7
```

**示例 13. `USES=shebangfix` 与 `SHEBANG_REGEX`**

要修复 `${WRKSRC}/scripts` 中所有以 **.pl**、**.sh** 或 **.cgi** 结尾的文件，可以执行：

```makefile
USES=	shebangfix
SHEBANG_REGEX=	./scripts/.*\.(sh|pl|cgi)
```

>**注意**
>
>`SHEBANG_REGEX` 是通过运行 `find -E` 来使用的，它使用现代正则表达式，也称为扩展正则表达式。更多信息参见 [re\_format(7)](https://man.freebsd.org/cgi/man.cgi?query=re_format&sektion=7&format=html)。

**示例 14. `USES=shebangfix` 与 `SHEBANG_GLOB`**

要修复 `${WRKSRC}` 中所有以 **.pl** 或 **.sh** 结尾的文件，可以执行：

```makefile
USES=	shebangfix
SHEBANG_GLOB=	*.sh *.pl
```

**示例 15. `USES=shebangfix` 与 `SHEBANG_FILES`**

要修复 `${WRKSRC}` 中的 **script/foobar.pl** 和 **script/\*.sh** 文件，可以执行：

```makefile
USES=	shebangfix
SHEBANG_FILES=	scripts/foobar.pl scripts/*.sh
```

## 17.116. `sqlite`

可能的参数： (无)， `2`， `3`

添加对 SQLite 的依赖。默认使用版本 3，但也可以使用 `:2` 修饰符来指定版本 2。

## 17.117. `sbrk`

可能的参数： (无)

标记该 port 在 `aarch64` 和 `riscv64` 上为 `BROKEN`。

## 17.118. `ssl`

可能的参数： (无)， `build`， `run`

提供对 OpenSSL 的支持。可以使用 `build` 或 `run` 来指定仅在构建时或运行时需要的依赖。这些变量对 port 可用，也会被添加到 `MAKE_ENV` 中：

`OPENSSLBASE` OpenSSL 安装基础路径。

`OPENSSLDIR` OpenSSL 配置文件路径。

`OPENSSLLIB` OpenSSL 库路径。

`OPENSSLINC` OpenSSL 包含文件路径。

`OPENSSLRPATH` 如果定义，链接器需要使用的 OpenSSL 库路径。


>**技巧**
>
>如果一个 port 在使用 OpenSSL flavor 时无法构建，请设置 `BROKEN_SSL` 变量，并可能设置 `BROKEN_SSL_REASON__flavor_`：
>
>```makefile
>BROKEN_SSL=	libressl
>BROKEN_SSL_REASON_libressl=	需要 OpenSSL 中才有的特性
>```

## 17.119. `tar`

可能的参数：（无），`Z`，`bz2`，`bzip2`，`lzma`，`tbz`，`tbz2`，`tgz`，`txz`，`xz`，`zst`，`zstd`

将 `EXTRACT_SUFX` 设置为 `.tar`，`.tar.Z`，`.tar.bz2`，`.tar.bz2`，`.tar.lzma`，`.tbz`，`.tbz2`，`.tgz`，`.txz`，`.tar.xz`，`.tar.zst` 或 `.tar.zstd`。

## 17.120. `tcl`

可能的参数：*version*，`wrapper`，`build`，`run`，`tea`

添加对 Tcl 的依赖。可以使用 *version* 来请求特定版本。版本可以为空、一个或多个精确版本号（当前为 `84`、`85` 或 `86`），或一个最小版本号（当前为 `84+`、`85+` 或 `86+`）。如果只请求一个非版本特定的 wrapper，可以使用 `wrapper`。可以使用 `build` 或 `run` 来指定构建时或运行时的依赖。要使用 Tcl 扩展架构构建该 Port，使用 `tea`。在包含 **bsd.port.pre.mk** 后，该 Port 可以使用以下变量检查结果：

* `TCL_VER`：选择的 Tcl 的主次版本
* `TCLSH`：Tcl 解释器的完整路径
* `TCL_LIBDIR`：Tcl 库的路径
* `TCL_INCLUDEDIR`：Tcl C 头文件的路径
* `TCL_PKG_LIB_PREFIX`：根据 TIP595 的库前缀
* `TCL_PKG_STUB_POSTFIX`：Stub 库后缀
* `TK_VER`：选择的 Tk 的主次版本
* `WISH`：Tk 解释器的完整路径
* `TK_LIBDIR`：Tk 库的路径
* `TK_INCLUDEDIR`：Tk C 头文件的路径

## 17.121. `terminfo`

可能的参数：（无）

将 [`@terminfo`](https://docs.freebsd.org/en/books/porters-handbook/plist/#plist-keywords-terminfo) 添加到 **plist**。当该 Port 安装 **\*.terminfo** 文件到 **\${PREFIX}/share/misc** 时使用。

## 17.122. `tex`

可能的参数：（无）

提供对 tex 的支持。加载所有与 TEX 相关的默认变量，并且不添加任何对其他 Ports 的依赖。

变量用于指定需要哪些 TEX 模块。

`USE_TEX` 运行时所需的 TEX 扩展列表。在扩展名后添加 `:build` 来添加构建时依赖，`:run` 添加运行时依赖，`:test` 添加测试时依赖，`:extract` 添加提取时依赖。例如：`base texmf:build source:run`

当前可能的参数如下：

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

## 17.123. `tk`

与 `tcl` 的参数相同

当同时使用 Tcl 和 Tk 时的小包装器。返回的变量与使用 Tcl 时相同。

## 17.124. `trigger`

可能的参数：（无）

提供对需要由 [pkg(8)](https://man.freebsd.org/cgi/man.cgi?query=pkg&sektion=8&format=html) 执行触发器的 Ports 的支持。触发器在事务结束时执行，如果条件满足。

可以通过 Ports 设置以下变量：

`TRIGGERS` 打包的触发器列表。默认为 `${PORTNAME}`。

触发器以 UCL 格式指定，通常放置在 Port 的 **files/** 目录中。

## 17.125. `uidfix`

可能的参数：（无）

更改构建系统的一些默认行为（主要是变量），以允许以普通用户身份安装该 Port。在使用 [`USES=fakeroot`](https://docs.freebsd.org/en/books/porters-handbook/uses/#uses-fakeroot) 或补丁之前，请先尝试此方法。

## 17.126. `uniquefiles`

可能的参数：（无），`dirs`

通过添加前缀或后缀使文件或目录“唯一”。如果使用 `dirs` 参数，Port 需要一个基于 `UNIQUE_PREFIX` 的前缀（且仅需前缀），适用于标准目录 `DOCSDIR`、`EXAMPLESDIR`、`DATADIR`、`WWWDIR`、`ETCDIR`。以下变量可供 Port 使用：

* `UNIQUE_PREFIX`：用于目录和文件的前缀。默认值：`${PKGNAMEPREFIX}`。
* `UNIQUE_PREFIX_FILES`：需要添加前缀的文件列表。默认值：空。
* `UNIQUE_SUFFIX`：用于文件的后缀。默认值：`${PKGNAMESUFFIX}`。
* `UNIQUE_SUFFIX_FILES`：需要添加后缀的文件列表。默认值：空。

## 17.127. `vala`

可能的参数：`build`，`lib`，`no_depend`

添加对 [lang/vala](https://cgit.freebsd.org/ports/tree/lang/vala/) 的构建或库依赖。`no_depend` 参数保留给 [lang/vala](https://cgit.freebsd.org/ports/tree/lang/vala/) 本身。

## 17.128. `varnish`

可能的参数：`4`（默认），`6`，`7`

处理 Varnish Cache 的依赖关系。添加对 [www/varnish\*](https://cgit.freebsd.org/ports/tree/www/varnish*/) 的依赖。

## 17.129. `waf`

可能的参数：（无）

为使用 `waf` 构建系统的 Ports 提供支持。

它意味着 `USES=python:build`。

以下变量会被导出以供 Port 使用：

`WAF_CMD`：`waf` 脚本的位置。如果 `waf` 脚本不在 **WRKSRC/waf** 中，请设置此项。

`CONFIGURE_TARGET`：`Configure` 目标。默认值：`configure`。

`ALL_TARGET`：`All` 目标。默认值：`build`。

`INSTALL_TARGET`：`Install` 目标。默认值：`install`。

## 17.130. `webplugin`

可能的参数：（无），`ARGS`

自动为支持 webplugin 框架的每个应用程序创建和删除符号链接。`ARGS` 可以是以下之一：

* `gecko`：支持基于 Gecko 的插件
* `native`：支持 Gecko、Opera 和 WebKit-GTK 插件
* `linux`：支持 Linux 插件
* `all`（默认，隐式）：支持所有插件类型
* （单独的条目）：仅支持列出的浏览器

以下变量可以调整：

* `WEBPLUGIN_FILES`：没有默认值，必须手动设置。要安装的插件文件。
* `WEBPLUGIN_DIR`：安装插件文件的目录，默认值 **PREFIX/lib/browser\_plugins/WEBPLUGIN\_NAME**。如果 Port 安装插件文件到默认目录以外的地方，请设置此项，以避免损坏的符号链接。
* `WEBPLUGIN_NAME`：安装插件文件的最终目录，默认值 `PKGBASE`。

## 17.131. `xfce`

可能的参数：（无），`gtk2`

为 Xfce 相关 Ports 提供支持。有关详细信息，请参阅 [Using Xfce](https://docs.freebsd.org/en/books/porters-handbook/special/#using-xfce)。

`gtk2` 参数指定该 Port 需要 GTK2 支持。它添加了一些核心组件提供的额外功能，例如 [x11/libxfce4menu](https://cgit.freebsd.org/ports/tree/x11/libxfce4menu/) 和 [x11-wm/xfce4-panel](https://cgit.freebsd.org/ports/tree/x11-wm/xfce4-panel/)。

## 17.132. `xorg`

可能的参数：（无）

提供一种简便的方法来依赖 X.org 组件。这些组件应列在 `USE_XORG` 中。可用的组件有：

**表 1. 可用的 X.Org 组件**

| 名称            | 描述                  |
| ------------- | ------------------- |
| `dmx`         | DMX 扩展库             |
| `fontenc`     | 字体编码库               |
| `fontutil`    | 在目录中创建 X 字体文件的索引    |
| `ice`         | X11 的客户端交换库         |
| `libfs`       | FS 库                |
| `pciaccess`   | 通用 PCI 访问库          |
| `pixman`      | 低级像素操作库             |
| `sm`          | X11 会话管理库           |
| `x11`         | X11 库               |
| `xau`         | X11 身份验证协议库         |
| `xaw`         | X Athena 小部件库       |
| `xaw6`        | X Athena 小部件库       |
| `xaw7`        | X Athena 小部件库       |
| `xbitmaps`    | X.Org 位图数据          |
| `xcb`         | X 协议 C 语言绑定 (XCB) 库 |
| `xcomposite`  | X 复合扩展库             |
| `xcursor`     | X 客户端光标加载库          |
| `xdamage`     | X 损坏扩展库             |
| `xdmcp`       | X 显示管理器控制协议库        |
| `xext`        | X11 扩展库             |
| `xfixes`      | X 修复扩展库             |
| `xfont`       | X 字体库               |
| `xfont2`      | X 字体库               |
| `xft`         | 客户端字体 API，用于 X 应用程序 |
| `xi`          | X 输入扩展库             |
| `xinerama`    | X11 Xinerama 库      |
| `xkbfile`     | XKB 文件库             |
| `xmu`         | X 杂项实用程序库           |
| `xmuu`        | X 杂项实用程序库           |
| `xorg-macros` | X.Org 开发 aclocal 宏  |
| `xorg-server` | X.Org X 服务器及相关程序    |
| `xorgproto`   | Xorg 协议头文件          |
| `xpm`         | X Pixmap 库          |
| `xpresent`    | X Present 扩展库       |
| `xrandr`      | X 调整大小和旋转扩展库        |
| `xrender`     | X 渲染扩展库             |
| `xres`        | X 资源使用库             |
| `xscrnsaver`  | XScrnSaver 库        |
| `xshmfence`   | 共享内存“SyncFence”同步原语 |
| `xt`          | X 工具包库              |
| `xtrans`      | X 的抽象网络代码           |
| `xtst`        | X 测试扩展              |
| `xv`          | X 视频扩展库             |
| `xvmc`        | X 视频扩展运动补偿库         |
| `xxf86dga`    | X DGA 扩展            |
| `xxf86vm`     | X Vidmode 扩展        |

## 17.133. `xorg-cat`

可能的参数：`app`，`data`，`doc`，`driver`，`font`，`lib`，`proto`，`util`，`xserver` 和（无）或单独的 `autotools`（默认），`meson`

提供对构建 Xorg 组件的支持。它会设置所需的常见依赖项和适当的配置环境。此选项仅适用于 Xorg 组件。

类别必须与上游类别匹配。

第二个参数是要使用的构建系统。默认是 autotools，但也支持 meson。

## 17.134. `zip`

可能的参数：（无），`infozip`

指示分发文件使用 ZIP 压缩算法。对于使用 InfoZip 算法的文件，必须传递 `infozip` 参数以设置适当的依赖关系。

