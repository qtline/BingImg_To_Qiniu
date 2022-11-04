# Bing每日一图抓取至七牛云（Shell版）BingImg_To_Qiniu

1.  通过解析BingAPI 获取Bing壁纸的url；
>   BingAPI地址：https://cn.bing.com/HPImageArchive.aspx?format=js&idx=1&n=1
2.  Shell环境下的JSON处理工具“jq” 处理获取的BingAPi JSON数据，并输出到清单列表。
>   jq工具：https://stedolan.github.io/jq
3.  然后通过七牛云官方工具qshell 抓取资源并存储到七牛空间中（不占本地空间）
>  七牛云qshell：https://github.com/qiniu/qshell 
4.  每日一图的数据输出为`yyyymmdd.json`同步上传至qiniu空间
5.  历史数据汇总bing原始数据为`bing.json` 自定义数据为`data.json`，同步上传至qiniu空间。

```bash
#!/bin/bash
# 1、通过解析BingAPI 获取Bing壁纸的url；
#   BingAPI地址：https://cn.bing.com/HPImageArchive.aspx?format=js&idx=1&n=1
# 2、Shell环境下的JSON处理工具“jq” 处理获取的BingAPi JSON数据，并输出到清单列表。
#   jq工具：https://stedolan.github.io/jq
# 3、然后通过七牛云官方工具qshell 抓取资源并存储到七牛空间中
#   七牛云qshell：https://github.com/qiniu/qshell 


# 全局部分
Bingjson=tmp/BingJson/$(date +"%Y%m%d").json
curl 'https://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1' |jq '[.images[0]]' > $Bingjson
# 获取当日BingAPI数据，并保存到/tmp目录，以当前日期（年月日）命名

startdate=`jq -r '.[0].startdate' <$Bingjson`
title=`jq -r '.[0].title' <$Bingjson`
urlbase=`jq -r '.[0].urlbase' <$Bingjson`
copyright=`jq -r '.[0].copyright' <$Bingjson`
copyrightlink=`jq -r '.[0].copyrightlink' <$Bingjson`

# 拼接Bing图片地址
bingurl="https://cn.bing.com"$urlbase
bingimgurl_1080=$bingurl"_1920x1080.jpg"
bingimgurl_1366=$bingurl"_1366x768.jpg"
bingimgurl_uhd=$bingurl"_UHD.jpg"

localurl_1080=$startdate'_1920x1080.jpg'
localurl_1366=$startdate'_1366x768.jpg'
localurl_uhd=$startdate'_UHD.jpg'
# 生成批量获取到清单文件
echo $bingimgurl_1080'|'$localurl_1080'' > tmp/UpList/$startdate.txt
echo $bingimgurl_1366'|'$localurl_1366'' >> tmp/UpList/$startdate.txt
echo $bingimgurl_uhd'|'$localurl_uhd'' >> tmp/UpList/$startdate.txt
# 通过上传清单文件上传每日图片
./qshell batchfetch avatar --sep "|" -i tmp/UpList/$startdate.txt

# 生成每日图像JSON数据
echo '{
    "date":"'$startdate'",
    "title":"'$title'",
    "copyright":"'$copyright'",
    "copyrightlink":"'$copyrightlink'",
    "bingimgurl_1080":"'$bingimgurl_1080'",
    "bingimgurl_1366":"'$bingimgurl_1366'",
    "bingimgurl_uhd":"'$bingimgurl_uhd'",
    "localurl_1080":"'$localurl_1080'",
    "localurl_1366":"'$localurl_1366'",
    "localurl_uhd":"'$localurl_uhd'"
}' > tmp/DataJson/$startdate.json
# 上传每日图像JSON数据
./qshell fput avatar $startdate.json tmp/DataJson/$startdate.json

# 合并历史Bing历史JSON 至 bing.json
jq -s '.' tmp/BingJson/* >bing.json
./qshell fput avatar bing.json bing.json

# 合并历史每日图像JSON数据JSON 至 data.json
jq -s '.' tmp/BingJson/* >data.json
./qshell fput avatar data.json data.json


```
