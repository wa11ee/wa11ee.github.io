---
layout: post
title: PostgresXL 集群搭建                              # Title of the page
hide_title: false                                  # Hide the title when displaying the post, but shown in lists of posts
excerpt_separator: <!--more-->
#feature-img: "assets/img/sample.png"              # Add a feature-image to the post
#thumbnail: "assets/img/avatar.png"  # Add a thumbnail image on blog view
#color: rgb(80,140,22)                             # Add the specified color as feature image, and change link colors in post
bootstrap: true                                   # Add bootstrap to the page
tags: [大数据]
---
PostgresSQl是个不错的关系型数据库，得益于UDF分析数据时也很方便，因此<!--more-->很多BI应用也选做为底层存储 。阿里腾讯也在此基础上研发出来面向大数据服务的产品Hologres、Tbase  
PostgresXL是PostgresSQL的开源分布式数据库的实现，包括分布式存储、分布式计算  [官方网站](https://www.postgres-xl.org/)  
本文将介绍如何安装PostgresXL集群。  
#### 软件版本
postgres-xl-9.5r1.6  
centos7.6  

#### 节点规划

| 节点          | 角色                       |
|:------------|:-------------------------|
| bi-test1    | gtmServer                |
| bi-test3    | gtmProxy、coord、datanode  |
| bi-test4    | gtmProxy、coord、datanode  |
| bi-test5    | gtmProxy、coord、datanode  |
 
1. **集群host文件**  
将每个节点主机名对应的IP地址加入集群主机Hosts中

2. **创建pgxl用户，配置免密登录**  
在每个节点创建pgxl用户
```
adduser pgxl && su pgxl
```
配置免密登录，`RSA公钥` 为bi-test1节点的ssh公钥
```
mkdir ~/.ssh && echo 'RSA公钥' >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_key
```

3. **创建编译目录**
```
cd ~ && mkdir pgxl_build_dir && cd pgxl_build_dir
```

4. **配置安装参数**
--with-pgport 指定pg端口
```
/home/pgxl/postgres-xl-9.5r1.6/configure  --prefix=/home/pgxl/pgxl --with-pgport=15432
```

5. **编译源码**
```
make
```

6. **安装 (注意两种方式)**  
`bi-test1`
```
make install-world
```
其余节点
```
make install-world
```

7. **环境变量配置**  
`vi ~/.bashrc`加入  
``` 
PATH=$PATH:/home/pgxl/pgxl/bin
export $PATH
```
```
source ~/.bashrc
```

8. **集群初始化**    
进入`bi-test1`  
运行`/home/pgxl/pgxl/bin/pgxc_ctl` 进入控制台  
输入`prepare`生成`pgxc_ctl.conf`配置文件  
手动编辑pgxc_ctl.conf配置文件  
注意每个节点的配置信息，端口，slave，hba entry，链接配置信息  
确认配置无误后运行`/home/pgxl/pgxl/bin/pgxc_ctl init all` 启动集群

pgxl表存储方式有复制表、分片表目前使用下来分片表有些莫名的bug感觉不太稳定  

