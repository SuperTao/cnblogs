工作中需要在github上保存项目，一个仓库中有多个分支，进行一些实验，方便后面操作。  

#### 参考链接

http://rogerdudler.github.io/git-guide/index.zh.html  

http://www.ruanyifeng.com/blog/2014/06/git_remote.html

#### 1、创建远程仓库

我在github账户上创建了一个git_test的仓库。

创建完成仓库之后，github会显示一些git的基本操作。

```
... or create a new repository on the command line

echo "# git_test" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:TonySudo/git_test.git
git push -u origin master

... or push an existing repository from the command line

git remote add origin git@github.com:TonySudo/git_test.git
git push -u origin master
```

仓库建好了，如何上传文件呢？我知道的有两种方法：

* 将github上的仓库clone到本地，添加文件之后再push。

* 在本地创建仓库，连接到远程的仓库,然后再push文件。 

下面分别介绍。

#### 2、克隆远程仓库到本地

```
# clone的时候，远程主机自动被命名为origin。
$ git clone git@github.com:TonySudo/git_test.git
Cloning into 'git_test'...
warning: You appear to have cloned an empty repository.
Checking connectivity... done.

# 克隆某个分支, 使用-b选项，进行分支的选择，branch是远端仓库分支的名字。
$ git clone -b branch https://github.com/TonySudo/git_test.git
Cloning into 'git_test'...
Unpacking objects: 100% (6/6), done.packing objects:  16% (1/6)

remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0
Checking connectivity... done.

# 上传文件

$ cd git_test/

# 创建文件
$ touch master

$ git add master

$ git commit -m "init push"
[master (root-commit) 436b48f] init push
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 master
 
# 上传到github 

$ git push
Counting objects: 3, done.
Writing objects: 100% (3/3), 203 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:TonySudo/git_test.git
 * [new branch]      master -> master
 
上传完成，在github的仓库中就会看到一个master文件。 
```

#### 3、本地仓库连接远程仓库

```
# 在本地创建一个目录用于当做仓库。

$ mkdir git

$ cd git

# 初始化本地仓库

$ git init
Initialized empty Git repository in C:/Users/Tony/Desktop/git/git/.git/

# 连接远程仓库

$ git remote add origin git@github.com:TonySudo/git_test.git

# origin 是定义的远程主机的名字， origin 是远程仓库的网址

如果git remote连接时出现错误

fatal: remote origin already exists.

解决方法参考：

http://blog.csdn.net/dengjianqiang2011/article/details/9260435

$ git remote rm origin 

再次运行之前的命令就可以成功。

# 查看远程仓库的名字：

$ git remote -v
origin  git@github.com:TonySudo/git_exe.git (fetch)
origin  git@github.com:TonySudo/git_exe.git (push)

# 上传文件

$ touch master

$ git add master

$ git commit -m "init push"
[master (root-commit) f88dd2b] init push
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 master

$ git push -u origin master

如果出现错误。
fatal: The current branch master has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin master

$ git push --set-upstream origin master
```

#### 4. 提取服务器上的更新

```
# 从服务器上下载更新，这只是下载下来，没有对源码进行更改。
#  默认取回所有的更新。
$ git fetch origin

# 取回某一个分支的更新,branch1是分支名,可以是master或者其他的。
$ git fetch origin branch1

# 将fetch下来的跟新和本地的分支进行合并。merge之后，本地的源码才会改变。
$ git merge

# git pull相当于执行了git fetch和git merge
# 将远程的branch1分支的内容下载到本地的master分支。
# 也可以将branch1更改为其他的，例如master. 
$ git pull origin branch1:master
```

#### 5、文件上传

```
# 将本地的分支branch上的更新上传到远端的master分支。
$ git push origin branch:master
Counting objects: 3, done.
Writing objects: 100% (3/3), 200 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:TonySudo/git_test.git
 * [new branch]      master -> master
有时候会出错，说明本地上的文件跟服务器上的不同步 
$ git push origin branch:master
To git@github.com:TonySudo/git_test.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:TonySudo/git_test.git'
hint: Updates were rejected because the tip of your current branch is behi
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

解决方法，先将服务器上的更新先下载到本地，这样就和服务器上的同步了，再进行提交即可。

将本地的branch分支的内容传送大远端仓库的branch分支
$ git push origin branch:branch
Counting objects: 2, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 255 bytes | 0 bytes/s, done.
Total 2 (delta 0), reused 0 (delta 0)
To git@github.com:TonySudo/git_test.git
 * [new branch]      branch -> branch
``` 
 
#### 6. 分支

创建一个名为branch1的分支

`git branch branch1`

切换到分支branch1

`git checkout branch1`

将本地master分支的文件上传到远端仓库的名为branch1的分支上。 如果远端这个分支不存在，就会创建这个分支。

`git push origin master:branch1`

Tony Liu

2017-2-8, Shenzhen