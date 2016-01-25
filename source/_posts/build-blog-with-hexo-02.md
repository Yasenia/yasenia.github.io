---
title: 使用 hexo 搭建个人博客 02
date: 2016-01-25 22:18:09
tags: [hexo, blog, github]
---

在我的上一篇文章[《使用 hexo 搭建个人博客 01》](http://www.jianshu.com/p/bdb333a4391d)中，介绍了使用 hexo 搭建博客的基本知识。本篇将主要介绍如何部署 hexo 到 Github Pages 服务，以及如何更换 hexo 的主题。

---

# 6. 部署 blog 为 Github Pages
[github](https://github.com) 是目前全球最大的代码托管仓库。Github Pages 是由 github 提供的一个免费的静态页面托管服务，只需要我们拥有一个 github 账号即可。
要将 hexo 生成的静态网页部署为 Github Pages，详细步骤如下：
1. 使用用户名为 `${username}` 的 github 账号，创建一个 repository，命名为 `${username}.github.io`。此处仓库名必须为该格式，否则无法使用 Github Pages 服务。
2. 使用 vim 或其他编辑器编辑 `~/blog` 目录下的 `_config.yml` 文件。在 shell 中执行：
``` shell
~/blog$ vim _config.yml
```
修改配置文件 deploy 相关配置：
``` yams
# ... other configs
deploy:
    type: git
    repository: git@github.com:${username}/${username}.github.io.git
    branch: master
```
这里的 repository url 可以在 github 仓库页面复制（本文使用的是ssh方式）。另外需要注意，yaml 配置文件对格式有严格的要求，必须保证缩进，且 `:` 后一定要加上空格，否则配置可能失效。
3. 生成 ssh 密钥对，在 shell 中执行：
``` shell
~/blog$ cd ~
~$ ssh-keygen -t rsa -C ${email}
```
此处 `${email}` 建议使用你的真实可用邮箱。接下来终端会分别提示你输入生成密钥目标文件夹，密钥使用密码以及确认密码。可以直接点击 `Enter` 使用缺省值。
执行成功后，缺省会生成一个 `~/.ssh` 目录，里面包含了 `id_rsa` 与 `id_rsa.pub` 两个文件，分别保存了 ssh 私钥与公钥。
4. 用任意文本编辑器打开 `~/.ssh/id_rsa.pub` 文件，拷贝里面的文本内容。登入 github，进入 `${username}.github.io` 仓库的 "Settings"，切换到 "Deploy keys" 选项卡，点击 "add deploy key" 按钮，此处 "title" 栏可以任意填写，"key" 栏粘贴本机生成的 ssh 公钥内容，勾选 "Allow write access" 选项，保存即可。
5. 安装 hexo-deployer-git 插件，在 shell 中执行：
``` shell
~$ cd blog
~/blog$ npm install hexo-deployer-git --save
```
6. 生成并部署 hexo 静态页面，在 shell 中执行：
``` shell
~/blog$ hexo generate
~/blog$ hexo deploy
```
这两条指令也可以等价简写为：
``` shell
~/blog$ hero g -d
```
成功执行后，用浏览器访问 `http://${username}.github.io`，就可以看到托管到 Github Pages 上的博客了。

---

# 7. 选择一款适合你的主题
hexo 默认的主题样式是 landscape，也许你希望使用更多样、更个性化的主题风格。在 github 上有许多开源的 hexo 主题，你只需要把它们克隆到 `~/blog/themes` 目录下，并在 `~/blog/_config.yml` 的 `theme` 属性中设置你希望使用的主题，然后重新生成页面部署即可。
这里推荐几个在 github 上 star 数较高的主题：
* [Yilia](https://github.com/litten/hexo-theme-yilia)
* [NexT](https://github.com/iissnan/hexo-theme-next)
* [Tranquilpeak](https://github.com/LouisBarranqueiro/hexo-theme-tranquilpeak)
* [Modernist](https://github.com/orderedlist/modernist)
* [Landscape plus](https://github.com/xiangming/landscape-plus)

当然，如果你自己就是一名优秀的前端工程师，甚至可以自己定制一份属于自己独一无二的 hexo-theme，你还可以把你的作品开源到 github 上供更多的 hexo 用户使用与学习。

---

# 8. 开始你的博客之旅
这里已经介绍了最基本的 hexo 知识与配置。希望能为每一个想要拥有自己博客的朋友们提供一些参考和帮助。
开始属于你的博客之旅吧！
