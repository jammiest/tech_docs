# location

nginx的location配置详解  
语法规则： `location [=|~|~*|^~] /uri/ { … }`
`=` 开头表示精确匹配  
`^~` 开头表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则`^~ /static/ /aa`匹配到（注意是空格）。  
`~` 开头表示区分大小写的正则匹配  
`~*`  开头表示不区分大小写的正则匹配  
`!~`和`!~*`分别为区分大小写不匹配及不区分大小写不匹配 的正则  
/ 通用匹配，任何请求都会匹配到。  

因此,大类型可以分为三种：

> `location = patt {}` [精准匹配]
> 
> `location patt{}`     [普通匹配]
> 
> `location ~ patt{}`  [正则匹配]

location命中过程:

?> 1.先进行**精准匹配**，如果命中立即返回结果并结束解析的过程；

?> 2.精准匹配未命中判断**普通匹配**，如果命中多个会记录下"最长的"命中结果，但不会结束解析；

?> 3.继续判断**正则匹配**，按照正则匹配设置的规则正则表达式进行匹配，如果有多个**正则匹配**由上到下进行匹配，一旦匹配成功一个会立即返回结果并结束解析；

?> ps:**普通匹配**的前后顺序是无所谓的，因为记录的是最长的结果，而正则匹配是按从上到下匹配的，这个需要注意！！！

```nginx
location = / {  
   #规则A  
}  
location = /login {  
   #规则B  
}  
location ^~ /static/ {  
   #规则C  
}  
location ~ \.(gif|jpg|png|js|css)$ {  
   #规则D  
}  
location ~* \.png$ {  
   #规则E  
}  
location !~ \.xhtml$ {  
   #规则F  
}  
location !~* \.xhtml$ {  
   #规则G  
}  
location / {  
   #规则H  
}  
```

那么产生的效果如下：  
访问根目录/， 比如`http://localhost/` 将匹配规则A  
访问 `http://localhost/login` 将匹配规则B，`http://localhost/register` 则匹配规则H  
访问 `http://localhost/static/a.html` 将匹配规则C  
访问 `http://localhost/a.gif`, `http://localhost/b.jpg` 将匹配规则D和规则E，但是规则D顺序优先，规则E不起作用，而 `http://localhost/static/c.png` 则优先匹配到 规则C  
访问 `http://localhost/a.PNG` 则匹配规则E， 而不会匹配规则D，因为规则E不区分大小写。  
访问 `http://localhost/a.xhtml` 不会匹配规则F和规则G，`http://localhost/a.XHTML`不会匹配规则G，因为不区分大小写。规则F，规则G属于排除法，符合匹配规则但是不会匹配到，所以想想看实际应用中哪里会用到。  
访问 `http://localhost/category/id/1111` 则最终匹配到规则H，因为以上规则都不匹配，这个时候应该是nginx转发请求给后端应用服务器，比如FastCGI（php），tomcat（jsp），nginx作为方向代理服务器存在。  
所以实际使用中，通常至少有三个匹配规则定义，如下：

```nginx
# 直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。  
# 这里是直接转发给后端应用服务器了，也可以是一个静态首页  
# 第一个必选规则  
location = / {  
    proxy_pass http://tomcat:8080/index  
}  
   
# 第二个必选规则是处理静态文件请求，这是nginx作为http服务器的强项  
# 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用  
location ^~ /static/ {  
    root /webroot/static/;  
}  
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {  
    root /webroot/res/;  
}  
   
# 第三个规则就是通用规则，用来转发动态请求到后端应用服务器  
# 非静态文件请求就默认是动态请求，自己根据实际把握  
# 毕竟目前的一些框架的流行，带.php,.jsp后缀的情况很少了  
location / {  
    proxy_pass http://tomcat:8080/  
}  
```