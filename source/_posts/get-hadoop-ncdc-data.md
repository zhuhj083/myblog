---
title: 获取hadoop权威指南第四版中的NCDC数据
date: 2018-05-08 16:05:12
tags: [hadoop,ncdc]
categories: hadoop
---
美国国家气候数据NCDC的FTP地址为:
[ftp://ftp.ncdc.noaa.gov/pub/data/gsod/2013/](ftp://ftp.ncdc.noaa.gov/pub/data/gsod/2013/)

可以通过以下shell脚本来获得
``` bash
vim getNcdcBigData.sh
```

内容如下：
``` bash
#!/bin/bash
for i in {1901..2014}
do
cd /home/xxxx/hapood/ncdc
wget --execute robots=off -r -np -nH --cut-dirs=4 -R index.html* ftp://ftp.ncdc.noaa.gov/pub/data/gsod/$i/
done
```
