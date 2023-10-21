+++
title = "在 githook 中调用 nodejs 脚本"
date = "2021-08-22T21:55:06+08:00"
categories = [ "编程",]
tags = [ "code", "nodejs",]
toc = "true"
+++

 
如何在 git hook 中调用 nodejs 脚本。主要踩坑在于不知道如何在 bash 中获取 node 脚本返回值，搜了好大一圈。

## 背景
微服务模式开发中，每个小组维护自己的应用，通过一个 nginx 入口反向代理所有的子应用，向用户开放一个站点.nginx 应用中需要维护各个子应用的代理，即 ng.conf 中的 location.
此外，一个应用需要配置 DEV,ST,UAT,PRD 四个环境的 location.目前的做法是 www/ngconf/目录下面分为 dev、st、uat、prd 四个文件夹，在文件夹内部每个小组各自维护一个 conf 文件。
每增加一个应用，需要在四个文件夹中自己小组的配置文件添加配置。随着应用越来越多，以及人员流动，会发生不同文件配置相同的 location entry. 
例如 A 应用上线一个功能需要依赖 B 应用，但是新人不知道 B 已经配置过了，所以又重复添加了一个，导致启动报错。

<!--more-->

## 需求
入口应用是一个 nodejs 应用，自然选择了 js 脚本检查 conf 文件 location path 配置是否重复。

## 实现
在 git hook 目录下，新增一个`pre-commit`文件，添加内容：

```bash
#!/usr/bin/sh
# 检查项目中同一个目录下面的nginx conf 所有location是否重复
if [ -e ./ngconf_check.js ]; then
    node ngconf_check.js
    if [[ $? != 0 ]]; then
        echo >&2 fix duplicate location entry in nginx conf
        exit 1
    fi
fi

# 如果项目有自定义pre-commit,执行
if [ -e ./.git/hooks/pre-commit ]; then
    ./.git/hooks/pre-commit "$@"
fi
exit 0
```
要点：bash 中`$?`表示获取上命令的返回值。这里获得的是 js 脚本的 process.exit(code) 返回的 code. 默认返回是 0.

ngconf_check.js:

```javascript
// 检查 ng conf 是否有重复的 location entry
const fs = require('fs');
const path = require('path');

const ConfEnvDirs = new Set();
ConfEnvDirs.add('dev');
ConfEnvDirs.add('st');
ConfEnvDirs.add('uat');
ConfEnvDirs.add('prd');
const NgconfPath = 'www/ngconf';

let result = 'pass';
countLocationInDir(NgconfPath);
if(result !== 'pass'){
    process.exit(1);//向 bash 返回 1
}

/**
 *
 * @param rootPath
 */
function countLocationInDir(rootPath) {
    if (!fs.existsSync(rootPath)) return;
    let dirs = fs.readdirSync(rootPath);
    dirs.forEach(dir => {
        let envDir = path.join(rootPath, dir);
        //只处理四个环境目录下的 conf 文件，每个目录用一个 map 记录
        const LocationEntryMap = new Map();//location entry -> file,line
        if (!(fs.statSync(envDir).isDirectory() && ConfEnvDirs.has(dir))) {
            return;
        }
        let confFiles = fs.readdirSync(envDir);
        confFiles.forEach(filename => {
            let fullPath = path.join(envDir, filename);
            if (fs.lstatSync(fullPath).isFile() && /[\w\W.].conf$/.test(filename)) {
                countLocationsInFile(fullPath, LocationEntryMap);
            }
        });
    });

}

/**
 *
 * @param confFile
 * @param countMap
 */
function countLocationsInFile(confFile, countMap) {
    let lines = fs.readFileSync(confFile, "utf-8")
        .split("\n")
        .filter(Boolean);
    lines.forEach((line, lineNumber) => {
        if (line.trim().startsWith("location")) {
            const arr = line.trim().split(' ');
            const locationEntry = arr[1];
            const entryInfo = `${confFile}, at line:${lineNumber}`;
            if (countMap.has(locationEntry)) {
                console.log(`ERROR: duplicate location entry: ${locationEntry}`);
                console.log(`location 1:${countMap.get(locationEntry)}`);
                console.log(`location 2:${entryInfo}`);                
                //修改 result 变量
                result='error';
            } else {
                countMap.set(locationEntry, entryInfo);
            }
        }
    });
}
```