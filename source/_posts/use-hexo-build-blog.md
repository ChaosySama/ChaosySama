---
title: Hexo 搭建个人博客
comments: true
categories: Tech
tags: [blog,hexo,next]
abbrlink: c9d3f490
date: 2019-09-14 14:04:32
---

作为程序猿，总会有一些经验总结，想要写下来，方便以后自己回顾。但是本地的话，换个环境或者换个端，就没法即时查看了。之前一直使用的是 `oneNote`，多端同步，除了一些常见的诟病，其实还是很方便的，不过对于和别人分享来说，实在是不太方便。于是就想到用 `hexo`  +  `github page` 搭建一个免费的博客。既能写，又能定制，还能分享，最重要的是免费。

<!--more-->

## hexo 的搭建

网上已经有各种从零搭建教程，具体可以参考：

[超详细Hexo+Github博客搭建小白教程][hexoUrl0]

[GitHub+Hexo 搭建个人网站详细教程][hexoUrl1]

上面两个其实内容差不多，我也是参考这两个教程搭建的。中间有些步骤想跳就跳，比如绑定域名啥啥的，初期的话可以先用起来，等到后面~~弃坑了，就不用麻烦了有新的需求了，再加上也不迟嘛。

## 主题的选择和优化

搭建完成后，选择了一个比较大众的主题 [Next][nextUrl] ，结合 Hexo 主题优化的教程：

[Hexo+Next个人博客主题优化][nextOptimize]

加一些方便的功能，修改一些样式之类的，总之自己觉得好用就好。拿不准的功能可以加上去看看，不好可以删嘛。

于是花点时间捣鼓捣鼓就变成现在这个样子了，不想花时间，直接开始写也可以，看各人喜好了。

## 文章的编写和发布

教程里写的很明白了，这里就不多描述了，总结下来就是下面几步：

```shell
hexo n "new article"
vim source/_posts/new-article.md
hexo g && hexo s
```

就可以在本地浏览了。

然后

```shell
hexo d
```

就可以去 github 上浏览了。（需要过几分钟）



[hexoUrl0]: https://zhuanlan.zhihu.com/p/35668237
[hexoUrl1]: https://zhuanlan.zhihu.com/p/26625249
[nextUrl]: https://theme-next.iissnan.com/
[nextOptimize]: https://www.jianshu.com/p/efbeddc5eb19


