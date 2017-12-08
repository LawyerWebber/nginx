1.nginx轻量级的http服务器，比apacha服务器的性能更高、稳定性更强

2.高并发连接的情况下，nginx采用最新的epoll网络I/O模型，apache采用的select网络I/O模型效率低下

3.安装lnmp环境
  virtualbox虚拟机，安装Linux（centos）
  配置centos第三方yum源（centos默认的标准源里没有nginx软件包）
	yum install -y wget  （complate代表完成，此时可以使用wget了）
	wget http://www.atomicorp.com/installers/atomic
	若长时间没反应，ctrl+c 终止
	sh atomic
	yes
  
  安装nginx
    yum install -y nginx 
  	service nginx start (OK，启动成功)
  	chkconfig  --level 235 nginx on（开机自动启动）

  安装mysql
    yum install -y mysql-server mysql-devel
    service mysqld start
    chkconfig --level 235 mysqld on

  安装php和所需组件使php支持mysql、fastfgi模式  
    yum install -y php lighttpd-fastcgi php-common php-devel php-fpm
    php-mysql
    service php-fpm start
    chkconfig --level 235 php-fpm on

4.nginx配置
  配置nginx支持php
    未进入nginx目录
      mv /etc/nginx/nginx.conf /etc/nginx/nginx.confbak (mv备份)将配置文件改为备份文件（修改配置文件前）
      cp /etc/nginx/nginx.conf.default /etc/nginx/nginx.conf（cp复制）
      由于原配置文件需要自己去写，因此可以使用默认的配置文件作为配置文件
      vi /etc/nginx/nginx.conf  （vi修改）
      修改nginx配置文件，添加fastcgi支持
    已经进入了nginx目录
      cd /etc/nginx
      ls
      直接写为
      mv nginx.conf nginx.confbak
      cp nginx.conf.defalut nginx.conf
      vi nginx.conf
    set nu 显示行号
    默认
    location /{
    	root html;(root目录)
    	index index.html index.html（默认访问的索引文件）
	}  
	改为
	location /{
		root /www;（/根目录）
		index index.php index.html inde.htm
	}
	#代表注释
	location ~ \.php$ {
		root /www;
		fastcgi_param SCRIPT_FILEMAME /www$fastcgi_script_name;
	}
	保存退出
	配置nginx支持php
	备份
	  cd /etc/
	  cp php.ini php.inibak(bak就是备份)
	  vi php.ini
	  在文件末尾(:$ 小写o到达最后一行 )添加cgi.fix_pathinfo = 1
	创建www目录
	   mkdir /www
	   cd /www/
	   vi index.php
	   <?php
	    phpinfo();  //测试php是否安装成功  
	   ?>
	重启nginx php-fpm
	   service nginx restart
	   service php-fpm restart
    查看本机ip(linux系统中)
       ifconfig
       在浏览器输入ip地址/info.php
       显示php界面环境搭建成功      	

5.nginx启动与关闭
  开启
    service nginx start
  关闭
    service nginx stop
  重启
    service nginx restart

6.nginx配置文件说明
  主配置文件
    /etc/nginx/nginx.conf
  扩展配置文件
    /etc/nginx/cong.d/*.conf，需要主配置文件加载
    添加这句话
    http {
		# Load config files from the /etc/nginx/conf.d directory
		include /etc/nginx/conf.d/*conf;
	}      
	conf.d下面的conf文件才能生效

7.nginx配置文件的架构
  #http服务器
  http{
  	#主机设置
  	server{
  		#默认访问请求
  		location/{
  		    #定义服务器的默认网站根目录
			root /www;
			#定义首页索引文件的位置
			index index.php index.html index.htm;
  		}
  		#侦听80端口
  		listen 80;
  		#定义使用www.xx.com域名访问
  		server_name www.xx.com;
  		#设定本虚拟主机的访问日志
  		access_log logs/www.xx.com.access.log main;

  	}
  }	
  #php脚本请求全部转发到fastcgi处理，使用fastcgi默认配置
  location ~ \php$ {}

8.显示目录
  开启目录列表
  打开nginx.conf文件，在location server或http段加入
  http{
  	autoindex on; //自动显示目录
  }
  // 只要更改配置项，需要重启nginx配置
  service nginx restart
  service php-fpm restart //管理器
  然后在浏览器中输入域名就能看到文件夹

9.开启php错误
  修改php配置文件
  vi /etc/php.ini
  display_errors = On
  修改php-fpm.conf配置文件
  vi /etc/php-fpm.d/www.conf
  php_flag[display_errors] = on
  重启nginx和php-fpm
  service nginx restart
  service php-fpm restart
  :/ 搜索     

10.虚拟主机
   两个或者两个以上的网站共同访问一台服务器
   在/www目录下面创建qq和hd两个目录（创建这两个网站），里面放入index.html
   通过虚拟主机www.qq.com（域名通过改host文件，上线后购买域名）访问qq目录，www.hd.com访问hd目录
   nginx下，一个server标签就是一个虚拟主机  
   rm -rf * 删除目录
   nginx.conf配置文件中的http{}中新增如下代码：
   server{
     listen 80;
     server_name www.qq.com;  //主机名称
     index index.html;
     root /www/qq;   //默认访问的目录
   }
   server{
   	 listen 80;
   	 server_name www.hd.com;
   	 index index.html;
   	 root /www/hd;
   }
   Linux系统查看ip是ifconfig
   动态ip,重启网卡或者电脑会改变
   修改了配置文件需要重启
   重启nginx，service nginx restart
   更改本地hosts文件指向测试主机
   浏览器访问不同地址就会出现不同页面
   windows操作系统host文件的地址
   c:/windows/system32/drivers/etc/hosts
   可能涉及到权限的问题
   因此复制hosts文件出来修改在粘贴回去

11.nginx反向代理
   当一个代理服务器能够代理外部网络上的主机访问内部网络时，这种代理服务的方式称为反向代理服务。
   此时代理服务器对外就表现为一个web服务器，外部网络就可以简单把它当作一个标准的web服务器而不需要特定的配置。
   不同之处在于，这个服务器没有保存任何网页的真实数据，所有静态网页或者cgi程序，都保存在内部的web服务器上。 
12.负载均衡
   通过反向代理实现负载均衡，负载均衡是大流量网站要做的措施，单从字面上的意思来理解就可以解释N台服务器平均分担负载，
   不会因为某台服务器负载高宕机而某台服务器闲置的情况。那么负载均衡的前提就是要多台服务器才能实现，
   也就是两台以上即可。
13.部署思路
   删除之前的虚拟主机配置server，在虚拟机里克隆三台CentOS，A服务器做为主服务器，域名直接解析到A服务器（192.168.1.10）上，
   由A服务器负载均衡到B服务器（192.168.1.100）与C服务器（192.168.1.200）上。这样就算B或者C服务器宕机也不影响网站访问，
   这样就不会担心在负载均衡模式下因为某台机子宕机和拖累整个站点了。
   测试域名：hd.com
   至少3台服务器
   A服务器IP：192.168.1.10（主）也称之为反向代理服务器
   B服务器IP: 192.168.1.100
   C服务器IP: 192.168.1.200
   修改host文件，使用hd域名
   由于测试，域名就随便使用一个hd.com用作测试，所以解析在hosts文件设置
   打开C：windows\system32/drivers/etc/hosts文件，在末尾添加192.168.1.10 hd.com
   ping baidu.com //测试是否有网络
14.主服务器部署（反向代理）
   A服务器nginx.conf配置，设置完毕记得重启nginx
   在http段加入以下代码，配置服务器集群server cluster
   下面weight(权重)代表2/5（5分之2的机率）访问192.168.1.100，3/5访问192.168.1.200
   upstream hd.com{
   server 192.168.1.100:80 weight=2;
   server 192.168.1.200:80 weight=3;
   }
   server{
   listen 80;
   server_name hd.com;
   location /{
     #反向代理地址
	 proxy_pass http://hd.com;
	 #设置主机头和客户端真实地址，以便服务器获取客户端真实ip
	 proxy_set_header Host $host;
	 proxy_set_header X-Real-IP $remote_addr;
	 proxy_set_header X-Forwarded-For $proxy_add_x_forward_for;
   }
   }
   远程连接服务器 xshell
15.从服务器部署
   B.C服务器nginx.conf配置，设置完毕后记得重启nginx
   在http段加入以下代码
   server{
   listen 80;
   server_name hd.com;
   index index.html;
   root /www;
   }   
   当访问hd.com的时候，为了区分是转向哪台服务器处理所以分别在B,C服务器下写一个
   不同内容的/www/index.html文件以作区分
   打开浏览器访问hd.com结果，刷新会发现所有的请求均分别被主服务器（192.168.1.10）
   分配到B服务器（192.168.1.100）与C服务器（192.168.1.200）上，实现了负载均衡
   连接服务器
   ssh root@服务器地址
























   
