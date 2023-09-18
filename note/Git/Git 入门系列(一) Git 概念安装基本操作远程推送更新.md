

[TOC]



# Git

## 1. 安装Git

官网下载地址： https://git-scm.com/ 

安装文档：https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git

安装完成后，右键git bash 输入 ==git --version== 

![image-20230918170822643](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230918170822643.png)

## 2. Git的基本使用

### 2.1  个人信息设置

```java
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
查看配置    git config -l
```



## 3. 了解.Git文件

git分支迭代版本号是由40位组成，前2位是文件夹，后38位是文件名。

查看文件：

git cat-file -p 331e7256bfc6ce6297ef518b8666b4b7ca91ab9f  

![image-20230913113951566](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230913113951566.png)

继续查看tree 的文件目录

![image-20230913114746404](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230913114746404.png)

这里的parent指的是上一次提交的版本号

![image-20230913114902306](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230913114902306.png)

这里的100644就是指普通文件。

## 4 .分支操作

不同的分支指向不同的版本。不同的版本包含着操作差异产生的文件。就是每个版本的文件。

![image-20230914112457206](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230914112457206.png)



![image-20230914113406047](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230914113406047.png)



## 5. Git 指令

```
git --version 查看git版本
git init 初始化文件夹为git仓库
git clone git@gitee.com:qbn66/note.git 克隆仓库
git remote add origin https://github.com/tingtingtina/gitStudy.git  添加远程仓库
git push -u origin master  推代码到远程仓库 origin是可以修改的
git remote -v 查看远程版本
git remote rename origin github  修改远程仓库名origin->github
git checkout -- ss.txt 注意一定要有“--”  舍弃文件改动
git reset HEAD <file> 将 stage 状态改为 unstage，也就是移出暂存区到工作区
git reset . 移除所有
####

git reset --soft HEAD^不删除工作区的改动，撤销commit，将内容存放在暂存区（add 之后），HEAD^ 或 HEAD~1 表示上一次 commit（HEAD~2就是上上次commit）， 也可以接commitid，如 git reset --soft 8f1fe9，撤销commit 到 「github commit」 那一次提交，之后修改的内容都在暂存区


```

