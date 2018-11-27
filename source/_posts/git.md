---
title: 'git常用'
date: "2018/05/11"
tags: [git]
categories: [开发常用工具]
copyright: true
---
场景1：关联远程并提交
`git init` 初始化本地
`git remote add origin url` 本地仓库关联远程仓库
`git checkout -b master`  创建分支master并切换到master分支
`git add`  添加
`git commit` 添加
`git push -u origin master`  推送本地到远程
场景2：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。
场景3：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，可以分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。
场景4：本地在cl分支开发，现在要合并master的内容使cl分支有最新的master代码。
      `git checkout master`切换分支
      `git pull`拉取并合并master分支最新代码 
      `git checkout cl`切换分支 
      `git merge master`更新最新master代码到cl分支
      
出错1：`fatal: 拒绝合并无关的历史`
解决：首先将远程仓库和本地仓库关联起来：
`git branch --set-upstream-to=origin/master master`
然后使用git pull整合远程仓库和本地仓库，
`git pull --allow-unrelated-histories    (忽略版本不同造成的影响)`

出错2：
```
To XXX
! [rejected] master -> master (non-fast-forward)
error: 无法推送一些引用到 'git@XXX.git'
提示：更新被拒绝，因为您当前分支的最新提交落后于其对应的远程分支。
```
解决：本地当前的分支和远程的对应分支不对应。远程分支文件多于本地文件，无法合并。需要手动拉取并合并，之后就可以正常push了。
`git fetch origin`  
`git merge origin/master（换成你要推送的分支）` 


1、修改后放在了修改区；add之后是暂存区；commit是本地仓库。
暂存区是存放下次将要提交的图表信息，索引。本地仓库存放原数据，对象数据库。
2、
`git branch` **查看所有分支**
`git branch` 分支名 **创建分支**
`git checkout` 分支名 **切换分**支
`git checkout -b newbranch` **创建并切换分支**
`git branch -d` 删除分支
`git branch -v` 查看分支详细信息（分支名及最后一个提交等）
`git checkout -- 文件名` 将本地文件返回到本地仓库的状态（一般在你push时，又不想提交本地某些文件修改的时候使用，移出来后与远程merge后本地被移出来的文件修改部分会被远程覆盖！    ）
`git checkout 版本号 文件名` 返回到文件的某版本（git log 查看所有版本）
`git checkout HEAD～num ` 返回到前num次提交
`git remote -v` 查看远程仓库地址命令

`git merge --abort `**取消本次合并**
`git fetch` **拉取远程分支**
`git merge origin master` **合并分支**
`git pull=git fetch+git merge` **拉取并合并**
<<<< 自己的 ==== 别人的 >>>>

git status 
git add  **暂存区**
git commit -m '描述' **本地仓库**
git push -u origin branch名字 **推送到远程**（第一次推送使用 -u，意思是推送远程并关联远程分支）

git reset HEAD 文件 将文件从暂存区撤回
git reset --mixed HEAD~ 将本地仓库的内容复制到暂存区
git reset --hard HEAD~ 将本地仓库的内容复制到暂存区和本地工作区（覆盖不可逆，要小心）
git reset --soft HEAD~ 将本地仓库的head指针指向前一次提交

git revert 版本号 撤销某版本的提交（与reset类似，但是reset会将某版本之后的提交全部抹去）

git cherry-pick 版本号 将某次提交内容合并到当前分支（只想合并某次提交的内容，其他次提交的内容不提交） 
git cherry-pick --continue 继续提交

git rebase 把一个分支的修改合并到当前分支

git diff 工作区、暂存区之间的差别
git diff --cached/staged 暂存区、本地仓库差别
git diff 两个分支/两次提交码 比较他们的区别

ssh-keygen 生成key
cat ~/.ssh/id_rsa.publs

vim .gitignore修改不想添加到git中的文件status不在提示

git init创建新的/重新初始化
git clone
git remote add origin url在本地关联远程仓库
git remote -v 等
./git就是本地仓库
git log 提交日志，本地远程都有
git log --graph --oneline 
git reflog 所有日志，只在本地

git config 添加用户等
git config --add user.name/user.email
git config -e 看用户