[TOC]

# git 创建本地分支，推送到远程
```
删除本地分支(必须保证不在删除的分支上，才能进行删除)
git branch -d dev 
切换到本地分支
git checkout 分支名 
例如：git checkout dev，这条命令表示从当前master分支切换到dev分支。
创建本地分支并切换
git checkout -b 分支名 
例如：git checkout -b dev，这条命令把创建本地分支和切换到该分支的功能结合起来了，即基于当前分支master创建本地分支dev并切换到该分支下。
提交本地分支到远程仓库
git push origin 本地分支名
例如： git push origin dev ，这条命令表示把本地dev分支提交到远程仓库，即创建了远程分支dev。
删除远程分支
git push --delete origin dev
```
# git修改分支名字
```
# Rename branch locally
git branch -m org_AF807 master
# Delete the old branch
git push origin :org_AF807
# Push the new branch, set local branch to track the new remote
git push --set-upstream origin master
```

# gitlab 基于其他仓库分支创建本地仓库基线
```
1. 在gitlab创建空的新仓库
git@10.155.100.55:root/AF8.0.8.git

2. 老仓库的代码路径
git@code.sangfor.org:AF/af.ngaf/project/AF8.0.8/AF8.0.8.git

3. 下载老仓库代码
git clone git@code.sangfor.org:AF/af.ngaf/project/AF8.0.8/AF8.0.8.git

4. 进入老仓库代码，切换到想要push的分支
cd AF8.0.8
git checkout -b branch1

5. 设置老仓库的url为新仓库路径
git remote set-url origin git@10.155.100.55:root/AF8.0.8.git

6. 推送本地分支到新仓库
推送主线: git push -u origin master
或者
推送分支: git push -u origin org_AF808

7. 分支修改名字，如果推送的是master就不用这一步
git branch -m org_AF808 master
git push --set-upstream origin master
在gitlab web上面把默认分支head修改为新推送的master，然后删掉老分支即可
```
# git 同步所有远程分支
```
git branch -r | grep -v '\->' | while read remote; do git branch --track "${remote#origin/}" "$remote"; done
git branch -r | grep -v '(\->|old-origin)' | while read remote; do git branch --track "${remote#origin/}" "$remote"; done
```

# git 以下命令将删除除master分支外的所有本地分支。
```
git branch | grep -v "master" | xargs git branch -D
```
# git 删除仓库远端分支
```
git push origin --delete old-origin/tag_OpenSSL_0_9_7i
```

#git基于tag拉分支
```
git 基于tag拉branch

1.执行:git origin fetch 获得最新.
2.通过:git branch <new-branch-name> <tag-name> 会根据tag创建新的分支.
例如:git branch newbranch ver1.0.0.1
会以tag ver1.0.0.1创建新的分支newbranch;
3.可以通过git checkout newbranch 切换到新的分支.
4.通过 git push origin newbranch 把本地创建的分支提交到远程仓库.

```
#git 清除远程仓库已经删除的本地分支 清除已经合并到master的本地分支
```
查看本地分支和追踪情况：
git remote show origin

git remote prune 来同步删除这些分支
git remote prune origin
```

# svn仓库推送到git
```
git init

find ./ -name ".svn" |xargs -i rm -rf {} 
find . -type d -empty -exec touch {}/.gitignore \;
git add -A -f

git push origin branch_name
```
# win下使用git修改文件可执行属性
```
git update-index --chmod=+x auto_build_gch.sh
```
# git下载代码但是不带git信息
```
git checkout-index --prefix=/home/ych/tmp/ -a
```
# git修改上次提交的方法
```
方法一: 用–amend选项 修改需要修改的地方
git add .
git commit –amend

注：这种方式可以比较方便的保持原有的Change-Id，推荐使用。

方法二: 先reset，再修改
这是可以完全控制上一次提交内容的方法。但在与Gerrit配合使用时，需特别注意保持同一个commit的多次提交的Change-Id是不变的。

否则，就需要Abondon之前的Change，产生一些垃圾不说，操作得不对，会使得简单的事情复杂化，甚至无法合并。

git reset HEAD^

重新修改
git add .
git commit -m “MSG”

特别注意：为了保持提交到Gerrit的Change不变，需要复制对应的Change-Id到commit msg的最后，可以到Gerrit上对应的Change去复制，参见图1。
```

# git设置名字和邮箱
```
git config user.name "姚楚宏93101"
git config user.email "93101@sangfor.com"
git config --global user.name "姚楚宏93101"
git config --global user.email "93101@sangfor.com"

git config --global user.name "air5005"
git config --global user.email "ych0510@126.com"
```
# git设置秘钥
```
ssh-keygen -t rsa -C "93101@sangfor.com" -b 4096
ssh-keygen -t rsa -C "ych0510@126.com" -b 4096
```
# 迁移git仓库流程
1. 修改本地仓库url为远端url
```
git remote set-url origin git@200.200.232.200:root/osc_fragrouter.git
```
2. 把需要push的分支都git checkout出来
可以使用下面脚本批量创建
```
git branch -r | grep -v '\->' | while read remote; do git branch --track "${remote#origin/}" "$remote"; done
```

3. 推送所有本地分支和tag
```
git push -u origin --all
git push -u origin --tags
```

# 私人仓库代码
```
git clone git@200.200.232.200:root/train_project.git
git clone git@200.200.232.200:root/app_example.git
git clone git@200.200.232.200:root/pa-base-lib.git
git clone git@200.200.232.200:root/snort.git
git clone git@200.200.232.200:root/libring.git
git clone git@200.200.232.200:root/pktring.git
git clone git@200.200.232.200:root/libmmap.git
git clone git@200.200.232.200:root/osc_openssl.git
git clone git@200.200.232.200:root/osc_nginx.git
git clone git@200.200.232.200:root/zr9101.git
git clone git@200.200.232.200:root/osc_libhtp.git
git clone git@200.200.232.200:root/upu.git
git clone git@200.200.232.200:root/dpdk.git
```


