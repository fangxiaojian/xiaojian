# git 操作指令

## git 上传至远程仓库

添加需要提交的文件

>  git add . 

（注：别忘记后面的.，此操作是把 文件夹下面的文件都添加进来）

提交到本地仓库

> git commit  -m  "提交信息"

（注：“提交信息”里面换成你需要，如“first commit”）

关联上远程仓库

> git remote add origin https://github.com/githubName/RepositoriesName

将你的代码上传到Github

> git push -u origin master   

（注：此操作目的是把本地仓库push到github上面，此步骤需要你输入帐号和密码）



## git 切换分支

> git checkout [分支名称]              // 如 git checkout jdk8u/jdk8u

若是报错：

> error: you need to resolve your current index first
> .gitignore: needs merge
> .hgignore: needs merge
> .jcheck/conf: needs merge
> Makefile: needs merge
> README: needs merge
> README-builds.html: needs merge
> THIRD_PARTY_README: needs merge
> configure: needs merge
> make/Main.gmk: needs merge
> make/common/JavaCompilation.gmk: needs merge
> make/common/MakeBase.gmk: needs merge
> make/common/NativeCompilation.gmk: needs merge
> make/common/support/ListPathsSafely-pre-compress.incl: needs merge
> make/common/support/ListPathsSafely-uncompress.sed: needs merge
> make/devkit/Makefile: needs merge
> make/devkit/Tools.gmk: needs merge
> make/jprt.properties: needs merge
> make/scripts/normalizer.pl: needs merge
> make/scripts/update_copyright_year.sh: needs merge

说明是文件冲突了

可以提交代码，或者重置之前代码以解决冲击

> git reset --hard

查看全部分支

> git branch -a

## git 版本回退

查看历史提交信息 

> git log

撤销单个文件的修改

> git reset HEAD [文件地址+文件名]   // 如：src/com/jay/example/testforgit/MainActivity.java
> git checkout [文件地址+文件名]   // 如：src/com/jay/example/testforgit/MainActivity.java



用**HEAD**代表当前版本，上一个版本就是**HEAD^**，再上一个版本就是**HEAD^^**依次类推 

> git reset --hard HEAD
> git reset --hard HEAD^
> git log

也可以 加版本号

> git reset --hard ad2080c

回退后，你突然后悔了，想回退回新的那个版本 

> git reflog
>
> git reset --hard ad2080c