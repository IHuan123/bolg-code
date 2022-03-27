---
title: npm run
toc: true
date: 2022-03-27 14:45:19
tags: [node,npm]
urlname:
categories: [node,npm]
recommend:
cover:
keywords: npm run如何执行的
top: 3
---

[原文：npm scripts 使用指南](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)

## npm脚本

npm 脚本是写在`package.json`文件里面`scripts`中的执行命令。

```json
{
  // ...
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server --config webpack.config.js"
  },
}
```

<!--more-->

这是一个package.jsong文件脚本命令的一个示例，`scripts`为一个对象，其中每项值都对应一个脚本命令。比如，`dev`命令对应的脚本是`webpack-dev-server --config webpack.config.js`。

命令行下使用`npm run dev`命令，就可以执行这段脚本。

使用不带任何参数的`npm run`就可以获得项目所有的脚本命令。

```bash
$ npm run
# 返回值如下
Lifecycle scripts included in demo@1.0.0:
  test
    echo "Error: no test specified" && exit 1

available via `npm run-script`:
  dev
    webpack-dev-server --config webpack.config.js
```



项目是通过`npm run dev` 命令就可以启动，但直接输入`webpack-dev-server --config webpack.config.js`时终端就会返回`bash: webpack-dev-server: command not found`这样的提示，这是因为在全局变量中没有相应的配置。

## npm run都做了什么

每当我们在执行`npm run`时，npm就会到package.json中找对应的脚本命令，并会新建一个Shell，在这个Shell中执行指定的脚本。这个新建的Shell会将`node_modules/.bin`这个目录加入`PATH`变量中去，在执行结束后又会将`PATH`恢复回去。

所以在`node_modules/.bin`中的所有脚步都可以直接使用脚步名，不需要加上路径。比如：`dev`脚本命令

```bash
"dev": "webpack-dev-server --config webpack.config.js"
```

不用写成

```bash
"dev": "./node_modules/.bin/webpack-dev-server --config webpack.config.js"
```

由于 npm 脚本的唯一要求就是可以在 Shell 执行，因此它不一定是 Node 脚本，任何可执行文件都可以写在里面。

npm 脚本的退出码，也遵守 Shell 脚本规则。如果退出码不是`0`，npm 就认为这个脚本执行失败。

## 简写

npm常用的简写

> - `npm start` === `npm run start`
> - `npm stop` === `npm run stop`
> - `npm test` === `npm run test`
> - `npm restart` ===`npm run stop && npm run restart && npm run start`



## npm默认值

npm对`start`和`install`提供了默认值。只要在package.json中没有定义这两个命令,就会执行

```json
"start": "node server.js"，
"install": "node-gyp rebuild"
```

`npm run start`的默认值是`node server.js`，前提是项目根目录下有`server.js`这个脚本；`npm run install`的默认值是`node-gyp rebuild`，前提是项目根目录下有`binding.gyp`文件。

## 钩子

npm 脚本有`pre`和`post`两个钩子。举例来说，`build`脚本命令的钩子就是`prebuild`和`postbuild`。

```json
"prebuild": "echo I run before the build script",
"build": "cross-env NODE_ENV=production webpack",
"postbuild": "echo I run after the build script"
```



用户执行`npm run build`的时候，会自动按照下面的顺序执行。

```bash
npm run prebuild && npm run build && npm run postbuild
```



因此，可以在这两个钩子里面，完成一些准备工作和清理工作。下面是一个例子。

```json
"clean": "rimraf ./dist && mkdir dist",
"prebuild": "npm run clean",
"build": "cross-env NODE_ENV=production webpack"
```



npm 默认提供下面这些钩子。

> - prepublish，postpublish
> - preinstall，postinstall
> - preuninstall，postuninstall
> - preversion，postversion
> - pretest，posttest
> - prestop，poststop
> - prestart，poststart
> - prerestart，postrestart

自定义的脚本命令也可以加上`pre`和`post`钩子。比如，`myscript`这个脚本命令，也有`premyscript`和`postmyscript`钩子。不过，双重的`pre`和`post`无效，比如`prepretest`和`postposttest`是无效的。

npm 提供一个`npm_lifecycle_event`变量，返回当前正在运行的脚本名称，比如`pretest`、`test`、`posttest`等等。所以，可以利用这个变量，在同一个脚本文件里面，为不同的`npm scripts`命令编写代码。请看下面的例子。

```javascript
const TARGET = process.env.npm_lifecycle_event;

if (TARGET === 'test') {
  console.log(`Running the test task!`);
}

if (TARGET === 'pretest') {
  console.log(`Running the pretest task!`);
}

if (TARGET === 'posttest') {
  console.log(`Running the posttest task!`);
}
```



注意，`prepublish`这个钩子不仅会在`npm publish`命令之前运行，还会在`npm install`（不带任何参数）命令之前运行。这种行为很容易让用户感到困惑，所以 npm 4 引入了一个新的钩子`prepare`，行为等同于`prepublish`，而从 npm 5 开始，`prepublish`将只在`npm publish`命令之前运行。

## 变量

npm 脚本有一个非常强大的功能，就是可以使用 npm 的内部变量。

通过`npm_package_`前缀，npm 脚本可以获取到`package.json`里面的字段。比如。

```json
{
  "name": "demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
    "config": {
    "type": "git",
    "url": "xxx"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server",
    "d": "npm run dev -- --config webpack.config.js",
    "predev": "echo I run before the build script",
    "postdev": "echo I run after the build script"
  },
}
```

变量`npm_package_name`返回`demo`，变量`npm_package_version`返回`1.0.0`，在代码中通过环境变量`process.env.npm_package_xxx`获取。如果是 Bash 脚本，可以用`$npm_package_name`和`$npm_package_version`取到这两个值。

```js
console.log(process.env.npm_package_name); // demo
console.log(process.env.npm_package_version); // 1.0.0
```

`npm_package_`前缀也支持嵌套的`package.json`字段。

```javascript
console.log(process.env.npm_package.config.type) //git
```

上面代码中，`scripts`字段的`test`脚本的值。

下面是另外一个例子。

> ```javascript
> "scripts": {
>   "install": "foo.js"
> }
> ```

上面代码中，`npm_package_scripts_install`变量的值等于`foo.js`。

然后，npm 脚本还可以通过`npm_config_`前缀，拿到 npm 的配置变量，即`npm config get xxx`命令返回的值。比如，当前模块的发行标签，可以通过`npm_config_tag`取到。

```json
"view": "echo $npm_config_tag",
```



注意，`package.json`里面的`config`对象，可以被环境变量覆盖。

```json
{ 
  "name" : "foo",
  "config" : { "port" : "8080" },
  "scripts" : { "start" : "node server.js" }
}
```



上面代码中，`npm_package_config_port`变量返回的是`8080`。这个值可以用下面的方法覆盖。

```bash
$ npm config set foo:port 80
```



最后，`env`命令可以列出所有环境变量。

```json
"env": "env"
```



## 通配符

由于 npm 脚本就是 Shell 脚本，因为可以使用 Shell 通配符。

```json
"lint": "jshint *.js"
"lint": "jshint **/*.js"
```

上面代码中，`*`表示任意文件名，`**`表示任意一层子目录。

如果要将通配符传入原始命令，防止被 Shell 转义，要将星号转义。

```json
"test": "tap test/\*.js"
```

## 传参

npm传参需要使用`--`标明，比如：

```json
  "scripts": {
    "dev": "webpack-dev-server",
  },
```

在`webpack-dev-server`我想要指定配置`config`文件时:

```bash
npm run dev -- --config webpack.config.js
```

## 执行顺序

npm 脚本执行多个任务

并行执行（即同时的平行执行），使用`&`符号:`$ npm run script1.js & npm run script2.js`

继发执行，排队依次执行，执行成功才执行下一个，可以使用`&&`符号:`$ npm run script1.js && npm run script2.js`

