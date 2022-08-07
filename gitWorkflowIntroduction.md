# Git 工作流介绍

## [全线性流程](./gitLinearWorkflow.md)
线性流程参考了Gitflow的分支方案, 摒弃了它的分支合并方式.  
### 优点
1. 清晰明了的代码提交历史
2. 代码冲突清晰, 容易解决 
### 缺点
1. 对变基操作要求高
2. 变基导致Commit ID经常产生变动
3. 不适用于多生产环境的代码


## Git Flow


### 优点
1. 当项目中需要同时维护多个生产版本时，该工作流模式非常理想
2. 由于遵循系统化的模式，因此分支命名容易理解

### 缺点
1. Git 的历史记录将变得异常混乱，可读性很差
2. master / develop 分支的割裂使CI/CD流程变得更加困难
3. 当项目维护单一生产环境版本时，该工作流则不适用

### 参考文献
[[1] Vincent Driessen: A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)  
[[2] 知乎: 图解Git Flow工作流](https://zhuanlan.zhihu.com/p/198066289)


## Github Flow

### 优点
1. 该工作流对于CI/CD流程友好
2. 是Git Flow的一种简版替换
3. 当项目维护单一生产环境版本时，该工作流适用

### 缺点
1. 生产环境对应的代码极易处于不稳定状态
2. 不适用于多生产环境的代码
3. Git 提交历史混乱， 可读性差

### 参考文献
[[1] Github 官方文档](https://docs.github.com/cn/get-started/quickstart/github-flow)
