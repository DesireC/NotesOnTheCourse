[<img src="../index.jpg" width = "80" height = "80"  />](../index.md#index)

<h1 id="git">Git命令</h1>

## 基础

`git help checkout` 查看git checkout的命令帮助
`git clone https://github.com/libgit2/libgit2` 克隆远程仓库到本地
`git status` 查看当前分支状态
`git add m.js`将m.js放到暂存区域
`git commit -m 'desc' `提交更新并且添加备注desc
`git commit -am 'desc' `跳过使用暂存区域直接提交更新并且添加备注desc
`git push` 推送更新到远程仓库
`git pull` 获取远程仓库的更新

## 分支管理

`git chekcout mobile` 如果存在mobile分支，切换到mobile分支；否则新建mobile分支
`git branch -d mobile `删除mobile分支
`git checkout -b mobile origin/mobile `拉取远程分支mobile到本地
`git chekcout -b mobile` 基于当前分支新建并切换到mobile分支
`git push origin mobile`将本地的mobile分支同步到远程仓库
`git push origin --delete mobile`删除远程的mobile分支
`git ls-remote `查看远程分支列表
`git merge mobile` 将mobile分支合并到当前分支

## 标签管理

`git tag` 查看本地标签列表
`git tag -a v1.4 -m 'my version 1.4'` 创建标签v1.4，并且添加标签说明“my version 1.4”
`git push origin v1.4` 将标签v1.4同步到远程仓库
`git push origin --tags` 将所有本地标签同步到远程仓库
`git tag -d v1.4` 删除标签v1.4
`git push origin :refs/tags/v1.4` 删除远程仓库标签v1.4
`git tag v1.4new v1.4` 复制v1.4标签，生成v1.4new标签
`git checkoout v1.4` 切换当前文件到v1.4（跟切换分支的命令一致）
`git checkout -b mobile v1.4` 基于v1.4标签生成mobile分支

## 其他

`git reset HEAD m.js` 取消m.js的暂存
`git checkout -- m.js` 撤销m.js的本地修改
`git reset --hard 版本序列号` 还原本地文件到某个版本
`git reset --hard HEAD~2`工作目录、暂存区域、本地仓库回退到上2个版本
`git log master ^origin/master` 查看到未传送到远程代码库的提交详情

### 如何只更新某个文件?

`git fetch origin master `先更新本地库(但不更新工作拷贝)
`git log -p master..origin/master` 查看差异
`git checkout origin/master -- path/to/file` 更新单个文件的工作拷贝

而 更新所有文件的工作拷贝 的命令如下： 
`git merge origin/master`

### 某个文件的某一行修改了，如何追踪修改人？

`git blame m.js`
`git gui blame m.js` 使用git GUI界面来查看每一行的变化，更直观

### 如何查看某个文件的历史修改？

`git log m.js`
`git show 版本号` 查看某个版本的修改详情

### 如何对某个文件进行版本回退？

```
git log m.js` 
`git reset 9aa51d89799716aa68cff3f30c26f8815408e926 m.js` 将m.js回退到某个版本
`git checkout m.js`
`git commit -m "m.js回退版本" `
或者
`git checkout 9aa51d89799716aa68cff3f30c26f8815408e926 m.js
```

### 如何对某个文件取消版本控制，但是不删除？

```
git rm --cached m.js`
然后更新 .gitignore 忽略掉目标文件，最后 
`git commit -m "We really don't want Git to track m.js anymore!"
```

### 如何查看差异？

`git diff HEAD`工作目录与上次提交时之间的所有差别
`git diff `查看尚未暂存的文件更新了哪些部分
`git diff filename` 查看尚未暂存的某个文件更新了哪些
`git diff --cached `查看已经暂存起来的文件和上次提交的版本之间的差异
`git diff --cached filename` 查看已经暂存起来的某个文件和上次提交的版本之间的差异
`git diff ffd98b291e0caa6c33575c1ef465eae661ce40c9 b8e7b00c02b95b320f14b625663fdecf2d63e74c` 查看某两个版本之间的差异
`git diff ffd98b291e0caa6c33575c1ef465eae661ce40c9:filename b8e7b00c02b95b320f14b625663fdecf2d63e74c:filename` 查看某两个版本的某个文件之间的差异