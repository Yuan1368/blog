---
title: "用electron做一个桌面工具程序（应该不算教程）"
date: 2022-04-09T10:56:24+08:00
tags: ["前端","工具","electron"]
categories: ["工具怎么用"]
---

> 本着有鱼能摸，不摸白不摸的原则，简单学了下 electron 做了一个密码加密生成器，主要作用实际上是用来给我自己的设置的密码再进行加密一下。我一直以来给不同应用设置密码的方法是使用这个应用的名称再加上一部分通用的内容作为密码，这样的话既能保证我能简单地记住每个应用的密码，又能保证给不同应用的密码是不相同的，避免“撞库”造成的泄密风险。但是这样做还是有点风险，一旦对方了解我设置密码的原则后，还是会泄密，最好的密码就是不要去记住它，所以我再给我设置的密码再加密一次，用不同的加密算法去加密，这样又能减少风险了。

## electron介绍

Electron 基于 Chromium 和 Node.js, 让你可以使用 HTML, CSS 和 JavaScript 构建应用，这也就意味着并不需要去学 windows 桌面端的 API 就能去制作一个桌面端的应用，而且它可以构建不同平台的应用，这样就能用一套方案，在 Linux 和 Windows 上都能做桌面应用了，大大提高了我们的开发效率。

## electron 安装

安装过程实际上很简单，按照 electron 官网提供的文档就能实现了[Quick Start | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/tutorial/quick-start)。

如果你已经完成了 electron 的安装过程，实际上你会发现，它其实与正常的 Node.js 的 web 应用构建没什么区别，只不过在这里多了一层 由 electron 提供的`app`来对页面包装一下，让 web 页面能够运行在 electron 内，而不是我们系统里的浏览器。换言之，electron 更像是一个只会渲染我们提供给它的页面的简单浏览器。

## 编写过程

我的需求实际上很简单，只需要一个输入框输入密码，一个输出框，和一个点击后就能把密码输出到输出框内的按钮，这个过程很简单，与我们写简单的 web 页面一致：

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8">
  <title>没啥用工具——密码生成器</title>
  <meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline';" />
  <style>
    .container {
      display: flex;
      padding: 0 2rem;
      flex-direction: column;
      justify-content: center;
      align-items: center;
    }

    .input-password,
    .output-text {
      outline-style: none;
      border: 1px solid #ccc;
      border-radius: 3px;
      padding: 0px 5px;
      width: 100px;
      font-size: 14px;
      font-weight: 700;
      font-family: "Microsoft soft";
    }

    .input-password:focus,
    .output-text:focus {
      border-color: #66afe9;
      outline: 0;
      -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075), 0 0 8px rgba(102, 175, 233, .6);
      box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075), 0 0 8px rgba(102, 175, 233, .6)
    }

    .output-text {
      height: 150px;
      margin-top: 10px;
      word-wrap: break-word;
    }

    .generate-button {
      padding: 1em 2em;
      margin-top: 10px;
      border: 0;
      background-color: #2cadf8;
      color: #fff;
      border-radius: 10px;
      cursor: pointer;
    }

    .generate-button:hover {
      color: #2cadf8;
      background-color: #fff;
      border: 2px #2cadf8 solid;
    }
  </style>
</head>

<body>
  <div class="container">
    <input type="password" class="input-password" id="input-password" />
    <div class="output-text" id="output-text"></div>
    <div class="button-groups">
      <button class="generate-button" id="generate-password">生成</button>
    </div>
  </div>
  <script>
    let MD5 = require("crypto-js/md5")

    let passwordButton = document.getElementById("generate-password");
    passwordButton.addEventListener("click", () => {
      let inputText = document.getElementById("input-password").value;
      let outputText = document.getElementById("output-text");
      console.log(outputText.childNodes);
      if (outputText.childNodes.length) {
        outputText.removeChild(outputText.childNodes[0]);
      }
      outputText.appendChild(document.createTextNode(MD5(inputText)));
    })
  </script>
</body>

</html>
```

这里我使用了`crypto-js`提供的 md5 加密算法，另外为了让页面显得不那么单调，还给输入框与按钮加了点样式进行修饰，实际上大部分的样式都是我 copy 过来的。

![image-20220409115404085](https://raw.githubusercontent.com/Yuan1368/imgs-repo/main/202204091154334.png)



为了能让我们的页面运行在桌面应用内，而不是在浏览器内，如上所述我们还需要一个 electron 把代理运行我们的页面，所以要新建一个`main.js`：

```js
const { app, BrowserWindow, Menu } = require('electron')

Menu.setApplicationMenu(null);

function createWindow() {
  const win = new BrowserWindow({
    width: 300,
    height: 300,
    resizable: false,
    webPreferences: {
      nodeIntegration: true,
      contextIsolation: false,
    }
  })
  win.loadFile('index.html')
}



app.whenReady().then(() => {
  createWindow()

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow()
    }
  })
})

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

```

由于 electron 默认生成的应用会由工具栏，所以我用了`Menu.setApplicationMenu(null);`将工具栏给取消掉。然后我用新建了一个`createWindow`函数，它的作用是给应用的视口一个指定的宽高，因为我并不想让应用能够全屏。另外由于我在页面中使用了`crypto-js`,这是通过 npm 进行安装的，所以需要允许页面使用 node.js ，也就是`nodeIntegration: true`。

很明显`win.loadFile('index.html')`作用就是加载我们的页面，当应用准备完毕时，就可以渲染页面了：`app.whenReady()`。

我们还需要编写一个脚本让 node.js 能够去执行生成应用的内容，这与正常 node.js 生成页面是一致的，在`package.json`中添加一条脚本：

```js
  "scripts": {
    "start": "electron-forge start",
  },
```

## 打包应用

有了页面内容后，我们还需要将它打包成一个可执行文件，这样我们就可以随时使用它，而不必通过打开项目运行脚本了。

```shell
yarn add --dev @electron-forge/cli
npx electron-forge import
```

以上过程安装了一个 electron 提供给我们的打包脚手架，然后在脚本内会多出两条命令，其中有一条是`"make": "electron-forge make"`，我们可以运行这条脚本，然后在项目中会多出一个`out`文件夹，里面就有我们想要的可执行文件。

![image-20220409121036255](https://raw.githubusercontent.com/Yuan1368/imgs-repo/main/202204091210309.png)

OK，整个过程就是这样简单，其它的一些详细配置可以通过官网提供的文档完成。