# GitHub Page + Hexo 搭建博客

## 准备

1. Node.js 环境
2. Git

### Node.js环境

1. 下载地址：<https://nodejs.org/dist/>，根据系统选择合适的版本
2. 安装完成后，确认版本信息
   - `node -v`
   - `npm -v`

> node.js版本升级请参考：<https://blog.csdn.net/tlbaba/article/details/79412433>

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

在当前目录下，命令行执行如下操作：

> hexo init

然后执行：

> hexo install

npm将会自动安装你需要的组件，只需要等待npm操作即可。

## Hexo初体验

1. 生成静态文件

   > hexo generate 或者 hexo g

2. 运行服务

   > hexo server 或者 hexo s

   启动服务后，会有如下提示：

   > INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.

   在浏览器中打开`http://localhost:4000/`，会看到如下页面：

   ![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/blog/blog/hexo.png)



## Hexo配置

Hexo配置文件是yaml语法格式，在配置的过程中保证语法正确。可以在`_config.yml`中修改大部分配置。

### Site 网站

| 参数        | 描述                                |
| ----------- | ----------------------------------- |
| titile      | 网站标题                            |
| subtitle    | 网站副标题                          |
| description | 网站描述                            |
| author      | 您的名字                            |
| language    | 网站使用的语言                      |
| timezone    | 网站时区。Hexo 默认使用您电脑的时区 |

### URL 网址

| 参数              | 描述                     | 默认值                      |
| ----------------- | ------------------------ | --------------------------- |
| url               | 网址                     |                             |
| root              | 网站根目录               |                             |
| permalink         | 文章的永久链接格式       | `:year/:month/:day/:title/` |
| permalink_default | 永久链接中各部分的默认值 |                             |

> 如果您的网站存放在子目录中，例如 `http://yoursite.com/blog`，则请将您的 `url` 设为 `http://yoursite.com/blog` 并把 `root` 设为 `/blog/`。



### Directory 目录

| 参数         | 描述                                                         | 默认值         |
| :----------- | :----------------------------------------------------------- | :------------- |
| source_dir   | 资源文件夹，这个文件夹用来存放内容。                         | `source`       |
| public_dir   | 公共文件夹，这个文件夹用于存放生成的站点文件。               | `public`       |
| tag_dir      | 标签文件夹                                                   | `tags`         |
| archive_dir  | 归档文件夹                                                   | `archives`     |
| category_dir | 分类文件夹                                                   | `categories`   |
| code_dir     | Include code 文件夹                                          | downloads/code |
| i18n_dir     | 国际化（i18n）文件夹                                         | `:lang`        |
| skip_render  | 跳过指定文件的渲染，您可使用 [glob 表达式](https://github.com/isaacs/node-glob)来匹配路径。 |                |

### Writing 文章

| 参数              | 描述                                                         | 默认值    |
| :---------------- | :----------------------------------------------------------- | :-------- |
| new_post_name     | 新文章的文件名称                                             | :title.md |
| default_layout    | 预设布局                                                     | post      |
| auto_spacing      | 在中文和英文之间加入空格                                     | false     |
| titlecase         | 把标题转换为 title case                                      | false     |
| external_link     | 在新标签中打开链接                                           | true      |
| filename_case     | 把文件名称转换为 (1) 小写或 (2) 大写                         | 0         |
| render_drafts     | 显示草稿                                                     | false     |
| post_asset_folder | 启动 [Asset 文件夹](https://xuanwo.io/2015/03/26/hexo-intor/asset-folders.html) | false     |
| relative_link     | 把链接改为与根目录的相对位址                                 | false     |
| future            | 显示未来的文章                                               | true      |
| highlight         | 代码块的设置                                                 |           |

### Category & Tag

| 参数             | 描述     | 默认值          |
| :--------------- | :------- | :-------------- |
| default_category | 默认分类 | `uncategorized` |
| category_map     | 分类别名 |                 |
| tag_map          | 标签别名 |                 |

### 日期 / 时间格式

Hexo 使用 [Moment.js](http://momentjs.com/) 来解析和显示时间。

| 参数        | 描述     | 默认值       |
| :---------- | :------- | :----------- |
| date_format | 日期格式 | `MMM D YYYY` |
| time_format | 时间格式 | `H:mm:ss`    |

### Pagination 分页

| 参数           | 描述                                | 默认值 |
| :------------- | :---------------------------------- | :----- |
| per_page       | 每页显示的文章量 (0 = 关闭分页功能) | `10`   |
| pagination_dir | 分页目录                            | `page` |

### Extensions 扩展

| 参数   | 描述                                |
| :----- | :---------------------------------- |
| theme  | 当前主题名称。值为`false`时禁用主题 |
| deploy | 部署部分的设置                      |

### Deployment

首先要为自己配置身份信息，配置全局的git信息，如下

> git config --global user.name "yourname"
> git config --global user.email "youremail"

在`_config.yml`文件中找到`Deployment`，做如下修改：

> deploy:
> 	type: git
> 	repo: git@github.com:yourname/yourname.github.io.git
> 	branch: master

如果使用git方式进行部署，执行`npm install hexo-deployer-git --save`来安装所需的插件。

在当前目录打开命令行，输入如下命令进行部署。

> hexo deploy 或者 hexo d

推送到git仓库后，就可以通过git pages进行访问。



## 添加新文章

打开Hexo目录下的`source`文件夹，所有的文章都会以md形式保存在`_post`文件夹中，只要在`_post`文件夹中新建md类型的文档，就能在执行`hexo g`的时候被渲染。 

在命令行执行如下命令会生成对应的文章：

> hexo new "My New Post" 或者 hexo n "My New Post"

新建的文章头会添加如下一些yml信息，如下所示：

> ---
>
> title: hello-world   //在此处添加你的标题。
> date: 2017-11-7 08:55:29   //在此处输入你编辑这篇文章的时间。
> categories: Code   //在此处输入这篇文章的分类。
> toc: true  //在此处设定是否开启目录，需要主题支持。
>
> ---

## 更换Hexo主题

可以在[此处](https://github.com/hexojs/hexo/wiki/Themes)寻找自己喜欢的主题下载所有的主题文件，保存到Hexo目录下的 `themes`文件夹下。然后再`_config.yml`文件中修改如下：

> `#` Extensions
>
> `##` Plugins: http://hexo.io/plugins/
>
> theme: landscape //themes文件夹中对应文件夹的名称

然后先执行`hexo clean`，然后重新`hexo g`，并且`hexo d`，很快就能看到新主题的效果了~



## 更换域名

### 注册域名

首先，需要注册一个域名。在中国的话，`.cn`全都需要进行备案，如果不想备案的话，请注册别的顶级域名。 然后，我们需要配置一下域名解析。推荐使用DNSPod的服务，比较稳定，解析速度比较快。在域名注册商出修改NS服务器地址为：

> f1g1ns1.dnspod.net
> f1g1ns2.dnspod.net

### hexo域名配置

在自己本地的hexo目录下的`source`文件夹中，新建一个`CNAME`文件*（注意，没有后缀名。）*，内容为`yourdomin.xxx`。然后再执行一下`hexo d -g`，重新上传自己的博客。

### GitHub-pages绑定自定义域名

在这一系列的操作中，包括修改DNS服务器，设置A解析等等，都需要一定的时间。短则10分钟，长则24小时，最长不会超过72小时。如果超过72小时，请检查自己的配置过程，或者修改自己本地的DNS服务器。