## nrm

**用途**：对镜像源的管理

**官网**：<https://www.npmjs.com/package/nrm>

**安装与使用**

```shell
npm install -g nrm // 安装
nrm ls // 查看所有镜像
nrm current // 查看当前镜像
nrm add <name> <url> // 增加镜像
nrm del <name> //移除镜像
nrm test <name> // 测试镜像源的响应时间
nrm use <name> // 使用指定的镜像
```

## nvm

**用途**：管理node的版本

**官网**： <https://github.com/nvm-sh/nvm>

**安装**

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash //安装

安装完如果有以下提示 复制并执行
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

$ nvm // 测试是否安装成功

```

**使用**

- 安装最新稳定版 `node`

  ```shell
  nvm install stable
  
  ```

  

- 安装指定版本

  ```shell
  nvm install <version>
  ```

  

- 删除已安装的指定版本

  ```shell
  nvm uninstall <version>
  ```

- 切换使用指定的版本 `node`

  ```shell
  // 临时版本 - 只在当前窗口生效指定版本
  nvm use <version>
  
  // 永久版本 - 所有窗口生效指定版本
  nvm alias default <version>
  
  ```

- 列出所有安装的版本

  ```shell
  nvm ls
  ```

- 列出所有远程服务器的版本

  ```shell
  nvm ls-remote
  ```

- 显示当前的版本

  ```shell
  nvm current
  ```

- 给不同的版本号添加别名

  ```shell
  nvm alias <name> <version>
  ```

- 删除已定义的别名

  ```shell
  nvm unalias <name>
  ```

- 在当前版本 `node` 环境下，重新全局安装指定版本号的 `npm` 包

  ```shell
  nvm reinstall-packages <version>
  ```

- 查看更多命令可在终端输入

  ```shell
  nvm
  ```

  

