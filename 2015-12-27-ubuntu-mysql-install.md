title: Ubuntu 安装 MySQL 简便方法 
date: 2015-12-27 15:34:56
categories: 
    - Linux
tags:
    - Ubuntu
    - Database
    - MySQL
    - Installation
---
Ubuntu 14.04 官方软件源默认的MySQL是5.5版本，可以选择从软件源安装5.6版本但是不能从软件源安装mysql-workbench，因为其依赖5.5版本的MySQL。  

<!-- more -->  

为了方便的安装和更新MySQL可以选择添加MySQL的官方软件仓库。[此地址](http://dev.mysql.com/downloads/repo/apt/ 'Download MySQL APT Repository')是适用于debian和ubuntu的APT repository，你也可以根据需要选择YUM repository或者SUSE repository。  
MySQL APT repository 包括以下软件包:  

- MySQL 5.7 (GA)
- MySQL 5.6 (GA)
- MySQL Workbench 6.3 (GA) - Ubuntu Only
- MySQL Router
- MySQL Utilities
- MySQL Connector / Python

下载“Ubuntu / Debian (Architecture Independent), DEB“ (mysql-apt-config_0.6.0-1_all.deb) 并安装。更新软件包列表后即可安装MySQL。
```bash
#安装APT repository
$ sudo dpkg -i mysql-apt-config_0.6.0-1_all.deb
#安装时出现配置选项，可以选择mysql的版本和是否包括其他软件包
#更新软件包列表
$ sudo apt-get update
#安装mysql-server
$ sudo apt-get install mysql-server mysql-workbench
```
