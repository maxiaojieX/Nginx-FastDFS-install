# FastDFS with Nginx 

#### 了解FastDFS、Nginx
FastDFS是一个开源的分布式文件系统，她对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。
FastDFS服务端有两个角色：跟踪器（tracker）和存储节点（storage）。跟踪器主要做调度工作，在访问上起负载均衡的作用。
Nginx是一个http服务可以独立提供http服务。可以做网页静态服务器。

#### 应用场景
FastDFS的HTTP服务较为简单，无法提供负载均衡等高性能的服务。

#### 安装教程 摘自(https://www.cnblogs.com/jym-sunshine/p/6397470.html)
befor{
	
	yum -y install gcc automake autoconf libtool make
	yum install gcc gcc-c++
	2.安装PCRE库 wget    https://netix.dl.sourceforge.net/project/pcre/pcre/8.40/pcre-8.40.tar.gz

	3.安装zlib库 wget http://zlib.net/zlib-1.2.11.tar.gz
	4.安装ssl wget https://www.openssl.org/source/openssl-1.0.1t.tar.gz

}
安装 FastDFS三步走
[root@localhost ~]# cd /usr/local/src
[root@localhost src]# ls
fastdfs-master-V5.05.zip         
fastdfs-nginx-module-master.zip  
libfastcommon-master.zip         
nginx-1.10.0.tar.gz              
ngx_cache_purge-2.3.tar.gz       
openssl-1.0.1t                   
2.1 安装 libfastcommon
解压 libfastcommon，命令：

[root@localhost src]# unzip libfastcommon-master.zip
[root@localhost src]# cd libfastcommon-master
[root@localhost libfastcommon-master]# ./make.sh
[root@localhost libfastcommon-master]# ./make.sh install
执行以上4步，安装完毕，即完成第一步。

2.2 安装 FastDFS
解压 libfastcommon，命令：

[root@localhost libfastcommon-master]# cd ..
[root@localhost src]# ls
fastdfs-master-V5.05.zip         openssl-1.0.1t.tar.gz
fastdfs-nginx-module-master.zip  pcre-8.39
libfastcommon-master             pcre-8.39.tar.gz
libfastcommon-master.zip         zlib-1.2.8
nginx-1.10.0.tar.gz              zlib-1.2.8.tar.gz
ngx_cache_purge-2.3.tar.gz       zlib-1.2.8.tar.gz.1
openssl-1.0.1t
[root@localhost src]# unzip fastdfs-master-V5.05.zip
[root@localhost src]# cd fastdfs-master-V5.05
-bash: cd: fastdfs-master-V5.05: 没有那个文件或目录
[root@localhost src]# ls
fastdfs-master                   openssl-1.0.1t
fastdfs-master-V5.05.zip         openssl-1.0.1t.tar.gz
fastdfs-nginx-module-master.zip  pcre-8.39
libfastcommon-master             pcre-8.39.tar.gz
libfastcommon-master.zip         zlib-1.2.8
nginx-1.10.0.tar.gz              zlib-1.2.8.tar.gz
ngx_cache_purge-2.3.tar.gz       zlib-1.2.8.tar.gz.1
[root@localhost src]# cd fastdfs-master
[root@localhost fastdfs-master]# ./make.sh
[root@localhost fastdfs-master]# ./make.sh install
执行以上几步，
显示：

[root@localhost fastdfs-master]# ./make.sh install
cc -Wall -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE -g -O -DDEBUG_FLAG        -c -o ../common/fdfs_global.o ../common/fdfs_global.c  -I../    common -I/usr/include/fastcommon
cc -Wall -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE -g -O -DDEBUG_FLAG        -c -o tracker_proto.o tracker_proto.c  -I../common -I/usr/  include/fastcommon
...
tracker_client.h storage_client.h storage_client1.h client_func.h       client_global.h fdfs_client.h /usr/include/fastdfs
if [ ! -f /etc/fdfs/client.conf.sample ]; then cp -f ../conf/       client.conf /etc/fdfs/client.conf.sample; fi
即安装完毕！

3 配置 Tracker 服务
上述安装成功后，在/etc/目录下会有一个fdfs的目录，进入它。会看到三个.sample后缀的文件，这是作者给我们的示例文件，我们需要把其中的tracker.conf.sample文件改为tracker.conf配置文件并修改它。看命令：

[root@localhost fastdfs-master]# cd /etc/fdfs
[root@localhost fdfs]# ls
client.conf.sample  storage.conf.sample  tracker.conf.sample
---
[root@localhost fdfs]# cp tracker.conf.sample tracker.conf
[root@localhost fdfs]# vim tracker.conf
打开tracker.conf文件，只需要找到你只需要该这两个参数就可以了。

# the base path to store data and log files
base_path=/data/fastdfs
# HTTP port on this tracker server
http.server_port=80
当然前提是你要有或先创建了/data/fastdfs目录。port=22122这个端口参数不建议修改，除非你已经占用它了。
修改完成保存并退出 vim ，这时候我们可以使用/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start来启动 Tracker服务，但是这个命令不够优雅，怎么做呢？使用ln -s 建立软链接：

ln -s /usr/bin/fdfs_trackerd /usr/local/bin
ln -s /usr/bin/stop.sh /usr/local/bin
ln -s /usr/bin/restart.sh /usr/local/bin
这时候我们就可以使用service fdfs_trackerd start来优雅地启动 Tracker服务了，是不是比刚才带目录的命令好记太多了（懒是社会生产力）。你也可以启动过服务看一下端口是否在监听，命令：

启动服务：service fdfs_trackerd start
查看监听：netstat -unltp|grep fdfs
启动命令如下：

[root@localhost fdfs]# mkdir /data
[root@localhost fdfs]# mkdir /data/fastdfs
[root@localhost fdfs]# /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start
看到22122端口正常被监听后，这时候就算 Tracker服务安装成功啦！

[root@localhost fdfs]# netstat -unltp | grep fdfs
tcp        0      0 0.0.0.0:22122           0.0.0.0:*
LISTEN      5895/fdfs_trackerd  
4、配置 Storage 服务
现在开始配置 Storage 服务，由于我这是单机器测试，你把 Storage 服务放在多台服务器也是可以的，它有 Group(组)的概念，同一组内服务器互备同步，这里不再演示。直接开始配置，依然是进入/etc/fdfs的目录操作，首先进入它。会看到三个.sample后缀的文件，我们需要把其中的storage.conf.sample文件改为storage.conf配置文件并修改它。还看命令：

cp storage.conf.sample storage.conf
vim storage.conf
指令操作：

[root@localhost fdfs]# cd /etc/fdfs
[root@localhost fdfs]# cp storage.conf.sample storage.conf
[root@localhost fdfs]# vim storage.conf
    
打开storage.conf文件后，找到这两个参数进行修改：

# the base path to store data and log files
base_path=/data/fastdfs/storage
# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
store_path0=/data/fastdfs/storage
#store_path1=/home/yuqing/fastdfs2
# tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
tracker_server=192.168.198.129:22122
当然你的/data/fastdfs目录下要有storage文件夹，没有就创建一个，不然会报错的，日志以及文件都会在这个下面，启动时候会自动生成许多文件夹。stroage的port=23000这个端口参数也不建议修改，默认就好，除非你已经占用它了。
修改完成保存并退出 vim ，这时候我们依然想优雅地启动 Storage服务，带目录的命令不够优雅，这里还是使用ln -s 建立软链接：

ln -s /usr/bin/fdfs_storaged /usr/local/bin
执行命令启动服务：

service fdfs_storaged start
出现了一个大大的error啦！！！要仔细看，错误提示是找不到文件夹，这就好办了嘛。创建一个文件夹再次启动看看。

[root@localhost fdfs]# /usr/bin/fdfs_storaged start
[2017-02-08 14:48:00] ERROR - file: shared_func.c, line: 968, /etc/fdfs/start is not a regular file
[2017-02-08 14:48:00] ERROR - file: process_ctrl.c, line: 230, load conf file "start" fail, ret code: 22
这次启动成功，没有错误了。查看一下监听：

netstat -unltp|grep fdfs
监听如下：

[root@localhost fdfs]# netstat -unltp|grep fdfs
tcp        0      0 0.0.0.0:22122           0.0.0.0:*               LISTEN      5895/fdfs_trackerd  
tcp        0      0 0.0.0.0:23000           0.0.0.0:*               LISTEN      6001/fdfs_storaged  
很好，22122 和 23000端口都在监听了，这个时候你去/data/fastdfs/storage文件夹下看的话，会出现一大堆文件夹，而且进去还有一大堆，哈哈，这就是存放文件的啦！

5服务监听测试
我们安装配置并启动了 Tracker 和 Storage 服务，也没有报错了。那他俩是不是在通信呢？我们可以监视一下：

/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
看到我横线处ACTIVE这样就 ok 啦！

    [root@localhost fdfs]# /usr/bin/fdfs_monitor /etc/fdfs/storage.conf
[2017-02-08 14:51:16] DEBUG - base_path=/data/fastdfs/storage, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=600s, use_storage_id=0, storage server id count: 0

server_count=1, server_index=0

tracker server is 192.168.160.130:22122

group count: 1

Group 1:
group name = group1
disk total space = 17878 MB
disk free space = 10198 MB
trunk free space = 0 MB
storage server count = 1
active server count = 1
storage server port = 23000
storage HTTP port = 8888
store path count = 1
subdir count per path = 256
current write server index = 0
current trunk file id = 0

Storage 1:
    id = 192.168.160.130
    ip_addr = 192.168.160.130 (localhost.localdomain)  ACTIVE
    http domain = 
    version = 5.08
    join time = 2017-02-08 14:49:53
    up time = 2017-02-08 14:49:53
其实这个时候你就可以进行上传测试了，但可能会下载不了，

6 安装 Nginx 和 fastdfs-nginx-module
解压 fastdfs-nginx-module ，记着这时候别用tar解压了，因为是 .zip 文件，正确命令：

unzip master.zip
1）配置 nginx 安装，加入fastdfs-nginx-module模块
这是和普通 Nginx 安装不一样的地方，因为加载了模块。
若nginx未安装，根据相关教程进行安装，若已安装，查看已安装的nginx的版本，找到源码的存放位置，然后配置fastdfs-nginx-module模块

[root@localhost src]# cd /home/roo/下载/nginx-1.11.6
[root@localhost nginx-1.11.6]# ./configure --add-module=/usr/local/src/fastdfs-nginx-module-master/src/
checking for OS
 + Linux 3.10.0-327.36.3.el7.x86_64 x86_64
checking for C compiler ... found
[root@localhost nginx-1.11.6]# make && make install
这时候，我们可以看一下 Nginx 下安装成功的版本及模块，命令：

/usr/local/nginx/sbin/nginx -V
配置 fastdfs-nginx-module 和 Nginx
1.配置mod-fastdfs.conf，并拷贝到/etc/fdfs文件目录下。
cd /usr/local/src/fastdfs-nginx-module-master/src/
vim mod_fastdfs.conf
cp mod_fastdfs.conf /etc/fdfs
修改mod-fastdfs.conf配置只需要修改我标注的这三个地方就行了，其他不需要也不建议改变。

# FastDFS tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
# valid only when load_fdfs_parameters_from_tracker is true
tracker_server=192.168.198.129:22122
# if the url / uri including the group name
# set to false when uri like /M00/00/00/xxx
# set to true when uri like ${group_name}/M00/00/00/xxx, such as group1/M00/xxx
# default value is false
url_have_group_name = true
    # store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
# must same as storage.conf
store_path0=/data/fastdfs/storage
#store_path1=/home/yuqing/fastdfs1

接着我们需要把fastdfs-5.05下面的配置中还没有存在/etc/fdfs中的拷贝进去

cd /usr/local/src/fastdfs-5.05/conf
cp anti-steal.jpg http.conf mime.types /etc/fdfs/
2.配置 Nginx。编辑nginx.conf文件：
cd /usr/local/nginx/conf
vi nginx.conf
在配置文件中加入：

location /group1/M00 {
    root /data/fastdfs/storage/;
    ngx_fastdfs_module;
}
由于我们配置了group1/M00的访问，我们需要建立一个group1文件夹，并建立M00到data的软链接。

mkdir /data/fastdfs/storage/data/group1
ln -s /data/fastdfs/storage/data /data/fastdfs/storage/data/group1/M00
启动 Nginx ，会打印出fastdfs模块的pid，看看日志是否报错，正常不会报错的

/usr/local/nginx/sbin/nginx
打开浏览器，访问一下发现并不能访问，也并没有报错，但显示如下画面。糟糕了，怎么办？对了，我好像没关闭防火墙。

开放80端口访问权限。在iptables中加入重启就行，或者你直接关闭防火墙，本地测试环境可以这么干，但到线上万万不能关闭防火墙的。

vi /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
重启防火墙，使设置生效：

service iptables restart
再次刷新浏览器，可以看到如下画面，说明我们 Nginx 结合 fastdfs-nginx-module 模块安装并配置成功啦！

我最后说一下怎么在已经安装过 Nginx 的服务器上安装配置 fastdfs-nginx-module 模块？ 因为，一般我们线上服务器都是已经安装过 Nginx 的，所以这个时候，我们就直接进入 Nginx 的存放目录，进行配置后编译，就不需要执行最后安装make install这一步了，接着重启就行了。

上传测试
完成上面的步骤后，我们已经安装配置完成了全部工作，接下来就是测试了。因为执行文件全部在/usr/bin目录下，我们切换到这里，并新建一个test.txt文件，随便写一点什么，我写了This is a test file. by:mafly这句话在里边。然后测试上传：

cd /usr/bin
vim test.txt
fdfs_test /etc/fdfs/client.conf upload test.txt
很不幸，并没有成功，报错了。

ERROR - file: shared_func.c, line: 960, open file /etc/fdfs/client.conf fail, errno: 2, error info: No such file or directory
ERROR - file: ../client/client_func.c, line: 402, load conf file "/etc/fdfs/client.conf" fail, ret code: 2
一般什么事情第一次都不是很顺利，这很正常，通过错误提示我看到，好像没有找到client.conf这个文件，现在想起来的确没有配置这个文件，那我们现在去配置一下图中的两个参数：

cd /etc/fdfs
cp client.conf.sample client.conf
vim client.conf
怎么还依然报错阿？？？
解决办法：{
	pkill -9 fdfs

	/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf

	/usr/bin/fdfs_storaged /etc/fdfs/storage.conf
}

upload file fail, error no: 2, error info: No such file or directory
哈哈，你是不是测试上传命令中要上传的test.txt文件路径有问题，嗯，那我改一下命令：

/usr/bin/fdfs_test /etc/fdfs/client.conf upload /usr/bin/test.txt
成功啦！！！ 返回文件信息及上传后的文件 HTTP 地址，你打开浏览器访问一下试试