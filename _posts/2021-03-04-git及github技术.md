---
title: Git及GitHub
layout: post
tags: 数据管理
categories: ''


---

## 20210304  .gitignore

###### 一、简介

我们做的每个Git项目中都需要一个“.gitignore”文件，这个文件的作用就是告诉Git哪些文件不需要添加到版本管理中。比如我们项目中的npm包(node_modules)，它在我们项目中是很重要的，但是它占的内存也是很大的，所以一般我们用Git管理的时候是不需要添加npm包的。

###### 二、常用的规则



```swift
/mtk/ 过滤整个文件夹
*.zip 过滤所有.zip文件
/mtk/do.c 过滤某个具体文件
```

以上规则意思是：被过滤掉的文件就不会出现在你的GitHub库中了，当然本地库中还有，只是push的时候不会上传。
 除了以上规则，它还可以指定要将哪些文件添加到版本管理中。



```swift
!src/   不过滤该文件夹
!*.zip   不过滤所有.zip文件
!/mtk/do.c 不过滤该文件
```

**1、配置语法：**
 以斜杠`/`开头表示目录；
 以星号`*`通配多个字符；
 以问号`?`通配单个字符
 以方括号`[]`包含单个字符的匹配列表；
 以叹号`!`表示不忽略(跟踪)匹配到的文件或目录；

此外，git 对于 .ignore 配置文件是按行从上到下进行规则匹配的，意味着如果前面的规则匹配的范围更大，则后面的规则将不会生效；

**2、示例说明**
 **a、规则：fd1/***
 说明：忽略目录 fd1 下的全部内容；注意，不管是根目录下的 /fd1/ 目录，还是某个子目录 /child/fd1/ 目录，都会被忽略；
 **b、规则：/fd1/***
 说明：忽略根目录下的 /fd1/ 目录的全部内容；
 **c、规则：**
 /*
 !.gitignore
 !/fw/bin/
 !/fw/sf/
 说明：忽略全部内容，但是不忽略 .gitignore 文件、根目录下的 /fw/bin/ 和 /fw/sf/ 目录；

###### 三、创建.gitignore文件

**1) 常规的windows操作**

- 根目录下创建gitignore.txt；
- 编辑gitignore.txt，写下你的规则，例如加上node_modules/；
- 打开命令行窗口，切换到根目录（可以直接在文件夹上面的地址栏输入cmd回车）；
- 执行命令ren gitignore.txt .gitignore。

**2) 用Git Bash**

- 根目录下右键选择“Git Bash Here”进入bash命令窗口；
- 输入`vim .gitignore`或`touch .gitignore`命令，打开文件（没有文件会自动创建）；
- 按i键切换到编辑状态，输入规则，例如node_modules/，然后按Esc键退出编辑，输入:wq保存退出。

如图：



```cpp
# dependencies  npm包文件
/node_modules

# production  打包文件
/build

# misc 
.DS_Store

npm-debug.log*
```

**.DS_Store：**这个文件是Mac OS X用来存储文件夹的一些诸如自定义图标，ICON位置尺寸，窗口位置，显示列表种类以及一些像窗体自定义背景样式，颜色这样的元信息。默认情况下，Mac OS X下的每个文件夹下应该都会生成一个，包括网络介质存储盘和U盘这样的外部设备。

![img](https:////upload-images.jianshu.io/upload_images/4434233-0157f5244c8cb047.png?imageMogr2/auto-orient/strip|imageView2/2/w/326/format/webp)

image.png


**npm-debug.log：**项目主目录下总是会出现这个文件，而且不止一个，原因是npm i 的时候，如果报错，就会增加一个此文件来显示报错信息，npm install的时候则不会出现。



最后需要强调的一点是，如果你不慎在创建.gitignore文件之前就push了项目，那么即使你在.gitignore文件中写入新的过滤规则，这些规则也不会起作用，Git仍然会对所有文件进行版本管理。
 简单来说，出现这种问题的原因就是Git已经开始管理这些文件了，所以你无法再通过过滤规则过滤它们。因此一定要养成在项目开始就创建.gitignore文件的习惯，否则一旦push，处理起来会非常麻烦。



作者：秉持本心
链接：https://www.jianshu.com/p/699ed86028c2
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

# git克隆私有仓库

参考链接：https://blog.csdn.net/weixin_33739541/article/details/93756300

## 1.1 Git 是什么 

　　Git 是分布式版本控制系统

　　工作原理/流程如下图所示

 ![img](D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\1473345-20190502191252991-683742966.png)

　　1. Workspace：工作区

　　2. Index / Stage：暂存区

　　3. Repository：仓库区（或本地仓库）

　　4. Remote：远程仓库

Git工作流程有：
(1) 在工作目录中添加、修改、删除文件；
(2) 将需要进行版本管理的文件放入**暂存区**；
(3) 将暂存区的文件提交到**Git仓库**中；

<img src="D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\image-20210401115535391.png" alt="image-20210401115535391" style="zoom:67%;" />

## 1.2 工作流程

**准备工作：** 账户初始化
	配置账户信息命令格式：

```
配置账户名：
git config --global user.name "GitHub用户名"

配置账户邮箱：
git config --global user.email "GitHub邮箱"
```

```
git config --list 查看设置信息
```

### 1.仓库搭建

#### 1.1本地仓库搭建

创建本地仓库的方法有两种：一种是创建全新的仓库，另一种是克隆远程仓库。

1、创建全新的仓库，需要用GIT管理的项目的根目录执行：

```
# 在当前目录新建一个Git代码库 
git init
```

2、执行后可以看到，仅仅在项目目录多出了一个.git目录，关于版本等的所有信息都在这个目录里面。

#### 1.2克隆远程仓库

1、另一种方式是克隆远程目录，由于是将远程服务器上的仓库完全镜像一份至本地！

```
# 克隆一个项目和它的整个代码历史(版本信息)$ git clone [url]  # https://gitee.com/kuangstudy/openclass.git
```

### 2.文件提交

**新建 hello_world.py 文件** ——> **写程序、保存** ——> **文件提交到暂存区** ——> **文件提交到Git仓库**

```
# 新建文件，并编写程序
touch hello_world.py
```

```
# 提交文件到暂存区
git add hello_world.py
```

```
# 提交到仓库
git commit -m "新增一个helloworld文件"
```

```
# 查看文件状态，每个阶段都可以使用
git status
```

## 1.3 版本前进与后退

首先查看版本提交记录：

```
git log
```

```
git reflog
```

![image-20210401142152277](D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\image-20210401142152277.png)

版本的前进和后退需要通过 指针 HEAD 来进行控制。

HEAD控制的方式有三种：
(1) 使用版本号进行操作(常用)：

```
命令格式为：
git reset --hard 版本号
```

(2) 使用^符号进行操作，只能后退：

```
命令： git reset --hard HEAD^ 表示后退一个版本
命令： git reset --hard HEAD^^ 表示后退两个版本
```

(3) 使用~符号进行操作，只能后退：

```
命令： git reset --hard HEAD~ 表示后退一个版本；
命令： git reset --hard HEAD~~ 表示后退两个版本；
命令： git reset --hard HEAD~5 表示后退5个版本；
命令： git reset --hard HEAD~n 表示后退n个版本；
```

注：命令参数有 soft, mixed, hard ** 三种类型：**
(1) 参数 soft 表示本地Git仓库移动HEAD指针。
(2) 参数 mixed 表示本地Git仓库移动HEAD指针，重置暂存区。
(3) 参数 hard 表示表示本地Git仓库移动HEAD指针，重置暂存区，重置工作区。

下面举例说明版本后退和前进。

使用版本号进行操作，其命令格式为：git reset --hard 版本号

后退到版本号为：629eeff的版本，命令为：

```
git reset --hard 629eeff
```

结果如下图所示：

![image-20210401142907731](C:\Users\zheng\AppData\Roaming\Typora\typora-user-images\image-20210401142907731.png)

当然也可以前进到刚才的版本：命令为：

```
git reset --hard dd7dc71
```

结果如下图所示：

<img src="D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\image-20210401143053706.png" alt="image-20210401143053706" style="zoom: 80%;" />

**TIP: 通过恢复到版本号位置，可以找回删除的文件。**

## 1.4 版本中的文件内容比较

对于版本之间同样文件内容比较可以使用命令 

```
git diff 
```

进行操作


比较文件的差异有多种情况：

第一种：**工作区和暂存区**文件比较，命令格式为：

```
 git diff 文件名
 # 举例如下
 git diff d123.txt 
```

第二种：**工作区和Git仓库文件**比较，命令格式为：

```
git diff HEAD 文件名
# 举例如下
git diff HEAD d123.txt
```

第三种：**工作区和Git仓库 的 不同版本文件**比较，命令格式为：

```
git diff 版本号 文件名
git diff 629eeff README.md
```

![image-20210401144646858](D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\image-20210401144646858.png)

注：不添加文件名进行比较时，将自动比较多个相同文件名文件。

## 1.5 删除文件

git删除文件用到的命令有以下几种情况：

命令：rm 文件名 表示删除的是工作区的文件。
命令：git rm 表示删除的是工作区和暂存区的文件。
命令：

```
# 表示当工作目录和暂存区的同一个文件存在不同内容时，执行命令 git rm -f 文件名 就可以强制删除工作区和暂存区的文件。
git rm -f 文件名 
```


命令：

```
# 表示只删除暂存区的文件并且保留工作目录的文件。
git rm --cached 文件名 
```

## 1.6 重命名文件

重命名文件有来两种情况：

1. 命令：`mv 旧文件名 新文件名` 表示将工作区文件重命名，暂存区和仓库文件名不变。
2. 命令(**常用**)：`git mv 旧文件名 新文件名` 表示将工作区和暂存区的文件重命名，仓库文件名不变。

## 1.7 创建分支与查看分支

分支管理中常用的命令如下：

**创建**分支命令格式：

```
git branch 分支名
```

**查看**分支命令格式：

```
git branch -v
# 对各个分支进行图形化显示
git log --decorate --all --oneline --graph
```

**切换**分支命令格式：

```
git checkout 分支名
```

**合并**分支命令格式：

```
git merge 分支名
```

  注： 需要注意当前所在分支

## 1.8 本地库管理远程库(GitHub)

本地库(**Git仓库**)可以管理远程库(**GitHub**)，一般地操作有**pull**, **clone** 和**pull** 操作，在实际协作开发项目是一般会有两种情况：

**第一种是团队之间互相认识，共同开发项目**。这样可以建立私人项目或者公开项目，然后邀请项目成员共同开发，这样其他人将看不到团队的项目。
例如：如下图所示，开发者A在GitHub上新建仓库，接着新建一个分支然后邀请团队成员开发者B在这个分支上提交代码。开发者B在本地库写好的项目代码推送到远程库的分支上，然后可以发送请求给开发者A，要求审核代码并合并分支，开发者A可以将代码拉取下来查看。如果觉得代码可以，那么直接合并代码即可，如果觉得代码不可以，要求开发者B进行修改知道合格为止。

**第二种是开源的项目，全网都可以看到**，任何开发者都可以对你的项目提出建议和修改，达到共同开发的目的。
例如：如下图所示，开发者A在GitHub上新建仓库，然后开发者C在GitHub上无意间看到了这个仓库，觉得挺好就Fork该仓库，于是开发者C就会出现该仓库。开发者C将这个仓库拉取(pull)本地库进行改进然后推送(push)到自己的远程库上，此时开发者A的仓库不会变化。
如果开发者C想要将自己的代码合并到开发者仓库中就要提出请求，然后开发者A进行审核，如果通过开发者A觉得可以就进行合并。
![image-20210401164650178](D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\image-20210401164650178.png)



本地库就是开发者将文件提交到Git仓库，此时Git仓库文件还在本地电脑。通过Git仓库可以管理GitHub远程仓库，例如Git仓库可以推送到GitHub仓库上，共享到社区，其他开发者可以看到这个仓库，然后可以提出Bug和改进建议达到共同开发的目的。

### (1) 建立本地库

开发者A新建develop_A文件夹，然后通过git命令初始化，接着新建`test_hello.cpp`文件提交到本地库。如下

```
git init
touch test_hello.cpp
git add test_hello.cpp
```

### (2) 建立远程库

开发者A在GitHub上新建一个项目(teamwork)， 如下图所示：

<img src="D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\image-20210401164934105.png" alt="image-20210401164934105" style="zoom:50%;" />

如下图所示，复制teamwork仓库地址：https://github.com/luohuayouyi666/teamwork.git

<img src="D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\image-20210401165009669.png" alt="image-20210401165009669" style="zoom:50%;" />

```
# 将远程仓库克隆到本地
git clone https://github.com/luohuayouyi666/teamwork.git
```



### (3) 本地库管理远程库的基本操作

2.1 本地库推送(push)到远程库(GitHub)

本地库推送到远程库命令格式为：

```
git push origin 分支名
```


现在开发者A将本地库推送到GitHub仓库上， 由前面的操作可知，teamwork仓库的地址为：
https://github.com/luohuayouyi666/teamwork.git

由于地址太长不太好操作，可以用一个**简单的名称(别名origin)**代替复较长的地址，
起别名的命令为格式为：

```
git remote add origin 远程库地址
```

例如：git remote add origin https://github.com/luohuayouyi666/teamwork.git 如下图所示，还可以通过命令：git remote -v 查看当前的别名 判断新名称(别名)是否更新成功。

![image-20210401165717359](D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\image-20210401165717359.png)

**注意：**
**更改远程地址别名**命令：

这里的origin是仓库地址的别名，可自行设定

```
git remote add origin GitHub仓库地址
```

**删除本地指定的远程地址别名**命令：

```
git remote remove origin
```

### (4)远程库克隆(clone)到本地库

远程库克隆(clone)到本地库命令格式为：

```
git clone 远程仓库地址
```

