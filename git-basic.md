# git basic

## 安装

```shell
# debian/ubuntu
sudo apt install git

# centos/rhel
sudo yum install git-core
```

查看版本信息

```shell
git --version
```

#### 配置

git的配置文件存在于三个位置：

- /etc/gitconfig（系统配置）
- ~/.gitconfig（用户特定配置）
- .git/config（项目特定配置）

从上到下，文件中的配置优先级依次升高，同名的配置会被覆盖。

若想正常使用git，至少要配置以下两个参数

```shell
# the option ''--global' instructs git to modify config in ~/.gitconfig
git config --global user.name = foo
git config --global user.email = foo@bar.com
```

查看配置

```shell
git config --list
```

> **提示：**
>
> 列出的配置项中，后面的会覆盖前面的

## 基本使用模式

#### 个人使用

开始一个新项目，创建一个新的库

```shell
# method 1
mkdir myrepo; cd myrepo; git init

# method 2
git init myrepo
```

创建第一个提交

```shell
vim file1
vim file2
git add file*
git commit

# show commit log
git log
```

修改文件，继续提交

```shell
vim file1
git diff # check modification
git add file1
git commit
```



#### 从头开始一个多人合作，共享中心库的项目

服务器上创建中心库

```shell
git init --bare
```

成员A创建自己的本地库，并做第一次提交

```shell
git init 
vim README
git add README
git commit
git remote add origion URL-TO-CENTRAL-BARE-REPO
git push origion master
```

其他成员可控中心库

```shell
git clone URL-TO-CENTRAL-BARE-REPO
```



## 常用命令

> 不同git命令或者参数操作的目标是不一样的。目标可能是
>
> - workdir 和staging area
> - staging area 和 repo

#### add/rm/restore/diff

```shell
# start tracking files or cache modifications
git add .
git add FILENAME ...

# rm tracked file in index
git rm --cached FILENAME ...

# rm files in work tree
git rm FILE ...

# Cancel modificaton in work tree
# work tree has the same content as staging area
git restore FILENAME ...

# restore files in index
# staging area and repo have the same state
git restore --staged FILENAME ...

# restore both index and work tree
git restore --worktree --staged FILE ...
git restore -W -S FILE ...

# compare work-dir and staging area
git diff

# compare staging area and repo
git diff --cached
```



#### 分支操作

```shell
# show all local branches. current branch is prefixed with an asterisk
git branch

# create a branch
git branch NAME

# switch to an other branch
git switch NAME
```



#### 远程git repo

```shell
# add a remote
git remote add NAME URL

# show remote
git remote show

# fetch and merge remote commit
git pull REMOTE BRANCH

# fetch only
git fetch REMOTE BRANCH

# publish commit to remote
git push REMOTE BRANCH
```

## 导出适合发布的代码包

```shell
git archive --format=tar --prefix=the-project/ HEAD \
	| gzip > the-project.tar.gz
```

