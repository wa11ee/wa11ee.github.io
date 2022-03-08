---
layout: post
title: CDH cm-agent 安装                              # Title of the page
hide_title: false                                  # Hide the title when displaying the post, but shown in lists of posts
excerpt_separator: <!--more-->
#feature-img: "assets/img/sample.png"              # Add a feature-image to the post
#thumbnail: "assets/img/avatar.png"  # Add a thumbnail image on blog view
#color: rgb(80,140,22)                             # Add the specified color as feature image, and change link colors in post
bootstrap: true                                   # Add bootstrap to the page
tags: [大数据]
---
最近在cdh计算节点上安装Mysql环境时不小心将cm-agent服务给卸载了  
起初就是因为Mysql和Mariadb依赖冲突<!--more-->，卸载Mariadb时将cm-agent服务也顺带删除了  
使用rpm命令卸载时一定要注意当前包和其他包的依赖关系  
建议用` rpm -e 文件名 --nodeps `方式
  

### 相关配置

*配置文件：*`/etc/cloudera-scm-agent/config.ini`  
*本机信息：*`/var/lib/cloudera-scm-agent/`  
* *uuid* 为当前节点主机ID，需要保持一致
* *cm_guid*: cdh集群ID，需要保持一致

*日志：*`/var/log/cloudera-scm-agent/cloudera-scm-agent.log`

这些配置文件我们保持不动

### 开始安装

从cdh主节点的httpd服务中下载cm-agent.rpm包

`rpm -ivh cm-agent.rpm --nodeps`  

修改`cm_guid`集群ID和之前一致  
修改 `config.ini` `clouderaserverIP` 主节点IP

### 重启服务

``` 
server启动
/opt/cloudera-manger/cm-5.15.1/etc/init.d/cloudera-scm-server start
agent启动
/opt/cloudera-manager/cm-5.15.1/etc/init.d/cloudera-scm-agent start
```
重启后检查cm-agent日志服务是否启动成功