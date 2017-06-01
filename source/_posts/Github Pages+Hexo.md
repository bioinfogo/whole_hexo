---
title: Github Pages + Hexo 构建静态blog
date: 2017-06-01 08:30:05
categories: Web

toc: true
---

# 配置过程
## 本地首次配置
本地配置node.js，以及git，这些直接在官网就可以下载，都是直接点击下一步安装。
本地配置git的用户名与邮箱：
```
git config --global user.name yourgitname
git config --global user.name yourgit@email.com
```
随后，为了与在线的github连接，需要配置ssh，并将生成的公钥粘贴到你的github账户中。
本地配置ssh
```
ssh-keygen -t rsa -C "yourgit@email.com"
```
成功后，在系统盘的user/your_user_name中找到.ssh文件夹，打开其中的id_rsa.pub文件，将所有内容复制。
接着，打开你的github 设置页页面，点击SSH and GPG keys，新建一个ssh key，将刚刚拷贝的内容粘贴过去即可。
这样的话，就可以在本地连接到你的github账号了，可以测试一下：
```
ssh -T git@github.com
```
如果显示成功的话，就说明配置完成了。

然后，在管理员命令行下，利用npm安装hexo
```
npm install hexo-cli -g
#等到安装成功后，运行：
npm install hexo --save
#测试是否安装成功：
hexo -v
```
成功后，可以开始配置hexo了。
```
#初始化：
hexo init
```
## 多终端的配置
如果想要在不同的终端都可以编写blog并上传，可以在多台电脑上配置git和node.js 等。
原理是：在第一台电脑上上传网站源代码到github 上，然后在各个电脑上克隆代码，修改之后渲染上传，然后将源代码再次保存在github上即可。
[方法](http://zhangnai.xin/2016/10/11/create-a-new-blog-dir/)
对于source，theme等文件夹，采用单独保存到一个repo 的方式进行。
也可以http://sufaith.com/2016/02/27/Hexo%E8%BF%81%E7%A7%BB/


## 注意事项
* 利用npm安装hexo时候，注意以管理员的身份进行，不然的话可能报错无法找到npm（其实是权限的问题）


# 问题及解决方案
## hexo module无法找到
在首次进行hexo g 的时候，报错：
> Cannot find module 'C:\Program FIles\Git\node_modules\hexo-cli\bin\hexo'

在网上查了半天，发现是由于这个module 在Git目录下没有找到导致的，解决方案如下：
在C盘中找到Users\_your_name_\AppData\Roaming\npm\node_modules\hexo-cli 这个文件夹
然后将其中的node_modules 文件夹拷贝到Git 安装目录下，即可。
## 本地server 无法访问
在我这里，是由于端口已经被占用导致的，更换端口即可。
```
hexo s -p 9966
```

## CNAME 大小写无法区分
在source下建立了CNAME文件，对于Pages进行了重定向，但上传到github后发现变成了小写。
因而导致了Github无法识别改文件，无法完成重定向。

解决方案：
先在Github上删除小写的cname文件
在你的clone 目录下，执行：
```
 git config core.ignorecase false
```
将ignorecase 设置为false，就可以区分大小写。但是在笔者的实践中，这样做无效。
估计是因为服务器端的信息还是没有更改。
最终问题的解决参考了[Yizhao He's Notes](http://1mhz.me/2015/hexo-deploy-case-sensitive/)
进入deploy_git/.git目录，修改其中的config文件，将ignorecase= ture 改为ignorecase= false
然后，删除整个deploy_git 文件夹，之后同步清空Github 上的信息，这一步很重要：
```
#在.deploy_git目录下
git commit -m 'clean all file'
git push
#回到上一目录，再次进行部署
cd ..
hexo clean
hexo deploy -g
```

