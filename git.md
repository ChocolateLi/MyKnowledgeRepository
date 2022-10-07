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

git remote set-url origin https://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@github.com/ChocolateLi/Pytorch.git/
```































