# myblog
为便于在多台机器上写博客，将我的hexo个人博客存放于此

## **A电脑**备份博客到github
1. 进入博客根目录。确认.gitignore文件的存在。内容如下：
```bash
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

2. 在博客根目录下，在`git bash`下依次执行
```bash
git init
git remote add origin <server>
```

3. 同步到远程git仓库，在`git bash`下以此执行

```bash
git add .                   #添加目录下所有文件
git commit -m “更新说明”     #提交并添加更新说明
git push -u origin master   #推送更新到远程仓库
```

## **B电脑**拉下远程仓库文件
在B电脑上先安装好nodejs、git、hexo，
执行
```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
建好hexo文件夹，再执行以下命令：
```bash
git init
git remote add origin <server>    #添加远程仓库
git fetch --all                   #从远程仓库抓取数据到本地
git reset --hard origin/master    #彻底回退到某个版本，本地的源码也会变为上一个版本的内容
```

## ***B电脑***发布博客后同步
在B电脑发布完博客之后，将博客备份同步到远程仓库。
执行以下命令：
```bash
git add                     #可以用git master 查看更改内容
git commit -m "更新信息"
git push -u origin master   #以后每次提交可以直接git pu
```
## 每次写博客时同步管理
每次写博客之前，先执行
```bash
git pull
```
进行同步更新。
发布完文章后同样按照上面的*发布博客后同步*。 同步到远程仓库。

## 常用命令整理
```bash
git pull                      #同步更新github仓库
hexo new "文章名"              #简写形式 hexo n "文章名"
hexo clean                    #清除旧的public文件夹
hexo generate                 #生成静态文件 简写形式 hexo g
hexo deploy                   #发布到github上 简写形式 hexo d
git add .                     #添加更改文件到缓存区
git commit -m "更新说明"       #提交到本地仓库
git push -u origin master     #推送到远程仓库进行备份
```
