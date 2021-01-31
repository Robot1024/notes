1.把所有遇到的问题和用的命令都记录了

------

当容器无法运行 systemctl  这时候需要使用特权启动**--privileged=true**，

docker run -itd **--privileged=true** -p 3022:22 -p 22122:22122 -p 23000:23000 -p 8888:8888  --name FastDFS centos:7 **/usr/sbin/init**

------

**安装ssh**

yum install -y openssl openssh-server

vim   /etc/ssh/sshd_config

将:

PermitRootLogin  yes

RSAAuthentication yes 

PubkeyAuthentication  yes

启动ssh的服务：

```
systemctl start sshd.service
```

设置开机自动启动ssh服务

```
systemctl enable sshd.service
```

------

**storage 启动失败**

connect to address 10.211.55.3  23000  ,网上都说是防火墙没关，

检查docker 容器的防火墙根本没有，一直在容器里找原因。

最后问题定位，主机重启过，主机防火墙开了。坑啊，再遇到就要想到不是容器的，就是主机的。

------

**root 密码无意别修改成未知密码**

只有百度经验讲的详细：

https://jingyan.baidu.com/article/f96699bb7adf11c94f3c1b4b.html

------

**使用nginx 1.15.6  可能会有问题，照网上做的**

一直这个问题：but it could not be found where specified (LUAJIT_LIB=/usr/local/lib, LUAJIT

这个是nginx 版本问题导致的，我装的nginx-1.15.6 ，换成nginx-1.4.6 就没了，也可能是前面其他地方操作有问题。

------

**2.安装libfastcommon基础库**

```
mkdir /root/fastdfs
cd /root/fastdfs
git clone https://github.com/happyfish100/libfastcommon.git --depth 1 cd libfastcommon/
./make.sh && ./make.sh install
```

------

**3.安装FastDFS**

```
cd /root/fastdfs
wget https://github.com/happyfish100/fastdfs/archive/V5.11.tar.gz tar -zxvf V5.11.tar.gz
cd fastdfs-5.11
./make.sh && ./make.sh install

#配置文件准备
cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf 
cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf 
cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf 
cp /root/fastdfs/fastdfs-5.11/conf/http.conf /etc/fdfs 
cp /root/fastdfs/fastdfs-5.11/conf/mime.types /etc/fdfs
```

```
vim /etc/fdfs/tracker.conf 
#需要修改的内容如下 
port=22122 
base_path=/home/fastdfs
```

```
vim /etc/fdfs/storage.conf
#需要修改的内容如下
port=23000
base_path=/home/fastdfs # 数据和日志文件存储根目录 
store_path0=/home/fastdfs # 第一个存储目录 
tracker_server=192.168.211.136:22122
# http访问文件的端口(默认8888,看情况修改,和nginx中保持一致) 
http.server_port=8888
```

**4.启动**

```
mkdir /home/fastdfs -p
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart 
查看所有运行的端口，如果没storage 没启动成功先去看日志，很有可能是防火墙的原因
netstat -ntlp
```

**5.测试上传**

```
vim /etc/fdfs/client.conf #需要修改的内容如下 base_path=/home/fastdfs

#tracker服务器IP和端口
tracker_server=192.168.211.136:22122
#保存后测试,返回ID表示成功 如:group1/M00/00/00/xxx.png 
/usr/bin/fdfs_upload_file  /etc/fdfs/client.conf  /root/fastdfs/1.png group1/M00/00/00/wKjTiF7h5EWASb5aAACGZa9JdFo611.png
```

**5.安装fastdfs-nginx-module**

```
cd /root/fastdfs
wget https://github.com/happyfish100/fastdfs-nginx-module/archive/V1.20.tar.gz 
解压
tar -xvf V1.20.tar.gz
cd fastdfs-nginx-module-1.20/src
vim config
修改第5行 和15行 修改成
ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/" 
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
```

```
cp mod_fastdfs.conf /etc/fdfs/
```

```
vim /etc/fdfs/mod_fastdfs.conf 
#需要修改的内容如下 
tracker_server=192.168.211.136:22122
url_have_group_name=true 
store_path0=/home/fastdfs
```

```
mkdir -p /var/temp/nginx/client
```

------

**7.安装nginx**

现在还能安装nginx，遇到问题有说要先安装LuaJit，要不会出现**no file package.perload['restyfastdfs']**,

后来竹导说是centOS7.9 太高，之后换成 centOS7.3 。一直认为restyfastdfs 是fastdfs model 模块nginx 没有加载成功，最后问题是restyfastdfs.lua 文件，在csdn 上下载一下放到其中一个目录就好。

![image-20210131012406408](https://tva1.sinaimg.cn/large/008eGmZEly1gn68eg8dg7j31mo0e04qp.jpg)

------

1.安装GraphicsMagick

```
wget http://ftp.icm.edu.pl/pub/unix/graphics
wget http://ftp.icm.edu.pl/pub/unix/graphics/GraphicsMagick/1.3/GraphicsMagick-1.3.18.tar.gz

tar  -xvf   GraphicsMagick-1.3.18.tar.gz
cd GraphicsMagick-1.3.18

./configure      -enable-shared -enable-lzw -without-perl -with-modules
make 
make install

gm version 出现下面的信息
```

**之前加 -enable-shared -enable-lzw -without-perl -with-modules 都会编译失败**

失败报错是：

**如果configure提示“configure: error: libltdl is required for modules build”**

yum install libtool-ltdl libtool-ltdl-devel  运行这个，然编译就可以过了，这也可能是之前埋下的坑把，之前只用

./configure 编译，然后make  make install

------

```
GraphicsMagick 1.3.18 2013-03-10 Q8 http://www.GraphicsMagick.org/
Copyright (C) 2002-2013 GraphicsMagick Group.
Additional copyrights and licenses apply to this software.
See http://www.GraphicsMagick.org/www/Copyright.html for details.

Feature Support:
  Thread Safe              yes
  Large Files (> 32 bit)   yes
  Large Memory (> 32 bit)  yes
  BZIP                     no
  DPS                      no
  FlashPix                 no
  FreeType                 no
  Ghostscript (Library)    no
  JBIG                     no
  JPEG-2000                no
  JPEG                     no
  Little CMS               no
  Loadable Modules         no
  OpenMP                   yes (201107)
  PNG                      no
  TIFF                     no
  TRIO                     no
  UMEM                     no
  WMF                      no
  X11                      no
  XML                      no
  ZLIB                     yes

Host type: x86_64-unknown-linux-gnu

Configured using the command:
  ./configure 

Final Build Parameters:
  CC       = gcc -std=gnu99
  CFLAGS   = -fopenmp -g -O2 -Wall -pthread
  CPPFLAGS = 
  CXX      = g++
  CXXFLAGS = -pthread
  LDFLAGS  = 
  LIBS     = -lz -lm -lgomp -lpthread
```

如果 png  jpg 都是no 需要添加支持 

```
yum install -y libpng-devel libpng
yum install -y  libjpeg-devel libjpeg
freetype 支持
yum -y install libjpeg libjpeg-devel libpng libpng-devel giflib giflib-devel freetype freetype-devel
```

```
./configure     -enable-shared -enable-lzw -without-perl -with-modules
make 
make install
```

------

**2.安装LuaJIT**

```
安装LuaJIT
下载  
wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz  
tar -xvf   LuaJIT-2.0.4.tar.gz  
cd  LuaJIT-2.0.4

别忘记了
make   
别忘记了
make install

忘记编译安装，然后编译安装nginx 可能会导致问题

export LUAJIT_LIB=/usr/local/lib
#这个就2.0 不是软件版本号luajit-2.0.4,之前改过，导致报错
export LUAJIT_INC=/usr/local/include/luajit-2.0  
```

3.分别下载和解压 lua-nginx-module-0.8.10.tar.gz  和 ngx_devel_kit-0.2.18.tar.gz

```
这些是model 不需要make 和 make install
wget -O lua-nginx-module-0.8.10.tar.gz https://github.com/chaoslawful/lua-nginx-module/archive/v0.8.10.tar.gz 
 wget -O ngx_devel_kit-0.2.18.tar.gz https://github.com/simpl/ngx_devel_kit/archive/v0.2.18.tar.gz 
 tar -xvf  lua-nginx-module-0.8.10.tar.gz
 tar -xvf  ngx_devel_kit-0.2.18.tar.gz 
```

------

4.下载安装 nginx  

**configure 报这个错时候 ：checking for PCRE library ... not found 这个错误**
解决办法：运行   yum -y install openssl openssl-devel

**/configure: error: the HTTP cache module requires md5 functions
from OpenSSL library.** 

解决办法：yum -y install openssl openssl-devel

```
还有nginx 从新编译./configure  make 后要备份原nginx，新编译生成的nginx 是在objs 
文件夹里备份旧的nginx程序
cp /usr/local/nginx/sbin/nginx/usr/local/nginx/sbin/nginx.bak
把新的nginx程序覆盖旧的
cp objs/nginx /usr/local/nginx/sbin/nginx
测试新的nginx程序是否正确
/usr/local/nginx/sbin/nginx -t
```

**还有不要再make intall ，这回覆盖，重点**

error.log 测试是时候尽量开启，设置成info 级别。

------

lua 的代码调试：不会调试只能打log ，log 会输出到nginx  error.log 里

lua 输出log 的写法：

ngx.log(ngx.ERR, " string:", "no")  

------

配置fastdfs文件系统时，出现错误No such file or directory。
```lua
-- 创建缩略图
local image_sizes = {"80x80", "800x600", "40x40", "60x60"};  
```

_80x80.jpg   _80*80.jpg  被坑哭。

```
 wget http://nginx.org/download/nginx-1.4.2.tar.gz
 tar -xvf  nginx-1.4.2.tar.gz
 cd  nginx-1.4.2
 
 nginx 添加module:
 ./configure  --add-module=/root/fastdfs/fastdfs-nginx-module-1.20/src --add-module=/root/fastdfs/lua-nginx-module-0.8.10 --add-module=/root/fastdfs/ngx_devel_kit-0.2.18 


别忘记了
make   
别忘记了
make install

ln -s /usr/local/lib/libluajit-5.1.so.2 /usr/lib/libluajit-5.1.so.2

cat /etc/ld.so.conf
include ld.so.conf.d/*.conf

#使用echo 会有问题，双引号也会被添加到/etc/ld.so.conf 
#可以使用vim  然后，手动加入/usr/local/lib
echo “/usr/local/lib” >> /etc/ld.so.conf
# ldconfig 这是一个命令，使/etc/ld.so.conf,动态链接库修改后重建加载
ldconfig

/usr/local/nginx/sbin/nginx -V
```

vim /usr/local/nginx/conf/nginx.conf
```
server {
        listen       8888;
        server_name  localhost;
        #location ~/group[0-9]/ {
        #    ngx_fastdfs_module;
        #}
        location / {
             root /home/fastdfs/data/00/00;
        }
        location /test {
            default_type text/html;
            content_by_lua '
            ngx.say("hello world")
            ngx.log(ngx.ERR,"err err")
            ';
        }
         # fastdfs 缩略图生成
        location ~/group[0-9]/M00 {
                alias /home/fastdfs/data;

                set $image_root "/home/fastdfs/data";
                if ($uri ~ "/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/(.*)") {
                  set $image_dir "$image_root/$3/$4/";
                  set $image_name "$5";
                  set $file "$image_dir$image_name";
                }
    
                if (!-f $file) {
                  # 关闭lua代码缓存，方便调试lua脚本
                  #lua_code_cache off;
                  content_by_lua_file  conf/img_crop.lua;
                }
                ngx_fastdfs_module;
        }
    
    }
```

img_crop.lua  脚本 

```
-- 写入文件
local function writefile(filename, info)
    local wfile=io.open(filename, "w") --写入文件(w覆盖)
    assert(wfile)  --打开时验证是否出错		
    wfile:write(info)  --写入传入的内容
    wfile:close()  --调用结束后记得关闭
end

-- 检测路径是否目录
local function is_dir(sPath)
    if type(sPath) ~= "string" then return false end

    local response = os.execute( "cd " .. sPath )
    if response == 0 then
        return true
    end
    return false

end

-- 检测文件是否存在
local file_exists = function(name)
    local f=io.open(name,"r")
    if f~=nil then io.close(f) return true else return false end
end

local area = nil
local originalUri = ngx.var.uri;
local originalFile = ngx.var.file;
local index = string.find(ngx.var.uri, "([0-9]+)x([0-9]+)");  
if index then 
    originalUri = string.sub(ngx.var.uri, 0, index-2);  
    area = string.sub(ngx.var.uri, index);  
    index = string.find(area, "([.])");  
    area = string.sub(area, 0, index-1);  

    local index = string.find(originalFile, "([0-9]+)x([0-9]+)");  
    originalFile = string.sub(originalFile, 0, index-2)

end

-- check original file
if not file_exists(originalFile) then
    local fileid = string.sub(originalUri, 2);
    -- main
    local fastdfs = require('restyfastdfs')
    local fdfs = fastdfs:new()
    fdfs:set_tracker("192.168.211.136", 22122)
    fdfs:set_timeout(1000)
    fdfs:set_tracker_keepalive(0, 100)
    fdfs:set_storage_keepalive(0, 100)
    local data = fdfs:do_download(fileid)
    if data then
       -- check image dir
        if not is_dir(ngx.var.image_dir) then
            os.execute("mkdir -p " .. ngx.var.image_dir)
        end
        writefile(originalFile, data)
    end
end

-- 创建缩略图
local image_sizes = {"80x80", "800x600", "40x40", "60x60"};  
function table.contains(table, element)  
    for _, value in pairs(table) do  
        if value == element then
            return true  
        end  
    end  
    return false  
end 

if table.contains(image_sizes, area) then  
    local command = "gm convert " .. originalFile  .. " -thumbnail " .. area .. " -background gray -gravity center -extent " .. area .. " " .. ngx.var.file;  
    os.execute(command);  
end;

if file_exists(ngx.var.file) then
    --ngx.req.set_uri(ngx.var.uri, true);  
    ngx.exec(ngx.var.uri)
else
    ngx.exit(404)
end
```

改变目录的权限

```
chmod -R 777 /home/fastdfs/data/
```

启动nginx 测试 

```
http://192.168.211.136:8888/test
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /root/fastdfs/1.png
将返回的路径使用 nginx 访问 我的访问路径是 
http://192.168.211.136:8888/group1/M00/00/00/wKjTiF8HDKSANWomAACGZa9JdFo395.png
http://192.168.211.136:8888/wKjTiF8HDKSANWomAACGZa9JdFo395.png
http://192.168.211.136:8888/wKjTiF8HDKSANWomAACGZa9JdFo395.png_300x200.png
```

