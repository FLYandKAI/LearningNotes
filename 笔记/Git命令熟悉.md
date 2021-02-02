# Git命令熟悉

## git merge和git merge --no-ff的区别

![git merge常用命令区别](C:\Users\LangFo.zheng\Desktop\git merge常用命令区别.png)

- --no-ff指的是强行关闭fast-forward方式。
- fast-forward方式就是当条件允许的时候，git直接把HEAD指针指向合并分支的头，完成合并。属于 “快进方式”，不过这种情况如果删除分支，则会丢失分支信息。因为在这个过程中没有创建commit 
-  git merge --squash 是用来把一些不必要commit进行压缩，比如说，你的分支在开发的时候写的 commit很乱，那么我们合并的时候不希望把这些历史commit带过来，于是使用--squash进行合并， 此时文件已经同合并后一样了，但不移动HEAD，不提交。需要进行一次额外的commit来“总结”一 下，然后完成最终的合并。

–no-ff

关闭fast-forward模式，在提交的时候，会创建一个merge的commit信息，然后合并的和master分支
 merge的不同行为，向后看，其实最终都会将代码合并到master分支，而区别仅仅只是分支上的简洁清晰的问题，然后，向前看，也就是我们使用`reset` 的时候，就会发现，不同的行为就带来了不同的影响

![git merge --no-fff](C:\Users\LangFo.zheng\Desktop\git merge --no-fff.png)

上图是使用 `merge --no-ff`的时候的效果，此时`git reset HEAD^ --hard` 的时候，整个分支会回退到 **dev2-commit-2**

![git merge -ff](C:\Users\LangFo.zheng\Desktop\git merge -ff.png)

上图是使用 **fast-forward** 模式的时候，即 `git merge` ，这时候 `git reset HEAD^ --hard`，整个分支会回退到 **dev1-commit-3**

## git stash 和unstash的使用

场景如下，你正在开发需求1时，突然线上发现了一个bug，需要立即修复。需求1的代码因为不完善，也没经过测试，所以你希望针对需求1所做的修改先暂时隐藏，这样就可以使用 stash功能了。

VCS-->git -->stash

这个时候针对需求1做的修改都会隐藏掉。现在假设你处理bug完毕。需要继续开发需求，现在需要unstash

VCS-->git-->Unstash,选中你刚刚的stash，选中Pop stash。点击pop stash即可。

## git commit --amend 用法详解

比如，leader不abandon代码，你回去之后，修改出问题的Java文件，修改好之后，git add 该出问题.java 

然后 git commit –amend –-no-edit, 

最后 git push origin HEAD:refs/for/branches。

当我们想要对上一次的提交进行修改时，我们可以使用git commit –amend命令。git commit –amend既可以对上次提交的内容进行修改，也可以修改提交说明。

**举个例子：**

Step1：我们先在工作区中创建两个文件a.txt和b.txt。并且add到暂存区，然后执行提交操作：

Step2：此时我们查看一下我们的提交日志：

可以看到我们的提交日志中显示最新提交有两个文件被改变。

Step3：此时我们发觉我们忘了创建文件c.txt，而我们认为c.txt应该和a.txt,b.txt一同提交，而且a.txt文件中应该有内容‘a’。于是我们在工作区中创建c.txt，并add到暂存区。并且修改a.txt（故意写错语法且没有将a.txt的修改add到暂存区）：

Step4：我们查看一下此时的提交日志，可以看到上次的提交0c35a不见了，并且新的提交11225好就是上次提交的修补提交，它就像是在上次提交被无视了，修改后重新进行提交了一样：

Step5：此时我们发现a.txt文件修改没有成功，于是我们还得进行一次对a.txt的修改，将a.txt add到stage，然后再执行一次与上一次类似的提交修补：

OK了，git commit –amend的用法大致就是这样。

**总结：git commit --amend 相当于上次提交错误的信息被覆盖了，gitk图形化界面上看不到上次提交的信息，git log上也看不到之前的信息，而add 后再commit 相当于重新加了一个信息。**



## 拉取服务端代码

//从远程的origin仓库的master分支下载到本地并新建一个分支temp

$  git fetch origin master:temp 

$ git diff temp //比较差异
$ git merge temp //合并
$ git branch -d temp // 删除临时分支

 

Push gerrit服务器失败，报unpacked错误时，

error: remote unpack failed: error Missing tree f96fac296102de08f1669b4febdc179a74a03e89

可尝试通过 git push --no-thin origin HEAD:refs/for/master 提交

--no-thin这个参数的含义是“在向服务器提交代码时不对信息进行压缩处理”