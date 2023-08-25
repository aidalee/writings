<!--
 * @Author: please
 * @Date: 2023-08-25 10:53:31
 * @LastEditors: please
 * @LastEditTime: 2023-08-25 17:10:29
 * @Description: 请填写简介
-->
## 先创建一个项目
pnpm create vite my-vue-app --template vue

## 根目录新建bin文件夹
  > bin 文件夹下新建`index.js`和`ui.js`，用**commander**库设置子命令,
  ```js
    // index.js
    #!/usr/bin/env node
    const program = require('commander')

    program
    .command('ui')
    .description('start GUI🚀')
    .action(()=>{
      // 子命令执行的函数 这里通过ui.js文件导出该函数
      require('./ui')()
    })

    // 同样的方式设置其它子命令
    // program
    // .command('other')
    // .description('other')
    // .action(()=>{
    //   fn()
    // })

    program.parse(process.argv);
  ```
  > 使用 **inquirer** 库来实现询问式的交互
  ```js
    //ui.js
    const inquirer = require('inquirer')
    const chalk = require('chalk')
    const path = require('path')
    const spawn = require('cross-spawn') // 用于生成子进程

    async function start () {
      console.log(chalk.green('🚀 Starting GUI at localhost:8088, wait a moment'));
      const serverDir = path.join(__dirname, '..', 'server')
      let server;
      server = spawn('node', [path.join(serverDir, 'app.js')])
      // 捕获子进程的输出
      server.stdout.on('data', (data) => {
        console.log(`${data}`)
      })
      // 捕获子进程的错误
      server.stderr.on('data', (data) => {
        console.log(`${data}`)
      })
      // 跟踪子进程的状态
      server.on('close', (code) => {
        console.log(`子进程退出，退出码 ${code}`)
      })
      // 监听子进程的退出
      process.on('exit', function() {
        server.kill()
      })
    }

    module.exports = async() => {
      inquirer.prompt([
        {
          type: 'list',
          name: 'department',
          message: '请选择部门',
          choices: [
            '云平台',
            '系统开发'
          ]
        },
        {
          type: 'input',
          name: 'username',
          message: '请输入姓名'
        }
      ]).then((answers)=>{
        // 获取用户输入参数
        const { department, username } = answers
        // 执行
        start()
      })
    }
  ```

## 根目录创建server文件夹
  > server下创建app.js,node框架我使用的是express，也可以换成你喜欢的koa、nest都行
  ```js
    const express = require('express')
    const path = require('path')
    const { openUrl } = require('./utils')
    const app = express()
    const PORT = 8088
    const server = require('http').Server(app)
    const CACHE_CONTROL = 'no-store, no-cache, must-revalidate, private'



    app.use(express.static(path.resolve(__dirname, '../dist'), { setHeaders }))

    function setHeaders(res, path, stat) {
      res.set('Cache-Control', CACHE_CONTROL)
    }

    server.listen(PORT, () => {
      openUrl(`http://localhost:${PORT}`);
    })

    module.exports = server
  ```
  > utils.js用来放开发用到的一些工具函数
  ```js
    const { exec } = require('child_process')
    /**
    * @description: 
    * @param {*} url
    * @return {*}
    */
    function openUrl (url) {
      switch (process.platform) {
        // window系统使用以下命令在浏览器打开url
        case "win32":
          exec(`start ${url}`)
          break;
        // mac系统使用以下命令在浏览器打开url 
        case "darwin":
          exec(`open ${url}`)
          break;
        default:
          exec(`open ${url}`)
      }
    }

    module.exports = {
      openUrl
    }
  ```
## package.json配置bin选项
  > package.json添加bin和files选项，配置如下:

  ```json
    {
      "name": "lemon-box",
      "private": true,
      "version": "0.0.0",
      "scripts": {
        "dev": "vite --port 8088",
        "build": "vite build",
        "preview": "vite preview"
      },
      "bin": {
        "lemon": "./bin/index.js"
      },
      "files": [
        "bin",
        "server",
        "dist",
        "package.json"
      ],
      "dependencies": {
        "chalk": "3.0.0",
        "commander": "^11.0.0",
        "cross-spawn": "^7.0.3",
        "express": "^4.18.2",
        "fs-extra": "^11.1.1",
        "inquirer": "8.2.5",
        "os": "^0.1.2",
        "vue": "^3.3.4",
        "vue-router": "^4.2.4"
      },
      "devDependencies": {
        "@vitejs/plugin-vue": "^4.2.3",
        "vite": "^4.4.5"
      }
    }
  ```
## 本地测试
  1. `npm link` 生成软链接
  2. 命令行执行 `lemon ui`,出现以下就👌
  ```shell
  ➜  lemon-box lemon ui,
  ? 请选择部门 云平台
  ? 请输入姓名 please
  🚀 Starting GUI at localhost:8088, wait a moment
  ```

其它更多丰富的功能在此基础上进行扩展就行

## 参考

[前端亮点 or 提效？开发一款 Node CLI 终端工具！](https://juejin.cn/post/7178666619135066170?searchId=20230825133000212A52F60C105F815FD7)