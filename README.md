# nas系统设置
## 1、进入系统修改密码：
1)修改root密码：passwd root  
2)修改ubuntu密码：passwd ubuntu  
## 2、安装docker
1）安装命令：apt update && apt install docker.io  
2）设置安装docker程序到外接U盘  
systemctl stop docker               # 停止 Docker 服务  
mkdir -p /mnt/sda1/docker           # 建立文件夹  
chmod 777 -R /mnt/sda1/docker           # 赋予权限  
vi /lib/systemd/system/docker.service   # 编辑配置文件  
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock  
插上一句 --graph="/mnt/sda1/docker" 变为如下:  
ExecStart=/usr/bin/dockerd --graph="/mnt/sda1/docker" -H fd:// --containerd=/run/containerd/containerd.sock  
systemctl daemon-reload  
systemctl start docker  
## 3、安装docker镜像
1）安装docker管理程序中文面版  
docker run --restart always -p 9000:8081 -d -v /var/run/docker.sock:/var/run/docker.sock -v /etc/docker/:/etc/docker/ wangbinxingkong/fast:latest  
安装完，访问http://NAS盒子IP:9000访问。  
2）安装DDNS-go  
docker run -d --restart=always --name ddns-go --net=host -v /mnt/sda1/ddns:/root jeessy/ddns-go -l :9877 -f 600  
安装完，访问http://NAS盒子IP:9877配置一下 。  
3）安装容器魔方  
docker run -d --name=wxedge --restart=always --privileged --net=host  --tmpfs /run --tmpfs /tmp -v 磁盘路径:/storage:rw  registry.hub.docker.com/onething1/wxedge  
“磁盘路径”是磁盘的目录，请根据自己实际磁盘目录更改，必须挂载到容器内的/storage目录，推荐磁盘是ext4文件系统，至少需要50G以上的空间，建议是固态硬盘。  
安装完，访问http://NAS盒子IP:18888激活绑定。  
## 4、输入用户名和密码后显示首页设置
vi /etc/nginx/sites-available/default  
在location的位置添加两句  
auth_basic "admin";  
auth_basic_user_file /etc/nginx/passwords.list;  
## 5、修改admin密码  
echo -n 'admin:' | tee /etc/nginx/passwords.list  
openssl passwd -apr1 | tee -a /etc/nginx/passwords.list  
nginx -s reload  
## 6、阿里云docker镜像加速器
sudo mkdir -p /etc/docker  
sudo tee /etc/docker/daemon.json <<-'EOF'  
{  
"registry-mirrors": ["https://z0nkeoyx.mirror.aliyuncs.com"] 
}  
EOF    
sudo systemctl daemon-reload  
sudo systemctl restart docker  
## 7、保存和重装后还原博客数据库
在 recoverbackup 重装系统之前, 一定要备份博客的数据库
数据库位置路径在 /var/www/html/blog/usr/623d62d5e5e96.db db 名称可能会不相同
操作如下:  
mkdir -p /mnt/sda1/blog-data  
chmod -R 777 /mnt/sda1/blog-data  
cp /var/www/html/blog/usr/*.db /mnt/sda1/blog-data/myblog.db  
把数据库文档移出来到U盘指定目录并重命名为 myblog.db  
下次重新安装博客时, 点击 "我准备好了, 开始下一步", 数据库文件路径输入 /mnt/sda1/blog-data/myblog.db 然后无需设置密码直接点击 "确认, 开始安装", 然后点击 "使用原数据" 这样就把数据还原了, 打开还是原来一样的所有文章。
注意: 如果你常常使用的是外网网址登陆博客, 请在后台设置里填写正确的外网网址。
否则首页将会造成js错乱而影响访问。（可用内网 IP 地址登陆去修改）
## 8、在线升级宝塔系统或网心云版系统
1、切换宝塔系统  
bash <(curl https://ecoo.top/upgrade.sh)  
2、切换网心云系统  
bash <(curl https://ecoo.top/mv23-wx.sh)  
## 9、盒子空间有限，简单几步把/var/www/html/转移到外接U盘或硬盘  
第1步  
查看外置硬盘路径df -h  
第2步  
给硬盘权限，建立html文件夹，把/var/www/html/目录内的文件复制到外置硬盘html目录内  
cd /mnt/sda1  
mkdir html  
chmod -R 777 /mnt/sda1/  
cp -r /var/www/html/* /mnt/sda1/html  
查看文件是否复制到硬盘  
ls /mnt/sda1/html  
第3步  
删除原来/var/www/内的文件  
rm -r /var/www/*  
第4步  
把/var/www/文件链接到外置硬盘并给权限  
ln -s /mnt/sda1/html /var/www  
chmod -R 777 /mnt/sda1/html  
软链接完成。
## 未完待续。
