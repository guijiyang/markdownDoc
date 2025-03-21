<!-- @import "../my-style.less" -->

# <center>git 使用方式</center>

## 代码格式说明
- **<>**:自定义的内容，用于替换实际的值，如
- **`<RemoteStorageLocalName>`**：远程仓库别名，例如：origin
- **`<BranchName>`**：分支名称，例如：master
- **`<ProjectMainDirectory>`**：项目主目录
- **`<SubmoduleDirectory>`**：子模块目录
- **A(B)**:表示B是和A等价的可替换的，如`--track(-t)`，表示`--track`和`-t`是等价的

## 创建新库

首先切换到库目录,然后初始化库
**初始化库**

```shell
git init
```

## 库编辑

**新增文件或者有已有变更**

```shell
git add 文件名
```

**提交到 git 上**

```shell
git commit -m "描述信息"
```

## 远程仓库

_关联远程仓库_

```shell
git remote add <RemoteStorageLocalName> <RemoteStorageURL>
```

_获取远程仓库上分支的内容_

```shell
git pull <RemoteStorageLocalName> <BranchName>
```

_将本地分支推送到远程仓库_

```shell
git push <RemoteStorageLocalName> <BranchName>
```

## 子模块

- 在项目的根目录添加子模块

```shell
cd <ProjectMainDirectory>
git submodule add <RemoteStorageURL>  <SubmoduleDirectory>
git submodule init
git submodule update
```

- 从远程仓库上拉取子模块的代码

```shell
#主工程代码当下来之后
git submodule update --init --recursive
git submodule foreach git pull origin master
```

## 分支

```shell
# 显示本地分支列表
git branch
# 显示远程仓库分支列表
git branch -r
# 显示所有分支列表
git branch -a
```

_创建分支_

```shell
git branch <BranchName>
# or
git switch <BranchName>
# or
git switch -c <BranchName>
```

_删除本地分支并同步远程仓库的分支_

```shell
# 首先需要切换到其他分支
git branch <OtherBranchName>
# 删除本地分支
git branch --delete(-d) <BranchName>
# 同步删除远程分支
git push <RemoteStorageLocalName> --delete(-d) <BranchName>
```

_切换或创建分支_

```shell
# 创建并切换到一个本地分支，同时将其跟踪到一个远程分支
git switch -c <BranchName> --track(-t) remotes/<RemoteStorageLocalName>/<BranchName>
# or
git switch -t remotes/<RemoteStorageLocalName>/<BranchName>

# 切换(创建)本地分支
git switch <BranchName>
```

_关联远程分支_

```shell
git branch --set-upstream-to=<RemoteStorageLocalName>/<RemoteBranchName> <BranchName>
# or
git branch -u <RemoteStorageLocalName>/<RemoteBranchName> <BranchName>
```

_合并分支_

```shell
# BranchName是需要合并到当前分支的其他分支。
git merge <BranchName>
```

_显示本地分支信息_
```shell
# 显示本地分支信息
git branch -v
# or 显示本地分支和远程分支信息
git branch -vv
```

## 修改最后一次提交
```shell
# 本地修改最后一次提交
# 覆盖最近一次修改内容
git commit --amend(-a)  
or
git reset --soft HEAD^ #移动HEAD指针到上一个版本，但不改变暂存区和工作区
git commit -C ORIG_HEAD #使用上一次commit的message
# or
git commit -c ORIG_HEAD #修改并使用上一次commit的message
#覆盖最近一次修改的log
git commit --amend -m <NewCommitMessage> 
#覆盖远程前一次提交
git push <RemoteStorageLocalName> <BranchName> --force 
```

## 撤销暂存区的提交

```shell
# 重置单个文件：
git reset <FileName>
# 重置所有文件：
git reset
# 重置单个文件：
git restore --staged <FileName>
# 重置所有文件：
git restore --staged .
```

## 版本回退

- 私有库版本回退

```shell
# 先用下面命令找到要回退的版本的commit id：
git reflog
# 接着回退版本:
git reset --hard <CommitId> #注意使用hard会将HEAD，暂存区和工作区都回退到指定版本，如果想保留这些修改，可以使用soft或mixed参数
# 如果你的错误提交已经推送到自己的远程分支了，那么就需要回滚远程分支了。
# 强制推送到远程分支：
git push -f <RemoteStorageLocalName>
```

CommitId是要回退的版本的ID，是一串16进制数字。

- 公共远程版本回退

```shell
# 撤销最近一次提交
git revert HEAD                     
# 撤销上上次的提交，注意：数字从0开始
git revert HEAD~1                   
# 撤销0ffaacc这次提交
git revert 0ffaacc                  
```

要注意以下几点：

- 1. `revert` 是撤销一次提交，所以后面的`commit id`是你需要回滚到的版本的前一次提交。

- 2. 使用`revert HEAD`是撤销最近的一次提交，如果你最近一次提交是用`revert`命令产生的，那么你再执行一次，就相当于撤销了上次的撤销操作，换句话说，你连续执行两次`revert HEAD`命令，就跟没执行是一样的。
- 3. 使用`revert HEAD~1` 表示撤销最近2次提交，这个数字是从0开始的，如果你之前撤销过产生了commit id，那么也会计算在内的。
- 4. 如果使用 `revert` 撤销的不是最近一次提交，那么一定会有代码冲突，需要你合并代码，合并代码只需要把当前的代码全部去掉，保留之前版本的代码就可以了。

`git revert` 命令的好处就是不会丢掉别人的提交，即使你撤销后覆盖了别人的提交，他更新代码后，可以在本地用 `reset` 向前回滚，找到自己的代码，然后拉一下分支，再回来合并上去就可以找回被你覆盖的提交了。
使用`revert`命令，如果不是撤销的最近一次提交，那么一定会有冲突。
解决冲突很简单，因为我们只想回到某次提交，因此需要把当前最新的代码去掉即可，也就是`HEAD`标记的代码：
```output
<<<<<<< HEAD
```

全部清空
第一次提交

- 5. 错的太远了直接将代码全部删掉，用正确代码替代

## 显示日志

```shell
# 显示粗略信息
git log
# 显示详细信息
git log -p
# 显示相对详细信息
git log --stat
# 显示某一次更改
git show <commitid>
# 显示某一次更改的某文件
git show <commitid> <filename>
# 查看缓存区和上次提交的不同
git diff --staged
# 对比工作目录和缓存区
git diff # 可是显示工作目录和暂存区之间的不同
# 对比工作目录和上一条提交
git diff HEAD # 可以显示工作目录和上一条提交之间不同，就是说：如果你现在把所有的文件都 add 然后 git commit，你将会提交什么
```

## 常见问题

- git pull 报错 error: Your local changes to the following files would be overwritten by merge：
    出现上述报错的原因是因为其他人修改了xxx.java文件并提交到了版本库，而我们本地也修改了xxx.java文件，这时进行pull自然就会产生冲突。
**解决方法**
执行git stash（IDEA中的菜单为Stash Changes）命令将工作区恢复到上次提交的内容，同时将本地和暂存区所做的修改备份，这样整个项目就回到了我们修改之前的状态，这时就可以正常git pull了，git pull完成后，执行git stash pop（IDEA中的菜单为Unstash Changes）命令将之前本地做的修改应用到当前工作区。
**相关命令**
`git stash`
备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到暂存区中。
`git pull`
拉取服务器上的代码到本地。
`git stash pop`
从暂存区读取最近一次保存的内容，恢复工作区的相关内容。由于可能存在多个Stash的内容，所以用栈来管理，pop会从最近的一个stash中读取内容并恢复。
`git stash list`
显示暂存区中的所有备份，可以利用这个列表来决定从那个地方恢复。
`git stash clear`
清空暂存区。
`git stash apply`
这条命令会恢复最近保存的进度。如果你有多个 stash，可以使用 `git stash apply stash@{n}` 来恢复特定的
`git stash drop stash@{n}`
删除一个存储的进度。如果不指定 stash@{n}，则默认删除最新的进度。
`git stash branch <BranchName>`
基于最新的 stash 创建一个新分支。这可以用于在 stash 进度上继续工作，而不是恢复进度并继续在当前分支上工作。
