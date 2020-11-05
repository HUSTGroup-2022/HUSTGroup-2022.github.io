---
title: Netlify+hexo+github多电脑多作者维护博客
date: 2020-11-05 16:06:53

categories:

- 前端
tags:

- Netlify
- hexo
---
[完成后的目录结构](https://github.com/HUSTGroup-2022/HUSTGroup-2022.github.io)。



### 准备工作
* 安装Node.js（Hexo基于NodeJs，其中npm工具对于网页部署很有用）
  [下载地址](https://nodejs.org/en/)，下载推荐安装版本LTS。
  下载好msi文件后，一路默认，唯独注意在Custom Setup这一步勾选 `Add to PATH` 添加到系统环境变量(新版本应该都已经默认勾选了)。
* 安装Git（源码托管）
  [下载地址](https://git-for-windows.github.io/)，exe一路默认安装到底。

### 安装hexo
hexo用于部署本博客。
首先创建一个新的文件夹，用于放置hexo的基本内容，在cmd窗口下cd到改目录下。

依次执行：
``` bash
npm i -g hexo
hexo -v
```

查看是否安装完成，此时cmd窗口输出：
```bash
(base) F:\xuhao\hexo>npm i -g hexo
C:\Users\Administrator\AppData\Roaming\npm\hexo -> C:\Users\Administrator\AppData\Roaming\npm\node_modules\hexo\bin\hexo
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@~2.1.2 (node_modules\hexo\node_modules\chokidar\node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.1.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ hexo@5.2.0
added 90 packages from 357 contributors in 39.189s

(base) F:\xuhao\hexo>hexo -v
hexo-cli: 4.2.0
os: Windows_NT 10.0.17134 win32 x64
node: 14.15.0
v8: 8.4.371.19-node.17
uv: 1.40.0
zlib: 1.2.11
brotli: 1.0.9
ares: 1.16.1
modules: 83
nghttp2: 1.41.0
napi: 7
llhttp: 2.1.3
openssl: 1.1.1g
cldr: 37.0
icu: 67.1
tz: 2020a
unicode: 13.0
```

利用github部署前还需要安装Deployer部署器，否则会报如下错误
``` bash
ERROR Deployer not found: git
```

继续在cmd窗口输入如下命令hexo-deployer-git
``` bash
npm install hexo-deployer-git --save
```


### 配置github
在本地创建ssh key，简化之后git提交代码，在Cmd中输入，注意替换成自己的邮箱：
```bash
ssh-keygen -t rsa -C "your_email@youremail.com"
```
一路回车到底，cmd提示在`C:\Users\Administrator/.ssh/id_rsa.pub.`路径下生成了id_rsa.pub.的公钥，找到该文件，以记事本形式打开，复制里面的key。
来到Github页面，登录自己的github账号之后，在右上角区域找到settings，左边选择`SSH and GPG keys`，然后New SSH key，title随便填写，粘贴id_rsa.pub里面的key，保存。
为了验证是否成功，在cmd页面输入
```bash
ssh -T git@github.com
```
会提示
```bash
Hi HUSTGroup-2022! You've successfully authenticated, but GitHub does not provide shell access.
```
代表已经连接成功。
此时已基本大功告成，但如果此时通过运行$ hexo deploy运行网站，会报出如下错误，意思是我们还没有设置user.email和user.name。
```bash
*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'Him@hongxiaoxin.(none)')
error: src refspec HEAD does not match any.
error: failed to push some refs to 'git@github.com:ghxiaoxiao/ghxiaoxiao.github.io'
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: Spawn failed
    at ChildProcess.<anonymous> (d:\hexo\blog\node_modules\hexo-util\lib\spawn.js:52:19)
    at emitTwo (events.js:126:13)
    at ChildProcess.emit (events.js:214:7)
    at ChildProcess.cp.emit (d:\hexo\blog\node_modules\cross-spawn\lib\enoent.js:40:29)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:198:12)
```
因此，进行如下设置，其中email是你的GitHub绑定的邮箱，用户名是你的GitHub用户名。
```bash
git config --global user.email "hustgroup_2022@163.com"
git config --global user.name "HUSTGroup-2022"
```

### clone博客源码或拉取博客更新到本地电脑
对于多电脑多作者维护博客模式，本机第一次进行博客维护时，应该首先clone托管在github上的博客源码。注意，`除了第一次，此后无需执行此clone步骤`。
在cmd窗口下，切换到自己希望存放博客源码的目录，执行：
```bash
git clone https://github.com/HUSTGroup-2022/HUSTGroup-2022.github.io
```
然后切换到该源码目录下，可以使用TAB键补全地址：
```bash
cd HUSTGroup-2022.github.io
```
`对于非第一次进行博客维护的主机，直接执行如下命令而不必执行clone命令，从github拉取最新的更新（因为其他电脑的其他作者可能更新了源码并且托管在github上)：`
首先执行
```bash
git branch
```
查看当前所在github 仓库的分支，若为`* source`则正确。因为我们在该博客仓库下建立了两个分支，分别为master和source，source中用于存放源码，master中用于存放静态网页。
若非source分支，输入
```bash
git checkout source
git branch
```
此时已切换到source分支。
最后执行
```bash
git pull
```
拉取更新（可能由其它电脑的其他作者更新了博客源码）。

### 更新博客
在本地HUSTGroup-2022.github.io文件夹下，进入source/_posts/文件夹，可以看到里面的.md（markdown文件）存放了网页的markdown源码。
在cmd窗口HUSTGroup-2022.github.io目录下，执行
```bash
npm install
```
安装package.json中依赖的包（由网站创建时所依赖或者维护加入的记录）。
然后执行
```bash
hexo new 1105testnew
```
建立一个新的.md文件（文件名自取）用于更新博客，可以在source/_posts/文件夹下看到多出了一个1105testnew.md文件。
编辑该文件，输入自己希望更新的网页内容。在3-hexo主题下添加多个categories或者tags标签应该换行，以'-'符号打头，例如：
```bash
categories:

- 前端
tags:

- Netlify
- hexo
```
最后，当.md文件更新完后，生成网页，将网页部署到github，同时将源码托管到github。依次执行（#后面是注释）：
```bash
hexo g #根据更新的源码生成新的网页
hexo deploy #根据生成的网页部署到github，同时由Netlify维护的博客网站会得到更新
git add . #将本机添加的新的网页源码加入git缓存，准备提交源码到github
git commit -m "1105" #添加提交到github的commit，“”内可输入任何注释内容，一般情况下不可省略
git push -u origin source #如果确定当前在source分支下，也可以仅仅执行 git push指令
```
首次在cmd里push可能会要求输入账号密码，按照提示依次完成，第一行提示之后会弹窗要求确认并打开网页确认：
```bash
info: please complete authentication in your browser...
Username for 'https://github.com':
Password for 'https://HUSTGroup-2022@github.com':
```
More info: [github建立多分支](https://www.pianshen.com/article/8216193907/)
