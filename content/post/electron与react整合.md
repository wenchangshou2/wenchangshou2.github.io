---
title: "Electron与react整合"
date: 2020-02-11T14:32:57+08:00
---

我们知道如果需要electron和vue.js开发，可以通过**electron-vue**直接创建一个带vue的electron的项目，那么今天我们来讲讲electron如何与react全家桶结合。

源码地址：https://github.com/wenchangshou2/electrol-react-redux.git

## 安装yarn

Yarn是facebook推出的一个更加靠近的的包管理工具。

**mac下安装**

> ```
> brew install yarn
> ```

**Linux下安装**

在ubuntu中默认的包仓库是没有yarn的包的，我们需要：

```shell
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
```

在我们导入证书和源之后，可以通过下列的命令直接安装

```shell
sudo apt-get update && sudo apt-get install yarn
```



## 安装create-react-app

create-react-app是一个创建react项目的工具，通过这个工具可以快速创建一个初始的react项目。

```shell
yarn global add create-react-app
```

## 创建react项目

```shell
create-react-app app
cd app
```

## 添加electron

```shell
yarn add electron
yarn add electron-builder -D
```

## 安装foreman帮助我们在命令行下运行应用程序

```shell
yarn global add foreman
```

## 安装create-react-app创建的项目依赖

```shell
yarn install
```

## 配置eslint

eslint配置你的代码的规范，配合atom或者vs code，能够实时的观测你的代码风格是否符合规范。

> touch .eslintrc.json

``` js
{
    "env": {
        "browser": true,
        "commonjs": true,
        "es6": true,
        "jest": true
    },
    "parserOptions": {
        "ecmaFeatures": {
            "jsx": true
        },
        "sourceType": "module"
    },
    "rules": {
        "no-const-assign": "warn",
        "no-this-before-super": "warn",
        "no-undef": "warn",
        "no-continue": "off",
        "no-unreachable": "warn",
        "no-unused-vars": "warn",
        "constructor-super": "warn",
        "valid-typeof": "warn",
        "space-before-function-paren":"off",
        "function-paren-newline":"off",
        "no-plusplus":"off",
        "quotes": [
            2,
            "single"
        ]
    },
    "parser": "babel-eslint",
    "extends": "airbnb",
    "plugins": [
        "react",
        "jsx-a11y",
        "import"
    ]
}
```

同时我们需要安装ESLint的相应的插件

> ```
> yarn add eslint eslint-config-airbnb eslint-plugin-jsx-a11y eslint-plugin-import eslint-plugin-react eslint-plugin-import
> ```



在经过上面的操作之后，我们需要对package.json进行修改来适配electron

我们暂时将内容修改成下面的内容

``` js
{
  "name": "qiniu-upload",
  "version": "0.1.0",
  "private": true,
  "homepage": "./",
  "main": "src/electron-starter.js",
  "dependencies": {
    "electron": "^1.8.4",
    "eslint": "^4.19.1",
    "eslint-config-airbnb": "^16.1.0",
    "eslint-plugin-import": "^2.10.0",
    "eslint-plugin-jsx-a11y": "^6.0.3",
    "eslint-plugin-react": "^7.7.0",
    "prop-types": "^15.6.1",
    "react": "^16.3.0",
    "react-dom": "^16.3.0",
    "react-scripts": "1.1.1",
  },
  "scripts": {
    "start": "nf start -p 3000",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject",
    "electron": "electron .",
    "electron-start": " node src/electron-wait-react",
    "react-start": "react-scripts start",
    "pack": "build --dir",
    "dist": "npm run build && build",
    "postinstall": "install-app-deps"
  },
  "devDependencies": {
    "electron-builder": "^20.8.1"
  },
  "build": {
    "appId": "com.electron.electron-with-create-react-app",
    "win": {
      "iconUrl": "https://cdn2.iconfinder.com/data/icons/designer-skills/128/react-256.png"
    },
    "directories": {
      "buildResources": "public"
    }
  }
}

```



上面的版本可能随时在变，不用太介意，用最新的版本就可以 

上面的我们将原本的start改成了react-start，对于其他的没有做任何的改变，现在还需要添加electron的处理内容

```
  "homepage": "./",
  "main": "src/electron-starter.js",
```

**和**

```json
  "build": {
    "appId": "com.electron.electron-with-create-react-app",
    "win": {
      "iconUrl": "https://cdn2.iconfinder.com/data/icons/designer-skills/128/react-256.png"
    },
    "directories": {
      "buildResources": "public"
    }
  }
```

修改之后最终的文件内容:

```js
{
  "name": "qiniu-upload",
  "version": "0.1.0",
  "private": true,
  "homepage": "./",
  "main": "src/electron-starter.js",
  "dependencies": {
    "electron": "^1.8.4",
    "eslint": "^4.19.1",
    "eslint-config-airbnb": "^16.1.0",
    "eslint-plugin-import": "^2.10.0",
    "eslint-plugin-jsx-a11y": "^6.0.3",
    "eslint-plugin-react": "^7.7.0",
    "prop-types": "^15.6.1",
    "react": "^16.3.0",
    "react-dom": "^16.3.0",
    "react-redux": "^5.0.7",
    "react-scripts": "1.1.1",
    "redux": "^3.7.2"
  },
  "scripts": {
    "start": "nf start -p 3000",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject",
    "electron": "electron .",
    "electron-start": " node src/electron-wait-react",
    "react-start": "react-scripts start",
    "pack": "build --dir",
    "dist": "npm run build && build",
    "postinstall": "install-app-deps"
  },
  "devDependencies": {
    "electron-builder": "^20.8.1"
  },
  "build": {
    "appId": "com.electron.electron-with-create-react-app",
    "win": {
      "iconUrl": "https://cdn2.iconfinder.com/data/icons/designer-skills/128/react-256.png"
    },
    "directories": {
      "buildResources": "public"
    }
  }
}

```

现在我们还需要在项目的根目录下面创建一个名为Procfile的文件，其内容如下：

```
react: npm run react-start
electron: npm run electron-start
```

## 继续完善

现在我们还需要完成src/electron-start.js以及src/start-react.js文件

> src/electron-start.js

```js
const electron = require('electron');
const { app, BrowserWindow } = electron;

const path = require('path');
const url = require('url');


let mainWindow;

function createWindow() {
  mainWindow = new BrowserWindow({ width: 800, height: 600 });

  const startUrl =
    process.env.ELECTRON_START_URL ||
    url.format({
      pathname: path.join(__dirname, '/../build/index.html'),
      protocol: 'file:',
      slashes: true,
    });
  mainWindow.loadURL(startUrl);
  mainWindow.webContents.openDevTools();

  mainWindow.on('closed', () => {
    mainWindow = null;
  });
}

app.on('ready', createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', () => {
  if (mainWindow === null) {
    createWindow();
  }
});

```

> src/start-react.js

```js
const net = require('net');
const childProcess = require('child_process');

const port = process.env.PORT ? process.env.PORT - 100 : 3000;

process.env.ELECTRON_START_URL = `http://localhost:${port}`;

const client = new net.Socket();

let startedElectron = false;
const tryConnection = () => {
  client.connect({ port }, () => {
    client.end();
    if (!startedElectron) {
      console.log('starting electron');
      startedElectron = true;
      const { exec } = childProcess;
      exec('npm run electron');
    }
  });
};

tryConnection();

client.on('error', () => {
  setTimeout(tryConnection, 1000);
});

```



## 启动

现在我们运行

> yarn start

现在你发现了react将以本地应用程序的方式运行。

![启动界面](http://o7ez1faxc.bkt.clouddn.com/react-start.png)

## 继续耕耘

上面的是一个初级的demo，我们现在需要继续整合react redux,来实现一个官方的Todo功能



> 我们下节内容来实现整合