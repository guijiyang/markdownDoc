<!-- @import "my-style.less" -->

#<center>git 使用方式</center>

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

**关联远程仓库**

```shell
git remote add <RemoteStorageLocalName> 地址
```

**获取远程仓库上主分支的内容**

```shell
git pull <RemoteStorageLocalName> master
```

**将当前分支设置为远程仓库的 master 分支**

```shell
git branch --set-upstream-to=<RemoteStorageLocalName>/master master
```

**推送到远程仓库**

```shell
#主支
git push origin master
#分支
git push origin 分支名
```

## 子模块

- 在项目的根目录添加子模块

```shell
cd project_main_directory
git submodule add git://***.git  submodule_directory
git submodule init
git submodule update
```

- 从服务器上当有子模块的代码

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
### 创建分支
```shell
git branch <NewBranchName>
# or
git switch <NewBranchName>
# or
git switch -c <NewBranchName>
```
### 删除分支
```shell
git branch -d <BranchName>
git push <RemoteStorageLocalName> --delete <BranchName>
```
### 切换分支
```shell
# 拉取远程仓库分支
git checkout --track remotes/<RemoteStorageLocalName>/<BranchName>
# or
git switch -t remotes/<RemoteStorageLocalName>/<BranchName>
# 切换本地分支
git checkout <BranchName> 
# or 
git switch <BranchName>
```

## 版本回退
- 私有库版本回退
```shell
# 先用下面命令找到要回退的版本的commit id：
git reflog
# 接着回退版本:
git reset --hard <CommitId>
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