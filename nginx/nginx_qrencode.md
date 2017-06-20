# 如何利用nginx+lua生成二维码

## 1. 目标
> 利用nginx做一个简单高效的二维码生成API。

## 2. 环境
### 2.1 操作系统
```
#cat /etc/redhat-release
CentOS Linux release 7.2.1511 (Core)
```
### 2.2 依赖

+ libpng 1.6.29
+ libqrencode 3.2.0
+ qrencode(libqrencode lua包装代码)
+ nginx 1.10

## 3 安装libpng
1) 下载libpng
```
#mkdir -p ~/install
#cd ~/install
#wget http://download.sourceforge.net/libpng/libpng-1.6.29.tar.gz
#tar xvfz libpng-1.6.29.tar.gz
```

2) 编译安装
```
#cd libpng-1.6.29
#./configure
#make && make install
```

3) 验证
```
#find / -name libpng.so
/usr/local/lib/libpng.so
```

4) 设置LD路径
```
#echo "/usr/local/lib" > /etc/ld.so.conf.d/usr_local_lib.conf
#ldconfig

```

## 3 安装libqrencode
1) 下载libqrencode
```
#mkdir -p ~/install
#cd ~/install
#wget https://fukuchi.org/works/qrencode/qrencode-3.4.4.tar.gz
#tar xvfz qrencode-3.4.4.tar.gz
```

2) 编译安装
```
#cd qrencode-3.4.4
#./configure
#make && make install
```

3) 验证
```
#find / -name libqrencode.so
/usr/local/lib/libqrencode.so
```

4) 设置LD路径
```
#echo "/usr/local/lib" > /etc/ld.so.conf.d/usr_local_lib.conf
#ldconfig

```

## 4 安装qrencode
1) 下载qrencode
```
#mkdir -p ~/install
#cd ~/install
#git clone https://github.com/harrisj/qrencoder.git
```

2) 编译
```
#cd qrencode
#gcc -fPIC -shared -o qrencode.so -lpng -lqrencode -I/usr/local/include/luajit-2.0 qrencode.c 

```

3) 安装
```
#cp qrencode.so /usr/local/lib/lua/5.1/
#ls -ltr /usr/local/lib/lua/5.1/
```

4) 验证
```
#lua test/test.lua
...
```

## 5 nginx
1) 配置nginx.conf
```
        location /qrcode {
            content_by_lua_block {
                local qr = require("qrencode")
                local args = ngx.req.get_uri_args()
                local text = args.text
        
                if text == nil or text== "" then
                    ngx.say('need a text param')
                    ngx.exit(404)
                end
        
                ngx.say(qr {
                    text=text,
                    size=4,
                    margin=2,
                    symversion=0,
                    dpi=78,
                    casesensitive=true,
                    foreground="000000",
                    background="3FAF6F"
                })
            }
        }
```

2) 测试
```
#curl 'http://127.0.0.1/qrcode?text=http://orangleliu.info'
显示二维码
```

3) 页面测试
qrcode.html
```
<html>
<head>
	<title>qrcode</title>
</head>
<body>
qrcode: <br/>
<img height="200" width="200" src="/qrcode?text=https://zb.51vv.com/weixin/user/663235.htm#roomId=140081"></img>
</body>
</html>
```
