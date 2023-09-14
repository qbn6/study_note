

[TOC]



# Git

## 1. 下载安装Git

官网地址： https://git-scm.com/ 

## 2 .认识Git

git分支迭代版本号是由40位组成，前2位是文件夹，后38位是文件名。

查看文件：

git cat-file -p 331e7256bfc6ce6297ef518b8666b4b7ca91ab9f  

![image-20230913113951566](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230913113951566.png)

继续查看tree 的文件目录

![image-20230913114746404](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230913114746404.png)

这里的parent指的是上一次提交的版本号

![image-20230913114902306](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230913114902306.png)

这里的100644就是指普通文件。

## 3 .分支操作

不同的分支指向不同的版本。不同的版本包含着操作差异产生的文件。就是每个版本的文件。

![image-20230914112457206](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230914112457206.png)



![image-20230914113406047](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230914113406047.png)



## 4. Git 指令

```
git --version 查看git版本
git init 初始化文件夹为git仓库
git clone git@gitee.com:qbn66/note.git 克隆仓库
```

