+++
title = '油猴脚本开发笔记'
date = '2024-09-07T13:11:44+08:00'
categories = ['编程']
tags = ['code','油猴']
toc = true
+++

以前都是用别人的油猴脚本，这次自己开发了一个自动化脚本，记录一些笔记。

<!--more-->

## 开发模式
直接油猴编辑器里面编写代码实际体验很差，包括代码补全等，理想的方式应该是在 vs code 开发，让 chrome 实时获取本地脚本。
1. 打开 chrome 扩展管理界面 - 油猴 详情页 - 打开“允许访问文件网址（Allow access to file URLs）”
2. 在油猴管理面板新增一个脚本，将`require`指向本地脚本。
   ```js
    // ==UserScript==
    // @name         DevInLocal
    // @namespace    http://tampermonkey.net/
    // @match        *//example.com
    // @require      file:///path/to/userscript.user.js
    // ==/UserScript==
   ```
   注意路径格式：
   ```js
   // windows
   // @require      file://C:\path\to\userscript.user.js
   // unix
   // @require      file:///path/to/userscript.user.js
   ```


### 脚手架与热更新
目前项目比较小，直接使用 js 更方面也容易维护。但是如果是需要在网页上面增加很多样式或者卡片的脚本，建议还是找一些脚手架。通过 react 等一些框架写代码，然后 build 之后生成一个 js 并自动打开 chrome 油猴安装页面，同时脚手架一般能支持自动将 npm 依赖转换成@require。有些脚手架还支持热更新，这样不用每次 build 后手动刷新目标网页才能生效。
目前比较好的开发插件是[vite-plugin-monkey](https://github.com/lisonge/vite-plugin-monkey/blob/main/README_zh.md)

此外还有一些热门脚本[awesome-userscripts](https://github.com/awesome-scripts/awesome-userscripts)

### 剪切板
读取剪切板这个动作的代码必须通过用户事件（鼠标或者键盘）去触发，在用户脚本中执行读取剪切板会被 chrome 拦截（安全考虑）。

## js 相关的经验

### 处理 iframe 的内容
无论是否同源，如果只是单纯处理 iframe 部分，则直接在@match 中 设置 iframe src 即可，脚本能自动匹配（其实就是两个不同的网页）。
对于非同源的 iframe，如果想要在主站读取 iframe 内容或者根据条件修改 iframe 内容则无法实现。唯一的方法是创建两个脚本，主站脚本发送 postMessage，iframe 脚本接收消息，参考如下
```js
// iframe.js
window.addEventListener('message', function(event) {
  // 验证消息的来源域名，确保安全
  if (event.origin === 'https://your-main-site.com') {
    if (event.data === 'clickButton') {
      // 获取并点击按钮
      const button = document.querySelector('button'); // 假设是第一个 button
      if (button) {
        button.click();
      }
    }
  }
});

// main.js
(function() {
  'use strict';
  // 获取 iframe 元素
  const iframe = document.querySelector('iframe'); 
  // 每隔几秒发送消息到 iframe 让其点击按钮
  setInterval(() => {
    if (iframe && iframe.contentWindow) {
      iframe.contentWindow.postMessage('clickButton', 'https://iframe-domain.com'); // 指定 iframe 的域名
    }
  }, 5000); // 每隔 5 秒点击一次按钮
})();
```
### div 选择器
现在的网页大部分是 react 或者 vue 编译生成的，里面的 class 大部分是动态变化的，也基本没有 id，所以大部分情况只能使用层级加一些特殊 class 来定位 div，所以最好用的其实是`querySelector`和`querySelectorAll`。注意前者返回的是第一个匹配，非数组。配合 css 选择器非常好用，例如下面示例：

```js
// div with name
let input = document.querySelector('input[name="pipelineName"]')
// div with id
let input = document.querySelector('input#pipelineNameSearchForm_customizeSearchInput')
// 带 css 伪类的，注意 这里 div > button.tab___3x3nt 返回的还是一个嵌套元素（不是 element 数组），取嵌套元素的第二个（从 1 开始）
let btn = document.querySelector('div > button.tab___3x3nt:nth-child(2)')
```

### 脚本执行时机
页面加载有几个重要事件。
1. 首先是 document 对象上面的`DOMContentLoaded`事件，表示页面元素加载完成，js（async 例外）加载执行完成，但是 css 和图片不一定。
   Firefox，Chrome 和 Opera 都会在 DOMContentLoaded 中自动填充表单。
```js
document.addEventListener("DOMContentLoaded", readyHandler);
```
2. 当整个页面，包括样式、图片和其他资源被加载完成时，会触发 window 对象上的 load 事件。可以通过 onload 属性获取此事件。
```js
 window.onload = function() {}
```
当访问者离开页面时，window 对象上的 unload 事件就会被触发。我们可以在那里做一些**不涉及延迟**的操作，例如关闭相关的弹出窗口。
`navigator.sendBeacon`是一个特殊方法：它在后台发送数据，转换到另外一个页面不会有延迟：浏览器离开页面，但仍然在执行 sendBeacon。
```js
 window.onunload = function(){
  navigator.sendBeacon("/analytics", JSON.stringify(analyticsData)); // must < 64kb
 }
```
3. `document.readyState` 属性可以为我们提供当前加载状态的信息。包含`loading` `interactive` `complete`三种状态 每种状态变化时 document 都会有一个`readystatechange`事件。

一般在油猴脚本中使用下面判断元素是否就绪：
```js
function main(){
    if (
        document.readyState === "complete" &&
        document.querySelector('div > button.tab___3x3nt:nth-child(2)')
    ) {
        // your script 
    } else {
        console.log(`document unready, try later`);
        setTimeout(main, 500);
    }
}
```
### input 事件和回车事件
由于现在的网站都是通过 react 或者 vue 开发的，input 默认的 input/change 事件被屏蔽了，直接修改 input 的 value 并不会真的修改输入框的值（只是输入框展示变化了）。所以需要通过 event 触发 input 事件。回车事件（KeyboardEvent）也类似：
```js
let input = document.querySelector('input[name="pipelineName"]')
input.value = "NewProject"

let ev = new Event("input", { bubbles: true, cancelable: false });
input.dispatchEvent(ev);
```

### 异步方法处理循环动作
由于 js 是单线程异步的，对于那种需要等待返回再执行下一次循环的场景需要特殊处理。
假设场景如下：向一个 input 输入一个项目名称，搜索后等待网页更新，检查搜索结果的 div，勾选第一个 checkbox，然后再输入下一个项目名称搜索，如此循环。每次循环需要等网页更新勾选完成后才能触发下一次搜索，否则搜索结果就被覆盖了。
下面是一个完整示例：
```js

(() => {
  "use strict";
  main();
})();

async function main() {
  await waitForDocReady();
  addCopyBtn();
}

// 这里使用了 Promise 和 setInterval 轮询页面就绪状态
// 使用 window.onload 应该也可以实现相同效果
function waitForDocReady() {
  return new Promise((resolve) => {
    const docReadyInterval = setInterval(() => {
      if (
        document.readyState === "complete" &&
        document.querySelector('input[name="pipelineName"]')
      ) {
        clearInterval(docReadyInterval);
        resolve();
      }
    }, 400);
  });
}

function addCopyBtn() {
  let button = document.createElement("button");
  button.id = "copyButton";
  button.textContent = "从剪切板复制";

  let element = document.querySelector(
    "div.page-title.highlight > span.text-right"
  );
  element.parentNode.insertBefore(button, element);

  button.addEventListener("click", copyBtnHandler);
  console.log(`button added`);
}

async function copyBtnHandler() {
  let input = document.querySelector('input[name="pipelineName"]');

  let text = await navigator.clipboard.readText();
  const lines = text.split("\n");
  console.log(`lines ${lines}`);

  for (const value of lines) {
    await searchAndCheck(input, value);
  }
}

async function searchAndCheck(input, value) {
  return new Promise((resolve) => {
    searchByInput(input, value);
    const checkInterval = setInterval(() => {
      let checkbox = document.querySelector("re-checkbox > label > input");
      if (checkbox) {
        checkbox.click();
        clearInterval(checkInterval);
        resolve();//这里很巧妙
      }
    }, 400);
  });
}

function searchByInput(input, value) {
  input.value = value;
  let ev = new Event("input", { bubbles: true, cancelable: false });
  input.dispatchEvent(ev);

  let event = new KeyboardEvent("keydown", {
    key: "Enter",
    code: "Enter",
    keyCode: 13,
    which: 13,
    bubbles: true,
  });

  input.focus();
  input.dispatchEvent(event);
  console.log("keydown dispatched");
}






```

