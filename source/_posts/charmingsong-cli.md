---
title: 个人脚手架设计搭建
thumbnail: https://ws1.sinaimg.cn/large/8449ed5dly1ga23grlzdmj20p00dwgm0.jpg
categories:
  - node
tag:
  - typescript
---

# 个人脚手架搭建 -- charmingsong-cli

## 目的

**为了解决多次构建相同功能的项目,在一定程度上需要定制化以及私有化设置**

## 设计

#### 问题

**为什么不用现成的脚手架生成？**

- 如果利用`vue`或者`react`的脚手架搭建项目可能创建完项目后还需要一系列的配置以及更改展示样式，并没有真正的解决问题。并且各自的脚手架只能生成各自的框架结构，其他项目不支持。例如，用`vue-cli`生成一个`react`项目,是不可以的。

**为什么不直接复制相同的项目？**

- 现有的项目中逻辑代码关联性很强，并且项目中可能存在一些基于项目环境的代码，直接复制项目更改名称工程量较大。环境错误不易发觉。

#### 整体思路

为完成所需功能，主要需要实现两个模块

- **工具**——cli
- **模板**——template

`cli`： 就是和用户交互部分的工具， 需要知道用户所需。

`template`： 项目模板，可以自己随意定制。

##### 设计目的

- 模板与工具分开，各自单独维护
- `cli`负责获取模板列表，下载、渲染模板
  - 模板可以自行增加删除修改，不需要重新发版新的包
  - 项目逻辑都放到模板中，模板可以设计私有化属性

###### cli 工作流程图

![](https://ws1.sinaimg.cn/large/8449ed5dly1ga234pwenpj20jn0p4wfo.jpg)

## 启程

> 代码是利用`typescript`实现的，具体代码可以查看项目地址 **[charmingsong-cli](https://github.com/web-songsong/charmingsong-cli)**
>
> _文章不会用大量的代码填充讲解，主要会记录一些功能性模块代码_

### 目录结构

```
.
├── .DS_Store
├── .eslintrc.json
├── .gitignore
├── .npmignore
├── .npmrc
├── README.md
├── cs-config.json
├── package-lock.json
├── package.json
├── src
│   ├── index.ts
│   ├── libs
│   │   ├── inquirer.ts
│   │   ├── program.ts
│   │   └── tools.ts
│   ├── typing.d.ts
│   └── utils
│       └── logger.ts
├── tsconfig.json
└── yarn.lock
```

- `tsconfig.json`： ts 的配置文件，可以查询官网，了解的具体设置。

- `package.json`

  ```javascript
  // ...
  "scripts": {
    "build": "tsc", // 打包
    "dev": "tsc -w", // 开发调试
    "eslint": "eslint src --ext .ts", // eslint 检查以 .ts 为后缀的文件
    // ...
  },
  "bin": {
    "cs": "bin/index.js"
    // 全局安装(npm i -g xxx) 全局运行时执行的文件 然后全局就可以使用命令 cs
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run eslint"
      // 主要是为了在提交commit的时候检测代码是否符合标准，如果感兴趣可以了解一下husky
    }
  },
  // ...

  > [husky](https://github.com/typicode/husky)
  ```

- `src/`： 代码文件目录

  - `index.ts` ： 主文件

    ```bash
    #!/usr/bin/env node

    // ...
    ```

    全局安装运行命令式， 在文件开头加上 `#!/usr/bin/env node` 在直接运行时会用本机中的 `node`环境来运行文件。

  - `typing.d.ts`： `typescript`的描述文件，个人理解就是用来将一些没有`ts`全局获取不到的包， 定义个环境提供给`typescript` 。 （深入学习 ts 后来补充 `xxx.d.ts` 文件所代表的意义）

  - `utils/`：公共方法目录

    - `logger.ts` ： 提供一个在终端输出特殊样式的方法

  - `libs/`：环境工具目录

    - `program.ts`： 终端命令定义工具， 主要就是使用[commander](https://www.npmjs.com/package/commander)包，对其进行私有化设置封装
    - `inquirer.ts`：终端交互式界面提示工具，获取用户的自定义信息，利用[inquirer](https://www.npmjs.com/package/inquirer)来实现，和 终端命令定义工具 搭配使用。

    - `tools.ts`：主要提供流程需要的函数方法， 如：下载模板，渲染模板等

### 实现流程

> 项目地址已经贴出来了， 所以不讲解每个文件的代码实现逻辑
>
> 因为每个文件都是各司其职， 这里只记录主文件的实现逻辑

#### 主文件解读

> _建议不懂的包自行去官网看一下，一方面不论我怎么讲解都没有官网讲解的深刻全面，另一方面我可能有的地方也不是很了解其原理本质，如果有错误的地方，欢迎指出，共同进步_

1. 利用`commander`包全局注册一个 `init` 的命令

   ```typescript
   import program from './libs/program'
   // ./libs/program文件就是对commander设置了一下基本信息，然后重命名、引入

   async function commandInit(appName: string) {}

   program.command('init <project-name>').action((appName: string) => {
     commandInit(appName)
   })

   // commander 的用法，可以设置命令行的参数
   // 当用户输入 之前 cs init xxx 的时候
   //   cs ==> package.json文件的设置的环境变量 我设置名称是  cs
   //   init ==> 是 cs 后面带的命令， 这里设置的是 init
   //   xxx ==> 这里是指文件名，是用户输入的参数，此处用法 commander 包有关， 建议仔细看一下用法
   //  commandInit ==> 如果用户输入命令是 init 时所执行的函数。

   program.parse(process.argv)
   // 将node的环境变量给 program ，简单来说就是将终端参数交个 commander 来解析

   if (!process.argv.slice(2).length) {
     program.outputHelp()
   }
   // 这里是对 cs 命令的 提示帮助， 如果用户在终端只输入 cs ，而不输入命令 那么展示 提示信息
   ```

2. 实现定义的命令函数 commandInit

   ```typescript
   import { join } from 'path'
   import { existsSync, mkdirSync } from 'fs'

   import Logger from './utils/logger' // 在终端输出信息，可以捕捉错误

   import {
     downTemplate, // 下载模板方法
     getMetaJson, // 下载需要的初始json文件方法
     writeTemplate // 渲染文件方法
   } from './libs/tools'
   // ./libs/tools 是方法库，具体的代码实现可以参考源代码。

   import prompt from './libs/inquirer' // inquirer封装，终端交互的工具

   import { metaName } from '../cs-config.json'
   // 获取 inquirer 所需要初始化的json信息，也就是最开始提问的json

   async function commandInit(appName: string) {
     const targetDir = join(process.cwd(), appName)
     // 查看目标目录是否存在

     existsSync(targetDir)
       ? Logger.fatal('The current directory already exists --> %s ', appName)
       : mkdirSync(targetDir)
     // 存在就对用户，输出错误提示：目录存在， 如果不存在，就创建用户所输入的名称的文件夹

     const baseInfo: MetaInfoMust = await prompt(await getMetaJson())
     // 获取 模板列表的json 提供给 prompt,来和用户进行交互

     const { template } = baseInfo
     // 获取用户选择的模板信息

     const templatePath = (await downTemplate(template)) as string
     Logger.success('template download success！')
     // 下载目标模板到本地， 并提示成功信息

     const templateMetaPath = join(templatePath, metaName)

     const templateMetaInfo = existsSync(templateMetaPath)
       ? await prompt(require(templateMetaPath))
       : {}
     // 如果模板中需要自定义设置， 读取配置，在和用户交互

     const metaInfo = { ...baseInfo, ...templateMetaInfo }
     // 获取两次的交互所得的用户信息

     await writeTemplate(templatePath, metaInfo, targetDir)
     Logger.success('success ok!')
     // 根据获取的用户自定义信息，向开始创建的目标目录写入、渲染模板并提示成功信息

     process.exit(1)
     // 退出命令程序
   }
   ```

#### 模板的设计

> 模板类型是根据[github](https://github.com/web-songsong)纪录的模板类型
>
> 模板类型以 `cs-templates-xxx`格式命名，可用模板列表可自行查看本项目 `master_meta`分支

##### 模板开发规则

> 可自行添加模板

##### 目录

```bash
.
├── README.md
├── meta.json
└── template
```

模板写入利用 `handlebars`,

例如：

```json
{
  "name": "{{projectName}}",
  "version": "{{version}}",
  "description": "{{description}}"
}
```

##### 默认交互

模板名称--`template`

项目名称--`projectName`

简介描述--`description`

版本--`version`

##### 自定义交互

> 可以自行配置`meta.json`,依据`inquirer`语法配置， 自动解析。

例如：

```json
[
  {
    "name": "testname",
    "type": "input",
    "message": "测试",
    "default": "test"
  }
]
```
