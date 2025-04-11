# github.io
personal home page, all about programming and code.

[zhi.moe](http://zhi.moe)

## how to add post
```bash
# git clone
git clone --recurse-submodules -j6 https://github.com/zhimoe/zhimoe.git
# or git clone zhimoe repo and then 
git submodule add https://github.com/zhimoe/hugo-paged.git themes/paged
# or git submodule update --init
# install hugo on local 
sudo apt install hugo
# add post by hugo
hugo new post/hello-2020.md 
# preview your site on local
hugo serve # this will generate a public folder for site and you should add public in .gitignore

# commit the changes and publish use github action or travis-ci
```

## how to change theme
```bash
# submodule is a independent repo,
# so you need commit/push change in submodule first and then 
# update(commit) the main project to refer a new submodule commit hash

# step 1
cd themes/paged
git add <stuff>
git commit -m "update hugo theme" && git push origin HEAD:master

# step 2
cd ../..
git add ./themes/paged && git commit -m "update submodule theme"
git push
```

## 注意 search 功能
如果 sidebar 的搜索不可用，可能原因是 summary 中包含特殊字符，导致 sitemap.xml 解析错误

## 图片链接
图片需要放在项目或者主题项目的`static`目录下面，编译后会将 static 下平移到 public 目录下面。引用图片需使用绝对路径 例如
```html
<img src="/img/logo.jpg"
```
```markdown
![mdict-py](/mdict/mdictpy.png)
```
