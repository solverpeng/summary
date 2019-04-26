# 搭建开发环境



## SDK

[下载地址](https://golang.org/dl/)

## 环境变量配置

若是通过标准包安装，则会自动配置好GOROOT，只需要配置GOPATH 

1. GOROOT: GO语言的安装根目录 
2. GOPATH: 若干工作区目录的路径，可以是一个目录，也可以是多个目录，每个目录都代表着一个工作区 

## GPATH目录结构

根据约定，GOPATH需要建立3个目录

1. bin：存放编译后生成的可执行文件
2. pkg：存放编译后生成的包文件
3. src：存放项目源码

## 整体目录结构

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/go-summary/go-structure.png)



