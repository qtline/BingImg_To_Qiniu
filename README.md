# Bing每日一图抓取至七牛云（Shell版）BingImg_To_Qiniu

1.  通过解析BingAPI 获取Bing壁纸的url；
>   BingAPI地址：https://cn.bing.com/HPImageArchive.aspx?format=js&idx=1&n=1
2.  Shell环境下的JSON处理工具“jq” 处理获取的BingAPi JSON数据，并输出到清单列表。
>   jq工具：https://stedolan.github.io/jq
3.  然后通过七牛云官方工具qshell 抓取资源并存储到七牛空间中（不占本地空间）
>  七牛云qshell：https://github.com/qiniu/qshell 
4.  每日一图的数据输出为`yyyymmdd.json`同步上传至qiniu空间
5.  历史数据汇总bing原始数据为`bing.json` 自定义数据为`data.json`，同步上传至qiniu空间。
