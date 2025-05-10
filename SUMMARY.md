# Table of contents

* [FreeBSD Port 开发者手册翻译项目](README.md)
* [编辑日志](bian-ji-ri-zhi.md)
* [译者说明](yi-zhe-shuo-ming.md)

## FreeBSD Port 开发者手册

* [FreeBSD Port 开发者手册](freebsd-kai-fa-zhe-shou-ce.md)

## 第 1 章 简介

* [1.1.简介](di-1-zhang-jian-jie.md)

## 第 2 章 制作新的 port

* [2.1.制作新的 port](di-2-zhang-zhi-zuo-xin-de-port.md)

## 第 3 章 简单的 port

* [3.1.编写 Makefile](di-3-zhang-jian-dan-de-port/3.1.-bian-xie-makefile.md)
* [3.2.编写描述文件](di-3-zhang-jian-dan-de-port/3.2.-bian-xie-miao-shu-wen-jian.md)
* [3.3.创建校验和文件](di-3-zhang-jian-dan-de-port/3.3.-chuang-jian-xiao-yan-he-wen-jian.md)
* [3.4.测试 port](di-3-zhang-jian-dan-de-port/3.4.-ce-shi-port.md)
* [3.5.用 portlint 来检查 port](di-3-zhang-jian-dan-de-port/3.5.-yong-portlint-lai-jian-cha-port.md)
* [3.6.提交新的 port](di-3-zhang-jian-dan-de-port/3.6.-ti-jiao-xin-de-port.md)

## 第 4 章 复杂的 Port

* [4.1 Port 工作原理](di-4-zhang-fu-za-de-port/4.1.-ta-shi-zen-mo-yun-zuo-de.md)
* [4.2.获取源代码](di-4-zhang-fu-za-de-port/4.2.-huo-qu-yuan-dai-ma.md)
* [4.3.修改 port](di-4-zhang-fu-za-de-port/4.3.-xiu-gai-port.md)
* [4.4.打补丁](di-4-zhang-fu-za-de-port/4.4.-da-bu-ding.md)
* [4.5.配置](di-4-zhang-fu-za-de-port/4.5.-pei-zhi.md)
* [4.6.处理用户输入](di-4-zhang-fu-za-de-port/4.6.-chu-li-yong-hu-shu-ru.md)

## 第 5 章 配置 Makefile

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

## 第 6 章 特殊情况

* [第 6 章 特殊情况](di-6-zhang-te-shu-qing-kuang/di-liu-zhang.md)


## 第 7 章 Flavors

* [7.1.Flavors 简介](di-7-zhang-flavors/7.1.-flavors-jian-jie.md)
* [7.2.使用 FLAVORS](di-7-zhang-flavors/7.2.-shi-yong-flavors.md)
* [7.3.USES=php 和 Flavors](di-7-zhang-flavors/7.3.usesphp-he-flavors.md)
* [7.4.USES=python 和 Flavors](di-7-zhang-flavors/7.4.usespython-he-flavors.md)
* [7.5.USES=lua 和 Flavors](di-7-zhang-flavors/7.5.useslua-he-flavors.md)

## 第 8 章 高级 pkg-plist 实践

* [8.1.根据 make 变量对 pkg-plist 进行修改](di-8-zhang-gao-ji-pkgplist-shi-jian/8.1.-gen-ju-make-bian-liang-dui-pkgplist-jin-hang-xiu-gai.md)
* [8.2.空目录](di-8-zhang-gao-ji-pkgplist-shi-jian/8.2.-kong-mu-lu.md)
* [8.3.配置文件](di-8-zhang-gao-ji-pkgplist-shi-jian/8.3.-pei-zhi-wen-jian.md)
* [8.4.动态与静态软件包列表](di-8-zhang-gao-ji-pkgplist-shi-jian/8.4.-dong-tai-yu-jing-tai-ruan-jian-bao-lie-biao.md)
* [8.5.自动创建软件包列表](di-8-zhang-gao-ji-pkgplist-shi-jian/8.5.-zi-dong-chuang-jian-ruan-jian-bao-lie-biao.md)
* [8.6.用关键词扩展软件包列表](di-8-zhang-gao-ji-pkgplist-shi-jian/8.6.-yong-guan-jian-ci-kuo-zhan-ruan-jian-bao-lie-biao.md)

## 第 9 章 pkg-\*

* [9.1.pkg-message（安装二进制包时显示的消息文件）](di-9-zhang-pkg/9.1.pkgmessage-an-zhuang-er-jin-zhi-bao-shi-xian-shi-de-xiao-xi-wen-jian.md)
* [9.2.pkg-install、pkg-pre-install 和 pkg-post-install（安装二进制包时执行的脚本文件）](di-9-zhang-pkg/9.2.-pkginstall-an-zhuang-er-jin-zhi-bao-shi-zhi-hang-de-jiao-ben-wen-jian.md)
* [9.3.pkg-deinstall、pkg-pre-deinstall 和 pkg-post-deinstall（卸载时执行的脚本文件）](di-9-zhang-pkg/9.3.pkgdeinstall-xie-zai-shi-zhi-hang-de-jiao-ben-wen-jian.md)
* [9.4.修改 pkg-\* 文件的名字](di-9-zhang-pkg/9.4.-xiu-gai-pkg-wen-jian-de-ming-zi.md)
* [9.5.使用 SUB\_FILES 和 SUB\_LIST](di-9-zhang-pkg/9.5.-shi-yong-subfiles-he-sublist.md)

## 第 10 章 测试 port

* [10.1.运行 make describe](di-10-zhang-ce-shi-port/10.1.-yun-hang-make-describe.md)
* [10.2.运行 make test](di-10-zhang-ce-shi-port/10.2-yun-hang-make-test.md)
* [10.3.Portclippy / Portfmt](di-10-zhang-ce-shi-port/10.3.portclippy-portfmt.md)
* [10.4.Portlint](di-10-zhang-ce-shi-port/10.4.portlint.md)
* [10.5.Port 工具](di-10-zhang-ce-shi-port/10.5.porttools.md)
* [10.6.PREFIX 和 DESTDIR](di-10-zhang-ce-shi-port/10.6.prefix-he-destdir.md)
* [10.7.Poudriere](di-10-zhang-ce-shi-port/10.7.poudriere.md)
* [10.8.调试 port](di-10-zhang-ce-shi-port/10.8.tiao-shi-port.md)

## 第 11 章 升级 port

* [11.1.使用 Git 制作补丁](di-11-zhang-sheng-ji-port/11.1.-shi-yong-git-zhi-zuo-bu-ding.md)
* [11.2.UPDATING 和 MOVED](di-11-zhang-sheng-ji-port/11.2.updating-he-moved.md)

## 第 12 章 安全

* [12.1.安全为何如此重要](di-12-zhang-an-quan/12.1.-wei-he-an-quan-ru-ci-zhong-yao.md)
* [12.2.修复安全漏洞](di-12-zhang-an-quan/12.2.-xiu-fu-an-quan-lou-dong.md)
* [12.3.向社区通报情况](di-12-zhang-an-quan/12.3.-rang-she-qu-le-jie-qing-kuang.md)

## 第 13 章 该做什么和不该做什么

* [第 13 章 该做什么和不该做什么](di-13-zhang-gai-zuo-shi-mo-he-bu-gai-zuo-shi-mo/di-13-zhang.md)

## 第14章 一个简单的 port

* [第 14 章 一个简单的 port](di-14-zhang-makefile-shi-li.md)

## 第 15 章 在 Port Makefile 中变量的顺序

* [第 15 章 在 Port Makefile 中变量的顺序](di-15-zhang-zai-port-makefile-zhong-bian-liang-de-shun-xu/di-15-zhang.md)

## 第 16 章 保持更新

* [第 16 章 保持更新](di-16-zhang-bao-chi-tong-bu/di-16-zhang.md)

## 第 17 章 使用 USES 宏

* [第 17 章 使用 USES 宏](di-17-zhang-shi-yong-uses-hong/di-17-zhang.md)

## 第 18 章 \_\_FreeBSD\_version 的值

* [第 18 章 \_\_FreeBSD\_version 的值](di-18-zhang-freebsdversion-de-zhi/18.md)


