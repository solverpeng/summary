# GitHub Page + Hexo 搭建博客

## 准备

1. Node.js 环境
2. Git

### Node.js环境

1. 下载地址：<https://nodejs.org/dist/>，根据系统选择合适的版本
2. 安装完成后，确认版本信息
   - `node -v`
   - `npm -v`

> 版本升级请参考：<https://blog.csdn.net/tlbaba/article/details/79412433>

### Git环境

不做过多说明，完成后，确认版本信息：`git --version`



## GitHub创建代码库

1. 新建公共仓库，名称为`yourname.github.io`，如我的是`solverpeng.github.io`。

2. 完成后，如下图所示
   ![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/blog/blog/github.png)

3. master分支新增index.html，然后通过 <https://solverpeng.github.io/> 访问能否正常显示。



## Hexo安装

1. 创建一个空文件夹，在当前目录打开命令行，执行如下命令，WARN属于正常，不影响使用。

   > npm install hexo-cli -g

2. 然后执行如下命令

   > npm install hexo --save

3. 确认是否成功

   > hexo -v

### 安装出现的问题

1. DeprecationWarning: fs.SyncWriteStream is deprecated

   - 原因：
     node和hexo插件的版本带来的问题：在node8.x的版本中，fs.SyncWriteStream被弃用了。

   - 解决，更新如下插件：

     > npm install hexo-fs --save
     > npm install hexo-deployer-git@0.3.1 --save
     > npm install hexo-renderer-ejs@0.3.1 --save
     > npm install hexo-server@0.2.2 --save
     >
     > npm audit fix

## 初始化Hexo

