---
title: 'git常用'
date: "2018/05/11"
tags: [git]
categories: [开发常用工具]
copyright: true
---
1、修改后放在了修改区；add之后是暂存区；commit是本地仓库。
暂存区是存放下次将要提交的图表信息，索引。本地仓库存放原数据，对象数据库。
2、
git branch **查看所有分支**
git branch 分支名 **创建分支**
git checkout 分支名 **切换分**支
git checkout -b newbranch **创建并切换分支**
git branch -d 删除分支
git branch -v 查看分支详细信息（分支名及最后一个提交等）
git checkout -- 文件名 将本地文件返回到本地仓库的状态（一般在你push时，又不想提交本地某些文件修改的时候使用，移出来后与远程merge后本地被移出来的文件修改部分会被远程覆盖！    ）
git checkout 版本号 文件名 返回到文件的某版本（git log 查看所有版本）
git checkout HEAD～num 返回到前num次提交

git merge --abort **取消本次合并**
git fetch **拉取远程分支**
git merge origin master **合并分支**
git pull=git fetch+git merge **拉取并合并**
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