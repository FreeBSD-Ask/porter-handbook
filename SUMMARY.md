# Table of contents

* [FreeBSD port 开发者手冊翻译项目](README.md)
* [编辑日志](bian-ji-ri-zhi.md)
* [译者说明](yi-zhe-shuo-ming.md)

## FreeBSD Port 开发者手册

* [FreeBSD Port 开发者手册](freebsd-kai-fa-zhe-shou-ce.md)

## 第1章 简介

* [1.1.简介](di-1-zhang-jian-jie.md)

## 第2章 制作新的 port

* [2.1.制作新的 port](di-2-zhang-zhi-zuo-xin-de-port.md)

## 第3章 简单的 port

* [3.1.编写 Makefile](di-3-zhang-jian-dan-de-port/3.1.-bian-xie-makefile.md)
* [3.2.编写描述文件](di-3-zhang-jian-dan-de-port/3.2.-bian-xie-miao-shu-wen-jian.md)
* [3.3.创建校验和文件](di-3-zhang-jian-dan-de-port/3.3.-chuang-jian-xiao-yan-he-wen-jian.md)
* [3.4.测试 port](di-3-zhang-jian-dan-de-port/3.4.-ce-shi-port.md)
* [3.5.用 portlint 来检查 port](di-3-zhang-jian-dan-de-port/3.5.-yong-portlint-lai-jian-cha-port.md)
* [3.6.提交新的 port](di-3-zhang-jian-dan-de-port/3.6.-ti-jiao-xin-de-port.md)

## 第4章 复杂的 Port

* [4.1.Port 是如何运行的](di-4-zhang-fu-za-de-port/4.1.-ta-shi-zen-mo-yun-zuo-de.md)
* [4.2.获取源代码](di-4-zhang-fu-za-de-port/4.2.-huo-qu-yuan-dai-ma.md)
* [4.3.修改 port](di-4-zhang-fu-za-de-port/4.3.-xiu-gai-port.md)
* [4.4.打补丁](di-4-zhang-fu-za-de-port/4.4.-da-bu-ding.md)
* [4.5.配置](di-4-zhang-fu-za-de-port/4.5.-pei-zhi.md)
* [4.6.处理用户输入](di-4-zhang-fu-za-de-port/4.6.-chu-li-yong-hu-shu-ru.md)

## 第5章 配置 Makefile

* [5.1.原始来源](di-5-zhang-pei-zhi-makefile/5.1.-zuo-zhe-fa-bu-de-dai-ma.md)
* [5.2.命名](di-5-zhang-pei-zhi-makefile/5.2.-ming-ming.md)
* [5.3.归类](di-5-zhang-pei-zhi-makefile/5.3.-gui-lei.md)
* [5.4.源代码包文件](di-5-zhang-pei-zhi-makefile/5.4.-yuan-dai-ma-bao-wen-jian.md)
* [5.5.维护者（MAINTAINER）](di-5-zhang-pei-zhi-makefile/5.5.-wei-hu-zhe-maintainer.md)
* [5.6.一句话说明（COMMENT）](di-5-zhang-pei-zhi-makefile/5.6.-yi-ju-hua-shuo-ming-comment.md)
* [5.7.项目网站](di-5-zhang-pei-zhi-makefile/5.7.-xiang-mu-wang-zhan.md)
* [5.8.许可证](di-5-zhang-pei-zhi-makefile/5.8.-xu-ke-zheng.md)
* [5.9.PORTSCOUT](di-5-zhang-pei-zhi-makefile/5.9.portscout.md)
* [5.10.依赖](di-5-zhang-pei-zhi-makefile/5.10.-yi-lai.md)
* [5.11.从属 port 和 MASTERDIR](di-5-zhang-pei-zhi-makefile/5.11.-cong-shu-port-he-masterdir.md)
* [5.12.man 手册](di-5-zhang-pei-zhi-makefile/5.12.man-shou-ce.md)
* [5.13.info 文件](di-5-zhang-pei-zhi-makefile/5.13.info-wen-jian.md)
* [5.14.Makefile 参数](di-5-zhang-pei-zhi-makefile/5.14.makefile-can-shu.md)
* [5.15.特殊的工作目录](di-5-zhang-pei-zhi-makefile/5.15.-te-shu-de-gong-zuo-mu-lu.md)
* [5.16.解决冲突](di-5-zhang-pei-zhi-makefile/5.16.-jie-jue-chong-tu.md)
* [5.17.安装文件](di-5-zhang-pei-zhi-makefile/5.17.-an-zhuang-wen-jian.md)
* [5.18. 使用 BINARY\_ALIAS 来重命名命令，而不是在编译中打补丁](di-5-zhang-pei-zhi-makefile/5.18.-shi-yong-binaryalias-lai-zhong-ming-ming-ming-ling-er-bu-shi-zai-bian-yi-zhong-da-bu-ding.md)

## 第6章 特殊情况

* [第6章 特殊情况](di-6-zhang-te-shu-qing-kuang/di-liu-zhang.md)


## 第7章 Flavors

* [7.1.Flavors 简介](di-7-zhang-flavors/7.1.-flavors-jian-jie.md)
* [7.2.使用 FLAVORS](di-7-zhang-flavors/7.2.-shi-yong-flavors.md)
* [7.3.USES=php 和 Flavors](di-7-zhang-flavors/7.3.usesphp-he-flavors.md)
* [7.4.USES=python 和 Flavors](di-7-zhang-flavors/7.4.usespython-he-flavors.md)
* [7.5.USES=lua 和 Flavors](di-7-zhang-flavors/7.5.useslua-he-flavors.md)

## 第8章 高级 pkg-plist 实践

* [8.1.根据 make 变量对 pkg-plist 进行修改](di-8-zhang-gao-ji-pkgplist-shi-jian/8.1.-gen-ju-make-bian-liang-dui-pkgplist-jin-hang-xiu-gai.md)
* [8.2.空目录](di-8-zhang-gao-ji-pkgplist-shi-jian/8.2.-kong-mu-lu.md)
* [8.3.配置文件](di-8-zhang-gao-ji-pkgplist-shi-jian/8.3.-pei-zhi-wen-jian.md)
* [8.4.动态与静态软件包列表](di-8-zhang-gao-ji-pkgplist-shi-jian/8.4.-dong-tai-yu-jing-tai-ruan-jian-bao-lie-biao.md)
* [8.5.自动创建软件包列表](di-8-zhang-gao-ji-pkgplist-shi-jian/8.5.-zi-dong-chuang-jian-ruan-jian-bao-lie-biao.md)
* [8.6.用关键词扩展软件包列表](di-8-zhang-gao-ji-pkgplist-shi-jian/8.6.-yong-guan-jian-ci-kuo-zhan-ruan-jian-bao-lie-biao.md)

## 第9章 pkg-\*

* [9.1.pkg-message（安装二进制包时显示的消息文件）](di-9-zhang-pkg/9.1.pkgmessage-an-zhuang-er-jin-zhi-bao-shi-xian-shi-de-xiao-xi-wen-jian.md)
* [9.2.pkg-install、pkg-pre-install 和 pkg-post-install（安装二进制包时执行的脚本文件）](di-9-zhang-pkg/9.2.-pkginstall-an-zhuang-er-jin-zhi-bao-shi-zhi-hang-de-jiao-ben-wen-jian.md)
* [9.3.pkg-deinstall、pkg-pre-deinstall 和 pkg-post-deinstall（卸载时执行的脚本文件）](di-9-zhang-pkg/9.3.pkgdeinstall-xie-zai-shi-zhi-hang-de-jiao-ben-wen-jian.md)
* [9.4.修改 pkg-\* 文件的名字](di-9-zhang-pkg/9.4.-xiu-gai-pkg-wen-jian-de-ming-zi.md)
* [9.5.使用 SUB\_FILES 和 SUB\_LIST](di-9-zhang-pkg/9.5.-shi-yong-subfiles-he-sublist.md)

## 第10章 测试 port

* [10.1.运行 make describe](di-10-zhang-ce-shi-port/10.1.-yun-hang-make-describe.md)
* [10.2.运行 make test](di-10-zhang-ce-shi-port/10.2-yun-hang-make-test.md)
* [10.3.Portclippy / Portfmt](di-10-zhang-ce-shi-port/10.3.portclippy-portfmt.md)
* [10.4.Portlint](di-10-zhang-ce-shi-port/10.4.portlint.md)
* [10.5.Port 工具](di-10-zhang-ce-shi-port/10.5.porttools.md)
* [10.6.PREFIX 和 DESTDIR](di-10-zhang-ce-shi-port/10.6.prefix-he-destdir.md)
* [10.7.Poudriere](di-10-zhang-ce-shi-port/10.7.poudriere.md)
* [10.8.调试 port](di-10-zhang-ce-shi-port/10.8.tiao-shi-port.md)

## 第11章 升级 port

* [11.1.使用 Git 制作补丁](di-11-zhang-sheng-ji-port/11.1.-shi-yong-git-zhi-zuo-bu-ding.md)
* [11.2.UPDATING 和 MOVED](di-11-zhang-sheng-ji-port/11.2.updating-he-moved.md)

## 第12章 安全

* [12.1.安全为何如此重要](di-12-zhang-an-quan/12.1.-wei-he-an-quan-ru-ci-zhong-yao.md)
* [12.2.修复安全漏洞](di-12-zhang-an-quan/12.2.-xiu-fu-an-quan-lou-dong.md)
* [12.3.让社区了解情况](di-12-zhang-an-quan/12.3.-rang-she-qu-le-jie-qing-kuang.md)

## 第13章 该做什么和不该做什么

* [13.1.简介](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.1.-jian-jie.md)
* [13.2.WRKDIR（联编时使用的临时目录）](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.2.wrkdir-lian-bian-shi-shi-yong-de-lin-shi-mu-lu.md)
* [13.3.WRKDIRPREFIX（用于联编的临时目录的父目录名)](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.3.wrkdirprefix-yong-yu-lian-bian-de-lin-shi-mu-lu-de-fu-mu-lu-ming.md)
* [13.4.区分不同的操作系统， 以及其版本](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.4.-qu-fen-bu-tong-de-cao-zuo-xi-tong-yi-ji-qi-ban-ben.md)
* [13.5.在 bsd.port.mk 之后写一些内容](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.5.-zai-bsd.port.mk-zhi-hou-xie-yi-xie-nei-rong.md)
* [13.6.在 wrapper 脚本中使用 exec 语句](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.6.-zai-wrapper-jiao-ben-zhong-shi-yong-exec-yu-ju.md)
* [13.7.理性行事](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.7.-li-xing-hang-shi.md)
* [13.8.遵循 CC 和 CXX 设置](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.8.-zun-xun-cc-he-cxx-she-zhi.md)
* [13.9.遵循 CFLAGS](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.9.-zun-xun-cflags.md)
* [13.10.冗长的编译日志](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.10.-rong-chang-de-bian-yi-ri-zhi.md)
* [13.11.反馈](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.11.-fan-kui.md)
* [13.12.README.html](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.12.readme.html.md)
* [13.13.使用 BROKEN、 FORBIDDEN 或 IGNORE 阻止用户安装 port](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.13.-shi-yong-broken-forbidden-huo-ignore-zu-zhi-yong-hu-an-zhuang-port.md)
* [13.14.适用架构考量](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.14.-shi-yong-jia-gou-kao-liang.md)
* [13.15.使用 DEPRECATED 或 EXPIRATION\_DATE 表示某个 port 将被删除](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.15.-shi-yong-deprecated-huo-expirationdate-biao-shi-mou-ge-port-jiang-bei-shan-chu.md)
* [13.16.避免使用 .error 结构](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.16.-bi-mian-shi-yong-.error-jie-gou.md)
* [13.17.sysctl 的用法](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.17.sysctl-de-yong-fa.md)
* [13.18.重新发布的 distfiles](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.18.-zhong-xin-fa-bu-de-distfiles.md)
* [13.19.使用 POSIX 标准](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.19.-shi-yong-posix-biao-zhun.md)
* [13.20.杂项](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/13.20.-za-xiang.md)

***

* [第14章 Makefile 示例](di-14-zhang-makefile-shi-li.md)

## 第15章 在 Port Makefile 中变量的顺序

* [15.1.PORTNAME 部分](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.1.portname-bu-fen.md)
* [15.2.PATCHFILES 部分](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.2.patchfiles-bu-fen.md)
* [15.3.MAINTAINER 部分](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.3.maintainer-bu-fen.md)
* [15.4.LICENSE（许可证）部分](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.4.-license-xu-ke-zheng-bu-fen.md)
* [15.5.一般的 BROKEN/IGNORE/DEPRECATED 消息](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.5.-yi-ban-de-brokenignoredeprecated-xiao-xi.md)
* [15.6.依赖部分](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.6.-yi-lai-bu-fen.md)
* [15.7.Flavors](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.7.flavors.md)
* [15.8.USES 和 USE\_x](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.8.uses-he-usex.md)
* [15.9.标准的 bsd.port.mk 变量](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.9.-biao-zhun-de-bsd.port.mk-bian-liang.md)
* [15.10.Options 和帮助文档](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.10.options-he-bang-zhu-wen-dang.md)
* [15.11.其余变量](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.11.-qi-yu-bian-liang.md)
* [15.12.Target](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/15.12.target.md)

## 第16章 保持同步

* [16.1.FreshPorts](di-16-zhang-bao-chi-tong-bu/16.1.freshports.md)
* [16.2.代码库的 Web 访问界面](di-16-zhang-bao-chi-tong-bu/16.2.-dai-ma-ku-de-web-fang-wen-jie-mian.md)
* [16.3.FreeBSD Ports 邮件列表](di-16-zhang-bao-chi-tong-bu/16.3.freebsd-ports-you-jian-lie-biao.md)
* [16.4.FreeBSD port 构建集群](di-16-zhang-bao-chi-tong-bu/16.4.-freebsd-port-gou-jian-ji-qun.md)
* [16.5.Portscout：FreeBSD Ports Distfile 扫描器](di-16-zhang-bao-chi-tong-bu/16.5.portscoutfreebsd-ports-distfile-sao-miao-qi.md)

## 第17章 使用 USES 宏

* [第17章 使用 USES 宏](di-17-zhang-shi-yong-uses-hong/di-17-zhang.md)


## 第18章 \_\_FreeBSD\_version 的值

* [第18章 \_\_FreeBSD\_version 的值](di-18-zhang-freebsdversion-de-zhi/18.md)


