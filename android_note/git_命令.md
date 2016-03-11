# GIT 命令

### git init 创建 

### git add xxx 

### git commit -m "xxx"

### git status 查看状态

### git diff xxx 查看变化

### git log

### git log --pretty=oneline 最近一次

### $ git reset --hard HEAD^ 

	Head 为当前版本。Head^为上一版本，Head～100倒退一百个版本
	
### git reflog 重现输入命令

### git checkout 

    其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

### git回退

	场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

	场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。

	场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。
	
### git rm test.txt

### 添加远程库

	要关联一个远程库，使用命令git remote add origin git@server-name:path/repo-name.git；

	关联后，使用命令git push -u origin master第一次推送master分支的所有内容；

	此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改；

	分布式版本系统的最大好处之一是在本地工作完全不需要考虑远程库的存在，也就是有没有联网都可以正常工作，而SVN在没有联网的时候是拒绝干活的！当有网络的时候，再把本地提交推送一下就完成了同步，真是太方便了！
	
### git clone + link

### git 

	Git鼓励大量使用分支：

	查看分支：git branch

	创建分支：git branch <name>

	切换分支：git checkout <name>

	创建+切换分支：git checkout -b <name>

	合并某分支到当前分支：git merge <name>

	删除分支：git branch -d <name>

## git 

	当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

	用git log --graph命令可以看到分支合并图。
