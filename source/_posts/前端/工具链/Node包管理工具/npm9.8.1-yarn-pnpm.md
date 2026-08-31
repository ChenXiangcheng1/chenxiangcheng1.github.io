# npm9.8.1

npm 是 **node 包管理工具**，在安装 Node.js 时，会自动安装 npm，但是 npm 的发布比 Node.js 更频繁，因此记得更新 npm。

[npm CLI 文档](https://docs.npmjs.com/cli/v9/commands)	|	[npm 包搜索引擎](https://www.npmjs.com/package/package)  
[npmleaderboard](https://www.npmleaderboard.org/)  

## npm配置

[.npmrc 配置](https://docs.npmjs.com/cli/v9/using-npm/config)

[npm 注册中心镜像](https://npmmirror.com/)


## 命令

<https://biaoyansu.com/20.4>

| npm 命令                           | 释义                                                         | 参数            | 释义                                                         | yarn命令                     | 释义                     | pnpm<br />(node>18.12) |
| ---------------------------------- | ------------------------------------------------------------ | --------------- | ------------------------------------------------------------ | ---------------------------- | ------------------------ | ---------------------- |
| ls                                 | 查看当前目录下的直接依赖包                                   | -g              | 全局包(nodejs分发包括npm、corepack)                                                       |                              |                          |                        |
|                                    |                                                              | -a              | 间接依赖包                                                   |                              |                          |                        |
| install                            |                                                              |                 |                                                              | install                      |                          | install                |
| run <CMD>                          | 运行 package.json 脚本                                       |                 |                                                              | <CMD>                        |                          | run <CMD>              |
| i                                  | **全局安装：**在命令行上运行。安装位置：在 node 根路径下，即 E:\Develop\nvm\v18.14.2\node_modules<br />**最佳实践**：会互调的包使用全局安装<br />推荐脚手架不要全局安装使用 npx 临时下载安装执行最新远程模块(package.json+入口脚本)<br /><br />**局域安装**，需要`require()`。安装位置：当前目录下创建 node_modules 文件夹内<br />**最佳实践**：`npm --registry https://registry.npm.taobao.org install` | --save = -S     | package.json里面的dependencies<br />生产环境需要依赖的包     | add                          |                          | add                    |
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
| | | | | | | exec 执行node_modules/.bin/xxcmd |
| |                                                              |                 |                                                              |                              |                          | pnpm-lock.yaml(lockfile)         |


# pnpm

pnpm 原理是利用了软硬连接，因为硬链接不能跨盘符，所以会在盘符根目录生成 `E:\.pnpm-store\v3(索引文件)` 、`pnpm store path`
`LOCALAPPDATA%/pnpm/store`

[pnpm官方文档](https://pnpm.io/zh/cli/add)

最佳实践：  
~~`npx pnpm install`~~, npx非常适合脚手架等一次性工具, 但pnpm项目需要指定packageManager, 因此还是要使用mise在项目目录安装pnpm

## 项目配置

```json
# npx pnpm init创建的package.json
{
  "name": "hello-typescript",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devEngines": {
    "packageManager": {
      "name": "pnpm",
      "version": "11.23.0",
      "onFail": "download"
    }
  },
  "packageManager": "pnpm@11.23.0",
  "type": "module"
}
```

```json
{
  "bin": { // npx --package <package-spec> xxbin
    "skills": "./bin/cli.mjs",
    "add-skill": "./bin/cli.mjs"
  },
  "scripts": {  // npm run xxscript会执行scripts
    "build": "tsc && shx chmod +x dist/*.js",
    "prepare": "npm run build",  // npm i会触发prepare(例: tsc编译 初始化git hooks)
    "watch": "tsc --watch",
    "test": "vitest run --coverage"
  },
  // module(.mjs) commonjs(.cjs)
  "type": "module"
}
```

TODO: 基于package.json, npm的生命周期?


## 命令

```bash

```

```bash
$ pnpm help -a
Version 11.23.0 (compiled to binary; bundled Node.js v26.0.0)
Usage: pnpm [command] [flags]
       pnpm [ -h | --help | -v | --version ]

Manage your dependencies:
      add                  Installs a package and any packages that it depends on. By default, any new package is installed as a prod dependency
      clean                Safely remove node_modules directories from all workspace projects
      dedupe               Perform an install removing older dependencies in the lockfile if a newer version can be used
      fetch                Fetch packages from a lockfile into virtual store, package manifest is ignored
      import               Generates a pnpm-lock.yaml from an npm package-lock.json (or npm-shrinkwrap.json) file
   i, install              Install all dependencies for a project
  it, install-test         Runs a pnpm install followed immediately by a pnpm test
  ln, link                 Connect the local project to another one
      prune                Removes extraneous packages
  rb, rebuild              Rebuild a package
  rm, remove               Removes packages from node_modules and from the project's package.json
      unlink               Unlinks a package. Like yarn unlink but pnpm re-installs the dependency after removing the external link
  up, update               Updates packages to their latest version based on the specified range

Patch your dependencies:
      patch                Prepare a package for patching
      patch-commit         Generate a patch out of a directory
      patch-remove         Remove existing patch files

Review your dependencies:
      audit                Checks for known security issues with the installed packages
      licenses             Check licenses in consumed packages
  ls, list                 Print all the versions of packages that are installed, as well as their dependencies, in a tree-structure
      outdated             Check for outdated packages
      why                  Shows all packages that depend on the specified package

Run your scripts:
      approve-builds       Approve dependencies for running scripts during installation
      create               Create a project from a "create-*" or "@foo/create-*" starter kit
      dlx                  Fetches a package from the registry without installing it as a dependency, hot loads it, and runs whatever default command binary it exposes
      exec                 Executes a shell command in scope of a project
      ignored-builds       Print the list of packages with blocked build scripts
      run                  Runs a defined package script
      start                Runs an arbitrary command specified in the package's "start" property of its "scripts" object
   t, test                 Runs a package's "test" script, if one was provided

Other:
      bin                  Prints the directory into which the executables of dependencies are linked
   c, config               Manage the pnpm configuration files
      deploy               Deploy a package from a workspace
      init                 Create a package.json file
      pack                 Create a tarball from a package
      publish              Publishes a package to the registry
      root                 Prints the effective modules directory
      stage                Stage packages for publishing

Manage your engines:
      env                  Manage Node.js versions (deprecated, use runtime)
  rt, runtime              Manage runtimes
      self-update          Updates pnpm to the latest version

Inspect your store:
      cat-file             Prints the contents of a file based on the hash value stored in the index file
      cat-index            Prints the index file of a specific package from the store
      find-hash            Experimental! Lists the packages that include the file with the specified hash

Manage your store:
      store add            Adds new packages to the pnpm store directly. Does not modify any projects or files outside the store
      store path           Prints the path to the active store directory
      store prune          Removes unreferenced (extraneous, orphan) packages from the store
      store status         Checks for modified packages in the store

Manage your cache:
      cache delete           Experimental! Deletes metadata cache for the specified package(s). Supports patterns
      cache list             Experimental! Lists the available packages metadata cache. Supports filtering by glob
      cache list-registries  Experimental! Lists all registries that have their metadata cache locally
      cache view             Experimental! Views information from the specified package's cache

Options:
  -r, --recursive          Run the command for each project in the workspace.
```

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



## skills

[官网](https://www.skills.sh/trending)

[skills#github](https://github.com/vercel-labs/skills): 对 `skills/<category>/<category>/<name>/SKILL.md` 搜索skill

```package.json
"bin": {
  "skills": "./bin/cli.mjs",
  "add-skill": "./bin/cli.mjs"
}  # 表示该npm package暴露了两个CLI命令skills、add-skill(等价)
```

入口: src/cli.ts



> 还不支持 deepseek harness



使用

```bash
npx skills --help

npx skills list

npx skills find --owner vercel --full-depth  # 交互式搜索

# package: https://github.com/vercel-labs/agent-skills git@github.com:vercel-labs/agent-skills.git ./my-local-skills
npx skills add <package> --list  # 不安装列出目标仓库里可用skill
GITHUB_TOKEN=xxx npx skills add <package> -g --skill <skills> --agent claude-code  # 默认安装到./<agent>/skills/，全局安装到~/<agent>/skills/	

npx skills use <package> --skill <skill> --agent claude-code
npx skills use <package>@<skill>  # 输出prompt

npx skills remove  # 交互式移除

npx skills update # 交互式升级

npx skills init <sub_dir>  # 在sub_dir创建一个SKILL.md模板
```



