---
title: 使用Travis CI自动化部署博客
comments: true
tags:
  - blog
  - hexo
  - travis
  - ci
categories: Tech
abbrlink: 56331
date: 2020-03-24 14:01:28
---

> 上一篇文章解决了写文章时候的各种琐碎的事情的痛点，这篇我们来解决一下剩下来的痛点：文章的部署，也就是环境搭建和一系列`hexo`相关操作。
>
> 同样先想一下我们的需求：
>
> - 不需要每次写完文章都用一系列`hexo`操作来部署
> - 在新的环境中能快速修改并提交部署，比如github上直接修改
>
> 也就是说，我们需要一个平台来帮我们监控代码提交，然后自动执行`hexo`的一系列操作，我们只需要提交`.md`文件到仓库即可。
>
> 这个需求就是所谓的持续集成(Continuous Integration)，简称CI

那么接下来，我们就研究一下怎么给我们的博客用上CI来简化我们的部署操作。

<!-- more -->

## 仓库构成

为了避免混乱，先说一下我们目前的仓库构成。首先，我们为了白嫖，并且主域名访问，所以创建了一个github_username.github.io的同名仓库，这个仓库下的master分支会被映射到外网访问。

这个仓库目前有3个分支：`master`，`hexo`，`img`

- hexo分支：我们存放hexo相关的配置，文章的仓库，也是我们下面使用CI主要用到的仓库。
- master分支：使用hexo生成，部署到的分支，也是github pages用来访问的分支。
- img分支：[上篇文章](https://chaosysama.github.io/20200318/use-typora-and-picgo-write-blog) 中创建的图床分支。

## Travis CI

提到CI，市面上有各种类似的产品，如果大家有兴趣可以自行研究一下。这里主要介绍一下[Travis CI](https://travis-ci.org/)，它的好处是，可以关联github仓库，很方便的设置需要持续集成的仓库。并且配置简单易懂，界面清爽，最重要的是免费！~~对，又可以白嫖了~~

#### 关联github账号

打开Travis CI的页面，点击Sign Up，就可以授权关联自己的github账号，之后会有一段时间用来拉取你的所有仓库，耐心等待即可。

#### 准备travis用的github token

在github页面，点击头像，选择Settings -> Developer settings -> Personal access token -> Generate new token。在Note栏中填写travis_ci用来标记这个token是给travis用的，下面的权限选择repo的所有就可以，其他都不用选择，travis也用不到（越少的权限越安全）。

![image-20200324112117866](https://raw.githubusercontent.com/ChaosySama/ChaosySama.github.io/img/image-20200324112117866.png)

之后就是生成token，复制页面中的token备用（注意，token只显示一次，刷新之后就看不到了，不过如果忘记或者没有复制，可以点击Regenerate token来重新生成）

#### travis中关联仓库&设置

在travis的dashboard页面中，点击左侧`+`号，关联自己的仓库。在需要的仓库右侧开启即可。开启后点击旁边的`Settings`按钮。

在`General`里把`Build pushed pull requests`关闭，不关也可以，这个用不上。

在`Environment Variables`里定义以下几个变量，方便之后配置使用：

- GIT_REPO_TOKEN: 上面生成的token
- GIT_NAME: github账号的用户名
- GIT_EMAIL: github账号的邮箱
- GIT_REPO: 需要操作的仓库的地址，用来`git clone`使用的，在哪找就不多说了，注意需要把前面的`http://`删除

这里的Branch都默认`All branches`即可，后面`DISPLAY VALUE IN BUILD LOG`保持关闭状态，这样不会有泄露信息的风险，之后点击`Add`按钮添加变量。

![image-20200324135440300](https://raw.githubusercontent.com/ChaosySama/ChaosySama.github.io/img/image-20200324135440300.png)

#### hexo分支的配置

travis需要根据`.travis.yml`文件来配置环境和执行我们需要的操作。

于是我们可以在仓库的`hexo`分支的根目录，创建一个配置文件`.travis.yml`，内容如下（可参考）：

```yml
language: node_js # 指定语言环境
node_js:
  - 10 # use nodejs v10 LTS
dist: trusty # 指定系统版本。trusty 是指 Ubuntu 14.04 发行版的名称
sudo: false # 是否需要 sudo 权限
cache: npm # 是否缓存npm相关文件，建议缓存

branches:
  only:
  - hexo # 表示只在hexo分支监控和执行travis构建

before_install:
  - export TZ='Asia/Shanghai' # 设置时区，配合下面的commit msg中的时间

install:
  - npm install # script执行前的操作，这里用来根据package.json安装环境依赖

script:
  - hexo clean
  - hexo generate

after_success:
  - git clone https://${GIT_REPO} .deploy_git
  - cd .deploy_git
  - git checkout master
  - cd ../
  - mv .deploy_git/.git/ ./public/   # 这一步之前的操作是为了保留master分支的提交记录，不然每次git init的话只有1条commit
  - cd ./public
#   - git init
  - git config user.name "${GIT_NAME}"
  - git config user.email "${GIT_EMAIL}"
  - git add .
  - git commit -m "Travis CI Auto Builder at `date +"%Y%m%d %H:%M"`"
  - git push --force --quiet "https://${GIT_REPO_TOKEN}@${GIT_REPO}" master:master

# deploy:
#   provider: master
#   skip-cleanup: true
#   github-token: $GIT_REPO_TOKEN
#   keep-history: true
#   on:
#     branch: hexo
#   local-dir: public

notifications:
  email: true
```

下面解释一下文件中注释掉的部分：

- deploy配置项

```yml
deploy:
    provider: gh-pages
    skip-cleanup: true
    github-token: $GIT_REPO_TOKEN
    keep-history: true
    on:
    branch: hexo
    local-dir: public
```

   这个是一开始踩坑的时候官网推荐的配置`yml`文件的方法，但是经过验证，这个只适用于非同名根仓库（即仓库名<span style="color: red">不是</span>github_username.github.io的仓库），而我们这个不属于这个范围，所以不能用这个配置项。如果是非同名仓库来做博客的话，需要在仓库的设置中开启github page，然后再用上面的配置即可。

   我们的仓库属于同名仓库，所以得用after_success方法，模拟手动push的方式来执行。

- after_success中的`- git init`

   如果仓库的提交记录看起来不干净，不想继承之前的提交记录 ~~想重新做人~~ 怎么办？就可以用这条来从头开始你的commit记录。

   这里如果需要使用的话，需要取消注释并且将下面几行注释掉:
   
```yml
   - cd ../
   - mv .deploy_git/.git/ ./public/
   - cd ./public
```

## 提交测试

`.travis.yml`文件配置完后，就可以push一波，看看travis那边的执行情况了。

```shell
git add .travis.yml
git commit -m 'test'
git push
```

多刷新几下可以看到，已经在构建中了。

![image-20200324142319011](https://raw.githubusercontent.com/ChaosySama/ChaosySama.github.io/img/image-20200324142319011.png)

如果构建没什么问题的话，大概1分钟不到，就构建完了，构建成功或者失败都会发邮件到邮箱中。

如果构建失败，可以查看Current选项卡下方的`Job log`，会有报错信息。

成功之后，去仓库的master分支，就可以看到commit记录中多了刚刚构建的一条。

大功告成。

## travis接入后的使用

既然travis已经成功接入了我们的博客，那么以后我们写完文章就可以更方便了，下面是大概的流程：

```shell
hexo n test-travis # 使用hexo n是为了创建hexo相关的头部，有时间，类别等信息可以自己编辑
typora source/_posts/test-travis.md # 用typora一顿写
git add .
git commit -m 'new test-travis.md' # 这里的commit msg是提交到hexo分支的
git push
```

然后就可以坐等邮件通知，之后刷新几次博客页面就能看到刚才的文章已经发布上去了。

大家快试试看！