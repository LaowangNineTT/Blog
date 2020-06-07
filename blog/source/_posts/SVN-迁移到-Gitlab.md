---
title: SVN 迁移到 Gitlab
date: 2018-08-06 09:46:27
tags:
---
> 自己搭建了 Gitlab 也有一段时间了，看了一下准备把一些项目陆续的迁移到 Gitlab 中。
> 
> 之前迁移了一次 CM，但是过程没有记录下来。
> 
> 这次记录一下过程
> 
> 参考一篇文章[CSDN Blog](https://blog.csdn.net/Hello_Mr_Cc/article/details/72742503)
> 
> 还有一篇[Blog](https://www.lovelucy.info/codebase-from-svn-to-git-migration-keep-commit-history.html)
> 
<!--more-->

### 准备工作

按照顺序执行，先把 svn 中的 author 导出成 user.txt

```bash
/www/htdocs/xxx » svn log ^/ --xml | grep -P "^<author" | sort -u | perl -pe 's/<author>(.*?)<\/author>/$1 = /' > users.txt
usage: grep [-abcDEFGHhIiJLlmnOoqRSsUVvwxZ] [-A num] [-B num] [-C[num]]
    [-e pattern] [-f file] [--binary-files=value] [--color=when]
    [--context[=num]] [--directories=action] [--label] [--line-buffered]
    [--null] [pattern] [file ...]
svn: E000032: Write error: Broken pipe
```

但是在执行的时候报错了，提示`Write error: Broken pipe`，看了网上的一些东西，描述是，在命令的执行中，如果是`xxx|xxx`这种，上一个命令的输出，给到了下一个命令的输入，如果上一个输出有问题，那么会提示错误信息。

后来去尝试不断的拆分命令，很快就定位到了 `grep -P` 这里，grep 是没有 -P 这个选项的。看了下 man page，没找到 P 选项的含义，所以这里去除了 -P 就 OK 了。

```bash
svn log ^/ --xml | grep "^<author" | sort -u | perl -pe 's/<author>(.*?)<\/author>/$1 = /'
```

得到的 user.txt 是
```bash
qingping.xxx=xxx<xxx@xx.cn>
```

这里把 Gitlab 中的 author 对应填入进去。

### 获取 SVN 的数据到本地

通过 git svn clone 命令把 SVN 仓库导入到 Git 中，

```bash
git svn clone svn://xxx.xx@svn.xx.cn:16/openapi/trunk/openapi.xx.xx --no-metadata --authors-file=user.txt /www/htdocs/openapi
```

通过以上命令就会在目标文件夹中生成一个 git 项目
```bash
------------------------------------------------------------
/www/htdocs/openapi(branch:master) » git branch -a
* master
  remotes/git-svn
------------------------------------------------------------
```

但是这里生成了一个本地的 master 分支，和一个 git-svn 的 remote，我总是看这个 git-svn 不顺眼，如何才能删掉呢？

```bash
/www/htdocs/openapi(branch:master) » git remote rm remotes/git-svn
error: Could not remove config section 'remote.remotes/git-svn'
```


发现这里有些奇怪，这里感觉有些奇怪，但是也没有

借鉴了博客里面的东西，使用```git clone```删除了里面原有的 git-svn remote.