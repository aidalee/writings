<!--
 * @Author: please
 * @Date: 2023-10-09 17:58:15
 * @LastEditors: please
 * @LastEditTime: 2023-10-09 17:58:18
 * @Description: 请填写简介
-->
要通过Node.js向GitLab仓库创建项目，可以使用GitLab API。以下是一个简单的示例：

1. 首先，确保已经安装了`node-gitlab`库。如果没有，请运行以下命令安装：

```bash
npm install gitlab
```

2. 创建一个名为`create-project.js`的文件，并在其中添加以下代码：

```javascript
const gitlab = require('gitlab');
const config = {
  url: 'https://gitlab.example.com', // 替换为你的GitLab实例的URL
  token: 'your-private-token' // 替换为你的私人访问令牌
};

const gitlabApi = gitlab(config);

const projectName = 'my-new-project'; // 替换为你想要的项目名称
const description = 'This is a new project created using Node.js'; // 项目描述

async function createProject() {
  try {
    const project = await gitlabApi.Projects.create({ name: projectName, description });
    console.log(`Project "${project.name}" created successfully.`);
  } catch (error) {
    console.error('Error creating project:', error);
  }
}

createProject();
```

3. 替换`config`对象中的`url`和`token`值为你的GitLab实例的URL和私人访问令牌。

4. 保存文件并在命令行中运行以下命令：

```bash
node create-project.js
```

如果一切正常，你应该会看到类似以下的输出：

```
Project "my-new-project" created successfully.
```