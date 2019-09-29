# Linux软件安装

## RPM安装命令

一、安装：rpm –ivh 包名

二、  升级：rpm –Uvh 包名

三、  删除：rpm –e 包名

四、  查询：rpm –q 包名

五、  校验：rpm –Vf 包名

## yum光盘安装

yum源文件：cd  /etc/yum.repos.d

光盘yum源文件打击：

1. mv CentOS-Base.repo CentOS-Base.repo.bak

2. vi CentOS-Media.repo 改baseurl=file:///mnt/cdrom和enabled=1

命令：

1. yum list：搜寻所有可安装的包

2. yum search 关键字：关键字搜寻

3. yum –y install 包名：安装

4. yum –y update 包名：升级

5. yum –y remove 包名：卸载

   *(ps：httpd安装顺序：httpd-tools，httpd，httpd-manual，httpd-devel；卸载也是安装这个顺序)*

   *服务器使用最小化安装，需要什么软件安装什么，尽量不卸载*

6. yum grouplist：列出所有可用的软件组列表

7. yum groupinstall 软件组名：安装指定软件组

8. yum groupremove 软件组名：卸载指定软件组

注：软件组名必须是英文（LANG=en_US切换成英文命令）LANG=zh_CN.utf8

## 源码包安装

1. 下载文件并解压缩，cd 进入文件目录

2. 检测环境、定义功能选项：./configure

   指定安装位置：./configure –prefix=/usr/local/apache2

3. 编译：make

   编译报错时执行：make clean

4. 编译安装：make install

5. 启动：/usr/local/apache2/bin/apachectl start