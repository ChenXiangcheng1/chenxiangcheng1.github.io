---
title: Nginx
date: 2024-03-18
update: 2024-03-18

tags: 
categories:
  - 服务中间件
  - Web中间件
keywords: Nginx
description: 关于Nginx的笔记,涉及Nginx简介、Nginx的使用
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_08_26_56.png
---

# Nginx1.25.2

一个HTTP和反向代理服务器，DNS和负载均衡可以交给Nginx或Docker处理

[官网](https://nginx.org/)	|	[其他人的Blog(已吸收)](https://cloud.tencent.com/developer/article/1581988)



## 命令

| 命令(在nginx.exe目录上执行)                                  | 释义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| nginx -h                                                     | 帮助                                                         |
| `start nginx -p "$NGINX_HOME"`(UNIX风格SHELL)<br />`start nginx -p "%NGINX_HOME%"`(Windows) | 启动命令，默认工作目录为当前目录                             |
| nginx -p "$NGINX_HOME" -t                                    | 测试配置文件                                                 |
| nginx -s [stop\|quit\|reload\|reopen]                        | stop立即停止<br />quit优雅停止<br />reload重新加载配置，热加载<br />reopen重新打开日志文件 |



## 正向代理和反向代理

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_08_23.png" alt="img" style="zoom:67%;" />

<table>
    <tr>
        <td><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_08_24.png" alt="img" style="zoom:100%;" /></td>
        <td><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_08_26_56.png" alt="img" style="zoom:100%;" /></td>
    </tr>
    <tr>
    	<td>正向代理即是客户端代理, 代理客户端, 服务端不知道实际发起请求的客户端信息</td>
        <td>反向代理即是服务端代理, 代理服务端, 客户端不知道实际提供服务的服务端信息</td>
    </tr>
    <tr>
    	<td>正向代理的应用场景：缓存、授权、代理记录用户访问记录，对外隐藏用户信息</td>
        <td>反向代理的应用场景：保证内网安全(公网访问地址设为反向代理服务器，Web服务器在内网隐藏)、负载均衡、动静分离，动态资源访问应用服务器，静态资源访问静态资源服务器和OSS等，缓存长时间不更新的资源、配置SSL证书启用HTTPS支持</td>
    </tr>
</table>
反向代理就是服务端代理，让Nginx代理服务器接收用户请求，Nginx根据域名即服务转发到内网服务器，通过location路径映射的proxy_pass指令实现。
upstream指令块实现负载均衡



## 配置

看配置文件：E:\Applications\Scoop\apps\nginx\current\conf\nginx.conf



## 面试题

**1 Nginx是如何实现高并发的？**

一个master主进程和多个worker工作进程，master进程负责收集、分发请求，worker进程负责处理请求。
Worker进程采用了AIO模型(利用了底层操作系统的I/O多路复用机制，通过epoll监听多个FD，当事件产生回调FD绑定的事件处理器)



# 未整理



```conf
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;

    server {
        listen       80;
        server_name  manage.atguigu.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

	    proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location / {
			    proxy_pass http://127.0.0.1:8081;
			    proxy_connect_timeout 600;
			    proxy_read_timeout 600;
        }
        
    }

    server {
        listen       80;
        server_name  image.atguigu.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

		proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location / {
	    proxy_pass http://192.168.220.119:80;
	    proxy_connect_timeout 600;
	    proxy_read_timeout 600;
        }
        
    }

# 以下是自己的配置
    server {
    		# 匹配的域名和端口号
    		# 端口号
        listen       80;
        # 服务名
        server_name  manager.chinasofti.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

		proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location / {
      		# 反向代理转发转发
			    proxy_pass http://127.0.0.1:8082;
			    proxy_connect_timeout 600;
			    proxy_read_timeout 600;
        }
        
    }
}

```





# FastDFS+Nginx 实现图片服务器

![1609719652880](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_08_28_15.png)



## Tracker 集群

编译 libfastcommon、FastDFS_v5.05.tar.gz（包括tracker和storage）

### 配置

配置：/etc/fdfs/tracker.conf

```conf
# is this config file disabled
# false for enabled
# true for disabled
disabled=false

# bind an address of this host
# empty for bind all addresses of this host
bind_addr=

// 配置tracker开启端口
# the tracker server port
port=22122

# connect timeout in seconds
# default value is 30s
connect_timeout=30

# network timeout in seconds
# default value is 30s
network_timeout=60

// 基础路径存储数据和日志
# the base path to store data and log files
# base_path=/home/yuqing/fastdfs
base_path=/home/fastdfs

# max concurrent connections this server supported
max_connections=256

# accept thread count
# default value is 1
# since V4.07
accept_threads=1

# work thread count, should <= max_connections
# default value is 4
# since V2.00
work_threads=4

# the method of selecting group to upload files
# 0: round robin
# 1: specify group
# 2: load balance, select the max free space group to upload file
store_lookup=2

# which group to upload file
# when store_lookup set to 1, must set store_group to the group name
store_group=group2

# which storage server to upload file
# 0: round robin (default)
# 1: the first server order by ip address
# 2: the first server order by priority (the minimal)
store_server=0

# which path(means disk or mount point) of the storage server to upload file
# 0: round robin
# 2: load balance, select the max free space path to upload file
store_path=0

# which storage server to download file
# 0: round robin (default)
# 1: the source storage server which the current file uploaded to
download_server=0

# reserved storage space for system or other applications.
# if the free(available) space of any stoarge server in 
# a group <= reserved_storage_space, 
# no file can be uploaded to this group.
# bytes unit can be one of follows:
### G or g for gigabyte(GB)
### M or m for megabyte(MB)
### K or k for kilobyte(KB)
### no unit for byte(B)
### XX.XX% as ratio such as reserved_storage_space = 10%
reserved_storage_space = 10%

#standard log level as syslog, case insensitive, value list:
### emerg for emergency
### alert
### crit for critical
### error
### warn for warning
### notice
### info
### debug
log_level=info

#unix group name to run this program, 
#not set (empty) means run by the group of current user
run_by_group=

#unix username to run this program,
#not set (empty) means run by current user
run_by_user=

# allow_hosts can ocur more than once, host can be hostname or ip address,
# "*" means match all ip addresses, can use range like this: 10.0.1.[1-15,20] or
# host[01-08,20-25].domain.com, for example:
# allow_hosts=10.0.1.[1-15,20]
# allow_hosts=host[01-08,20-25].domain.com
allow_hosts=*

# sync log buff to disk every interval seconds
# default value is 10 seconds
sync_log_buff_interval = 10

# check storage server alive interval seconds
check_active_interval = 120

# thread stack size, should >= 64KB
# default value is 64KB
thread_stack_size = 64KB

# auto adjust when the ip address of the storage server changed
# default value is true
storage_ip_changed_auto_adjust = true

# storage sync file max delay seconds
# default value is 86400 seconds (one day)
# since V2.00
storage_sync_file_max_delay = 86400

# the max time of storage sync a file
# default value is 300 seconds
# since V2.00
storage_sync_file_max_time = 300

# if use a trunk file to store several small files
# default value is false
# since V3.00
use_trunk_file = false 

# the min slot size, should <= 4KB
# default value is 256 bytes
# since V3.00
slot_min_size = 256

# the max slot size, should > slot_min_size
# store the upload file to trunk file when it's size <=  this value
# default value is 16MB
# since V3.00
slot_max_size = 16MB

# the trunk file size, should >= 4MB
# default value is 64MB
# since V3.00
trunk_file_size = 64MB

# if create trunk file advancely
# default value is false
# since V3.06
trunk_create_file_advance = false

# the time base to create trunk file
# the time format: HH:MM
# default value is 02:00
# since V3.06
trunk_create_file_time_base = 02:00

# the interval of create trunk file, unit: second
# default value is 38400 (one day)
# since V3.06
trunk_create_file_interval = 86400

# the threshold to create trunk file
# when the free trunk file size less than the threshold, will create 
# the trunk files
# default value is 0
# since V3.06
trunk_create_file_space_threshold = 20G

# if check trunk space occupying when loading trunk free spaces
# the occupied spaces will be ignored
# default value is false
# since V3.09
# NOTICE: set this parameter to true will slow the loading of trunk spaces 
# when startup. you should set this parameter to true when neccessary.
trunk_init_check_occupying = false

# if ignore storage_trunk.dat, reload from trunk binlog
# default value is false
# since V3.10
# set to true once for version upgrade when your version less than V3.10
trunk_init_reload_from_binlog = false

# the min interval for compressing the trunk binlog file
# unit: second
# default value is 0, 0 means never compress
# FastDFS compress the trunk binlog when trunk init and trunk destroy
# recommand to set this parameter to 86400 (one day)
# since V5.01
trunk_compress_binlog_min_interval = 0

# if use storage ID instead of IP address
# default value is false
# since V4.00
use_storage_id = false

# specify storage ids filename, can use relative or absolute path
# since V4.00
storage_ids_filename = storage_ids.conf

# id type of the storage server in the filename, values are:
## ip: the ip address of the storage server
## id: the server id of the storage server
# this paramter is valid only when use_storage_id set to true
# default value is ip
# since V4.03
id_type_in_filename = ip

# if store slave file use symbol link
# default value is false
# since V4.01
store_slave_file_use_link = false

# if rotate the error log every day
# default value is false
# since V4.02
rotate_error_log = false

# rotate error log time base, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
# default value is 00:00
# since V4.02
error_log_rotate_time=00:00

# rotate error log when the log file exceeds this size
# 0 means never rotates log file by log file size
# default value is 0
# since V4.02
rotate_error_log_size = 0

# keep days of the log files
# 0 means do not delete old log files
# default value is 0
log_file_keep_days = 0

# if use connection pool
# default value is false
# since V4.05
use_connection_pool = false

# connections whose the idle time exceeds this time will be closed
# unit: second
# default value is 3600
# since V4.05
connection_pool_max_idle_time = 3600

# HTTP port on this tracker server
http.server_port=8080

# check storage HTTP server alive interval seconds
# <= 0 for never check
# default value is 30
http.check_alive_interval=30

# check storage HTTP server alive type, values are:
#   tcp : connect to the storge server with HTTP port only, 
#        do not request and get response
#   http: storage check alive url must return http status 200
# default value is tcp
http.check_alive_type=tcp

# check storage HTTP server alive uri/url
# NOTE: storage embed HTTP server support uri: /status.html
http.check_alive_uri=/status.html
```

### 启动Tracker

[root@localhost fdfs]# /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart

多次restart，启动日志显示先停止2952进程（实际环境可能不是2952）再反复启动，如下图：11269

![1609724275120](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_08_28_27.png)

如果没有显示上图要确认原有进程是否正常停止。



## FastDFS--storage 集群

编译 libfastcommon、FastDFS_v5.05.tar.gz（包括tracker和storage）

配置文件：/etc/fdfs/storage.conf

```conf
# is this config file disabled
# false for enabled
# true for disabled
disabled=false

# the name of the group this storage server belongs to
#
# comment or remove this item for fetching from tracker server,
# in this case, use_storage_id must set to true in tracker.conf,
# and storage_ids.conf must be configed correctly.
group_name=group1

# bind an address of this host
# empty for bind all addresses of this host
bind_addr=

# if bind an address of this host when connect to other servers 
# (this ?torage server as a client)
# true for binding the address configed by above parameter: "bind_addr"
# false for binding any address of this host
client_bind=true

# the storage server port
port=23000

# connect timeout in seconds
# default value is 30s
connect_timeout=30

# network timeout in seconds
# default value is 30s
network_timeout=60

# heart beat interval in seconds
heart_beat_interval=30

# disk usage report interval in seconds
stat_report_interval=60

# the base path to store data and log files
# base_path=/home/yuqing/fastdfs
base_path=/home/fastdfs

# max concurrent connections the server supported
# default value is 256
# more max_connections means more memory will be used
max_connections=256

# the buff size to recv / send data
# this parameter must more than 8KB
# default value is 64KB
# since V2.00
buff_size = 256KB

# accept thread count
# default value is 1
# since V4.07
accept_threads=1

# work thread count, should <= max_connections
# work thread deal network io
# default value is 4
# since V2.00
work_threads=4

# if disk read / write separated
##  false for mixed read and write
##  true for separated read and write
# default value is true
# since V2.00
disk_rw_separated = true

# disk reader thread count per store base path
# for mixed read / write, this parameter can be 0
# default value is 1
# since V2.00
disk_reader_threads = 1

# disk writer thread count per store base path
# for mixed read / write, this parameter can be 0
# default value is 1
# since V2.00
disk_writer_threads = 1

# when no entry to sync, try read binlog again after X milliseconds
# must > 0, default value is 200ms
sync_wait_msec=50

# after sync a file, usleep milliseconds
# 0 for sync successively (never call usleep)
sync_interval=0

# storage sync start time of a day, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
sync_start_time=00:00

# storage sync end time of a day, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
sync_end_time=23:59

# write to the mark file after sync N files
# default value is 500
write_mark_file_freq=500

# path(disk or mount point) count, default value is 1
store_path_count=1

// 基础路径存储数据和日志
# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
# store_path0=/home/yuqing/fastdfs
store_path0=/home/fastdfs/fdfs_storage
#store_path1=/home/yuqing/fastdfs2

# subdir_count  * subdir_count directories will be auto created under each 
# store_path (disk), value can be 1 to 256, default value is 256
subdir_count_per_path=256

// 配置tracker_server的ip和端口
# tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
# tracker_server=192.168.209.121:22122
tracker_server=192.168.231.133:22122

#standard log level as syslog, case insensitive, value list:
### emerg for emergency
### alert
### crit for critical
### error
### warn for warning
### notice
### info
### debug
log_level=info

#unix group name to run this program, 
#not set (empty) means run by the group of current user
run_by_group=

#unix username to run this program,
#not set (empty) means run by current user
run_by_user=

# allow_hosts can ocur more than once, host can be hostname or ip address,
# "*" means match all ip addresses, can use range like this: 10.0.1.[1-15,20] or
# host[01-08,20-25].domain.com, for example:
# allow_hosts=10.0.1.[1-15,20]
# allow_hosts=host[01-08,20-25].domain.com
allow_hosts=*

# the mode of the files distributed to the data path
# 0: round robin(default)
# 1: random, distributted by hash code
file_distribute_path_mode=0

# valid when file_distribute_to_path is set to 0 (round robin), 
# when the written file count reaches this number, then rotate to next path
# default value is 100
file_distribute_rotate_count=100

# call fsync to disk when write big file
# 0: never call fsync
# other: call fsync when written bytes >= this bytes
# default value is 0 (never call fsync)
fsync_after_written_bytes=0

# sync log buff to disk every interval seconds
# must > 0, default value is 10 seconds
sync_log_buff_interval=10

# sync binlog buff / cache to disk every interval seconds
# default value is 60 seconds
sync_binlog_buff_interval=10

# sync storage stat info to disk every interval seconds
# default value is 300 seconds
sync_stat_file_interval=300

# thread stack size, should >= 512KB
# default value is 512KB
thread_stack_size=512KB

# the priority as a source server for uploading file.
# the lower this value, the higher its uploading priority.
# default value is 10
upload_priority=10

# the NIC alias prefix, such as eth in Linux, you can see it by ifconfig -a
# multi aliases split by comma. empty value means auto set by OS type
# default values is empty
if_alias_prefix=

# if check file duplicate, when set to true, use FastDHT to store file indexes
# 1 or yes: need check
# 0 or no: do not check
# default value is 0
check_file_duplicate=0

# file signature method for check file duplicate
## hash: four 32 bits hash code
## md5: MD5 signature
# default value is hash
# since V4.01
file_signature_method=hash

# namespace for storing file indexes (key-value pairs)
# this item must be set when check_file_duplicate is true / on
key_namespace=FastDFS

# set keep_alive to 1 to enable persistent connection with FastDHT servers
# default value is 0 (short connection)
keep_alive=0

# you can use "#include filename" (not include double quotes) directive to 
# load FastDHT server list, when the filename is a relative path such as 
# pure filename, the base path is the base path of current/this config file.
# must set FastDHT server list when check_file_duplicate is true / on
# please see INSTALL of FastDHT for detail
##include /home/yuqing/fastdht/conf/fdht_servers.conf

# if log to access log
# default value is false
# since V4.00
use_access_log = false

# if rotate the access log every day
# default value is false
# since V4.00
rotate_access_log = false

# rotate access log time base, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
# default value is 00:00
# since V4.00
access_log_rotate_time=00:00

# if rotate the error log every day
# default value is false
# since V4.02
rotate_error_log = false

# rotate error log time base, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
# default value is 00:00
# since V4.02
error_log_rotate_time=00:00

# rotate access log when the log file exceeds this size
# 0 means never rotates log file by log file size
# default value is 0
# since V4.02
rotate_access_log_size = 0

# rotate error log when the log file exceeds this size
# 0 means never rotates log file by log file size
# default value is 0
# since V4.02
rotate_error_log_size = 0

# keep days of the log files
# 0 means do not delete old log files
# default value is 0
log_file_keep_days = 0

# if skip the invalid record when sync file
# default value is false
# since V4.02
file_sync_skip_invalid_record=false

# if use connection pool
# default value is false
# since V4.05
use_connection_pool = false

# connections whose the idle time exceeds this time will be closed
# unit: second
# default value is 3600
# since V4.05
connection_pool_max_idle_time = 3600

# use the ip address of this storage server if domain_name is empty,
# else this domain name will ocur in the url redirected by the tracker server
http.domain_name=

# the port of the web server on this storage server
http.server_port=8888

```

### 启动

[root@localhost fdfs]# /usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart &

多次restart ，启动的日志显示先停止2071进程（实际环境可能不是2071）再启动，如下图：11350

![1609726302300](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_08_28_37.png)

如果没有显示上图要确认原有进程是否正常停止。

## fdfs_test 客户端 测试

FastDFS安装成功可通过/usr/bin/fdfs_test测试上传、下载等操作。

配置：/etc/fdfs/client.conf

base_path=/home/fastdfs
tracker_server=192.168.220.119:22122

测试上传： /usr/bin/fdfs_test /etc/fdfs/client.conf upload xxxxyyy

返回信息：
http://192.168.231.133/group1/M00/00/00/wKjnhV_r45yAD1HNAAAF0ekQv9Q16.conf
对应storage服务器上的
/home/fastdfs/fdfs_storage/data/00/00/wKjcd1lpCHmAFpyoAADDWkRy8JA815_big.log

由于现在还没有和nginx整合无法使用http下载。



## Nginx 192.168.220.119反向代理给fastdfs-nginx-module

在每个tracker上安装nginx，的主要目的是做负载均衡及实现高可用。

配置文件：/usr/local/nginx/conf/nginx.conf

```conf
server {
    # 匹配的域名和端口号
    # 端口号
    listen       80;
    server_name  192.168.220.119;

    location /group1/M00/{
        # 反向代理转发转发
        ngx_fastdfs_module;
    }
}
```

### 启动

/usr/local/nginx/sbin/nginx

会显示 ngx_http_fastdfs_set pid=14040

## fastdfs-nginx-module

配置：/usr/local/fastdfs-nginx-module/src/config

```txt
ngx_addon_name=ngx_http_fastdfs_module
HTTP_MODULES="$HTTP_MODULES ngx_http_fastdfs_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_fastdfs_module.c"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
CORE_LIBS="$CORE_LIBS -L/usr/lib -lfastcommon -lfdfsclient"
CFLAGS="$CFLAGS -D_FILE_OFFSET_BITS=64 -DFDFS_OUTPUT_CHUNK_SIZE='256*1024' -DFDFS_MOD_CONF_FILENAME='\"/etc/fdfs/mod_fastdfs.conf\"'"
```

配置：/etc/fdfs/mod_fastdfs.conf

```txt
# connect timeout in seconds
# default value is 30s
connect_timeout=2

# network recv and send timeout in seconds
# default value is 30s
network_timeout=30

// 基础路径存储数据和日志
# the base path to store log files
# base_path=/tmp
base_path=/home/fastdfs

# if load FastDFS parameters from tracker server
# since V1.12
# default value is false
load_fdfs_parameters_from_tracker=true

# storage sync file max delay seconds
# same as tracker.conf
# valid only when load_fdfs_parameters_from_tracker is false
# since V1.12
# default value is 86400 seconds (one day)
storage_sync_file_max_delay = 86400

# if use storage ID instead of IP address
# same as tracker.conf
# valid only when load_fdfs_parameters_from_tracker is false
# default value is false
# since V1.13
use_storage_id = false

# specify storage ids filename, can use relative or absolute path
# same as tracker.conf
# valid only when load_fdfs_parameters_from_tracker is false
# since V1.13
storage_ids_filename = storage_ids.conf

多个tracker配置多行
# FastDFS tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
# valid only when load_fdfs_parameters_from_tracker is true
# tracker_server=tracker:22122
tracker_server=192.168.231.133:22122

# the port of the local storage server
# the default value is 23000
storage_server_port=23000

# the group name of the local storage server
group_name=group1

#url中包含group名称
# if the url / uri including the group name
# set to false when uri like /M00/00/00/xxx
# set to true when uri like ${group_name}/M00/00/00/xxx, such as group1/M00/xxx
# default value is false
# url_have_group_name = false
url_have_group_name = true

# path(disk or mount point) count, default value is 1
# must same as storage.conf
store_path_count=1

#指定文件存储路径
# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
# must same as storage.conf
# store_path0=/home/yuqing/fastdfs
store_path0=/home/fastdfs/fdfs_storage
#store_path1=/home/yuqing/fastdfs1

# standard log level as syslog, case insensitive, value list:
### emerg for emergency
### alert
### crit for critical
### error
### warn for warning
### notice
### info
### debug
log_level=info

# set the log filename, such as /usr/local/apache2/logs/mod_fastdfs.log
# empty for output to stderr (apache and nginx error_log file)
log_filename=

# response mode when the file not exist in the local file system
## proxy: get the content from other storage server, then send to client
## redirect: redirect to the original storage server (HTTP Header is Location)
response_mode=proxy

# the NIC alias prefix, such as eth in Linux, you can see it by ifconfig -a
# multi aliases split by comma. empty value means auto set by OS type
# this paramter used to get all ip address of the local host
# default values is empty
if_alias_prefix=

# use "#include" directive to include HTTP config file
# NOTE: #include is an include directive, do NOT remove the # before include
#include http.conf


# if support flv
# default value is false
# since v1.15
flv_support = true

# flv file extension name
# default value is flv
# since v1.15
flv_extension = flv


# set the group count
# set to none zero to support multi-group
# set to 0  for single group only
# groups settings section as [group1], [group2], ..., [groupN]
# default value is 0
# since v1.14
group_count = 0

# group settings for group #1
# since v1.14
# when support multi-group, uncomment following section
#[group1]
#group_name=group1
#storage_server_port=23000
#store_path_count=2
#store_path0=/home/yuqing/fastdfs
#store_path1=/home/yuqing/fastdfs1

# group settings for group #2
# since v1.14
# when support multi-group, uncomment following section as neccessary
#[group2]
#group_name=group2
#storage_server_port=23000
#store_path_count=1
#store_path0=/home/yuqing/fastdfs
```



## Java客户端：FastFileStorageClient

```pom
<dependency>
    <groupId>com.github.tobato</groupId>
    <artifactId>fastdfs-client</artifactId>
    <fastdfs-client.version>1.26.6</fastdfs-client.version>
</dependency>
```



### 解决jmx重复注册bean的问题

```java
package com.chinasofti.manager.config;

import com.github.tobato.fastdfs.FdfsClientConfig;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableMBeanExport;
import org.springframework.context.annotation.Import;
import org.springframework.jmx.support.RegistrationPolicy;

/**
 * @date 2021/1/4 @Created by cxc
 */
@Configuration
@Import(FdfsClientConfig.class)
// 解决jmx重复注册bean的问题
@EnableMBeanExport(registration = RegistrationPolicy.IGNORE_EXISTING)
public class FastDFSConfig {
    // 导入依赖组件
}

```



# 关于课程项目

## 开机启动了

tracker  fdfs_storaged  nginx

/usr/local/nginx/sbin/nginx

但是好像有问题

## 自己需要启动的

tomcat zookeeper 

## 前端控件

ztree、WebUploader、Ueditor、

Datatables：分页



# 架构



![1610588109415](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_02_09.png)

![1610588469196](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_02_08.png)

![1610587714744](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_02_07.png)





