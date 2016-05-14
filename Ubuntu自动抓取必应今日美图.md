---
title: Ubuntu自动抓取必应今日美图
date: 2016-05-14 21:15:23
tags:
---
创建脚本```~/yourpath/bing.sh```，内容如下：

``` bash
#提取URL
url=$(expr "$(curl http://www.bing.com/ |grep hprichbg)" : ".*g_img={url:'\(.*\)',id.*")
echo $url
#提取图片名称
filename=$(expr "$url" : ".*/\(.*\)")
echo $filename
#本地图片地址
localpath="~/Pictures/$filename"
echo $localpath
#下载至本地
cd ~/Pictures/
wget "www.bing.com$url"
```

添加定时任务：

``` bash
crontab -e
```

在编辑器中加入以下内容：

``` bash
# m h  dom mon dow   command
30 0 * * * bash ~/yourpath/bing.sh
```

保存并退出，需要注意的是代码中的文件路径需要自行修改，请勿直接复制粘贴。

PS:如需将图片同步至Windows PC，可以借助第三方工具Dropbox-Uploader（详细使用说明请参考GitHub），在本机添加一项定时任务，代码如下：

``` bash
for file in ~/Pictures/*
do
        echo $file
        if test -f $file
        then
                ./dropbox_uploader.sh upload $file /Pictures/$(basename $file)
        fi
done
```
