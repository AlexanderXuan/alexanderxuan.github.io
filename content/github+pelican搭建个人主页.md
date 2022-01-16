Title: github与pelican搭建个人主页
Date: 2021-12-07 08:30
Category: 个人主页搭建
Tags: 个人主页, python, pelican, github
Author: Alexander Xuan
Summary: github与pelican搭建个人主页

### github与pelican搭建个人主页

就我自己而言，搭建个人主页主要是为了记录一些自己的随笔，然后发布到自己的个人主页上，写作工具是本地的markdown工具，所以不需要太复杂的功能，静态的网页足够了，只要可以直接解析我的markdown文件生成html进行展示就行，所以这里采用静态网页生成器来完成这个工作，而github则可以解析静态网页内容，直接生成我们的个人网站，而且网站内容还可以进行版本管理，更方便一些。



#### github个人主页

首先需要在github上创建一个`<username>.github.io`的空仓库，我这里使用master主分支来管理整个网站的生成源码，gh-pages分支管理网站显示内容，所以需要在仓库设置中找到pages选项，将网站解析source分支变更为gh-pages。这样稍等一下就可以解析我们的网站内容了。

#### 静态网页生成器

静态网页生成器其实包含很多的工具，各种语言的实现都有，像比较出名的有Ruby的jekell，go的hugo以及很多的js的生成器，这个[网站](https://staticsitegenerators.net/)列举了基本所有语言的所有静态网页生成器，我目前主要用python进行工作，所以就先选择一个我尽量熟悉一些的语言编写的工具，这里选择Pelican来作为静态网页生成器。需求就是我写md文件，它帮我生成可用于github解析的html内容，我直接上传github就可以更新我的主页。

##### pelican配置

首先需要安装相关python包，python环境的安装这里不做解释：

```pip install pelican ghp-import Markdown```

pelican是静态网站生成器，Markdown是解析md文件需要的包，ghp-import主要是为了方便同步网页内容分支。

然后将创建的空仓库拉取到本地：

```git clone <https://GitHub.com/username/username.github.io> blog```

然后在blog目录下执行：

```pelican-quickstart```

这里需要做一些配置，但是都是比较简单的配置，可以按如下的配置进行选择：

```
This script will help you create a new Pelican-based website.

Please answer the following questions so this script can generate the files
needed by Pelican.

> Where do you want to create your new web site? [.]  
> What will be the title of this web site? My blog
> Who will be the author of this web site? username
> What will be the default language of this web site? [en]
> Do you want to specify a URL prefix? e.g., http://example.com   (Y/n) n
> Do you want to enable article pagination? (Y/n)
> How many articles per page do you want? [10]
> What is your time zone? [Europe/Paris] UTC
> Do you want to generate a Fabfile/Makefile to automate generation and publishing? (Y/n) y
> Do you want an auto-reload & simpleHTTP script to assist with theme and site development? (Y/n) y
> Do you want to upload your website using FTP? (y/N) n
> Do you want to upload your website using SSH? (y/N) n
> Do you want to upload your website using Dropbox? (y/N) n
> Do you want to upload your website using S3? (y/N) n
> Do you want to upload your website using Rackspace Cloud Files? (y/N) n
> Do you want to upload your website using GitHub Pages? (y/N) y
> Is this your personal page (username.github.io)? (y/N) y
Done. Your new project is available at /Users/username/blog
```



既可生成编译网站的源码，文件结构如下：

```
pelicanblog
├── Makefile
├── __pycache__
│   ├── pelicanconf.cpython-38.pyc
│   └── publishconf.cpython-38.pyc
├── content
│   └── first.md
├── output
│   ├── archives.html
│   ├── author
│   ├── authors.html
│   ├── categories.html
│   ├── category
│   ├── feeds
│   ├── index.html
│   ├── pelican_course.html
│   ├── tag
│   ├── tags.html
│   └── theme
├── pelicanconf.py
├── publishconf.py
└── tasks.py
```

两个conf文件是配置文件，pelicanconf.py是主要配置文件，始终有效，publishconf.py需要使用命令```pelican content -s publishconf.py```才生效，之后只要在content里面添加md内容即可，我这里的output是已经生成的网页内容。

更新content中的内容后在这个文件夹下执行```make html ```就可以生成output中的内容。

github将主分支推送上去之后本地执行```ghp-import output -b gh-pages```即可生成gh-pages分支，然后切到新分支将分支推送到github即可，然后就可以访问username.github.io来访问自己的主页了。



注意md中的内容需要添加一些信息以供解析使用：

```
Title: 用Pelican+GitHubPages搭建静态博客
Slug: static-blog
Date: 2018-10-03 09:20
Category: posts
Tags: blog,pelican,ipynb
Author: Zodiac Wang
Summary: 本文介绍如何搭建静态博客
```

这些字段的含义为：

- Title——文章标题
- Slug——文章在服务器上的路径，如 slug 是 first-post，而博客地址是 jupyter-blog.com, 则文章地址为 http://www.jupyter-blog.com/first-post
- Date——文章发布日期
- Category——文章的类别——可以是任何东西
- Tags——文章的标签
- Author——作者名字
- Summary——文章摘要



