##  远程仓库管理

检出仓库：$ git clone git://github.com/jquery/jquery.git

查看远程仓库：$ git remote -v

添加远程仓库：$ git remote add `name` `url`

> git remote add origin git@git.jtjr.com:h5_credit_grp/credit_account_v2.git

删除远程仓库：$ git remote rm `name`

修改远程仓库：$ git remote set-url —push `name` `newUrl`

拉取远程仓库：$ git pull `remoteName` `localBranchName`

推送远程仓库：$ git push `remoteName` `localBranchName`

## 分支管理

#### 创建分支、切换分支

创建`dev`分支，然后切换到`dev`分支：

```
$ git checkout -b dev
Switched to a new branch 'dev'
```

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：

```
$ git branch dev
$ git checkout dev
Switched to branch 'dev'
```

然后，用`git branch`命令查看当前分支：

```
$ git branch
* dev
  master
```

`git branch`命令会列出所有分支，当前分支前面会标一个`*`号。

`git branch -r`可以查看远程分支

#### 合并分支

把`dev`分支的工作成果合并到`master`分支上

```
$ git checkout master
Switched to branch 'master'
$ git merge dev
Updating d17efd8..fec145a
Fast-forward
 readme.txt |    1 +
 1 file changed, 1 insertion(+)
```

#### 删除分支

```
$ git branch -d dev
Deleted branch dev (was fec145a).
```

#### 创建远程分支(本地分支**push**到远程)

```
$ git push origin [name]
```

#### 删除远程分支

```
$ git push origin :heads/[name]
```

我从master分支创建了一个issue5560分支，做了一些修改后，使用**git push** origin master提交，但是显示的结果却是'Everything up-to-date'，发生问题的原因是**git push** origin master 在没有track远程分支的本地分支中默认提交的master分支，因为master分支默认指向了origin master 分支，这里要使用**git push**origin issue5560：master 就可以把issue5560推送到远程的master分支了。

    如果想把本地的某个分支test提交到远程仓库，并作为远程仓库的master分支，或者作为另外一个名叫test的分支，那么可以这么做。

$ **git push** origin test:master         // 提交本地test分支作为远程的master分支 //好像只写这一句，远程的github就会自动创建一个test分支
$ **git push** origin test:test              // 提交本地test分支作为远程的test分支

如果想删除远程的分支呢？类似于上面，如果:左边的分支为空，那么将删除:右边的远程的分支。

$ **git push** origin :test              // 刚提交到远程的test将被删除，但是本地还会保存的，不用担心

------



#### 追踪分支

在Git中‘追踪分支’是用与联系本地分支和远程分支的. 如果你在’追踪分支'(Tracking Branches)上执行推送(push)或拉取(pull)时,　它会自动推送(push)或拉取(pull)到关联的远程分支上.

如果你经常要从远程仓库里拉取(pull)分支到本地,并且不想很麻烦的使用"git pull "这种格式; 那么就应当使用‘追踪分支'(Tracking Branches).

‘git clone‘命令会自动在本地建立一个'master'分支，它是'origin/master'的‘追踪分支’. 而'origin/master'就是被克隆(clone)仓库的'master'分支.

译者注: origin一般是指原始仓库地址的别名.

你可以在使用'git branch'命令时加上'--track'参数, 来手动创建一个'追踪分支'.

```
git checkout -b dev-track --track origin/master
```

当你运行下命令时:

```
$ git pull dev-track
```

它会自动从‘origin'抓取(fetch)内容，再把远程的'origin/experimental'分支合并进(merge)本地的'experimental'分支.

当要把修改推送(push)到origin时, 它会将你本地的'experimental'分支中的修改推送到origin的‘experimental'分支里,　而无需指定它(origin).



可以通过`git branch -vv`来查看分支是否为追踪分支

```
$ git branch -vv
  dev 2976dcf first commit
* dev-track    2976dcf [origin/master] first commit
  master       2976dcf [origin/master] first commit
```



------

[使用中的一些奇技淫巧](https://mp.weixin.qq.com/s/x-hCxYQVYFfHoekXCDo1Dw)