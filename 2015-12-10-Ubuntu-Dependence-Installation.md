title: Ubuntu Dependence Installation 
date: 2015-12-10 17:10:24
categories: 
    - Linux
tags: 
- Ubuntu
- Installation
- Script 
----------
Ubuntu 依赖安装
========================== 
Ubuntu是一款很棒的系统，广泛用于科学计算和生物信息学研究中。但是仅仅安装系统是没有用的，更好的完成工作需要安装相应的软件，在Ubuntu 下安装软件少不了先安装依赖，这里我提供自己重装系统后安装软件和以来的脚本，可以给广大生物信息学同学提供一个参考。大家如果什么建议和补充欢迎和我交流！  

<!-- more -->  

安装脚本[下载地址在此][udi]  
此脚本包安装的内容包括：  

- 一些开发工具，如 build-essential，gcc，make，git 等
- 一些常用IDE，如 eclipse，qt-sdk，vim等
- 一些视频软件和解码器，如 vlc gstreamer1.0-libav等
- 文泉驿字体
- python的相关包
- 常用系统插件，如 nautilus-open-terminal p7zip icedtea-7-plugin
- chromium浏览器
- 信息学相关包，如：openbabel，pymol， r-bioc-biobase等
-一些其他软件的依赖包，如：skype，amber等

由于apt-get有很好的依赖解决能力，所以有些包并没有显示安装，而是通过安装依赖其的一些包达到自动安装，比如没有直接安装R，而是选择安装 r-bioc-biobase，通过依赖关系解决R包的安装问题，减少此文件的重复。大家可以选择性的安装自己需要的软件或者依赖。
如果大家对此脚本有什么修改意见和建议，或者自己有一些软件安装经验，欢迎和我交流，我会及时更新，方便大家更好的使用ubuntu。

[udi]:https://github.com/maoyibo/mybscript/blob/master/ubuntu_dependence_install.sh "Download page"
