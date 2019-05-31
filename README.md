# hello-world
Apache ⽇志切割 + 过滤指定类型的⽇志
1、vim /usr/local/apache2/etc/httpd.conf ⼤概在298⾏左右
插⼊⼀下两⾏：
ErrorLog "|/usr/local/apache2/bin/rotatelogs -l /usr/local/apache2/logs/
error_%Y%m%d.log 86400"
CustomLog "|/usr/local/apache2/bin/rotate-logs -l /usr/local/apache2/logs/
access_%Y%m%d.log 86400" combined 
上⾯两⾏分别是错误⽇志轮替和访问⽇志轮替
2、vim /usr/local/apache2/conf/httpd.conf 找到Document
关键字
在directory标签下写⼊：
CustomLog "|/usr/local ... _%Y%m%d.log 86400" combined env=!image￾request 
#定义过滤标签为image-request
SetEnvIf Request_URI ".*\.gif$" image-request
SetEnvIf Request_URI ".*\.jpg$" image-request
#过滤.gif和.jpg结尾的资源
Apache 配置静态缓存
客户端缓存资源时间，取决于服务器端设置缓存时间
第⼀种⽅法：
vim /usr/local/apache2/conf/httpd.conf
取消注释开启⼀个叫expires.c的模块 
移动到配置⽂件最后，插⼊如下内容：
<IfModule mod_expires.c>
 ExpiresActive on
 ExpiresByType image/gif "access plus 1 days"
 ExpiresByType image/jpeg "access plus 24 
hours"
 ExpiresByType image/png "access plus 24 
hours"
 ExpiresDefault "now plus 0 min"
</IfModule>
 缓存图像类/gif⽂件⼀天，图像类/jpeg24⼩时，图像类/png24⼩
时，其它默认没有缓存
第⼆种⽅法：
vim /usr/local/apache2/conf/httpd.conf ⽂件最后插⼊
mod_headers.c模块默认已加载
<IfModule mod_headers.c>
 #html,htm,txt⽂件缓存⼀个⼩时
 <filesmatch "\.(html|htm|txt)$">
 header set cache-control "max-age=3600"
 </filesmatch>
 #css,js,swf类⽂件缓存⼀个星期
 <filesmatch "\.(css|js|swf)$">
 header set cache-control "max-age=604800"
 </filesmatch>
 #ico,gig,jpg,jpeg,png,flv,pdf类⽂件缓存⼀年
 <filesmatch "\.(ico|gif|jpg|jpeg|png|flv|pdf)
$">
 header set cache-control "max￾age=29030400"
 </filesmatch>
</IfModule>
重启服务测试
/usr/local/apache2/bin/apachectl -M 
/usr/local/apache2/bin/apachectl -t 
/usr/local/apache2/bin/apachectl restart
使⽤curl 命令测试，格式为：curl -x192.168.1.61:80 ‘http:/www.avtb.com/htdocs/mazhen.jpg’ 
-I
：curl -x192.168.1.61:80 ‘192.168.1.61/htdocs/mazhen.jpg’ -I
测试缓存
禁⽌解析PHP
专门在主配置⽂件声明⼀个标签
<Directory /usr/local/apache2/htdocs/blog>
 php_admin_flag engine off
 <filesmatch "(.*)php"> 
 Order deny,allow
 Deny from all
 Deny from all
 </filesmatch>
</Directory>
检查配置⽂件语法
重启Apache服务
