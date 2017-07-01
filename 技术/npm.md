# 技术学习 npm

<!-- toc -->

<!--
  - 升级`npm`
  - 获取`npm`包安装路径：
  - 使用`package.json`来管理安装的包
  - 包安装方式
  - 包卸载方式
-->

## 升级`npm`

```bash
npm install npm@latest -g
```

## 获取`npm`包安装路径：

```bash
npm config get prefix
```

## 使用`package.json`来管理安装的包

用做问卷的方式创建一个`package.json`：

```bash
npm init
```

使用参数`--yes`或`-y`可以跳过问卷，快速生成：

```bash
npm init --yes
```

`package.json`中的依赖参数：

- "dependencies": 生产环境中依赖的包，使用`npm install <package_name> --save`
- "devDependencies": 开发/测试环境中使用的包，使用`npm install <package_name> --save-dev`

## 包安装方式

- 本地安装：在自己的模块中使用，利用`Node`的`require`
- 全局安装：命令行方式使用包，视作命令行工具

本地安装：

```bash
npm install <package_name>
```

全局安装：

```bash
npm install -g <package_name>
```

全局升级：

```bash
npm update -g <package_name>
```

## 包卸载方式

从`node_modules`中移除某个包：

```bash
npm uninstall <package_name>
```

从`dependencies`参数中移除

```bash
npm uninstall --save <package_name>
```

从`devDependencies`参数中移除

```bash
npm uninstall --save-dev <package_name>
```

全局卸载：

```bash
npm uninstall -g <package_name>
```

## 列出所有安装的包

```bash
npm list
```

## 根据 package.json 移除项目不依赖的包

```bash
npm prune
```
