# 使用Gitlab规范化提交消息

Gitlab 提供了两种类型的钩子： 服务器端钩子和客户端钩子  
这里主要利用服务器端钩子来实现对提交消息的规范化操作  
  
## Server Hooks 服务器端钩子
服务器端钩子分为三种：  
1. pre_receive
2. post_receive
3. update

### `pre_receive`
客户端推送代码后, gitlab 首先调用这个钩子, 如果以非零值退出则所有的更新都不会被接受.  
对提交信息的规范化主要通过该钩子函数实现, 一旦提交信息不满足要求则拒绝更新.  
代码格式化要求和代码风格检查也可以在此处实现.  

## 参考文献
1. [Gitlab - Server Hook 官方文档](https://docs.gitlab.com/ee/administration/server_hooks.html)


