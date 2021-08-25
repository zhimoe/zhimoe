---
title: "在githook中调用nodejs脚本"
date: "2021-08-22T21:55:06+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - nodejs
---
如何在git hook中调用nodejs脚本。主要采坑在于不知道如何在bash中获取node脚本返回值，搜了好大一圈。
<!--more-->
## 背景
微服务模式开发中，每个小组维护自己的应用，通过一个nginx入口反向代理所有的子应用，向用户开放一个站点。nginx应用中需要维护各个子应用的代理，即ng.conf中的location。
此外，一个应用需要配置DEV,ST,UAT,PRD四个环境的location。目前的做法是www/ngconf/目录下面分为dev、st、uat、prd四个文件夹，在文件夹内部每个小组各自维护一个conf文件。
每增加一个应用，需要在四个文件夹中自己小组的配置文件添加配置。随着应用越来越多，以及人员流动。会发生不同文件配置相同的location entry。 例如A应用上线一个功能需要依赖B应用，但是新人不知道B已经配置过了，所以又重复添加了一个，导致启动报错。

## 需求
入口应用是一个nodejs应用，自然选择了js脚本检查conf文件location path是否重复。

## 实现
在git hook目录下，新增一个`pre-commit`文件,添加一下内容：
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

# 如果项目有自定义pre-commit，执行
if [ -e ././githooks/pre-commit ]; then
    ./.git/hooks/pre-commit "$@"
fi
exit 0
```
要点： bash中`$?`表示获取上命令的返回值。这里获得的是js脚本的process.exit(code)返回的code。 默认返回是0。
ngconf_check.js：

```
// 检查ng conf是否有重复的location entry
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
    process.exit(1);//向bash返回1
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
        //只处理四个环境目录下的conf文件,每个目录用一个map记录
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
                //修改result变量
                result='error';
            } else {
                countMap.set(locationEntry, entryInfo);
            }
        }
    });
}
```