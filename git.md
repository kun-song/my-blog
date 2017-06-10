# Git

## 配置别名

如果经常使用 Git，则有的命令如 `checkout` 比较长，每次手动输入太麻烦，此时可以为 Git 配置快捷键，大大提高效率。

Git 的配置文件分为三个层次：
1. `/etc/gitconfig` 文件包含所有用户的 Git 配置。
2. `~/.gitconfig` 文件包含特定用户的 Git 配置。
3. `.git/config` 文件，只对该版本库生效。

一般在 `~/.gitconfig` 里配置即可，打开 `.gitconfig` 文件，里面已经存在一些配置：

```
[core]
        excludesfile = /Users/satansk/.gitignore_global
[difftool "sourcetree"]
        cmd = opendiff \"$LOCAL\" \"$REMOTE\"
        path =
[mergetool "sourcetree"]
        cmd = /Applications/SourceTree.app/Contents/Resources/opendiff-w.sh \"$LOCAL\" \"$REMOTE\" -ancestor \"$BASE\" -merge \"$MERGED\"
        trustExitCode = true
[user]
        name = satansk
        email = satansk@hotmail.com
        username = satansk
```

要添加别名配置，只要在配置文件下方按照如下格式添加即可：

```
[alias]
        ck = checkout
```

配置结束后，就可以用 `git ck` 替代 `git checkout` 了.

## 分支

### 基于分支的开发流程

#### 每次开发新特性，都应该新建一个分支：

```
git checkout -b xx
```
* 该命令实际相当于 `git branck xx` + `git checkout xx`。

#### 新特性开发完成后，切换到主分支，然后将开发分支合并到主分支：

```
// 切换回主分支
git checkout master

// 合并分支
git merge --no-ff xx
```
* Git 默认使用 **快进式合并**，会直接将 `Master` 分支指向 `xx` 分支，只保留一条时间线。
* 此处使用 `--no-ff` 选项，将保留分支自己的时间线，版本管理更加清晰。

#### 分支合并之后，删除开发分支

```
git branch -d xx
```


* 查看所有分支：`git branch`
* 创建分支：`git branch xx`
* 切换分支：`git checkout xx`
* 删除分支：`git branck -d xx`

注意，创建 + 切换分支常常用 `git checkout -b xx` 代替。
