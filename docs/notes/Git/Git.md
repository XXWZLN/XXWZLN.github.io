# Git

## Git Config

git config 分为

- 系统级(--system)：为全部用户设置
- 用户级(--global)：为当前用户设置
- 仓库级(--local)：例如为当前仓库设置远程仓库

`git config --show-origin -l` 会列出当前的所有级别的设置，以及每个设置所处的配置文件

配置用户名和邮箱

```shell
git config --global user.name "xxx"  
git config --global user.email xxx@outlook.com   
```

配置push相关

```shell
git config --global --add push.default current
git config --global --add push.autoSetupRemote true
```

你会收获两个好处。1）不需要「git push origin xxx」，只要「git push」，2）再也不会遇到「no upstream branch」的报错，也不需要「git push –set-upstream origin test && git push」。因为我们执行 git push 的大部分场景都是 push 到同名的 remote branch。来源是 [Auto setup remote branch and never again see an error about the missing upstream | pawelgrzybek.com](https://pawelgrzybek.com/auto-setup-remote-branch-and-never-again-see-an-error-about-the-missing-upstream/)。

## Ignore

全局ignore: ~/.gitignore_global，MacOS中建议屏蔽`.DS_Store`

```
*~
.DS_Store
```

在主目录下建立`.gitignore`文件

| 匹配示例     | 解释                                                                           |
| ------------ | ------------------------------------------------------------------------------ |
| bin/         | 忽略当前路径下的 bin 文件夹，该文件夹下的所有内容都会被忽略，不忽略 bin 文件。 |
| /bin         | 忽略根目录下的 bin 文件。                                                      |
| /*.c         | 忽略 cat.c，不忽略 build/cat.c。                                               |
| debug/*.obj  | 忽略 debug/io.obj，不忽略 debug/common/io.obj 和 tools/debug/io.obj。          |
| **/foo       | 忽略 /foo, a/foo, a/b/foo 等。                                                 |
| a/**/b       | 忽略 a/b, a/x/b, a/x/y/b 等。                                                  |
| !/bin/run.sh | 不忽略 bin 目录下的 run.sh 文件。                                              |
| *.log        | 忽略所有 .log 文件。                                                           |
| config.php   | 忽略当前路径的 config.php 文件。                                               |

- `git check-ignore -v <file>`：查看某个文件是否被忽略，以及匹配的规则
- [gitignore 模板](https://github.com/github/gitignore)

## remove file

添加文件到暂存区用`git add .`，删除分为两种

```shell
# 可以删除本地版本库中的文件和本地存储的文件
git rm <file>
# 仅删除本地版本库中的文件（文件变为untracked状态）
git rm -cached <file>
```

## Commit Message Format

> Reference
>
> - [angular/CONTRIBUTING.md at main · angular/angular (github.com)](https://github.com/angular/angular/blob/main/CONTRIBUTING.md#-commit-message-format)
>
> - [Commit message 和 Change log 编写指南 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

简单来说基本的格式是

```
<type>(<scope>): <subject>
// 空一行
<body>
```

`type`是基本的提交类型：

- feat：新功能（feature）
- fix：修补bug
- docs：文档（documentation）
- style： 格式（不影响代码运行的变动）
- refactor：重构（即不是新增功能，也不是修改bug的代码变动）
- test：增加测试
- build：构建过程或辅助工具的变动

`scope`用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

`subject` 是简述

`body` 是详细描述

## branch

```shell
# create branch
git branch <name>
# create branch and into it
git checkout -b <name>

# delete local branch
git branch -d <name>
# delete remote branch
git push origin --delete <name>

# switch to branch
git checkout <name>

# show branch
git branch
# show remote branch
git branch -r

# rename branch
git branch -m <old> <new>
```

## git stash

```shell
# create a stash stack after git add, then working diretory is clean
git stash
# recommend command below to create
git stash save "message"

# list all the stash stack
git stash list

# get back to the last working diretory (won't delete this stash)
git stash apply

# get back to one of the previous working diretory
git stash apply atash@{2}

# get back and delete this stash
git stash pop

# delete stash
git stash drop stash@{2}
```

## Checkout

`git checkout` 可以将当前的工作区切换到任意一个提交的记录上：

- `git checkout <branch>` 切换到branch的最新一个提交
- `git checkout <commit-id>` 切换到指定的 commit 提交
- `git checkout <tag>` 切换到指定的 tag 。因为tag类似一个有特殊名称的commit记录，所以会直接切换到 tag 指向的commit记录。

> Tag & Branch
>
> tag 和 branch 都是控制版本和开发进度的工具，两者相互独立。tag 管辖的 commit 提交范围仅限一个 commit，而 branch 管辖一整个分支中的所有 commit。

## detached HEAD

前面说到 branch 的管辖范围是一个分支中的所有 commit记录，每一个新的 commit 记录都会自动添加到当前branch head 的后面并将 head 新的记录上。如果当前工作区所在的commit记录(Head)不是一个分支的最后一个记录，就没有办法将提交的记录添加到分支的后面，这种情况就是`detached HEAD`。出现这种情况的一个场景就是：当前 head 位于`commit_n`处，但想回到之前开发的某个提交`commitn - 4` 处开发，又不想废弃当前的开发进度，一个合理的想法是用`git checkout`将 HEAD 移动到之前的 commit，这时会出现 `detached HEAD`。解决方法就是到达这个老commit之后新建一个分支，然后进行接下来的开发。

## Get Back

在开发中经常遇到想要回到之前开发状态的情况，这里根据可能的场景讨论一下对应的操作

1. `git reset --soft <commit-id>`：不想更改工作区的代码，但是对最新的提交不满意，想回到之前的提交（本地的代码不变，但修改 Git 的追踪记录）
2. `git reset --hard <commit-id>`：代码写死了，不知道问题在哪，想将整个项目回退到之前的状态（本地的代码和 Git 记录全部回退）
3. `git revert <commit-id>`：想要废弃新开发的 feature，但不想抹去代码记录，以防以后要再次使用这个feature，让工作区和 Git 记录“回到原先的状态”，但 Git记录不会抹去。（慎用）具体原理见官方文档
4. `git checkout <commit-id>`：现在是 2.0 版本，但想回到 1.0 版本做长期的维护（需要新建分支，避免 detached HEAD）

>如果当前commit只有一个父commit（没有 merge），`HEAD~ == 上一个 commit` `HEAD~2 == 上 2 个 commit` 

## link remote repo

```shell
# link remote repo
git remote add origin [url]

# show linked remote repo
git remote show origin

# remove remote repo
git remote remove origin
```

## push to github

```python
# push local branch to remote repo's default branch(master)
git push origin <local-branch>
# push local branch to specific remote branch
git push origin <local-branch>:<remote-branch>
# if local branch name is same as remote branch
git push origin <local-branch>

# if remote repo is empty use '-u'
# -u, --set-upstream    set upstream for git pull/status
git push -u origin master
```

`git push` 的作用是将指定的本地分支推送到指定的远程分支，可以用`git remote show origin` 和 `git branch -vv` 显示本地分支和远程分支的对应关系。在新建远程分支的时候需要使用`-u`设置这种映射。

```shell
git config --global --add push.default current
git config --global --add push.autoSetupRemote true
```

- 一般操作是将本地分支推送到同名的远程分支，使用`git push origin <local-branch>`。但是这不方便，因为通常都是在当前分支下推送当前分支。使用第一个设置就可以只输入`git push` 推送当前分支到远程
- 如果是第一次推送这个分支，需要命令`-u`，这也有点麻烦，使用第二个设置，也只需要输入`git push`就行

Reference：[Auto setup remote branch and never again see an error about the missing upstream | pawelgrzybek.com](https://pawelgrzybek.com/auto-setup-remote-branch-and-never-again-see-an-error-about-the-missing-upstream/)

## pull from github

```shell
# fetch specific branch and merge it to current branch
git pull origin <remote-branch>
```

使用`git pull`可以将远程仓库的最新状态拉取下来，并和当前所在的本地仓库合并。通常来说本地仓库和远程仓库的名称相同，如果在分支main下，执行`git pull origin main`就可以将本地仓库的main更新为最新的云端版本。如果本地的提交和远程的最新版本冲突，需要手动解决冲突。所有的pull操作必须在本地暂存区没有修改的时候执行。

如果要拉取一个本地没有的远程分支，应该执行

```shell
git fetch
git checkout -b <new-local-branch> origin/<remote-branch>
```

而不是`git pull`

按照惯例`new-local-branch` 和 `remote-branch` 应该相同

## Download specific part of repo 

https://github.com/download-directory/download-directory.github.io.git

https://github.com/MinhasKamal/DownGit.git

## License

[添加许可到仓库 - GitHub 文档](https://docs.github.com/zh/communities/setting-up-your-project-for-healthy-contributions/adding-a-license-to-a-repository)

### 软件的开源许可

![img](./assets/free_software_licenses.png)

### CC（Creative Commons）

用于“知识共享”，不用于软件

使用方法：按照 Github 官方文档创建许可证文件，选择CC 许可证即可。如果要使用 CC 的指定版本，可以直接到[CC 官网](https://creativecommons.org/share-your-work/cclicenses/)获得许可证文件，粘贴到 `LICENSE`文件中即可。

可能比较常见的版本：

- CC BY-SA：*ShareAlike，需要采用相同许可证
- CC BY：Attribution，需要标明原作者
- CC 0：Public Domain，进入公共领域

> reference:

> - https://www.bilibili.com/video/BV12u4y177vG/?spm_id_from=333.999.0.0&vd_source=fdd56007771480ca58f731c9abc561aa