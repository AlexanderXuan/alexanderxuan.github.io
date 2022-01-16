Title: 项目部署之多服务器部署工具fabric
Date: 2021-12-10 15:30
Category: python小技巧
Tags: 项目部署, python, fabric
Author: Alexander Xuan
Summary: 项目部署之多服务器部署工具fabric介绍

### 需求

项目部署的时候有时会遇到同样的项目部署到很多服务器的问题，这个时候就需要重复将项目拉取到每个服务器，每次更新都要做一次这样的繁琐操作，所以需求就是多服务器部署项目时希望可以一次操作将所有的服务都进行项目拉取和部署。

这样的工具可能不止一个，但是目前比较熟悉的语言是python，所以还是希望使用一个python的工具来进行这样的操作，经过一点时间的调研，发现使用fabric可以比较简单的实现这样的功能，fabric的教程比较不好看，但是我们需要使用的功能也不算很难，所以暂时就先使用fabric。



### fabric使用简单例程

首先是安装fabric，fabric有三个版本，我们这里使用兼容python3的fabric3，它是兼容fabric1和fabric2的，安装方式如下：

```shell
pip install fabric3
```

首先fabric是可以实现直接执行本地命令的，这里还是要先介绍一下，因为有时可能会有本地直接提交测试，然后远程拉取更新部署的一条龙运行部署需求。

下面都是在本地新建python文件fabfile.py里进行编写的例子。

本地命令执行简单例程：

```python
from fabric import *

def prepare_deploy():
    with lcd('~'):
        local("ls")
        local("cd ~")
        local("ls | wc -l")

```

然后执行`fab prepare_deploy`

例程没什么特别的意义，只是切换目录到用户目录，然后执行了三个简单的命令，但这也就是说执行git相关命令，测试相关命令等。

远程连接及命令执行简单例程：

```python
from fabric.api import *


env.hosts = ['username@ip:port']
env.passwords = {
    'username@ip:port': 'password'
}

def prepare_deploy():
    with lcd('~'):
        local("ls")
        local("cd ~")
        local("ls | wc -l")


def prepare_deploy_remote():
    with cd('/data/path'):
        run("ls")
        # put('./test_fabric.txt', '/data3/xuan')	# sftp上传文件
        get('/data3/xuan/test_fabric.txt', './')	# sftp下载文件
```

然后执行`fab prepare_deploy_remote`

既可以在设置的远程服务器执行命令，包括git等命令。

env.hosts和env.passwords列表中可以添加很多的远程主机，执行程序时会逐个执行同样的命令。这样其实就可以实现多个服务器的同时部署了。

如果写了`if  __name__ == "main":`的话就不需要`fab function`了，直接运行程序即可。



在服务器非常多的情况下，如果使用git部署拉取到每一台的话，不想保存个人信息到部署机器或者不想将每台部署机器都添加到gitlab中的话，可能需要输入非常多次的git密码，所以其实可以在一台服务器上拉取，然后使用sftp传输方式来更新到其他服务器上，这样就可以实现只拉取一次，输入一次git密码的部署效果。

我们的开发服务器与部署服务器网络是互通的，所以其实本地开发完直接sftp推送到部署机器即可。

补充fabric通过tmux运行离线任务的例子：
```python
def prepare_deploy_remote():
    with cd('/data/path'):
        run("ls")
        # put('./test_fabric.txt', '/data3/xuan')	# sftp上传文件
        get('/data3/xuan/test_fabric.txt', './')	# sftp下载文件
```