- 添加子包

  ```shell
  lerna create <pkgName>
  ```

  

- 给子包添加依赖

  ```shell
  
  
  ```

  

- 清空所有子包的node_modules

  ```shell
  lerna clean
  ```

  

由于yarn和lerna在功能上有较多的重叠,我们采用yarn官方推荐的做法,用yarn来处理依赖问题，用lerna来处理发布问题。能用yarn做的就用yarn做吧



- `devDependencies`用于本地环境开发 --save-dev
- `dependencies`用户发布环境 --save
- `devDependencies`是只会在开发环境下依赖的模块，生产环境不会被打入包内。通过`NODE_ENV=developement`或`NODE_ENV=production`指定开发还是生产环境。
- `dependencies依赖的包不仅开发环境能使用，生产环境也能使用`

## 流程

- GitHub 新建空项目仓库

- 克隆到本地

- 全局安装lerna 工具  npm install lerna -g

- 使用 dumi 初始化一个站点模式的组件库开发脚手架。yarn create @umijs/dumi-lib --site

- 生产的文件目录中src文件夹是用来放组件源码的。由于要做的是一个存在多包的项目所以删掉src,使用lerna init初始化为lerna的monorepo单体项目。可以看到目录中生成了packages文件夹。后面开发的子包都放在这个文件夹下。

- 在packages下创建子包。可以手动创建，也可以使用命令初始化创建。

  ### learn 两种模式

  - 固定模式：所有包是统一的版本号，每次升级，所有包版本统一更新，不管这个包内容改变与否

    具体体现在，`lerna` 的配置文件 `lerna.json` 中永远会存在一个确定版本号：

    ```json
    {
        "version": "0.0.1"
    }
    ```

    

  - 独立模式：每个包是单独的版本号，每次lerna 触发发布命令，每个包的版本都会单独变化

    具体体现在，`lerna` 的配置文件 `lerna.json` 中没有一个确定版本号，而是：

    ```json
    {
        "version": "independent"
    }
    
    ```

    

  ### learn.json

  ```json
  {
      "useWorkspaces": true, // 使用 workspaces 配置。此项为 true 的话，将使用 package.json 的 "workspaces"，下面的 "packages" 字段将不生效
      "version": "0.1.0", // 所有包版本号，独立模式-"independent"
      "npmClient": "cnpm", // npm client，可设置为 cnpm、yarn 等
      "packages": [ // 包所在目录，可指定多个
          "packages/*"
      ],
      "command": { // lerna 命令相关配置
          "publish": { // 发布相关
              "ignoreChanges": [ // 指定文件或目录的变更，不触发 publish
                  ".gitignore",
                  "*.log",
                  "*.md"
              ]
          },
          "bootstrap": { // bootstrap 相关
              "ignore": "npm-*",  // 不受 bootstrap 影响的包
              "npmClientArgs": [ // bootstr 执行参数
                  "--no-package-lock"
              ]
          }
      }
  }
  ```

  ### 关于yarn workspaces https://classic.yarnpkg.com/en/docs/workspaces

  在插件使用 `dependencies` 声明依赖库的特点：

  - 如果用户显式依赖了核心库，则可以忽略各插件的 `peerDependency` 声明；
  - 如果用户没有显式依赖核心库，则按照插件 `peerDependencies` 中声明的版本将库安装到项目根目录中；
  - 当用户依赖的版本、各插件依赖的版本之间不相互兼容，会报错让用户自行修复；

  ###  learn 命令

  - 创建子包： learn create <pkgName>
  - 安装公共依赖：yarn add -WD <pkgName>
  - 给指定package 安装依赖：yarn workspace module-1 add <pkgName>
  - 给所有 package 安装依赖：lerna add <pkgName>
  - workspace 之间的依赖：lerna add module-2 --scope module-1

## 参考文档

https://cloud.tencent.com/developer/article/1854807?from=15425

https://www.jianshu.com/p/3fff3be52395