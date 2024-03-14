# git

## centos安装配置git

1.安装

```shell
 yum install git
 或：sudo yum install git
```

2.验证

```shell
git --version
```

3.配置基本信息

```shell
//配置基本信息
[root@localhost ~]# git config --global user.name "chenli"
[root@localhost ~]# git config --global user.email 954357018@qq.com
//查看配置
[root@localhost ~]# git config --list
user.name=flymegoc
user.email=343672271@qq.com
[root@localhost ~]#
```



## git常用命令原理解释

[我用四个命令，总结了 Git 的所有套路](https://mp.weixin.qq.com/s/VdeQpFCL3GGsfOKrIRW6Hw)



## git命令练习网站

[learning git](https://learngitbranching.js.org/?locale=zh_CN)



## git基本命令

### 1.拉取github项目

```shell
git clone https://github.com/ChocolateLi/travel_spider.git
```

### 2.代码提交命令

```shell
git add .
git commit -m "一些描述"
git push 
```

### 3.git merge命令

[合并分支----git merge命令应用的三种情景](https://blog.csdn.net/qq_42780289/article/details/97945300)

[git分支-新建与合并](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)



**整个开发流程：**

1.先创建和切换要开发的分支

```git
git checkout -b dev_15534
```

2.在新的分支上开发完后，进行提交

```git
git status
git add .
git commit -m '提交内容'
```

3.切换回主分支，记得拉取一下最新的主分区代码

```git
git checkout main
git pull
```

4.将新分支提交的内容合并到主分支

```git 
git merger dev_15534
```

5.将主分区的内容提交到远程仓库

```git
git push
```

6.删除分支

```git
git branch -d dev_15534
```



**演示流程**

![](F:\github\MyKnowledgeRepository\img\git_img\git merge.png)

### 4.git branch 命令

创建新分支

```git
git branch 新分支名称
```

切换分支

```git
git checkout 新分支名称

//创建和切换一起操作
git checkout -b 新分支
```

查看本地分支

```git
git branch
```

查看远程分支

```git
git branch -r
```

查看所有分支（本地和远程）

```git
git branch -a
```

删除分支

```git
git branch -d 分支名
```



### 5.git reset 命令

在Git中，`git reset`命令主要用于撤销之前的一些操作，比如提交。它可以根据不同的参数选项，对工作区和暂存区进行操作，从而达到不同的效果。理解`git reset`命令之前，先简单了解一下Git的三个主要区域：

1. **工作区（Working Directory）**：就是你在电脑里能看到的文件。
2. **暂存区（Staging Area/Index）**：是Git用来准备下一次提交的文件改动的地方。你可以将一些更改添加到暂存区中，这样下次提交时，Git会将暂存区里的改动作为提交内容。（git add 命令的东西到暂存区）
3. **仓库（Repository）**：是Git用来保存项目的元数据和对象数据库的地方。提交（Commit）会将暂存区的内容永久保存到仓库的历史记录中。

`git reset`命令主要有三个选项，分别影响这些区域：

1. `git reset --soft <commit>`

- **作用**：将当前分支的HEAD指向另一个commit，但不改变暂存区和工作区。
- **适用场景**：当你想要撤销刚才的提交，但不想丢失工作进度时，可以使用这个选项。它只会改变Git历史记录，而不触及你的文件。
- **示例**：如果你想要撤销最近的一次提交，并保留更改在暂存区中，可以使用：
  ```bash
  git reset --soft HEAD~1
  ```

2. `git reset --mixed <commit>`（默认选项）

- **作用**：将HEAD指向另一个commit，并更新暂存区，但不改变工作区。
- **适用场景**：当你想要撤销最近的提交，并且想要重新审视和调整即将提交的文件时。这个命令会将改动回滚到暂存区，让你有机会重新`git add`文件。
- **示例**：如果你想要撤销最近的一次提交，并将更改回滚到暂存区，可以使用：
  ```bash
  git reset --mixed HEAD~1
  ```
  或者简写为（因为`--mixed`是默认选项）：
  ```bash
  git reset HEAD~1
  ```

3. `git reset --hard <commit>`

- **作用**：将HEAD指向另一个commit，并更新暂存区和工作区，使之与HEAD指向的commit一致。
- **适用场景**：当你想彻底撤销最近的提交和所有更改时，包括没有提交的工作进度。
- **示例**：如果你想要撤销最近的一次提交，并且不保留任何更改，可以使用：
  ```bash
  git reset --hard HEAD~1
  ```

使用`git reset`时，需要谨慎选择合适的选项，因为`--hard`选项会丢失所有未提交的更改。在执行操作前，确保不会意外丢失重要数据。



总结：

1.撤销git commit，并撤销git add .操作，但不撤销本地修改的代码。

```git
git reset --mixed HEAD^
```

2.撤销 git commit，但不撤销git add .操作。

```git
git reset --soft HEAD^
```

3.撤销commit 、撤销 git add. 操作，同时撤销本地修改的代码。（慎重使用）

```
git reset --hard HEAD^
```





## git问题解决方案

### 1.Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.

背景：git push origin main。然后输入用户名和用户密码，发生的错误

说明：发生这个错误的意思是原先密码不能用了，现在必须要使用个人访问令牌(personal access token)

解决方案：

1.生成自己的token

在个人主页找到settings

![](D:\github\MyKnowledgeRepository\img\git_img\token1.png)



选择开发者设置`Developer setting`

![](D:\github\MyKnowledgeRepository\img\git_img\token2.png)

选择个人访问令牌`Personal access tokens`，然后选中生成令牌`Generate new token`

![](D:\github\MyKnowledgeRepository\img\git_img\token3.png)

设置永远不会过期，勾选repo，点击generate

![](D:\github\MyKnowledgeRepository\img\git_img\token4.png)



记得把你的token保存下来，因为你再次刷新网页的时候，你已经没有办法看到它了，所以我还没有彻底搞清楚这个`token`的使用，后续还会继续探索！



2.把上面的生成的token添加到远程仓库链接中，这样就可以避免同一个仓库每次提交代码都要输入`token`了

```
git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git

<your_token>：换成你自己得到的token
<USERNAME>：是你自己github的用户名
<REPO>：是你的仓库名称

例如：
git remote set-url origin https://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@github.com/shliang0603/Yolov4_DeepSocial.git/


git remote set-url origin https://***********@github.com/ChocolateLi/MyKnowledgeRepository.git/

git remote set-url origin https://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@github.com/ChocolateLi/Pytorch.git

```

### 2.git pull发生冲突

git pull 命令发生以下错误

![](https://img2018.cnblogs.com/blog/773489/201905/773489-20190508113231265-2020653666.png)

目前git的报错提示已经相关友好了，可以直观的发现，这里可以通过commit的方式解决这个冲突问题

如果不想commit它，那就通过以下方案解决：

放弃本地修改，直接覆盖

```shell
git reset --hard
git pull
```





























