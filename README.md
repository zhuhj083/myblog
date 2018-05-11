# myblog
为便于在多台机器上写博客，将我的hexo个人博客存放于此

## A电脑备份博客内容到github
1. 进入博客目录。确认.gitignore文件的存在。内容如下：
```bash
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```
2. 在博客根目录下，在git bash下依次执行`git init`和 `git remote add origin <server>`
3. 同步到远程git仓库，在git bash下以此执行
```bash
git add .                   #添加目录下所有文件 
git commit -m “更新说明”     #提交并添加更新说明 
git push -u origin master   #推送更新到远程仓库
```

## B电脑拉下远程仓库文件
在B电脑上先安装好nodejs、git、hexo，然后建好hexo文件夹，（然后选做：将备份到远程仓库的文件及文件夹删除），然后执行以下命令：
```bash
git init 
git remote add origin <server> 
git fetch --all 
git reset --hard origin/master
```

## 发布博客后同步
在B电脑发布完博客之后，将博客备份同步到远程仓库。 
执行以下命令：
```bash
git add                     #可以用git master 查看更改内容  
git commit -m "更新信息"  
git push -u origin master   #以后每次提交可以直接git pu
```
## 平时同步管理
每次写博客的时候，先执行
```bash
git pull
```
进行同步更新。 
发布完文章后同样按照上面的 发布博客后同步。 同步到远程仓库。

## 常用命令整理
```bash
git pull                      #同步更新
hexo new post "新建文章"       #简写形式 hexo n "新建文章"
hexo clean                    #清除旧的public文件夹
hexo generate                 #生成静态文件 简写形式 hexo g
hexo deploy                   #发布到github上 简写形式 hexo d
git add .                     #添加更改文件到缓存区
git commit -m "更新说明"       #提交到本地仓库
git push -u origin master     #推送到远程仓库进行备份
```



