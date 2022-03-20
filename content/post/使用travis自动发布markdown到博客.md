---
title: "使用travis自动发布markdown到博客"
date: "2019-03-30T10:56:47+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - github
toc: true
---
如何使用github pages和github actions构建静态个人博客站点

<!--more-->

### update at 2021
更新：github开放action功能后,travis-ci已经没有必要了,目前博客使用zhimoe仓库管理源码,使用action编译后将public目录同步到zhimoe.github.io仓库的gh-pages分支.
注意,由于使用了jsdelivr的cdn功能,切换分支后theme的相关静态文件的path也要修改.

github给个人和组织免费提供github pages功能. 就是说如果有个repo的名字为 zhimoe.github.io (zhimoe 为你的github username), 那么这个repo里面的master或者gh-pages分支的内容如果存在index.html, 那么其他人可以通过 https://zhimoe.github.io 访问这个站点.
<!--more-->
借助于一些static gen工具,你可以将你的markdown转换为一个静态网站(html,js,css). 然后把静态网站的内容上传到刚说的repo中, 就有一个自己的博客站点了. static gen工具非常多, github推荐的是[Jekyll(ruby)](https://www.staticgen.com/), 主流的还有hexo(js)和hugo(go), hexo因为是基于js的,所以高质量的主题多(因为做主题是需要js,css技能), hugo的编译快些, 但是好看的主题不多. 高质量的主题除了美观可能还需要考虑移动端(responsive),评论, 访问统计等各种功能. 每个gen工具都有自己的主题站点. hugo的主题在这里找: [hugo themes](https://themes.gohugo.io/).

制作github pages站点的一般做法是把代码(放图片和markdown)放在master分支,static gen编译后的(html,js,css,image)内容放在gh-pages分支.然后在settings里面设置. 这样就可以得到一个站点了. 这么做有个缺点,就是markdown文件会被别人整个下载过去,之前就遇到过一次. 正好github现在有3个免费私有仓库. 所以我把源码放在私有仓库zhimoe.github.io.src里面,而编译后的内容发布的 https://zhimoe.github.io上面去.

自动编译发布这个过程就是持续集成(continue integration,CI)了, 即我提交一个markdown文件,我的主页会自动看到这篇文章, 不需要我在本地编译再提交编译结果文件.[travis-ci](travis-ci.com) 提供了免费的github CI服务. 使用github账号登录就会有提示操作. 这里勾选私有仓库zhimoe.github.io.src, 然后在项目里面添加.travis.yml文件告诉travis如何编译和发布内容到个人站点.


### markdown渲染设置
hugo使用BlackFriday渲染markdown文件,默认的设置有几个过于严格:
- 没有硬换行,需要使用`\`来表示换行
- 标题和`#`之间必须有空格
- 代码块前面必须有空行

在config.toml可以修改这些配置:

```toml
# markdown解析引擎blackfriday配置, 
# extensions :  noEmptyLineBeforeBlock-代码块前面无需空行,hardLineBreak-换行无需使用backslash
# extensionsmask spaceHeaders-标题之间无需空格
[blackfriday]
  angledQuotes = true
  extensions = ["hardLineBreak","noEmptyLineBeforeBlock"]
  extensionsmask = ["spaceHeaders"]
  fractions = false
  plainIDAnchors = true
```

### travis-ci配置
就是从一个私有的源码仓库编译,然后将编译后的文件强制覆盖到个人主页(即username.github.io这个仓库)的仓库中.具体的配置就不说了,注意是需要一个github的personal-access-key. 下面是.travis.yml内容:

```yaml
dist: xenial
language: python
python: 3.7

# Handle git submodules yourself
git:
    submodules: false
# Use sed to replace the SSH URL with the public URL, then initialize submodules
before_install:
  - sudo apt update -qq
  - sudo apt -yq install apt-transport-https
  - echo -e "Host github.com\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
  - git config --global user.email ${GITHUB_EMAIL}
  - git config --global user.name ${GITHUB_USERNAME}
  - sed -i 's/git@github.com:/https:\/\/github.com\//' .gitmodules
  - git submodule update --init --recursive

install:
  # install latest hugo release version
  # - wget -qO- https://api.github.com/repos/gohugoio/hugo/releases/latest | sed -r -n '/browser_download_url/{/Linux-64bit.deb/{s@[^:]*:[[:space:]]*"([^"]*)".*@\1@g;p;q}}' | xargs wget
  # use local hugo pkg for speed
  - sudo dpkg -i hugo*.deb
  - rm -rf public 2> /dev/null

  # compile src to dist
script:
  - hugo -d ./dist/

after_success:
  - git clone https://zhimoe:${GITHUB_TOKEN}@github.com/zhimoe/zhimoe.github.io.git
  - cd zhimoe.github.io 
  - git rm -rf . && git clean -fxd 
  - mv -v ../dist/* .
  - git add .
  - git commit -m "update site"
  - git remote set-url origin https://zhimoe:${GITHUB_TOKEN}@github.com/zhimoe/zhimoe.github.io.git
  - git remote -v
  - git push -q -f

```
要点:
- 在项目的源码中放了hugo的deb安装包,省去下载的过程
- 主题以submodules放在themes目录中,所以编译前一定要`git submodule update --init --recursive`更新主题到本地.
- 目标repo的远程仓库一定要在push前重新设置:`git remote set-url origin xxx`
