---
title: linux下转DOC转PDF转SWF实现百度的文库预览功能
categories: "服务" #文章分類目錄 可以省略
date: 2017-07-21 10:24:47
tags: ["服务","功能","office在线阅读"]
---

<!-- ## linux下转DOC转PDF转SWF实现百度的文库预览功能 -->
![Alt text](/img/zaixianyulan.png)

<!--more-->

http://www.php100.com/html/webkaifa/PHP/PHPyingyong/2013/0708/13625.html


``` bash
CentOS 6.6下安装OpenOffice4.0
最近由于项目需要，要在公司服务器上安装Openoffice，网上搜了一些资料后成功安装，现分享给大家。
1、首先先下载好需要的rpm包：Apache_OpenOffice_4.0.0_Linux_x86-64_install-rpm_zh-CN.tar.gz
或直接命令下载：wget http://heanet.dl.sourceforge.net/project/openofficeorg.mirror/4.0.0/binaries/zh-CN/Apache_OpenOffice_4.0.0_Linux_x86-64_install-rpm_zh-CN.tar.gz
放到服务器的目录下（我放到了opt下）
2、将下载的openoffice解压（我直接解压到opt目录）：tar -zxvf Apache_OpenOffice_4.0.0_Linux_x86-64_install-rpm_zh-CN.tar.gz
3、解压后生成文件夹zh-CN 进到RPMS目录下，直接yum localinstall *.rpm
4、再装RPMS/desktop-integration目录下的openoffice4.0-redhat-menus-4.0-9702.noarch.rpm：yum localinstall openoffice4.0-redhat-menus-4.0-9702.noarch.rpm
5、安装完成直接启动Openoffice服务：
临时启动   /opt/openoffice4/program/soffice -headless -accept="socket,host=127.0.0.1,port=8100;urp;" -nofirststartwizard
一直后台启动 nohup  /opt/openoffice4/program/soffice -headless -accept="socket,host=127.0.0.1,port=8100;urp;" -nofirststartwizard
6、查看服务是否启动（端口8100是否被soffice占用）：netstat -lnp |grep 8100
显示结果：tcp        0      0 127.0.0.1:8100              0.0.0.0:*                   LISTEN      19501/soffice.bin

大功告成！！！

===========================================================================================
安装OpenOffice SDK3.3
– wget http://ftp.nluug.nl/office/openoffice/stable/3.3.0/OOo-SDK_3.3.0_Linux_x86-64_install-rpm_en-US.tar.gz
ar zxvf OOo-SDK_3.3.0_Linux_x86-64_install-rpm_en-US.tar.gz 
cd OOO330_m20_native_packed-1_en-US.9567/
cd RPMS/
sudo rpm -ivh *.rpm --nodeps --force

========================================================================================
3. 安装jodconverter.2.2.2 ，安装了这个之后就已经可以实现DOC转PDF了。


解压之后拖到服务器
java -jar /FILE_SERVER/jodconverter-2.2.2/lib/jodconverter-cli-2.2.2.jar ./demo.doc ./2.pdf
测试

解决中文乱码
http://os.chinaunix.net/a2006/0831/1003/000001003262.shtml

首先把字体文件链接到存放字体的目录中
    cd /usr/share/fonts
    ln -s /home/fwolf/tools/fonts xpfonts
    cd xpfonts
    mkfontscale
    mkfontdir
    这样作和把字体拷贝到/usr/share/fonts的一个目录下的效果是一样的。后面的两个mkfont命令是生成xpfonts目录下所包含的字体的索引信息。然后运行fc-cache命令更新字体缓存：
    fc-cache
========================================================================================
安装swftools
sudo yum install gcc* automake zlib-devel libjpeg-devel giflib-devel freetype-devel
wget http://www.swftools.org/swftools-0.9.2.tar.gz
tar vxzf swftools-0.9.2.tar.gz
cd swftools-0.9.2
./configure --prefix=/usr/swftools
make
make install
sudo ln -s /usr/swftools/bin/pdf2swf /usr/bin/pdf2swf

可能出现的异常
修改一个源文件错误
这个时候，遇到报错
jpeg.c:463: error: conflicting types for ‘jpeg_load_from_mem’
jpeg.h:15: error: previous declaration of ‘jpeg_load_from_mem’ was here
make[1]: *** [jpeg.o] Error 1
原来是函数的定义和头文件的声明有点冲突，解决方式比较简单，修改 jpeg.c 的 463行：
改为：
int jpeg_load_from_mem(unsigned char*_data, int size, unsigned char**dest, int*width, int*height)

继续即可，

make install 时报错
swftools rm：无效选项 -- o

[appadmin@iZ256b708ljZ swftools-0.9.2]$ find ./ |xargs grep -r -i "default_viewer"

./swfs/Makefile
./swfs/Makefile.in
vi 打开
找/default_viewer
把这两个文件的 -o -L 去掉

简单的转换
[appadmin@iZ256b708ljZ ~]$ pdf2swf 1.pdf -o 1.swf

有些pdf中的图形转换效果不好，会产生过多shape，这种情况下可以使用 -s poly2bitmap 的参数，将图形转成点阵。生成的swf尺寸少了。 带简单导航的：
[appadmin@iZ256b708ljZ demo]$ pdf2swf -o 1.swf -z -s flashversion=7 -s simpleviewer -t 1.pdf

```