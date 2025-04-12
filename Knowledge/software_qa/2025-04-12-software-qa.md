---
title:
  - Rewrite Your Git History
date: 2025-04-12
tags:
  - softwareqa/git
mtrace:
  - 2025-04-12
---

# Rewrite Your Git History

昨天（其实是今天凌晨）我彻底把git的记录给重新写了一遍。因为以前我有用Github Desktop提交的记录，那个里面配置的邮箱是什么noreply，应该是自动分配的。我要彻底换成我的。

另外，我的邮箱从`www.spreadzhao@outlook.com`换成了`spreadzhao@outlook.com`，导致我github上几乎所有的contribution记录都没了。所以，这次彻底重来一遍。

看到了有`git-filter-repo`这个工具。[用法](https://stackoverflow.com/a/60364176/19270892)是：

```shell
git-filter-repo --name-callback 'return name.replace(b"OldName", b"NewName")' --email-callback 'return email.replace(b"old@email.com", b"new@email.com")'
```

看样子我们是相当于传了两个lambda一样。一个是改名字的回调，另一个是改邮箱的回调。上面的命令会收到警告，告诉你需要加上`--force`才能继续。

通过这种方式，我们还能用来验证修改是否完成。

因为这个仓库里所有的提交都是我，所以其实我可以用下面的命令来验证是否还有没改过的提交：

```shell
git filter-repo --commit-callback '
if commit.author_email != b"spreadzhao@outlook.com":
    print(f"Commit: {commit.original_id.decode()}")
    print(f"Author: {commit.author_name.decode()} <{commit.author_email.decode()}>")
    print(f"Date: {commit.author_date.decode()}")
    print(f"Message: {commit.message.decode()}")
    print("-" * 40)
'
```

这样，所有提交记录中邮箱不是`spreadzhao@outlook.com`的就会被过滤出来。同样也可以用来过滤名字：

```shell
git filter-repo --commit-callback '
if commit.author_name != b"SpreadZhao":
    print(f"Commit: {commit.original_id.decode()}")
    print(f"Author: {commit.author_name.decode()} <{commit.author_email.decode()}>")
    print(f"Date: {commit.author_date.decode()}")
    print(f"Message: {commit.message.decode()}")
    print("-" * 40)
'
```

注意，当修改过记录之后，所有被修改的记录的hash值都会变。所以，我直接把所有的remote仓库都删了，重新提交了一变。在多人仓库中，最好不要这么干。

`git-filter-repo`的使用说明：[git-filter-repo(1)](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html)