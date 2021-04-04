# Nginx容器FastDFS模块部署

前置环境

- Nginx容器：参考 [Nginx容器构建](/person/虚拟化/Docker/Nginx容器构建)

## 构建环境说明

Nginx官方容器不包含nginx源代码，因此无法直接使用nginx源码的./configure方法构建FastDFS模块，需要到nginx官网下载对应版本的nginx源码到容器内，通过源码执行构建和编译指令

### Step1 安装依赖

```sh
apt-get install libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev
```

### Step2  [libfastcommon安装](https://github.com/happyfish100/libfastcommon/releases)

> 解压及安装

```sh
tar zxf libfastcommon-1.0.39.tar.gz
cd libfastcommon-1.0.39
./make.sh
./make.sh install
```

### Step3 下载[fastdfs的Ninx模块](https://github.com/happyfish100/fastdfs-nginx-module/releases)

```sh
tar zxf fastdfs-nginx-module-1.20.tar.gz
cd fastdfs-nginx-module-1.20/src
```

> 因为libfastcommon安装位置问题，需要修改fastdfs的Ninx的配置文件config，否则会在不到头文件

```sh
# 替换config文件中的/usr/local/include为/usr/include/fastcommon
sed -i 's/\/usr\/local\/include/\/usr\/include\/fastcommon/g' config
```

## 查看当前容器nginx配置信息

```
root@service:/# nginx -V
	nginx version: nginx/1.17.3
	built by gcc 6.3.0 20170516 (Debian 6.3.0-18+deb9u1) 
	built with OpenSSL 1.1.0k  28 May 2019
	TLS SNI support enabled
	configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-g -O2 -fdebug-prefix-map=/data/builder/debuild/nginx-1.17.0/debian/debuild-base/nginx-1.17.0=. -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie'
```

> 在Nginx源码配置参数追加`--add-module=/root/fastdfs-nginx-module-1.20/src`，执行./configure

```sh
./configure --add-module=/root/fastdfs-nginx-module-1.20/src --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-g -O2 -fdebug-prefix-map=/data/builder/debuild/nginx-1.17.0/debian/debuild-base/nginx-1.17.0=. -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie'
```

## Step4 编译Nginx

```sh
make && make install
```

> 再次查看`nginx -V`,发现会多了`--add-module=/root/fastdfs-nginx-module-1.20/src`

### Step5 下载[FastDFS源码](https://github.com/happyfish100/fastdfs/releases)并解压并复制配置文件

```sh
tar zxf fastdfs-5.11.tar.gz
cp fastdfs-5.11/conf/* /etc/fdfs/
```

## 配置fdfs配置文件（可选）

- /etc/fdfs/mod_fastdfs.conf

## 重启Nginx容器

## Nginx配置文件参考

```conf
server
{
	listen 80 ;
	server_name  img.juo.com;
#                access_log   /var/log/nginx/i.juo.access.log;
#                error_log    /var/log/nginx/i.juo.error.log;
	server_name_in_redirect off;
	location / {
		root /var/local/www/fastdfs/data;
		add_header Access-Control-Allow-Origin *;
		add_header Access-Control-Allow-Headers X-Requested-With;
		add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
	}

	location /group1/M00/ {
		root /var/local/www/fastdfs/data;
		ngx_fastdfs_module;
	}
}
```
