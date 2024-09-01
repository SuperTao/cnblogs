使用git的时候，需要删除几个id，会对到之前的代码，但又想保留现在的代码，以便后面从新合并，所以就将现在的代码打包成patch，留到下次合并。

#### 参考链接

http://www.jianshu.com/p/e5d801b936b6

http://www.cnblogs.com/y041039/articles/2411600.html

#### 生成patch

可以用git diff命令或者git format-patch生成patch。

##### git format-patch
```
git format-patch <id>：从id这个版本到现在的patch。

如果有多个id的log信息，那么就会生成多个patch文件。

git patch命令生成的patch是根据时间节点来的，而且在打patch的时候，一个patch对应一个时间节点。

使用git log查看id信息。

git format-patch HEAD^ # 最近的1次commit的patch
git format-patch HEAD^^ # 最近的2次commit的patch
git format-patch HEAD^^^ # 最近的3次commit的patch
git format-patch HEAD^^^^ # 最近的4次commit的patch
```

##### git diff

```
git diff命令不管有几个时间节点，都只会生成一个patch文件。

git diff <id1> <id2> > file.patch

旧的id1在前。

```

#### 使用patch

先检查patch文件：`git apply --stat newpatch.patch`

检查能否应用成功：`git apply --check newpatch.patch`

使用git am会有patch里面的log信息，包括comment等。

打补丁

`git am --signoff < newpatch.patch` 或者 `git am --s < newpatch.patch`

`git apply newpatch.patch`

使用git apply 命令打入patch就不会记录commit的记录，而使用git　am的记录就会有记录。

#### patch失败

先检查patch文件：`git apply --stat newpatch.patch`

检查能否应用成功：`git apply --check newpatch.patch`

如果不成功，我的解决方法：

git am newpatch.patch

运行这条命令之后，是不会打patch，因为前面的`git apply --check`已经检查出不成功，git am也会进行检查，不成功就不会打patch。但是会进入git am的操作。

git apply --reject newpatch.patch 这条命令会将可以打的patch打上，然后不能打patch的文件在对应的目录生成.rej文件。对比rej文件，手动更改patch。

` git status`查看当前状态。

```
当前不在任何分支上。
您正处于 am 操作过程中。
  （解决冲突，然后运行 "git am --continue"）
  （使用 "git am --skip" 跳过此补丁）
  （使用 "git am --abort" 恢复原有分支）

尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git checkout -- <文件>..." 丢弃工作区的改动）

        修改：     CORE/MAC/src/pe/lim/limSendManagementFrames.c

未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）

        CORE/MAC/src/pe/lim/limSendManagementFrames.c.rej
```		

然后git add将更改的文件加入stage,并将.rej文件删除。

最后使用

`git am --continue`.这样patch就成功打入了，而且也会记录里面的commit信息。



Tony Liu

2017-12-20