---
title: Typescript 构建命令行工具上手指南
date: 2017-11-28 20:17:10
tags: [typescript, javascript]
---

这篇小教程里演示使用 [TypeScript](https://www.typescriptlang.org/) 构建命令行工具，利用 [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)/[await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) 进行非阻塞操作，利用 [mocha](https://mochajs.org/) 自动化测试以及 [travis-ci](https://travis-ci.org/) 进行持续集成。
<!-- more -->

# Intro

最近 TJ 发布了 [node-prune](https://github.com/tj/node-prune) 进行对 `node_modules` 里冗余文件的清理，但项目由 Go 写成，于是我移植了一个 JavaScript 版本。可以搭配[源码](https://github.com/xg-wang/pruner-cli)配合继续阅读文章。项目由 TypeScript 构建，npm 发布时自动转译成 JavaScript，因此可以随意使用 async/await 等等最新语法和类型检测。同时利用 [ts-node](https://github.com/TypeStrong/ts-node)直接在 TypeScript 环境下进行调试。项目代码量很少，适合作为类似小工具的模版～

# TypeScript Setup

项目最终的结构如下，源码放在 `src/` 目录下，最终转码到 `lib/` 目录发布到 npm，`test/`目录下是测试代码。

```plain
.
├── src/
├── test/
├── lib/
├── node_modules/
├── LICENSE
├── README.md
├── package-lock.json
├── package.json
└── tsconfig.json
```

首先使用 `npm init` 进行项目的初始化并安装 TypeScript

```bash
npm i typescript -D
```

输入完毕后打开 `package.json` 添加：

```json
{
    "scripts": {
        "build": "tsc",
        "dev": "tsc -w",
        "prepare": "npm run build",
    }
},
```

`tsc` 是 TypeScript 转译的命令，`-w` 参数可以观察源码的变化持续转译，`prepare` 指令会在 `npm install` 和 `npm publish` 之前执行，用来保证发布时是最新转译的代码。

为了告诉 `tsc` 如何进行转译，需要一个配置文件 [tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)，在命令行 `tsc --init` 可以生成一个默认的模版。如果要支持 Node6.x, 7.x，修改 `"target": "ES2015"`，对于 Node 项目。模块生成需要 `"module": "commonjs"`，之后指定以下包含的 ts 文件即可。

```json
{
  "compilerOptions": {
    "target": "ES2015",
    "module": "commonjs",
    "outDir": "./lib",
    "strict": true
  },
  "exclude": [
    "node_modules",
    "lib"
  ],
  "include":[
    "src/**/*.ts"
  ]
}
```

现在我们就可以愉快地写 TS 代码啦，试验一下写一个 walk 所有文件和文件夹的函数，

```ts
import * as fs from 'fs-extra';
import * as path from 'path';

export async function walk(dir: string, prunerF: (p: string, s: fs.Stats) => void): Promise<void> {
  let s = await fs.lstat(dir);
  if (!s.isDirectory()) return;

  const items = await fs.readdir(dir);
  for (let item of items) {
    const itemPath = path.join(dir, item);
    const s = await fs.lstat(itemPath);
    const pruned = await prunerF(itemPath, s);
    if (!pruned && s.isDirectory()) {
      await walk(itemPath, prunerF);
    }
  }
}
```

注意这里我们可以直接使用 async/await，并对传入参数进行类型标记，如果你使用有插件支持的编辑器，这时可以感受到智能补全带来的愉快体验了。

`npm build` 一下可以在 `lib/` 里面看到转译出来的 ES2015 代码，稍微看一眼，可以发现 async 是通过定义了一个 `__awaiter` 函数来进行的，感兴趣的同学可以自行研究。

```js
var __awaiter = (this && this.__awaiter) || function (thisArg, _arguments, P, generator) {
    return new (P || (P = Promise))(function (resolve, reject) {
        function fulfilled(value) { try { step(generator.next(value)); } catch (e) { reject(e); } }
        function rejected(value) { try { step(generator["throw"](value)); } catch (e) { reject(e); } }
        function step(result) { result.done ? resolve(result.value) : new P(function (resolve) { resolve(result.value); }).then(fulfilled, rejected); }
        step((generator = generator.apply(thisArg, _arguments || [])).next());
    });
};
```

# CLI

通过 `node file.js` 可以运行一个脚本，但要怎样安装一个 npm 的可执行文件呢，根据 npmjs.com 的[描述](https://docs.npmjs.com/files/package.json#bin)，在 `package.json` 中添加

```json
{
    "bin": {
        "prune": "lib/cli.js"
    }
}
```

之后就可以安装一个 `prune` 到系统路径中，而执行的文件顶部需要添加 `#!/usr/bin/env node`，否则不会被识别为 Node 脚本。

为了读取命令行传入的参数，可以使用 args，在 `src/cli.ts` 中这段代码

```ts

const argv = yargs
  .usage('Prune node_modules files and dependencies\n\nUsage: node-prune <path>')
  .option('config', {
    alias: 'c',
    description: '<filename> config file name',
    default: '.prune.json',
    type: 'string'
  })
  .option('dryrun', {
    alias: 'd',
    description: 'dry run',
    default: 'false',
    type: 'boolean'
  })
  .option('verbose', {
    description: 'log pruned file info',
    default: 'false',
    type: 'boolean'
  })
  .help('help').alias('help', 'h')
  .version('version', '0.1.0').alias('version', 'v')
  .argv;

const path = argv._[0] || 'node_modules';
const configs = {
  config: argv.config,
  dryrun: argv.dryrun,
  verbose: argv.verbose
};
```

可以生成这样的结果：

```bash
$ prune -h
Prune node_modules files and dependencies

Usage: node-prune <path>

Options:
  --config, -c   <filename> config file name   [string] [default: ".prune.json"]
  --dryrun, -d   dry run                            [boolean] [default: "false"]
  --verbose      log pruned file info               [boolean] [default: "false"]
  --help, -h     Show help                                             [boolean]
  --version, -v  Show version number                                   [boolean]
```

获取参数后就可以 import 你写的业务代码进行操作，这里略去，可以在 [Github](https://github.com/xg-wang/pruner-cli/blob/master/src/Pruner.ts) 看一下例子。

之后我们可以通过 `npm publis` 发布后 `npm install -g pruner-cli` 下载安装工具之后执行 `prune` 直接调用。

# Async Test

测试代码同样需要使用 TypeScript 以及 async/await。选用 mocha 进行 BDD 风格的测试。chai 是一个 assertions 库，搭配使用效果佳。

```bash
$ npm install mocha ts-node -g
$ npm install mocha chai ts-node --save-dev
```

正常使用 mocha 会在 `test/` 目录下执行 JavaScript，为了跳过转译直接测试 TS 代码，我们可以绑定 `ts-node` 直接执行测试代码，在 `package.json` 中加入：

```json
{
    "scripts": {
        "test": "mocha -r ts-node/register test/**/*.spec.ts"
    },
}
```

于是 `test/` 下所有的 `*.spec.ts` 都会被测试，并且可以使用 await 异步，expect 实现 BDD。一个[例子](https://github.com/xg-wang/pruner-cli/blob/master/test/Walker.spec.ts)。

# Travis

在项目下添加 `.travis.yml` 后添加项目语言的配置

```yml
language: node_js
node_js:
  - "6"
  - "7"
  - "8"
  - "9"
```

然后在 https://travis-ci.org/ 添加自己开源的项目就可以在每次 push 时自动测试编译和 test。

![travis](/2017/11/Typescript-构建命令行工具上手指南/travis.png)

点击 `build|passing` 后将图片的链接贴到项目的 README 中就可以在 Github 上显示 CI 的状态了！
