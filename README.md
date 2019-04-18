## 下载用到的包

下载`libfastcommon`、`fastdfs`、`ngnix`、`fastdfs-nginx-module`： 

```
wget https://github.com/chenjie222/fastfs_install/blob/master/package/V1.0.39.tar.gz
wget https://github.com/chenjie222/fastfs_install/blob/master/package/V1.20.tar.gz
wget https://github.com/chenjie222/fastfs_install/blob/master/package/V5.11.tar.gz
wget https://github.com/chenjie222/fastfs_install/blob/master/package/nginx-1.14.1.tar.gz
```

## 安装 `libfastcommon`

```
tar -xvf V1.0.39.tar.gz
cd libfastcommon-1.0.39
./make.sh
./make.sh install 
```

## 安装 `fastdfs`

```
tar -xvf V5.11.tar.gz
cd fastdfs-5.11
./make.sh
./make.sh install
```

安装好后，程序是在`/usr/bin`目录下： 

```
/usr/bin/fdfs_trackerd
```

而配置文件是在`/etc/fdfs`目录下 

但是这些配置文件是不全的，而且都是模板，所以需要从`fastdfs`包中拷贝过来，并修改配置： 

```
cp /root/fastdfs-5.11/conf/* /etc/fdfs
```

这里我的配置文件都在`conf`里，可以直接拿来用

## 修改配置

**vi /etc/fdfs/tracker.conf：**

以下是我的配置，如果安装可以改成你需要的 

```
# the tracker server port
port=22122

# the base path to store data and log files
base_path=/fastdfs/tracker

# HTTP port on this tracker server
http.server_port=9270
```

**vi /etc/fdfs/storage.conf：** 

```
# storage所属的组
group_name=group1

# the storage server port
port=23000

# the base path to store data and log files
base_path=/fastdfs/storage/data1

# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
store_path0=/fastdfs/storage/data1
store_path1=/fastdfs/storage/data2
store_path2=/fastdfs/storage/data3

# tracker服务器，虽然是同一台机器上，但是不能写127.0.0.1。这项配置可以出现一次或多次
tracker_server=10.110.156.60:22122   

# the port of the web server on this storage server
http.server_port=http.server_port=6667
```

 **vi /etc/fdfs/client.conf：** 

```
# the base path to store log files
base_path=/fastdfs/tracker
# tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
tracker_server=10.110.156.60:22122 
#HTTP settings
http.tracker_server_port=6666
```

**vi /etc/fdfs/mod_fastdfs.conf：** 

```
# the base path to store log files
base_path=/tmp

# FastDFS tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
# valid only when load_fdfs_parameters_from_tracker is true
tracker_server=10.110.156.60:22122

# the port of the local storage server
# the default value is 23000
storage_server_port=23000

# the group name of the local storage server
group_name=group1

# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
# must same as storage.conf
store_path0=/fastdfs/storage/data1
store_path1=/fastdfs/storage/data2
store_path2=/fastdfs/storage/data3
```

## 启动`tracker`和`storage`：

```
# 启动
fdfs_trackerd /etc/fdfs/tracker.conf start
fdfs_storaged /etc/fdfs/storage.conf start
# 查看日志
tail -f  /fastdfs/tracker/logs/strackerd.log
tail -f /fastdfs/storage/data1/logs/storaged.log
# 如果日志显示有错误信息，需要根据信息来查找错误原因
```

## 用`fdfs_test`测试上传

```
[root@oceanpacific logs]# fdfs_test /etc/fdfs/client.conf upload /root/fastdfs-5.11/conf/anti-steal.jpg
This is FastDFS client test program v5.11

Copyright (C) 2008, Happy Fish / YuQing

FastDFS may be copied only under the terms of the GNU General
Public License V3, which may be found in the FastDFS source kit.
Please visit the FastDFS Home Page http://www.csource.org/ 
for more detail.

[2019-04-18 14:35:56] DEBUG - base_path=/fastdfs/tracker, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group: 
	server 1. group_name=, ip_addr=10.110.156.60, port=23000

group_name=group1, ip_addr=10.110.156.60, port=23000
storage_upload_by_filename
group_name=group1, remote_filename=M01/01/6C/Cm6cPFy4GsyAAm2pAABdrSqbHGQ061.jpg
source ip address: 10.110.156.60
file timestamp=2019-04-18 14:35:56
file size=23981
file crc32=714808420
example file url: http://10.110.156.60:6666/group1/M01/01/6C/Cm6cPFy4GsyAAm2pAABdrSqbHGQ061.jpg
storage_upload_slave_by_filename
group_name=group1, remote_filename=M01/01/6C/Cm6cPFy4GsyAAm2pAABdrSqbHGQ061_big.jpg
source ip address: 10.110.156.60
file timestamp=2019-04-18 14:35:56
file size=23981
file crc32=714808420
example file url: http://10.110.156.60:6666/group1/M01/01/6C/Cm6cPFy4GsyAAm2pAABdrSqbHGQ061_big.jpg

```

## 安装nginx

```
tar -xvf nginx-1.14.1.tar.gz
nginx-1.14.1.tar.gz
./configure
make && make install
/usr/local/nginx/sbin/nginx -V #显示版本号，代表安装成功
```



## 安装`fastdfs-nginx-module`

```
tar -xvf V1.20.tar.gz
cd fastdfs-nginx-module-1.20/
cd .. && cd nginx-1.14.1
./configure --add-module=../fastdfs-nginx-module-1.20/src/ #这个时候会报错
make && make install
```

修改fastdfs-nginx-module-1.20/src/config

```
vim fastdfs-nginx-module-1.20/src/config
ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
```

再次执行安装fastdfs-nginx-module

确认是否安装ok

/usr/local/nginx/sbin/nginx -V

```
[root@oceanpacific ~]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.14.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC) 
configure arguments: --prefix=/usr/local/nginx --add-module=/root/fastdfs-nginx-module-1.20/src
```

配置nginx.conf

vim /usr/local/nginx/conf/nginx.conf

```
添加	      
location ~/group[0-9]/M0[0-9] {
	        #alias /fastdfs/storage/data1/data;
          ngx_fastdfs_module;
	      }
```

启动nginx

```
/usr/local/nginx/sbin/nginx
```

访问刚才上传的图片

```
http://192.168.3.30:8090/group1/M01/01/6C/Cm6cPFy4GsyAAm2pAABdrSqbHGQ061_big.jpg
# 如果能看到图片则表示安装fastdfs-nginx-module成功
```

