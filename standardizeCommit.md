# 使用Gitlab规范化提交消息

Gitlab 提供了两种类型的钩子： 服务器端钩子和客户端钩子  
这里主要利用服务器端钩子来实现对提交消息的规范化操作  
  
## Server Hooks 服务器端钩子
服务器端钩子## 服务端钩子
#### pre-receive
处理来自客户端的推送操作时, 最先被调用的钩子就是 `pre-receive`.  
它将从标准输入中获取该次推送的三个参数  
```
read line
echo "$line"
-----------------------------------------
# 输出: 父 Commit ID | 当前提交的Commit ID | 推送的分支名
76beee36ecaa1b027dec0dce5390b1a4941fb167 999341425bed6b0b15c97cc8b87598b9d334b8af refs/heads/master
```  
如果它以非零值退出, 所有的推送内容都不会被接受.  
你可以用这个钩子阻止对引用进行非快进（non-fast-forward）的更新, 或者对该推送所修改的所有引用和文件进行访问控制.  

#### update  
`update` 钩子和 `pre-receive` 钩子十分类似, 不同之处在于`update`会为每一个将被更新的引用[分支, 标签]各运行一次.  
假如推送者同时向多个分支推送内容, `pre-receive` 只运行一次，相比之下 `update` 则会为每一个被推送的分支各运行一次.  
区别与`pre-receive`, `update` 接受/提供了三个参数
```
echo $1, $2, $3
-----------------------------------------
# 输出: 推送的分支名 | 父Commit ID | 当前Commit ID
refs/heads/master 76beee36ecaa1b027dec0dce5390b1a4941fb167 999341425bed6b0b15c97cc8b87598b9d334b8af
```
如果 `update` 钩子以非零值退出, 只有相应的那一个分支会被拒绝, 其余的分支依然会被更新. 

#### post-receive
`post-receive` 挂钩在整个推送过程完结以后被触发. 可以用来更新其他系统服务或者通知用户.  
和 `pre-receive` 相同它们都从标准输入获取参数.  
它的应用场景包括给某个邮件列表发信, 通知持续集成(continous integration)的服务器, 或者更新问题追踪系统(ticket-tracking system).   甚至可以通过分析提交信息来决定某个问题(ticket/issue)是否应该被开启, 修改或者关闭.  
该钩子无法终止推送进程, 不过客户端在该钩子结束运行之前都将保持连接状态, 所以如果你想使用该钩子进行其他操作, 请谨慎使用它, 因为它将耗费你很长的一段时间.  


## 客户端钩子

### 提交工作流的钩子
#### pre-commit
`pre-commit` 钩子在提交Commit信息前运行.  
它用于检查即将提交的快照. 例如: 检查是否有所遗漏, 确保测试运行以及核查代码.  
如果该钩子以非零值退出, Git 将放弃此次提交, 不过你可以用 `git commit --no-verify` 来绕过这个环节.  
你可以利用该钩子, 来检查代码风格是否一致, 尾随空白字符是否存在(自带的钩子样例就是这么做的).  


#### prepare-commit-msg
`prepare-commit-msg` 钩子在启动提交信息编辑器之前, 默认信息被创建之后运行. 它允许你编辑提交者所看到的默认信息.   
```
# commit 时使用 -e 选项启用编辑器之前, -t 选项加载模板信息之后.
git commit -e -t template.txt
# 变基启用 -i 交互界面
git rebase -i
```

该钩子接收一些选项: 存有当前提交信息的文件的路径, 提交类型和修补提交的提交的 SHA-1 校验.  
它对一般的提交来说并没有什么用, 然而对那些会自动产生默认信息的提交, 如提交信息模板, 合并提交, 压缩提交和修订提交等非常实用.  你可以结合提交模板来使用它, 动态地插入信息.  

#### commit-msg
`commit-msg` 钩子接收一个参数, 此参数即上文提到的, 存有当前提交信息的临时文件的路径.   
如果该钩子脚本以非零值退出, Git 将放弃提交.   
因此, 该钩子可以用来在提交通过前验证项目状态或提交信息.  

#### post-commit
`post-commit` 钩子在整个提交过程完成后运行.   
它不接收任何参数, 但你可以很容易地通过运行 `git log -1 HEAD` 来获得最后一次的提交信息.  
该钩子一般用于通知之类的事情. 

### Email工作流的钩子
你可以给电子邮件工作流设置三个客户端钩子.  
它们都是由 `git am` 命令调用的, 如果你需要通过电子邮件接收由 `git format-patch` 产生的补丁, 这些钩子也许用得上. 
#### applypatch-msg
第一个运行的钩子是 `applypatch-msg`. 它接收单个参数: 包含请求合并信息的临时文件的名字.  
如果脚本返回非零值, Git 将放弃该补丁.  
你可以用该脚本来确保提交信息符合格式, 或直接用脚本修正格式错误.  

#### pre-applypatch
在 `git am` 运行期间被调用的是 `pre-applypatch`.  
有些难以理解的是, 它正好运行于应用补丁之后, 产生提交之前, 所以你可以用它在提交前检查快照.  
你可以用这个脚本运行测试或检查工作区. 如果有什么遗漏或测试未能通过, 脚本会以非零值退出, 中断 git am 的运行, 这样补丁就不会被提交.


#### post-applypatch
`post-applypatch` 运行于提交产生之后, 是在 `git am` 运行期间最后被调用的钩子.  
你可以用它把结果通知给一个小组或所拉取的补丁的作者, 但你没办法用它停止打补丁的过程. 


### 其它类型的钩子
#### pre-rebase
`pre-rebase` 钩子运行于变基之前, 以非零值退出可以中止变基的过程.  
你可以使用这个钩子来禁止对已经推送的提交变基, Git 自带的 `pre-rebase` 钩子示例就是这么写的. 
#### post-rewrite
`post-rewrite` 钩子被那些会替换提交记录的命令调用.  
比如 `git commit --amend` 和 `git rebase`(不包括 `git filter-branch`).  
它唯一的参数是触发重写的命令名, 同时从标准输入中接受一系列重写的提交记录.  
这个钩子的用途很大程度上跟 `post-checkout` 和 `post-merge` 差不多. 

#### pre-push
`pre-push` 钩子会在 `git push` 运行期间触发, 远程引用已被更新之后, 但还未传输任何文件对象时被调用.  
它接受远程分支的名字和位置作为参数, 同时从标准输入中读取一系列待更新的引用.  
你可以在推送开始之前, 用它验证对引用的更新操作(一个非零的退出码将终止推送过程)

#### post-checkout
在 `git checkout` 成功运行后, `post-checkout` 钩子会被调用.  
你可以根据你的项目环境用它调整你的工作目录. 其中包括处理大型文件, 自动生成文档或进行其他类似这样的操作.  

#### post-merge
在 `git merge` 成功运行后, `post-merge` 钩子会被调用.  
你可以用它恢复 Git 无法跟踪的工作区数据, 比如权限数据.  
这个钩子也可以用来验证某些在 Git 控制之外的文件是否存在, 这样你就能在工作区改变时, 把这些文件复制进来.  

#### pre-auto-gc
`pre-auto-gc`钩子在垃圾回收发生之前被调用, 可用于通知用户.  

> 注[1]: 需要注意的是, 克隆某个版本库时, 它的客户端钩子==并不==随同复制. 如果需要靠这些脚本来强制维持某种策略, 建议你在服务器端实现这一功能.   
> 注[2]: 在git-v2.9.* 起, 引入了一个新的参数`core.hooksPath`, 它可以为本地仓库指定具体的客户端钩子的位置.  
> 可以使用如下命令来启用它: `git config --global core.hooksPath "~/custom_hooks"`  


## GITLAB 环境变量
> 注: 仅服务器端的钩子可用

环境变量 | 描述
---|---
`GL_ID` | 当前提交推送作者的gitlab标识
`GL_PROJECT_PATH` |	gitlab的项目路径
`GL_PROTOCOL` |	推送协议
`GL_REPOSITORY` | 项目ID
`GL_USERNAME` |	当前提交推送作者的gitlab用户名

对于`pre-receive` 和 `post-receive`钩子可使用的额外环境变量:  

环境变量 | 描述
---|---
`GIT_ALTERNATE_OBJECT_DIRECTORIES` | 隔离环境中的备用对象目录
`GIT_OBJECT_DIRECTORY` | 隔离环境中的GitLab项目路径
`GIT_PUSH_OPTION_COUNT` | 推送选项的数量
`GIT_PUSH_OPTION_<i>` | 推送选项


## 参考文献

[[1] Git SCM 文档](https://git-scm.com/docs/githooks)  
[[2] Gitlab Server Hooks 文档](https://docs.gitlab.com/ee/administration/server_hooks.html)


