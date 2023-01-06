---
layout: post
title: rbenv gem bundle 博客环境搭建                             # Title of the page
hide_title: false                                  # Hide the title when displaying the post, but shown in lists of posts
excerpt_separator: <!--more-->
#feature-img: "assets/img/sample.png"              # Add a feature-image to the post
#thumbnail: "assets/img/avatar.png"  # Add a thumbnail image on blog view
#color: rgb(80,140,22)                             # Add the specified color as feature image, and change link colors in post
bootstrap: true                                   # Add bootstrap to the page
tags: [linux]
---
#### 安装 rbenv 
<!--more-->
```
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="~/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(~/.rbenv/bin/rbenv init -)"' >> ~/.bashrc
source ~/.bashrc
git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
apt install -y build-essential libssl-dev zlib1g-dev
```  
#### 安装 ruby
```
rbenv install 2.7.1
rbenv global 2.7.1
```
#### 安装 bundler
```
gem install bundler
```
#### 安装项目依赖
```
cd gitbasedir
bundle install
```
#### 运行项目
```
bundle exec jekyll serve --port 4000
```
