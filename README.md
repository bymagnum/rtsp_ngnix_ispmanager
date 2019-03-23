
# rtsp + nginx + ispmanager + CentOS 6.8 x86-64

1. Инсталируем ОС.

2. Инсталируем ISPmanager (любая версия)

3. Устанавливаем через SSH: <br>
<pre>
yum -y groupinstall 'Development Tools'
yum -y install libxml2-devel libxslt-devel gd-devel perl-ExtUtils-Embed GeoIP-devel
yum install nano -y
yum install vim -y
yum install hg -y
yum install cmake -y
yum install gcc gcc+ -y
yum install openssl openssl-devel -y
yum install php-devel -y
yum install git -y
yum install libxslt-devel -y
yum install pcre-devel -y
yum install libpcre3 libpcre3-dev libssl-dev -y
yum install rtmpdump -y
yum update -y
</pre>

пожалуй этого достаточно будет для работы сервера в режиме

4. Устанавливаем nginx (именно так и ставим, не через панель): <br>
<pre>
/usr/local/ispmgr/sbin/pkgctl install nginx
</pre>

5. Открываем файл ispmgr.conf (он находится по пути /usr/local/ispmgr/etc). Дописываем: FSEncoding UTF-8 и сохраняем файл.<br>
Данная опция необходима стала для редактирования файлов внутри панели ISPmanager. Открывает файлы сразу в кодировке UTF-8.

6. Переходим к сборке и компилированию nginx + rtmp <br>
скачиваем nginx:
<pre>
wget http://nginx.org/download/nginx-1.10.2.tar.gz 
</pre>
распаковываем:
<pre>
tar -xzvf nginx-1.10.2.tar.gz
</pre>
скачиваем nginx-rtmp-module (https://github.com/arut/nginx-rtmp-module):
<pre>
wget https://github.com/arut/nginx-rtmp-module/zipball/master -O nginx-rtmp-module-master.zip
</pre>
распаковываем:
<pre>
unzip nginx-rtmp-module-master.zip -d nginx-rtmp-module-master
</pre>
переходим в папку распакованного nginx-1.10.2:
<pre>
cd nginx-1.10.2
</pre>
собираем
(необходимо изменить arut-nginx-rtmp-module-43f1e42 - последние значения на свои):
<pre>
./configure --prefix=/usr --add-module=../nginx-rtmp-module-master/arut-nginx-rtmp-module-43f1e42/ --pid-path=/var/run/nginx.pid --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-http_ssl_module
</pre>
если ошибок нет, идем дальше:
<pre>
make
make install
</pre>
переходим в root папку:
<pre>
cd
</pre>

7. (ИНФОРМАЦИЯ НЕ АКТУАЛЬНА!!! Пункт 7.1) <strike>Устанавливаем ffmpeg. (http://firstwiki.ru/index.php/Установка_FFMPEG)</strike> <br>
<strike>Импортируем и устанавливаем нужные репозитории:</strike>
<strike><pre>
rpm --import https://raw.githubusercontent.com/example42/puppet-yum/master/files/CentOS.6/rpm-gpg/RPM-GPG-KEY.atrpms
rpm -Uvh https://www.mirrorservice.org/sites/dl.atrpms.net/el6.7-x86_64/atrpms/stable/atrpms-repo-6-7.el6.x86_64.rpm
</pre></strike>
<strike>Устраняем ошибки в импортируемых репозиториях</strike>
<strike><pre>
sed -i 's,http://dl,https://www.mirrorservice.org/sites/dl,' /etc/yum.repos.d/atrpms*.repo
</pre></strike>
<strike>Устанавливаем FFmpeg</strike>
<strike><pre>
yum install ffmpeg ffmpeg-compat ffmpeg-compat-devel ffmpeg-devel ffmpeg-libs
</pre></strike>


7.1 Устанавливаем ffmpeg.

Удаляем файлы repo atrpms (offline)
<pre>
/etc/yum.repos.d
</pre>
Далее:
<pre>
yum repolist - посмотреть список (уточняем есть ли в списке atrpms)
yum clean all - очистить кеш 
yum makecache - создать кеш
</pre>

Создаем repo: 
<pre>
yum install http://repo.okay.com.mx/centos/6/x86_64/release/okay-release-1-1.noarch.rpm
</pre>
Или руками пишем любым редактором: 
<pre>
[okay]
name=Extra OKay Packages for Enterprise Linux - $basearch
baseurl=http://repo.okay.com.mx/centos/$releasever/$basearch/release
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-OKAY

[okay-debuginfo]
name=Extra OKay Packages for Enterprise Linux - $basearch - Debug
baseurl=http://repo.okay.com.mx/centos/$releasever/$basearch/debug
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-OKAY
gpgcheck=1
</pre>

Далее:
<pre>
yum install ffmpeg
</pre>
Если ранее производились какие либо действия по установке ffmpeg - то возвращаемся к очистке yum и созданию нового кеша yum.


8. Это важно! Перезапуск не выполнять из панели, т.к. нам надо подружить панель с ngnix. <br>
Для тех кто забыл:
<pre>
/etc/init.d/nginx start
/etc/init.d/nginx restart
/etc/init.d/nginx stop
</pre>

9. Далее идем редактировать nginx.conf (лежит /etc/nginx/nginx.conf), приводим примерно к такому виду (может отличаться от моего): <br>
<pre>
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

    #gzip  on;

    log_format isp '$bytes_sent $request_length';


    server {
        listen       80;
        server_name  localhost;

        location /stat {
                rtmp_stat all;
                rtmp_stat_stylesheet stat.xsl;
        }
 
        location /stat.xsl {
                root /etc/nginx/;
        }


        location /dash {
                root /tmp;
                add_header Cache-Control no-cache;
        }


        location /hls {
                types {
                        application/vnd.apple.mpegurl m3u8;
                        video/mp2t ts;
                }
                root /tmp;
                add_header Cache-Control no-cache;
                add_header 'Access-Control-Allow-Origin' '*';
        }

        #charset koi8-r;
        #access_log  logs/host.access.log  main;

        location / {
                root /tmp;
                rtmp_control all;
                #root   html;
                index  index.html index.htm;
        }



        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

rtmp {
    access_log /var/log/nginx/rtmp_access.log;
    server {
        listen 1935;
        ping 30s;

 

       application hls {
            live on;
            hls on;
            hls_fragment 3s;
            hls_playlist_length 60;
            hls_path /tmp/hls;
            exec_static ffmpeg -i "rtsp://guest:12345456@11.22.33.44:123/cam/realmonitor?channel=1&subtype=0" -c copy -f flv rtmp://127.0.0.1:1935/hls/stream;
        }

    }
}
</pre>

10. После настройки сервера, берем плеер, который умеет поддерживать HLS формат и указываем путь до плейлиста. В нашем случае, это http://11.22.33.44/hls/stream.m3u8 (ip сервера свой)


CROSS DOMAIN:
<pre>
add_header 'Access-Control-Allow-Origin' '*';
</pre>


hls примеры:
1. (смазывает растяжкой)
<pre>
ffmpeg -loglevel verbose -re -i "rtsp://guest:12345456@11.22.33.44:123/cam/realmonitor?channel=1&subtype=0"  -vcodec libx264 -vprofile baseline -acodec libmp3lame -ar 44100 -ac 1 -f flv rtmp://127.0.0.1:1935/hls/stream
</pre>

2. (смазывает сеткой)
<pre>
ffmpeg -re -i "rtsp://guest:12345456@11.22.33.44:123/cam/realmonitor?channel=1&subtype=0" -acodec copy -bsf:a aac_adtstoasc -vcodec copy -f flv rtmp://127.0.0.1:1935/hls/stream
</pre>

3. (профит)
<pre>
ffmpeg -i "rtsp://guest:12345456@11.22.33.44:123/cam/realmonitor?channel=1&subtype=0" -c copy -f flv rtmp://127.0.0.1:1935/hls/stream
</pre>



Для добавления камер:

1. Идем в папку tmp, создаем каталог, с названием hls2.

2. Добавляем в секцию rtmp для нарезки плейлиста:
<pre>
     application papina {
            live on;
            hls on;
            hls_fragment 3s;
            hls_playlist_length 60;
            hls_path /tmp/hls2;
            exec_static ffmpeg -i "rtsp://guest:12345456@11.22.33.44:123/cam/realmonitor?channel=1&subtype=0" -c copy -f flv rtmp://127.0.0.1:1935/hls2/stream;
        }
</pre>     
3. А для того чтобы все это дело отдать в веб, в секцию server пишем: 
<pre>
        location /hls2 {
                types {
                        application/vnd.apple.mpegurl m3u8;
                        video/mp2t ts;
                }
                root /tmp;
                add_header Cache-Control no-cache;
                add_header 'Access-Control-Allow-Origin' '*';
        }
</pre>


Пармаетр:
<pre>
add_header 'Access-Control-Allow-Origin' '*'; 
</pre>
требуется для того, чтобы Ваше видео возможно было вставить на любом сайте (политика защиты браузера)*

