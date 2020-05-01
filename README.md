# github.io
personal home page, all about programming and code.

[cod3fn](http://cod3fn.com) stands for code use function

## how to add post
```bash
# git clone
git clone 
# add theme 
git submodule add https://github.com/cod3fn/hugo-theme-next.git themes/next
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
cd theme/next
git add <stuff>
git commit -m "update hugo theme"
git push

# step 2
cd github.io/
git add ./themes/next
git commit -m "updated submodule theme"
git push
```