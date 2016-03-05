---
layout: post
title: Jekyll本地搭建开发环境以及Github部署流程
subtitle: "使用Jekyll本地写博客，并部署到Github，绑定个人域名"
author: pizida
date: 2016-03-04 07:48:51 +0800
categories: technology
tag: 
   - 技术
---

* TOC
{:toc}


## 前言
  博客从wordpres迁移到Jekyll上来了，整个过程还是很顺利的。Jekyll是什么？它是一个简单静态博客生成工具，为什么是静态，因为它是不需要数据库的，直接编写通过<a href="http://wowubuntu.com/markdown/">markdown</a>编写静态文件，生成Html页面，它的优点是提升了页面的响应速度，并且让博主可以只专注于写文章，不用再去考虑如何排版（markdown语法很好的解决了排版问题）。那么为什么要迁移到Jekyll呢？不是因为跟风，也不是为了提高自我，完全就是瞎折腾！

## 本地搭建Jekyll
本次安装是在MacOs或Linux下进行的。

### 使用gem安装Jekyll
```ruby
gem install jekyll
```
这样就把jekyll安装完成了。
如果本地没有gem，那么需要先安装Ruby和gem，这里就不介绍了。

### 使用Jekyll创建你的博客站点
```ruby
jekyll new blog  #创建你的站点
控制台可以看见类似 `New jekyll site installed in /home/user/blog.`的提示
```

### 开启Jekyll服务
```ruby
cd blog    	 #进入blog目录,记得一定要进入创建的目录，否则服务无法开启
jekyll serve 	 #启动你的http服务 
```

本地服务开启后，Jekyll服务默认端口是4000，所以我打开浏览器，输入：<a href="http://localhost:4000">http://localhost:4000</a> 即可访问

### 使用Jekyll写博文
我们进入blog目录后，会发现Jekyl的结构如下：

```ruby
.
├ about.md
├ _config.yml
├ css
│   └ main.scss
├ feed.xml
├ _includes
│   ├ footer.html
│   ├ header.html
│   ├ head.html
│   ├ icon-github.html
│   ├ icon-github.svg
│   ├ icon-twitter.html
│   └ icon-twitter.svg
├ index.html
├ _layouts
│   ├ default.html
│   ├ page.html
│   └ post.html
├ _posts
│   └ 2016-03-04-welcome-to-jekyll.markdown
└ _sass
    
    ├ _layout.scss
    └ _syntax-highlighting.scss

```       
我们进入_post目录，撰写的markdown语法的博文都放在这里。默认会有一篇测试文章：2016-03-04-welcome-to-jekyll.markdown.
拷贝测试博文

```ruby
cp 2016-03-04-welcome-to-jekyll.markdown 2016-03-04-test.markdown
```

然后刷新一下浏览器、发现页面并没有变化.因为我们还没有通过Jekyll build去生成。
```ruby
jekyll build
```
默认情况下，服务会以前台的方式挂起，如果希望用后台进程运行服务，我们可以使用 `--detach`参数，缩写参数为`-B`(应该是Background的首字母)

```ruby
jekyll serve build --detach
```
或者

```ruby
jekyll serve build -B
```
**值得注意的是：如果用vagrant虚拟机去安装jekyll，那么启动服务时还需要加上-H参数，指定访问主机号为0.0.0.0，即`jekyll serve build -B -H 0.0.0.0`,否则vagrant下可能启动失败**

再查看首页，发现已经有两篇文章了！
好的，打开复制的 2016-03-04-test.markdown-test.markdown,可以看见开头有如下几个东东。

```ruby
---
layout: post
title:  "Welcome to Jekyll!"
date:   2016-03-04 10:52:19 +0100
categories: jekyll update
---
```

`layout`表示使用的是post布局，`title`是博客标题，`date`是自动生成的日期，`categories`是该文章生成html文件后存放的目录，可以去_site/jekyll/update下找到对应日期下面的html文档。当然你也可以只设置jekyll单一的目录，甚至是更多级别的目录，用空格分开即可。头信息设置完成后就可以书写正文了。

如果每次都输入这些头信息，还要去整理格式，那么一定很烦躁，这种重复性的东西我们就把它自动化，通过Rakefile去解决，它类似shell这样的脚本，可以使用交互模式。以下是我的Rakefile,可以复制后命名为Rakefile，放在站点根目录直接使用，也可以修改为适合自己的Rakefile：

```ruby
task :default => :new

require 'fileutils'

desc "创建新 post"
task :new do
  puts "请输入要创建的 post URL："
	@url = STDIN.gets.chomp
	puts "请输入 post 标题："
	@name = STDIN.gets.chomp
	puts "请输入 post 子标题："
	@subtitle = STDIN.gets.chomp
	puts "请输入 post 分类，以空格分隔："
	@categories = STDIN.gets.chomp
	puts "请输入 post 标签："
	@tag = STDIN.gets.chomp
	@slug = "#{@url}"
	@slug = @slug.downcase.strip.gsub(' ', '-')
	@date = Time.now.strftime("%F")
	@post_name = "_posts/#{@date}-#{@slug}.md"
	if File.exist?(@post_name)
			abort("文件名已经存在！创建失败")
	end
	FileUtils.touch(@post_name)
	open(@post_name, 'a') do |file|
			file.puts "---"
			file.puts "layout: post"
			file.puts "title: #{@name}"
			file.puts "subtitle: #{@subtitle}"
			file.puts "author: pizida"
			file.puts "date: #{Time.now}"
			file.puts "categories: #{@categories}"
			file.puts "tag: #{@tag}"
			file.puts "---"
	end
	exec "vi #{@post_name}"
end
```
如何使用Rake，输入一下命令：

```ruby
rake new
```
rake会启动交互模式，让你依次输入title，subtitle，author，categories，tag等信息，并为你创建好具有头信息的markdown文件。如下一样:

```shell
请输入要创建的 post URL：
testurl
请输入 post 标题：
testpost
请输入 post 子标题：
subtitle    
请输入 post 分类，以空格分隔：
jekyll
请输入 post 标签：
技术
```
我们查看`_post`目录，发现已经有一篇**2016-03-05-testurl.md**文章，打开看下

```ruby
---
layout: post
title: testpost
subtitle: subtitle
author: pizida
date: 2016-03-05 07:31:27 +0800
categories: jekyll
tag: 技术
---
```
已经为我们创建好头信息了。

本地搭建jekyll和写博文大致就是如此，有更多疑问可到官网<a href="https://jekyllrb.com/">https://jekyllrb.com/</a>或中文镜像站<a href="http://jekyllcn.com/">http://jekyllcn.com/</a>查阅资料。
***

## 利用gitHub搭建项目

### 创建我们的仓库
我们repository name设置为testblog，选择Publi仓库类型，勾选上初始化README文件，这个文件稍后需要用到。
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_3924717F-AF28-47BA-83C5-F4722090B464.png" alt="创建git仓库"/>

### 为仓库创建Github Pages
入库仓库后，选择**setting**
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_DF969E91-9438-4F57-BA3F-C09EC5F0E3D6.png" alt="创建Github Pages"/>

点击**Launch automatic page generator**
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_3F514A8A-3464-47B5-89AE-61FF6BC665B8.png" alt="生成pages"/>

然后编辑下标题和描述，选择模板，点击**Publish**
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_1.png"/>

如此，我们已经可以通过浏览器输入 http://username.github.io/testblog访问博客主页，注意**username**为你自己的账户名。

### 将本地jekyll代码部署到Github上的仓库
前面我们已经详细地说明如何搭建Jekyll，我们可以在本地开发测试，推送代码到仓库，发布到线上。




