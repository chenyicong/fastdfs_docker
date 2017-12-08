# fastdfs_docker
the docker image of fastdfs

注:该镜像是从 https://github.com/LionHeartFXCX/fastdfsOnDocker frok过来的，只是原先的storage镜像没有编译with-http_image_filter_module模块，导致nginx无法支持图片缩略图的动态生成，本镜像就是从原先的storage镜像昨启动的容器中，重新编译storage模块的nginx。加入with-http_image_filter_module模块，并commit到本地仓库，形成新的镜像。最后的镜像托管在aliyun的私有镜像仓库中,如果你有阿里云账号，可以登录后从这里下载:registry.cn-hangzhou.aliyuncs.com/chenyc-ak/fdfs_tracker:v32  registry.cn-hangzhou.aliyuncs.com/chenyc-ak/fdfs_storage:v32两个镜像资源。

一、fastdfs是一个开源的分布式文件系统，其主要分为tracker节点和storage节点，本镜像部署了storage节点，并在镜像内构建了一个nginx服务器，支持http服务。在docker的环境中，tracker节点部署在了与storage节点相同的服务器上。

二、镜像配置:

具体的镜像配置文件，参考fdfs文件夹里面的各个文件，其中各个配置文件的归属如下:
tracker.conf:        tracker的配置文件
storage.conf：       storage的配置文件
mod_fastdfs.conf:    nginx模块fastdfs-nginx-module的配置文件
nginx_conf文件夹:     里面含tracker  storage两个文件夹，分别用于为tracker、storage 提供http服务的nginx的配置文件
client.conf ：       client 的配置文件，主要用来跟tracker通信。。
storage_ids.conf:    fastdfs-nginx-module的配置文件,根据id来查找storage-ser

实际部署时,执行以下命令:
1、mkdir -p /home/ak_storage  /home/ak_storage/mod_fdfs_logs /home/ak_track /home/ak_client，并将fdfs_conf文件夹整体拷贝到/etc下面。
2、修改/etc/fdfs/storage.conf中tracker server的ip地址 <tracker_server> 为你环境中真实的tracker所在的地址；
   修改/etc/fdfs/nginx_conf/tracker/nginx.conf中nginx反向代理的storage ser的ip地址；
   修改/etc/fdfs/client.conftracker server的ip地址 <tracker_server> 为你环境中真实的tracker所在的地址。
   
三、启动镜像：

3.1、tracker 服务容器的启动过程：
docker run -itd --name tracker --net=host -v /etc/fdfs:/etc/fdfs -v /etc/fdfs/nginx_conf/tracker:/usr/local/nginx/conf -v /home/ak_tracker:/home/ak_tracker registry.alauda.cn/lionheart/fastdfs_tracker

docker exec tracker fdfs_trackerd /etc/fdfs/tracker.conf 
docker exec tracker /usr/local/nginx/sbin/nginx
------------------------------------------------------
3.2、storage 服务容器的启动过程：
docker run -itd --name storage --net=host -v /etc/fdfs:/etc/fdfs -v /etc/fdfs/nginx_conf/storage:/usr/local/nginx/conf -v /home/ak_storage:/home/ak_storage registry.alauda.cn/lionheart/fastdfs_storage

docker exec storage fdfs_storaged /etc/fdfs/storage.conf 
docker exec storage /usr/local/nginx/sbin/nginx

ln -s /home/ak_storage/data  /home/ak_storage/data/M00
------------------------------------------------------
3.3 测试容器启动是否ok：
docker run -itd --name client --net=host -v /etc/fdfs:/etc/fdfs -v /home/client:/home/client -v /home/ak_client:/home/ak_client registry.alauda.cn/lionheart/fastdfs_storage

docker exec client fdfs_test /etc/fdfs/client.conf upload /home/client/test.jpg  
-----------------------------------------------------
3.4、结果如下:
group_name=group1, ip_addr=12x.xx.x6.xx7, port=23000
storage_upload_by_filename
group_name=group1, remote_filename=M00/00/00/rBI9eloqaKeAOj6yAAQZg0M4szY298.jpg
source ip address: 172.18.61.122
file timestamp=2017-12-08 10:25:43
file size=268675
file crc32=1127789366
example file url: http://xxxxxx:8090/group1/M00/00/00/rBI9eloqaKeAOj6yAAQZg0M4szY298.jpg
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/rBI9eloqaKeAOj6yAAQZg0M4szY298_big.jpg
source ip address: 172.18.61.122
file timestamp=2017-12-08 10:25:43
file size=268675
file crc32=1127789366
example file url: http://xxxx:8090/group1/M00/00/00/rBI9eloqaKeAOj6yAAQZg0M4szY298_big.jpg

此时，你可以通过浏览器查看原图:http://xxxxx:8090/group1/M00/00/00/rBI9eloqaKeAOj6yAAQZg0M4szY298.jpg
             查看动态缩略图: http://xxxxx:8090/group1/M00/00/00/rBI9eloqaKeAOj6yAAQZg0M4szY298_20x20.jpg
                           http://xxxxx:8090/group1/M00/00/00/rBI9eloqaKeAOj6yAAQZg0M4szY298_50x40.jpg
                           ... 缩略图任意大小均支持

    
     
     

