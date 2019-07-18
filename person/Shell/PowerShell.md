# PowerShell


## 常用方法

> `Get-Help Invoke-RestMethod`
获取`Invoke-RestMethod`命令帮助文档

> `Get-Help Invoke-RestMethod -Online`
获取`Invoke-RestMethod`联机帮助文档

## 命令参考说明

### Invoke-RestMethod 获取网络请求
```powershell
# GET http://www.bing.com?q=how+many+feet+in+a+mile HTTP/1.1
$r = Invoke-WebRequest -Uri http://www.bing.com?q=how+many+feet+in+a+mile
# 打印Content内容
$r.Content

# POST http://www.bing.com?q=how+many+feet+in+a+mile HTTP/1.1
# -Method 后面可跟 Default, Delete, Get, Head, Merge, Options, Patch, Post, Put, Trace
$r = Invoke-WebRequest -Uri http://www.bing.com?q=how+many+feet+in+a+mile -Method POST

# GET http://www.bing.com?q=how+many+feet+in+a+mile HTTP/1.1
# 携带header参数
$r = Invoke-WebRequest -Uri http://www.bing.com?q=how+many+feet+in+a+mile -Headers @{"Date" = "Wed, 09 Mar 2016 03:32:47 GMT"} -UseBasicParsing

# GET http://card.juooo5.com/Cardproduct/getDiscountScheduleList HTTP/1.1
# 格式化Content数据
$content = Invoke-WebRequest -Uri http://card.juooo5.com/Cardproduct/getDiscountScheduleList | Select-Object Content
$content.content | ConvertFrom-Json | ConvertTo-Json

# POST http://card.juooo5.com/Cardproduct/getDiscountScheduleList HTTP/1.1
# Body：{page=1,cityId=1,cid[0]=1,cid[1]=1}
$content = Invoke-WebRequest -Uri http://card.juooo5.com/Cardproduct/getDiscountScheduleList -Method POST -Body {page=1;cityId=1;cid[0]=3;cid[1]=5} | Select-Object Content
# or
$body = @{
 "UserSessionId"="12345678"
 "OptionalEmail"="MyEmail@gmail.com"
} | ConvertTo-Json

$header = @{
 "Accept"="application/json"
 "connectapitoken"="97fe6ab5b1a640909551e36a071ce9ed"
 "Content-Type"="application/json"
} 
$content = Invoke-WebRequest -Uri http://card.juooo5.com/Cardproduct/getDiscountScheduleList -Method POST -Body $body -Headers $header | Select-Object Content
```