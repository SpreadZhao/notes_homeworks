# 1. Apache服务器配置

## 修改配置文件

该文件位于Apache目录下的`conf/httpd.conf`。打开并编辑这个文件：

```
Define SRVROOT "D:/greenprogram/Apache24"
ServerRoot "${SRVROOT}"
Listen 80
ServerName localhost:80
```

## 安装Apache服务

使用管理员权限打开终端，并输入如下命令：

```shell
httpd -k install -n "Apache24"
```

其中，Apache24是我们可以自定义的服务名。

## 启动Apache服务

在此电脑的服务中打开，或者直接`net start Apache24`都可以打开Apache服务。另外，我们也可以通过`/bin/ApacheMonitor.exe`来检测Apache服务。图形化界面很简洁明了。

## 测试服务

当在浏览器中输入`localhost:80`的时候，显示`It works!`就代表启动成功了。



# 2. git使用技巧

## 已经推送过的文件，但是本地发现他不用提交(比如clion的cmake-build-debug)

* 可使用如下代码来操作：

  ```git
  git rm --cached <files>
  ```

* 如果删除的文件不在当前目录下，而在子目录下，需要递归操作：

  ```git
  git rm -r --cached <d1/d2/files>
  ```
  
* 如果不想直接删，只想列出删了什么，就加一个-n

  ```git
  git rm -r -n --cached <d1/d2/files>
  ```

* 然后还需要提交这次修改

  ```git
  git commit -m "deleted files"
  ```

* 最后push

  ```git
  git push
  ```

另外如果不想再继续提交，可以在删除这个文件之后添加到`.gitignore`。比如就是本仓库中的`.obsidian`和`.trash`文件夹在云端删除后可以新建`.gitignore`并这样填写：

```git
.obsidian/
.trash/
```

然后再正常进行提交就不会再提交已经删掉的文件了。

---

## 本地的文件删掉了，我在远程仓库也不要了，怎么把这个删除后的状态同步到远程仓库

* 如果目录中包含中文，使用如下命令配置

  ```git
  git config --global core.quotepath false
  ```

* 删除了本地文件，查看状态

  ```git
  git status
  ```

* 这时候会看到这样的情况

![[gitst.png]]

* 然后，提交这次改动

  ```git
  git add -A
  ```

* 然后正常commit+push即可删除，这里-A表示删除mode

---

## 手动添加Git Bash Here到右键菜单

* 打开`regedit`

* 定位到`Computer\HKEY_CLASSES_ROOT\Directory\Background`

* 如图添加键值对

![[gitbash.png]]

* ~~然后再添加图标和程序目录就好了~~

![[gitbash2.png]]

* 上面这么做是错的！`command`应该建在子项中，参考`cmd`是咋做的

![[gitbash3.png]]

  不过`icon`确实是加在那里

---

## 不clone仓库而添加远程仓库

* `git init`在当前目录下生成`.git`文件夹

* 在那边建好仓库，复制地址

* `git remote add origin <地址>`

  origin就是仓库的名字，可以随便起，只是给本地一个提示罢了，origin是clone下来默认的名字

* 如果改地址，那就`git remote set-url <仓库名> <新地址>`

---

## gitee和github同步仓库，一次提交两次更新

* 首先有一个仓库，然后在另一个上面先import

* 然后在本地的`.git`里加上另一个的url

  ![[newurl.png]]

* 这样在`remote -v`的时候就能看到多个地址

  ![[remote.png]]

  > 我们能看到github的只支持push操作不支持pull操作

* 然后在修改提交过后就会有两次提交记录在shell中了

![[push.png]]

# 3. Obsidian

## pdf导出

安装minimal主题之后，代码段为黑色，并且表格非常难看，之后找到了这篇文章：

[PDF and print style reset with code syntax highlighting - Share & showcase - Obsidian Forum](https://forum.obsidian.md/t/pdf-and-print-style-reset-with-code-syntax-highlighting/31761)

因此新建如下代码片段放到`.obsidian/snippets`目录下，并在obsidian中应用即可：

```css
@media print {
  h1, h2, h3, h4, h5, h6, p, ul, li, ol {
    font-size: initial;
    font-weight: initial;
    font-family: initial;
    color: initial !important;
    background: none !important;
    outline: none !important;
    border: none !important;
    text-shadow: none !important;
  }

  th, td {
    font-size: initial;
    font-weight: initial;
    font-family: initial;
    color: initial !important;
    background: none !important;
    outline: none !important;
    text-shadow: none !important;
    border: 1px solid darkgray !important;
  }

  a {
    font-size: initial;
    font-weight: initial;
    font-family: initial;
    color: blue !important;
    text-decoration: underline !important;
    background: none !important;
    outline: none !important;
    border: none !important;
    text-shadow: none !important;
  }

  a[aria-label]::after {
    display: inline !important;
    content: " (" attr(aria-label) ")" !important;
    color: #666 !important;
    vertical-align: super !important;
    font-size: 70% !important;
    text-decoration: none !important;
  }

  pre,
  code span,
  code {
    color: black !important;
    background-color: white !important;
  }

  code {
    border: 1px solid darkgray !important;
    padding: 0 0.2em !important;
    line-height: initial !important;
    border-radius: 0 !important;
  }

  pre {
    border: 1px solid darkgray !important;
    margin: 1em 0px !important;
    padding: 0.5em !important;
    border-radius: 0 !important;
  }

  pre > code {
    font-size: 12px !important;
    border: none !important;
    border-radius: 0 !important;
    padding: 0 !important;
  }

  pre > code .token.em { font-style: italic !important; }
  pre > code .token.link { text-decoration: underline !important; }
  pre > code .token.strikethrough { text-decoration: line-through !important; }
  pre > code .token { color: #000 !important; }
  pre > code .token.keyword { color: #708 !important; }
  pre > code .token.number { color: #164 !important; }
  pre > code .token.variable {  }
  pre > code .token.punctuation {  }
  pre > code .token.property {  }
  pre > code .token.operator {  }
  pre > code .token.def { color: #00f !important; }
  pre > code .token.atom { color: #219 !important; }
  pre > code .token.variable-2 { color: #05a !important; }
  pre > code .token.type { color: #085 !important; }
  pre > code .token.comment { color: #a50 !important; }
  pre > code .token.string { color: #a11 !important; }
  pre > code .token.string-2 { color: #f50 !important; }
  pre > code .token.meta { color: #555 !important; }
  pre > code .token.qualifier { color: #555 !important; }
  pre > code .token.builtin { color: #30a !important; }
  pre > code .token.bracket { color: #997 !important; }
  pre > code .token.tag { color: #170 !important; }
  pre > code .token.attribute { color: #00c !important; }
  pre > code .token.hr { color: #999 !important; }
  pre > code .token.link { color: #00c !important; }
}
```



#TODO 
- [ ] 学习Dataview插件的使用

# 4. MySQL的配置

下载MySQL的绿色版。然后解压缩，在里面会发现下面的结构：

![[sql1.png]]

* 注：`my.ini`本来是没有的。

然后新建`my.ini`，并写入如下配置：

```yaml
[mysqld]
# 设置3306端口
port=3306

# 设置安装目录
basedir=D:\greenprogram\mysql

# 设置数据存放目录
datadir=D:\greenprogram\mysql\data

# 允许最大连接数
max_connections=200

# 允许链接失败的次数
max_connect_errors=10000

# 字符集
character-set-server=utf8mb4

# 默认存储引擎
default-storage-engine=INNODB

[mysql]
# 客户端默认字符集
default-character-set=utf8mb4

[client]
# 设置mysql客户端连接服务端时默认使用的端口和默认字符集
port=3306
default-character-set=utf8mb4
```

然后开始初始化MySQL。在`bin/`目录下使用`mysqld`执行初始化命令(**管理员**)：

```shell
mysqld --initialize --console
```

这是初始化MySQL，并将结果打印在控制台上。此时会出现一个**初始密码**，需要记住。

然后还要注册MySQL服务并启动它。

```shell
mysqld --install <serviceName>
net start <serviceName>
```

最后就可以用初始化密码登录了。

```shell
mysql -h localhost -u root -p
```

然后还要修改密码：

```shell
mysql> alter user 'root'@'localhost' identified with mysql_native_password by 'New Password';
```

退出：

```shell
quit
```

# 5. Windows

## 备份/恢复驱动

备份：cmd管理员

```shell
DISM /Online /Export-Driver /Destination:"D:\DriverBackup"
```

如果是win10 1607以上的版本，也可以用：

```shell
pnputil /export-driver * D:\DriverBackup
```

用powershell的话，还有别的招：

```shell
Export-WindowsDriver -Online -Destination D:\DriverBackup
```

恢复在设备管理器那里就能恢复了，自动搜索。

# 6. Linux

## Ubuntu调整字体大小

安装`gnome-tweaks`工具即可，之后便会出现Tweaks工具。在里面就能设置字体大小了。