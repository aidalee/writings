## react native基础设置概览

![](https://tva1.sinaimg.cn/large/008i3skNly1grz7zm9d51j30rs0dqqqx.jpg)

## 脚手架概览

![](https://tva1.sinaimg.cn/large/008i3skNly1grz8f7lhd5j31cc0rgtcm.jpg)

## 打包平台界面

![](https://tva1.sinaimg.cn/large/008i3skNly1grz8qyt985j313c0ryad0.jpg)

打包平台使用 node 开发，包括以下核心功能

![](https://tva1.sinaimg.cn/large/008i3skNly1grz95k21dkj30rs0dy0z7.jpg)

发布平台例图：
![](https://tva1.sinaimg.cn/large/008i3skNly1grz9o155saj30rs0e8tie.jpg)

发布平台同样使用node开发：

![](https://tva1.sinaimg.cn/large/008i3skNly1grz9o2hpxkj30rs0fkjyf.jpg)

功能点分析：

- 提供热更新分发功能
  - 按渠道分发，便于测试：按ios/安卓/...等渠道发布
  - 增量更新节省网络资源：使用bsdiff算法实现增量更新
  - 提供必要的API给devops：集成到公司的devops平台，形成了前端CICD的闭环

打包发布流程图：

![](https://tva1.sinaimg.cn/large/008i3skNly1grz9ws393hj30rs0eytgf.jpg)

监控平台图例：

![](https://tva1.sinaimg.cn/large/008i3skNly1grza7t2vnzj30rs0eq46a.jpg)

上报session 查看操作信息等

![](https://tva1.sinaimg.cn/large/008i3skNly1grzain7zl1j30rs0gadlr.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNly1grzaiotq88j30rs0csqd1.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNly1grzaiqg6pzj30rs0bun65.jpg)

展望：rn具有快速迭代更新的优势/探索与其他端的融合