# npm9.8.1

npm 是 **node 包管理工具**，在安装 Node.js 时，会自动安装 npm，但是 npm 的发布比 Node.js 更频繁，因此记得更新 npm。

[npm CLI 文档](https://docs.npmjs.com/cli/v9/commands)	|	[npm 包搜索引擎](https://www.npmjs.com/package/package)



## npm配置

[.npmrc 配置](https://docs.npmjs.com/cli/v9/using-npm/config)

[npm 注册中心镜像](https://npmmirror.com/)



## 命令

https://biaoyansu.com/20.4

| npm 命令                           | 释义                                                         | 参数            | 释义                                                         | yarn命令                     | 释义                     | pnpm<br />(node>18.12) |
| ---------------------------------- | ------------------------------------------------------------ | --------------- | ------------------------------------------------------------ | ---------------------------- | ------------------------ | ---------------------- |
| ls                                 | 查看当前目录下的直接依赖包                                   | -g              | 全局包                                                       |                              |                          |                        |
|                                    |                                                              | -a              | 间接依赖包                                                   |                              |                          |                        |
| install                            |                                                              |                 |                                                              | install                      |                          | install                |
| run <CMD>                          | 运行 package.json 脚本                                       |                 |                                                              | <CMD>                        |                          | run <CMD>              |
| i                                  | **全局安装：**在命令行上运行。安装位置：在 node 根路径下，即 E:\Develop\nvm\v18.14.2\node_modules<br />**最佳实践**：会互调的包使用全局安装<br />推荐脚手架不要全局安装使用 npx 临时下载安装执行最新远程模块(package.json+入口脚本)<br /><br />**局域安装**，需要`require()`。安装位置：当前目录下创建 node_modules 文件夹<br />**最佳实践**：`npm --registry https://registry.npm.taobao.org install` | --save = -S     | package.json里面的dependencies<br />生产环境需要依赖的包     | add                          |                          | add                    |
|                                    |                                                              | --save-dev = -D | packege.json里面的devDependencies<br />只有开发环境下需要依赖的库，线上生产环境上并不需要他们，如开发中所使用的外部的测试或者文档框架，vue-cli脚手架 | add --dev                    |                          |                        |
| `npm install npm@latest -g`        | 升级 npm                                                     |                 |                                                              | `npm install npm@latest -g`  | 升级 yarn                |                        |
| rebuild                            | 重新编译依赖包                                               |                 |                                                              |                              |                          |                        |
| rm -rf node_modules && npm install | 重新下载依赖包                                               |                 |                                                              | upgrade<br />install --force | 更新可升级<br />忽略缓存 |                        |
| uninstall                          |                                                              |                 |                                                              | remove                       |                          |                        |
| cache clean --force                | 下载失败时，清理缓存                                         |                 |                                                              | cache clean                  |                          |                        |
|                                    |                                                              |                 |                                                              |                              |                          |                        |
| init -y (项目目录下)               | **初始化项目**，创建包信息<br />会创建pakeage.json文件记录项目开发中所要用到的包，以及项目的详细信息<br />使版本迭代和项目移植更加方便，防止误删包<br />pakeage.json中不要添加注释 |                 |                                                              | init                         |                          |                        |
| npm key                            | 执行value中的命令（pakeage.json中script）                    |                 |                                                              |                              |                          |                        |
| npm help <command>                 | 帮助                                                         |                 |                                                              |                              |                          |                        |
|                                    |                                                              |                 |                                                              |                              |                          |                        |
|                                    |                                                              |                 |                                                              |                              |                          | pnpm-lock.yaml         |

pnpm 原理是利用了软硬连接，因为硬链接不能跨盘符，所以会在盘符根目录生成 `E:\.pnpm-store\v3(索引文件)` 、`pnpm store path`
`LOCALAPPDATA%/pnpm/store`

[pnpm官方文档](https://pnpm.io/zh/cli/add)

最佳实践：`npx pnpm install`



# 第三方包

## nodemon

用于热编译代码，自动重启项目工程

| 命令                                   | 释义   |
| -------------------------------------- | ------ |
| nodemon ./server.js [localhost] [8080] | 启动js |
| nodemon -help                          | 帮助   |
|                                        |        |



## es-checker

侦测运行环境支持哪些 ES6 的功能。



## npm-check-updates

[github](https://github.com/raineorshine/npm-check-updates)

`npx npm-check-updates`
