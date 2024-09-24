---
title: GithubPages + Hugo 搭建博客
description: How do I create my blog easily
slug: build-my-blog
date: 2024-09-24 00:00:00
image: cover.jpg
categories:
  - Other
tags: 
weight: 0
---
封面来自KiTA的[墨染の桜](https://www.pixiv.net/artworks/92462879)

---
# 为什么选择更换博客
本人上一个博客大约在2023年3到4月左右部署，使用的是Typecho框架下的Joe主题，域名在Namesilo购买，服务器算是白嫖的同学的（甚至证书和cdn之类的也是他帮忙搞的），他的个人博客贴在这里：[Glucy2的个人博客](https://glucy2.com/)。总而言之，感激不尽。    
这个博客平稳运营了两年，但是由于本人的摆烂行径，在两年内除了一篇初始博客之外并没有写其他文章，几乎接近荒废。最近重拾学习，准备重新开始写博客。  
两年前选择框架时，选择了动态博客是因为支持评论系统，但是由于不好意思继续白嫖服务器，顺便薅薅github羊毛，准备更换到GithubPages做静态博客。   
然后本人发现了静态博客也可以通过第三方的托管与插件实现评论系统，最后的阻碍也可以被解决，便最终决定重新开始。  
# 框架选择
静态博客框架在简单查询后，Jekyll，Hexo，Hugo 最终加入决赛圈。
首先是Hexo和Hugo的比较，最终结论是两者功能都比较相似，但是Hugo在生成时较Hexo更加快速，所以Hexo被我放弃了。
然后是Jekyll 和 Hugo，经过既不仔细也不严谨的搜索后，选择了Hugo，理由是部署流程较为简单不需要过多操心。
## 主题选择
主题使用Stack，来源于[Hugo的官方主题列表](https://themes.gohugo.io/)，网站按Github的star数排序，简单浏览前几个主题后选择比较对眼的stack。
从stack的[快速开始仓库](https://github.com/CaiJimmy/hugo-theme-stack-starter)fork到我的账户并命名为`usrname.github.io` 后clone到本地，就完成了基本部署。
在settings-->pages设置中选择`gh-pages` 分支就可以浏览主题模板了。
# 一些小主题更改
## 基本设置
在`config.toml` 中，更改如下
```toml
# Change baseurl before deploy
baseurl = "https://foth0626.github.io"
languageCode = "en"
title = "FOTH0626"


defaultContentLanguage = "en"

hasCJKLanguage = true

[pagination]
pagerSize = 20

ignoreFiles = [
"content/post/Templates/*"
]
```
我来解释一下我的更改
- `baseurl`  应该被解析到个人的域名（即`usrname.github.io`）
- `title` 会成为你的名字
- `hasCJKLanguage` 开启后会正确识别CJK字符，对于中文写作者来说必开
- `pagerSize` 决定一页显示多少篇博文
- `ignoreFiles` 是因为个人的写作软件设置，稍后解释
## 个人信息（左栏）
在`menu.toml` 中更改如下
```toml
[[social]]
identifier = "github"
name = "GitHub"
url = "https://github.com/FOTH0626"
 
[social.params]
icon = "brand-github"

[[social]]
identifier = "bilibili"
name = "Bilibili"
url = "https://space.bilibili.com/351177307"

[social.params]
icon = "brand-bilibili"

[[social]]
identifier = "mail"
name = "Gmail"
url = "mailto:fothli0626@gmail.com"

[social.params]
icon = "mail"
```

- `identifier`为唯一标识符
- `name`为网页中图标名称
- `url` 表示图标应该解析到的位置
- `icon` 代表图标，本人从[tabler](https://tabler.io/icons)寻找icon后复制到本地的`asset/icons` 文件夹下，这是默认的icon查找位置，需要名称与文件名对应
- ---
在`params.toml` 更改如下
```toml
favicon = "/heart-code.png"

subtitle = "小小的人偶，想要摘月亮"

[sidebar.avatar]
src = "img/foth.jpg"

[comments]
enabled = false
```
- 将网页的标签页icon更改，默认目录为`root/static` 
- 更改个人签名（来自Mygo二创[十万客服](https://www.bilibili.com/video/BV1Ry421i7jL)）
- 更改个人头像（本人多个平台同头像，头像来自于本人非常喜欢的东方up主kk的一篇专栏中东方颜百选的预览图）
- 关闭评论功能（虽然本来就没有配置，但是最底下有评论系统会导致不必要的误解，而且看了不爽），可能在不久的未来配置（大概率参考此篇[博客](https://www.pseudoyu.com/zh/2024/07/22/free_commenting_system_using_remark42_and_flyio/)）
## 更改url逻辑
`permalinks.toml`
```toml
post = "/article/:slug/"
```
文章的url会被创建为/article/name，更好区分
## 文章目录设置
```toml
[tableOfContents]
endLevel = 5
ordered = false 
startLevel = 1
```
现在目录会从一级标题到五级标题的无序列表排列

# 写作环境
我在之前提到关于这个设置是因为本人的写作软件的问题
```toml
ignoreFiles = [
"content/post/Templates/*"
]
```
现在我来详细解释，本人的写作软件为Obsidian，它有许多的插件，而我选择了一个名为Templater的插件用来快捷生成满足hugo前置格式的markdown文章，它本身需要一个模板但是模板文章明显不可以作为博客，所以我选择无视这个文件。我的模板如下
```markdown
---
title: ""
description: ""
slug: ""
date:
  <% tp.date.now("YYYY-MM-DD HH:mm:ss") %>
image:
categories:
tags:
weight: 0
---
```
最有效的是它会记录下我的博客文件创建日期，而且我可以在软件通过下拉框选择文件与tag，而不需要手动输入。
# 个人域名，CNAME与DNS
个人域名购买自Namesilo，一年续费五六刀，相当便宜。在githubpages设置中添加cname即可正确解析到foth0626.github.io  
DNS还没配，等待后续更新。