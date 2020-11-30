### Git 教程

+ #### Git是一个分布式的版本控制系统（可以自动记录文件的改动情况）


### 本地仓库

#### 创建版本库

> + ##### git init 将当前目录变成git可以管理的仓库
>
> + ##### git add fileName 将某个文件添加到仓库，用 . 代替fileName就是添加当前目录下所有文件
>
> + ##### git commit -m "message" 将已添加的文件提交到仓库，并为本次提交添加message说明

#### 时光穿梭

##### 状态查看

+ ###### git status 可以查看当前仓库的状态

+ ###### git diff fileName 查看该文件的改动情况

##### 版本回退

+ ##### git log 可以查看提交日志，git log --pretty=oneline查看简洁版。其中一长串字符串是版本号，HEAD表示当前版本，HEAD^表示上一版本，HEAD~100表示上100个版本

+ ##### git reset --hard 版本号，可以跳跃到该版本（写前几位就行），既可以跳到过去，也可以跳向未来（回退之后后悔）。

+ ##### git reflog 可以查看历史的每一次命令和版本号，以便确认跳到哪个版本

#### 工作区与暂存区

+ ###### 工作区就是写代码的文件

+ ###### 工作区中间有一个.git文件，该文件就是版本库

+ ###### 版本库中有一个暂存区stage，和一个master主分支。

+ ###### 当使用git add时，将文件添加到暂存区；当使用git commit时，将暂存区中的文件提交到master分支。

#### 撤销修改

+ ###### git checkout -- file 将file文件在工作区的修改全部撤销；

+ ###### 当file自修改后还没被提交到暂存区，工作区会撤销到和版本库一样的状态；

+ ###### 当file被提交到暂存区，撤销修改到添加暂存区后的状态。

+ ###### 当提交到缓存区时，使用git reset HEAD file 将提交到暂存区的内容撤销，然后使用git checkout -- file撤销工作区。

#### 删除文件

+ ###### 工作区删除file文件后，使用git rm file删除暂存区中file文件；

+ ###### 如果工作区误删文件，可以使用git checkout -- file还原。

### 远程仓库（github或者自己搭建服务器）

+ ##### 先有本地仓租，再有远程库的情况：

+ ##### 在GitHub新建git仓库，然后在本地工作区，运行 git remote add origin git@github.com:userName/pro.git添加远程库。其中userName是自己的用户名，pro是刚建的仓库；

+ ##### git push -u origin master 将本地仓退推送到远程仓库（将当前分支master推送到远程，并使用u参数将本地master与远程master关联起来）。从此以后本地修改，提交到远程仓库只需要使用 git push origin master。

+ ##### 先有远程仓租，再有本地库，从远程库克隆

+ ##### git clone git@github.com:userName/pro.git将远程仓库的代码clone下来。

+ ##### git 传输支持ssh方式（ git@github.com:userName/pro.git）和https方式（https://github.com/userName/pro.git)。其中ssh协议最快（但是要配置SSH KEY）

#### 分支管理（不同分支之间互不干扰）

##### 创建合并分支

+ ###### 当只有一条分支时，HEAD指向master，master指向提交；也就是说利用HEAD就能确定当前分支的提交点。每次提交时master分支都会向前移动一步；

  ![image-20201015142419632](images.assets\image-20201015142419632.png)

+ ###### 当新建一个分支dev时，会指向与master相同的提交，HEAD会指向dev，就表示当前分支在dev上面。

+ ###### 当dev提交时，dev指针向前移动一步，但master指针不变

  ![image-20201015142729847](images.assets\image-20201015142729847.png)

+ ###### 当dev分支上的工作完成了，就可以把dev合并到master分支上，可以简单的将master指针指向dev的提交，然后把HEAD指向master。

+ ###### git branch dev(新建dev分支)、git switch dev (切换到dev分支)、git branch(查看分支)、git merge dev(在主分支上合并dev分支)、git branch -d dev(删除dev分支)

##### 解决冲突

+ ###### 当新建的分支和主分支都各自做了修改，并提交的时候，合并分支时会出现冲突。

+ ###### 当出现冲突时，需要手动将合并失败的文件修改成我们希望的内容，再提交。

+ ###### 可以使用 git log --graph 查看分支合并图

##### 分支合并策略

+ ###### 普通的 git merge dev合并分支，使用Fast forward模式，直接将master指针指向dev提交的位置，删除dev时会丢掉dev的分支信息。

+ ###### 当使用 no-ff模式合并分支时，会创建一个新的commit，合并后如图。git merge --no-ff -m "message" dev

  ![image-20201015145151781](images.assets\image-20201015145151781.png)

##### Bug分支

+ ###### 当手头工作还没完成需要去完成其他工作（bug)时，就需要暂存当前的工作任务，去开发其他工作；

+ ###### 使用 git stash 将当前工作现场存储起来，再去开发其他任务(bug)；

+ ###### 其他任务完成后，再使用git stash list 查看暂存的任，并使用git stash apply或git stash pop恢复，前者恢复不删除stash(调用git stash drop手动删)，后者恢复后删除stash。

+ ###### git cherry-pick commit_id 可以将commit_id的提交复制到当前分支。

+ ###### git branch -D branchName,可以用来强行删除一个没有合并过的分支。

### 实操

```bash
git clone url  #克隆项目

git commit        #需要先commit一下才能新建branch
git branch test   #新建branch
git checkout test #切换分支
git add .         
git commit
git push origin test:test  #将本地的test分支提交到远程test分支上
```



### 遇到问题

#### git push origin master 报错

##### 错误

```bash
$ git push origin master
To github.com:xiaoyangLee/LearnJava.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@github.com:xiaoyangLee/LearnJava.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

##### 解决

主要是远程仓库的版本超前本地仓库的版本，需要pull下来

```bash
git pull --rebase origin master
```

