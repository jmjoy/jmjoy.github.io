+++
date = "2017-01-10T13:22:50+08:00"
draft = false
title = "linux工具集"
categories = ["linux"]
+++

持续更新ing……

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**

- [Deepin](#deepin)
    - [交换CapsLock和Esc键](#交换capslock和esc键)
    - [dconf-editor](#dconf-editor)
- [VCS](#vcs)
    - [rabbitvcs](#rabbitvcs)
- [编程语言](#编程语言)
    - [Rust](#rust)
        - [国内源](#国内源)
            - [USTC的源](#ustc的源)
    - [php](#php)
        - [composer](#composer)
        - [phpmd](#phpmd)
    - [javascript](#javascript)
- [编辑器](#编辑器)
    - [Emacs](#emacs)
        - [Spacemacs](#spacemacs)
            - [Spacemacs相关工具](#spacemacs相关工具)
                - [Fira Code 字体](#fira-code-字体)
                - [搜索工具](#搜索工具)
                - [GNU global](#gnu-global)
            - [tern](#tern)
- [数据库相关](#数据库相关)
    - [客户端](#客户端)
        - [MySQL Workbench](#mysql-workbench)
- [Docker](#docker)
    - [docker-compose](#docker-compose)
    - [推荐镜像](#推荐镜像)
        - [laraedit-docker](#laraedit-docker)
- [坚果云](#坚果云)

<!-- markdown-toc end -->

# Deepin #

[deepin官方网站](https://www.deepin.org/)

## 交换CapsLock和Esc键 ##

Vim键盘手专用

``` shell
gsettings set com.deepin.dde.keybinding.mediakey capslock "['']" && \
gsettings set com.deepin.dde.keyboard layout-options "['caps:swapescape']"
```

## dconf-editor ##

图形化的配置编辑器 **be careful**

``` shell
sudo apt install dconf-editor
```

-------------------------------------------------------------------------------

# VCS #

``` shell
sudo apt install git git-extras subversion
```

[git-extras: 非常实用的git命令扩展集](https://github.com/tj/git-extras/blob/master/Commands.md)

## rabbitvcs ##

linux下的TortoiseSVN / TortoiseGit替代方案

``` shell
sudo apt install rabbitvcs-cli rabbitvcs-core rabbitvcs-gedit rabbitvcs-nautilus
```

-------------------------------------------------------------------------------

# 编程语言 #

## Rust ##

[Rust官方网站](https://www.rust-lang.org/zh-CN/)
[安装Rust](https://www.rust-lang.org/zh-CN/install.html)

``` shell
curl https://sh.rustup.rs -sSf | sh
```

### 国内源 ###

#### USTC的源 ####

[rustup: The Rust toolchain installer](https://lug.ustc.edu.cn/wiki/mirrors/help/rust-static)

``` shell
export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
```

[cargo: Rust's package manager](https://lug.ustc.edu.cn/wiki/mirrors/help/rust-crates)

更改 $HOME/.cargo/config 为以下内容:

``` toml
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```

## php ##

``` shell
sudo apt install php7.0-cli php7.0-dev php7.0-pgsql php7.0-sqlite3 php7.0-gd \
     php7.0-curl php7.0-mcrypt php7.0-imap php7.0-mysql php7.0-readline php-xdebug php-common \
     php7.0-mbstring php7.0-xml php7.0-zip
```

### composer ###

[composer: PHP包管理器](https://getcomposer.org/)

[安装composer](https://getcomposer.org/download/)

``` shell
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
php -r "if (hash_file('SHA384', 'composer-setup.php') === '55d6ead61b29c7bdee5cccfb50076874187bd9f21f65d8991d46ec5cc90518f447387fb9f76ebae1fbbacf329e583e30') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" && \
php composer-setup.php && \
php -r "unlink('composer-setup.php');"
```

### phpmd ###

语法检查工具，作为编辑器外部工具

[phpmd官方网站](https://phpmd.org/)

``` shell
composer global require phpmd/phpmd
```

## javascript ##

TODO

-------------------------------------------------------------------------------

# 编辑器 #

## Emacs ##

[Emacs官方网站](https://www.gnu.org/software/emacs/)

``` shellsession
sudo apt install emacs
```

### Spacemacs ###

[Spacemacs官方网站](http://spacemacs.org/)

安装

``` shell
git clone https://github.com/syl20bnr/spacemacs ~/.emacs.d
```

#### Spacemacs相关工具 ####

##### Fira Code 字体 #####

https://github.com/tonsky/FiraCode

[下载](https://github.com/tonsky/FiraCode/releases)

##### 搜索工具 #####

``` shell
sudo apt install silversearcher-ag ack-grep
```

##### GNU global #####

[GNU global: 代码跳转TAGS生成工具](https://www.gnu.org/software/global/)

*由于apt源的版本比较老，所以通过编译安装最新*

[下载](https://www.gnu.org/software/global/download.html)

安装编译依赖

``` shell
sudo apt install make bison autoconf texinfo flex gperf libtool-bin libncurses5-dev
```

编译

``` shell
sh reconf.sh && ./configure && make
sudo make install
```

#### tern ####

``` shell
sudo npm install -g tern
```

-------------------------------------------------------------------------------

# 数据库相关 #

## 客户端 ##

### MySQL Workbench ###

[MySQL Workbench官方网站](https://www.mysql.com/products/workbench/)

# Docker #

[安装Docker](https://docs.docker.com/engine/installation/linux/ubuntu/)

## docker-compose ##

[安装docker-compose](https://docs.docker.com/compose/install/)

``` shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.9.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose
```

## 推荐镜像 ##

### laraedit-docker ###

Dockerized version of Laravel Homestead.

https://hub.docker.com/r/laraedit/laraedit/

-------------------------------------------------------------------------------

# 坚果云 #

国内比较优秀的跨平台同步云盘

[官方网站](https://www.jianguoyun.com/)
