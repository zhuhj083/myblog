---
title: python制作微信好友照片墙
date: 2018-12-24 19:08:52
tags: [python,wechat]
categories: [python]
---

# 安装itchat和pillow库
```bash
sudo pip install itchat
sudo pip install pillow
```

# 代码如下
wechat_head_img.py
```bash
import itchat
import math
import os
from PIL import Image

itchat.auto_login(hotReload=True)
friends = itchat.get_friends(update=True)

num = 0
if not os.path.exists("headImg"):
    os.mkdir("headImg")
for friend  in friends:
    img = itchat.get_head_img(userName=friend["UserName"])
    if len(img) == 0:
        print("skip %d,friend=%s" % (num,friend["NickName"]) )
        continue
    print("%d,friend=%s" % (num,friend["NickName"]))
    fileImage = open('headImg' + "/" + str(num) + ".jpg",'web')
    fileImage.write(img)
    fileImage.close()
    num += 1

all_image = os.listdir('headImg')
print("There %d images" % len(all_image))
each_size = int(math.sqrt(float(640*640)/len(all_image)))
lines = int(640 / each_size)
image = Image.new('RGBA',(640,640))
x = 0
y = 0
for i in range(0,len(all_image)):
    imagePath = 'headImg'+"/"+str(i) + ".jpg"
    if not os.path.isfile(imagePath):
        continue
    img = Image.open(imagePath)
    img = img.resize((each_size,each_size),Image.ANTIALIAS)
    image.paste( img , (x*each_size,y*each_size) )
    x += 1
    if x == lines:
        x = 0
        y += 1

image = image.convert('RGB')
image.save('headImg'+"/"+"all.jpg")
itchat.send_image('headImg'+"/"+"all.jpg" , 'filehelper')

```

# 直接执行
```bash
# 添加执行权限
sudo chmod +x wechat_head_img.py
# 执行
python wechat_head_img.py
```
