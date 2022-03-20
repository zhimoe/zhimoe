# github.io
personal home page, all about programming and code.

[zhi.moe](http://zhi.moe)

## how to add post
```bash
# git clone
git clone 
# add theme 
git submodule add https://github.com/zhimoe/hugo-theme-next.git themes/next
# update theme
git submodule update --init --recursive
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
cd themes/next
git add <stuff>
git commit -m "update hugo theme" && git push origin HEAD:master

# step 2
cd ../..
git add ./themes/next && git commit -m "update submodule theme"
git push
```

## 注意search功能
如果sidebar的搜索不可用,可能原因是summary中包含特殊字符,导致sitemap.xml解析错误