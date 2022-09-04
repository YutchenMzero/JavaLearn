## Git
[在线演示学习工具](https://oschina.gitee.io/learn-git-branching/ )
### 基本知识
#### 对待数据的方式
 Git采用的是直接记录快照的方式，而非差异比较。Git 更像是把数据看作是对小型文件系统的一组快照。 每次你提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个快照流。
#### Git 的三种状态
1. 已提交（committed）：数据已经安全的保存在本地数据库中。
2. 已修改（modified）：已修改表示修改了文件，但还没保存到数据库中。
3. 已暂存（staged）：表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。
其基本的工作流程为：
1. 在**工作目录**中修改文件。
2. 暂存文件，将文件的快照放入**暂存区域**。
3. 提交更新，找到暂存区域的文件，将快照永久性存储到**Git仓库**目录。
#### HEAD
HEAD 是一个对当前检出记录的符号引用,也就是指向你正在其基础上进行工作的提交记录。它总是指向当前分支上最近一次提交记录且通常情况下是指向分支名的。大多数修改提交树的 Git 命令都是从改变 HEAD 的指向开始的。
1. 分离的HEAD：即让其指向了某个具体的提交记录而不是分支名。可用checkout命令指定提交记录的哈希值实现
2. 相对引用：通过`^`（向上移动一个提交记录）和 `~<num>`（向上移动多个提交记录）快速定义到所需引用的位置。可以将分支名或者HEAD作为相对引用的参考位置。

### 常用命令与操作
#### 获取git仓库
1. 在现有目录中初始化仓库: 进入项目目录运行 `git init `命令,该命令将创建一个名为 `.git `的子目录
2. 从一个服务器克隆一个现有的 Git 仓库(并自定义名字): `git clone <url> [name]`
#### 更新记录
1. 检测当前文件状态：`git status`
2. 提出更改（把它们添加到暂存区）：`git add <filename> `(针对特定文件)、`git add *`(所有文件)、`git add *.txt`（支持通配符，所有 .txt 文件）
3. 提交更新: `git commit -m "代码提交信息"` （每次准备提交前，先用 `git status `看下，是不是都已暂存起来了， 然后再运行提交命令）
4. 跳过使用暂存区域更新的方式 : `git commit -a -m "代码提交信息"`。 `git commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过` git add `步骤。
5. 移除文件 ：`git rm <filename>` （从暂存区域移除，然后提交。）
6. 对文件重命名 ：`git mv <name1> <name2>`
7. 查看提交历史`git log`
#### 远程仓库
1. 将仓库连接到远程服务器`git remote add origin <server>`
2. 将改动提交到远程仓库`git push origin <branch_name>`
3. 移除远程仓库 test1:`git remote rm test1`
#### 撤销操作
1. 撤销上一次提交，并重新提交`git commit --amend`
2. 取消暂存的文件`git reset <filename>`
3. 回退提交记录到指定位置`git reset <commit_location>`(reset命令对远程分支无效)
4. 撤销更改并分享（新建一个提交，该提交与要撤回到的提交状态相同）`git revert <commit_location>`
5. 撤消对文件的修改（暂存区恢复到工作区）:`git checkout -- <filename>`
#### 拉取到本地
1. 将远程的branch分支的最新版本取到本地的origin/branch分支里，不会合并到本地的branch分支`git fetch origin <branch>`(即拉取到本地仓库，但不到工作区)
2. 将远程的branch分支的最新版本取到本地的origin/branch分支里，同时合并到本地的branch分支`git pull origin <branch>`
#### 分支
分支是用来将特性开发绝缘开来的。在你创建仓库的时候，master 是“默认”的分支。在其他分支上进行开发，完成后再将它们合并到主分支上。我们通常在开发新功能、修复一个紧急 bug 等等时候会选择创建分支。单分支开发好还是多分支开发好，还是要看具体场景来说。创建一个名字叫做 test 的分支。
1. 创建分支`git branch <branch_name>`
2. 切换分支`git checkout <branch_name>`(当你切换分支的时候，Git 会重置你的工作目录，使其看起来像回到了你在那个分支上最后一次提交的样子。 Git 会自动添加、删除、修改文件以确保此时你的工作目录和这个分支最后一次提交时的样子一模一样)
3. 创建并切换分支`git checkout -b <branch_name  >`
4. 合并分支到当前分支`git merge <branch_name>`
5. 将并行分支合并为线性分支`git rebase <branch_name>`
6. 强制修改分支的位置`git branch -f <branch_name> <commit_location>`,是分支指向另一个提交。
#### HEAD操作
1. 查看HEAD指向`cat .git/HEAD`,如果 HEAD 指向的是一个引用，还可以用`git symbolic-ref HEAD`查看它的指向.
#### 整理提交记录
1. 将一些提交复制到HEAD下面`git cherry-pick <commit_location>`
2. 使用`git rebase <commit_location> -i`整理所设定提交位置之后的所有提交

#### github相关问题
1. 遭遇如`Failed to connect to github.com port 443`的问题，若使用vpn得话，可使用`git config --global http.proxy <ip:socket>`配置git的代理，`<ip:socket>`应与所用的代理一致
