---
title: 使用 hexo 搭建个人博客 01
date: 2016-01-25 10:40:28
tags: [hexo, blog, github]
---

# 1. hexo 概览
根据 [hexo](https://hexo.io) 官方的描述，它是一个快速、简洁且高效的博客框架。

<!-- more -->

* hexo 是基于 [nodeJs](https://nodejs.org/en/) 生成网页的，这使得它的渲染速度相比同类型的静态博客框架 [jekyll](http://jekyll.bootcss.com) 等要迅速的多（后者基于 [ruby](http://www.ruby-lang.org/en/) 进行页面生成）。
* hexo 良好的集成了 [Github Pages](https://pages.github.com) 。这使得用户可以轻松的将自己的博客部署到 [github](https://github.com) 之上，享受免费、稳定且无限流量的服务，而不需要搭建自己的服务器。
* hexo 提供的完整的 [Markdown](http://daringfireball.net/projects/markdown/) [语法](https://github.com/othree/markdown-syntax-zhtw)支持，这使得用户可以更专注于编写文章的内容而非把精力花费在调整修改令人头疼文本格式。
* hexo 支持自定义主题，你也可以在 github 上找到许多优秀的开源的 hexo 主题应用于你的博客之中。


---

# 2. 环境配置
* nodeJs
由于 hexo 是采用 nodeJs 来生成静态网页的，因此想要使用 hexo，nodeJs 是必需的。登入 nodeJs 官网，你可以根据自己的系统，选择合适的版本进行安装。
* git
如果你希望将自己博客中的文章部署到 github 上，那么 [git](http://git-scm.com/download/) 是必不可少的。通常而言，OS X 与绝大多数 Linux 发行版都内置了 git。如果你的机器是 Windows 系统，请访问官网下载安装最新版本的 git，并将 `$GIT_HOME/bin` 路径添加到系统的环境变量当中。

---

# 3. 安装 hexo
nodeJs 安装成功后，我们可以很方便的通过 node 包管理器（npm）来下载安装 hexo 及其所依赖的插件。
在 shell 中执行指令：
``` Bash
~$ npm install hexo-cli -g
```
上述指令执行成功后，hexo 客户端程序就已经在你的机器上成功安装了。

---

# 4. 初始化你的 blog
在 shell 中执行指令：
``` Bash
~$ hexo init blog
```
hexo 会在当前目录下生成一个名为 `blog` 目录（当然你也可以换成其它你喜欢的名称），此博客相关的配置、主题与文本内容等都在此目录下由 hexo 进行管理。
接下来，在 shell 中执行:
``` Bash
~$ cd blog
~/blog$ npm install
```
这会将你的工作目录切换到 `~/blog` 之下，并根据该目录下的 `package.json` 配置安装 hexo 所需的依赖包。

通常，hexo 会自动生成一篇名为 Hello World 的缺省博客，它的位置为：
 `~/blog/source/_posts/hello-world.md`
在 shell 中执行以下两条指令：
``` Bash
~/blog$ hexo generate
~/blog$ hexo server
```
第一条指令使 hexo 根据 `~/blog` 目录中的配置以及文本内容生成相应的网页文件。你可以在 `~/blog/public` 目录下看到生成的 html、css等静态文件。第二条指令则是在本机的 `4000` 端口启动 hexo 服务。
成功执行上述指令后，打开浏览器，访问 `localhost:4000`，如果你能看到缺省情况下的 hexo 博客页面。则说明你的 hexo 博客已经搭建成功了。如果你希望退出服务，输入 `ctrl + C` 即可。

---

# 5. 创建和编辑文章
当你完成上述步骤后，本地的 hexo 已经基本搭建完成了。
现在，我们希望新建一篇名为 `Hello Hexo` 的文章。在 shell 中执行：
``` Bash
~/blog$ hexo new "Hello Hexo"
```
这样，我们就完成了文章的新建，我们可以用 vim 或其他编辑器打开位于 `~/blog/source/_posts` 目录中的 `Hello-Hexo.md` 文件。在 shell 中执行：
``` Bash
~/blog$ vim source/_posts/Hello-Hexo.md
```
打开文章后，我们可以看到如下内容：
> \---
> title: Hello Hexo
> date: 2016-01-25 09:12:21
> tags:
> \---

在两个 `---` 之间的是 yaml 格式的文章配置，其中可以指定文章标题、日期、标签等。
在第二个 `---` 之后，我们可以使用 Markdown 语法编写这篇博客正文。
文章编写完成后，重新生成网页并启动 hexo 服务，在 shell 中执行：
``` Bash
~/blog$ hexo generate
~/blog$ hexo server
```
现在，重新访问 `localhost:4000`，是不是已经看到你的新博客了呢？