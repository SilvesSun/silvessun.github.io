---
title: react开启less支持以及antd按需引入
date: 2019-05-17 15:46:30
tags:
- less
- 环境搭建
- antd按需引入
categories:
- React
---

系统基本信息
```
node v10.15.3
npm 6.5.0
yarn 1.15.2
webpack 4.29.6
```

# React开启less支持

使用`create-react-app`脚手架新建一个`react`项目

`create-react-app react-blog-with-redux`

然后进入到对应目录. 安装对应的包

`yarn add less less-loader`

还需要在`webpack`的配置文件中添加相应的配置, 故运行 `yarn eject` 暴露相关的配置

此时多出一个`config`文件夹, 结构如下:

```shell
config/
├── env.js
├── jest
│   ├── cssTransform.js
│   └── fileTransform.js
├── modules.js
├── paths.js
├── pnpTs.js
├── webpack.config.js
└── webpackDevServer.config.js
```

在`webpack.config.js`中作如下修改

- 修改`cssRegex`
  ```JavaScript
  const cssRegex = /\.css$/; //修改为

  // =>

  const cssRegex = /\.(css|less)$/;
  ```

- 添加less模块

  ```JavaScript
  // 在getStyleLoaders最后添加less-loader

  {loader: require.resolve('less-loader')}
  ```

![less](https://supcoder.net/lessloder.jpg)


# antd按需引入
## 安装antd
`yarn add antd`

## 使用 babel-plugin-import 来进行按需加载
`yarn add babel-plugin-import --save-dev`

## 编辑`package.json`中`babel`配置项

添加`plugins`, 如图
![antd](https://supcoder.net/antdBabel.png)

现在就可以按需引入了


