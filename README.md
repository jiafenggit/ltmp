# ltmp
CentOs6.5配置LTMP服务(Tengine,php,mysql)


一、配置防火墙

vim /etc/sysconfig/iptables

# 允许80端口通过防火墙

-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT

# 允许3306端口通过防火墙

-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT

# 重启防火墙使配置生效

/etc/init.d/iptables restart

备注: COMMIT ~ COMMIT 之间方可生效

二、安装Tengine

# 删除系统自带的软件包

yum remove httpd* php*

# 编译库

yum install gcc-c++

# 安装依赖库

yum -y install zlib zlib-devel openssl openssl-devel pcre pcre-devel

# 安装ngx_cache_purge模块

wget  http://labs.frickle.com/files/ngx_cache_purge-2.1.tar.gz  && tar -zxvf ngx_cache_purge-2.1.tar.gz && rm -rf ngx_cache_purge-2.1.tar.gz

# 下载Tengine

wget  http://tengine.taobao.org/download/tengine-2.0.0.tar.gz

# 解压,重命名,删除包

tar -zxvf tengine-2.0.0.tar.gz && rm -rf tengine-2.0.0.tar.gz

# 安装Tengine

cd tengine-2.0.0 && ./configure --add-module=/usr/local/ngx_cache_purge-2.1 --with-http_stub_status_module --prefix=/usr/local/nginx --user=nginx --group=nginx && make && make install

# 设置nginx开机启动 (配置文件 -> nginx开机启动.txt)

vim /etc/rc.d/init.d/nginx

chmod 775 /etc/rc.d/init.d/nginx

chkconfig  --level 012345 nginx on

# 添加nginx组与添加nginx用户并启动nginx

groupadd nginx && useradd -g nginx nginx && service nginx start

三、安装MySQL

# 输入Y即可自动安装,直到安装完成

yum install mysql mysql-server

# 启动MySQL

/etc/init.d/mysqld start

# 设为开机启动

chkconfig mysqld on

# 拷贝配置文件（注意：如果/etc目录下面默认有一个my.cnf，直接覆盖即可）

cp /usr/share/mysql/my-medium.cnf /etc/my.cnf

# 为root账户设置密码回车，根据提示输入Y，输入2次密码，回车，根据提示一路输入Y，最后出现：Thanks for using MySQL!

mysql_secure_installation

# MySql密码设置完成，重新启动 MySQL：(restart:重启, stop:停止, start:启动)

/etc/init.d/mysqld (restart|stop|start)

service mysqld (restart|stop|start)

四、安装PHP5

# 根据提示输入Y直到安装完成

yum install php php-fpm

# 这里选择以上安装包进行安装，根据提示输入Y回车

yum install php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt

# 设置php-fpm开机启动

chkconfig php-fpm on

# 启动php-fpm (restart:重启, stop:停止, start:启动)

/etc/init.d/php-fpm (restart|stop|start)

service php-fpm (restart|stop|start)

五、配置nginx支持php

# 备份原有配置文件

cp /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.backup

# 编辑 nginx.conf (配置文件 -> nginx.conf.txt)

vim /usr/local/nginx/conf/nginx.conf

# 增加代理配置文件 (配置文件 -> proxy(代理配置).conf.txt)

vim /usr/local/nginx/conf/proxy.conf

# 增加主机配置文件 (配置文件 -> mysvrhost(主机配置).conf.txt)

vim /usr/local/nginx/conf/mysvrhost.conf

# 增加压缩配置文件 (配置文件 -> mysvrhost(主机配置).conf.txt)

vim  /usr/local/nginx/conf/gzip.conf

# 增加虚拟机配置文件 (配置文件 -> 虚拟机.conf.txt)

vim  /usr/local/nginx/conf/conf.d/xxx.conf

# 优化TCP设置 (配置文件 -> 优化TCP设置.txt)

vim /etc/sysctl.conf

/sbin/sysctl -p

# 重启nginx

service nginx restart

六、php配置

# 编辑 php.ini

vim /etc/php.ini

# 在946行 把前面的分号去掉，改为date.timezone = PRC

date.timezone = PRC

# 在386行 列出PHP可以禁用的函数，如果某些程序需要用到这个函数，可以删除，取消禁用

disable_functions =

passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,getservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd,posix_getegid,posix_geteuid,posix_getgid,posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid,posix_getppid,posix_getpwnam,posix_getpwuid,posix_getrlimit,posix_getsid,posix_getuid,posix_isatty,posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid,posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname

# 在432行 禁止显示php版本的信息

expose_php = Off

七、配置php-fpm

# 备份原有配置文件并编辑

cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/www.conf.backup && vim /etc/php-fpm.d/www.conf

# 修改用户为nginx

user = nginx

# 修改组为nginx

group = nginx

八、新建站点

# 进入 /home/webapps

cd  /home/webapps

# 新建站点

mkdir  www.xxx.com/{backup,logs,public}  && cd  www.xxx.com

# 设置Nginx运行权限

chown nginx.nginx -R public

# 新建测试文件

vim public/index.php

# 文件内容

<?php

 phpinfo();

九、重启nginx,php-fpm

# 重启nginx

service nginx restart

# 重启php-fpm

service php-fpm restart
