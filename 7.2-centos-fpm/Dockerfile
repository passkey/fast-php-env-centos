# ------------------------------------------------------------------------------------
# @link https://hub.docker.com/_/php/         php image
# @link https://github.com/docker-library/php php dockerfiles
# ------------------------------------------------------------------------------------
# @build-example docker build . -f Dockerfile -t passkey/php:7.2-centos-fpm
# ------------------------------------------------------------------------------------
# @run-container docker run -d -p 10080:80 --name tfpm passkey/php:7.2-centos-fpm
#                docker exec -it tfpm /bin/sh
# ------------------------------------------------------------------------------------

FROM passkey/php:7.2-centos-cli
LABEL maintainer="gengwh <8505569@qq.com>" version="1.0"

##
# ---------- env settings ----------
##

ENV NGINX_VERSION=1.14.1

##
# ---------- building ----------
##

RUN cd /tmp \
        && curl -SL "http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -o nginx-${NGINX_VERSION}.tar.gz \
        && tar -zxf nginx-${NGINX_VERSION}.tar.gz \
        && cd /tmp/nginx-${NGINX_VERSION} \
        && ./configure \
            --prefix=/usr/local/nginx \
            --conf-path=/etc/nginx/nginx.conf \
            --pid-path=/run/nginx/nginx.pid \
            --lock-path=/run/nginx/nginx.lock \
            --user=root --group=root \
            --with-http_ssl_module --with-http_sub_module --with-http_dav_module --with-http_flv_module \
            --with-http_gzip_static_module --with-http_stub_status_module \
            --with-http_v2_module --with-file-aio --with-http_realip_module --with-pcre \
        && make -j `grep 'processor' /proc/cpuinfo | wc -l` \
        && make install \
        && ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx \
        && rm -rf /tmp/* \
        # - config nginx
        && mkdir -p /etc/nginx/conf.d /var/log/nginx \
        && cd /etc/nginx \
        && { \
            echo "user root;"; \
            echo "worker_processes auto;"; \
            echo "pcre_jit on;"; \
            echo "error_log /var/log/nginx/error.log warn;"; \
            echo "events {"; \
            echo "  use epoll;"; \
            echo "  worker_connections 65535;"; \
            echo "}"; \
            echo "http {"; \
            echo "  include /etc/nginx/mime.types;"; \
            echo "  default_type application/octet-stream;"; \
            echo "  server_tokens off;"; \
            echo "  client_max_body_size 1m;"; \
            echo "  keepalive_timeout 65;"; \
            echo "  sendfile on;"; \
            echo "  tcp_nodelay on;"; \
            echo "  ssl_prefer_server_ciphers on;"; \
            echo "  ssl_session_cache shared:SSL:2m;"; \
            echo "  gzip_vary on;"; \
            echo "  log_format main '\$remote_addr - \$remote_user [\$time_local] "\$request" \$status \$body_bytes_sent "\$http_referer" "\$http_user_agent" "\$http_x_forwarded_for"';"; \
            echo "  access_log /var/log/nginx/access.log main;"; \
            echo "  include /etc/nginx/conf.d/*.conf;"; \
            echo "}"; \
        } | tee nginx.conf \
        && { \
            echo "location ~ .php {"; \
            echo "  fastcgi_pass   127.0.0.1:9000;"; \
            echo "  fastcgi_index  index.php;"; \
            echo "  fastcgi_split_path_info ^((?U).+.php)(/?.+)$;"; \
            echo "  fastcgi_param  SCRIPT_FILENAME  \$document_root\$fastcgi_script_name;"; \
            echo "  fastcgi_param  PATH_INFO        \$fastcgi_path_info;"; \
            echo "  fastcgi_param  PATH_TRANSLATED  \$document_root\$fastcgi_path_info;"; \
            echo "  include        fastcgi_params;"; \
            echo "}"; \
        } | tee fastcgi_php.conf \
        && { \
            echo "server {"; \
            echo "  listen       80;"; \
            echo "  server_name  localhost;"; \
            echo "  charset      utf-8;"; \
            echo "  index index.html index.htm index.php;"; \
            echo "  root  /var/www;"; \
            echo "  include fastcgi_php.conf;"; \
            echo "}"; \
        } | tee conf.d/default.conf \
        # - config PHP-FPM
        && ln -s /usr/local/php/sbin/php-fpm /usr/bin/php-fpm \
        && mv /etc/php7/php-fpm.conf.default /etc/php7/php-fpm.conf \
        && mkdir -p /var/log/php7 \
        && echo "zend_extension=opcache.so" > /etc/php7/conf.d/00_opcache.ini \
        && cd /etc/php7 \
        && { \
            echo "[global]"; \
            echo "pid = /var/run/php-fpm.pid"; \
            echo "error_log = /var/log/php7/php-fpm.log"; \
            echo "[www]"; \
            echo "listen = 0.0.0.0:9000"; \
            echo "user = root"; \
            echo "group = root"; \
            echo "pm = dynamic"; \
            echo "pm.max_children = 200"; \
            echo "pm.start_servers = 50"; \
            echo "pm.min_spare_servers = 20"; \
            echo "pm.max_spare_servers = 50"; \
            echo "request_slowlog_timeout = 3"; \
            echo "slowlog = /var/log/php7/slow.log"; \
            echo "rlimit_files = 51200"; \
            echo "rlimit_core = 0"; \
            echo "catch_workers_output = yes"; \
            echo "env[APP_ENV] = _APP_ENV_"; \
        } | tee php-fpm.d/www.conf \
        && { \
            echo "expose_php = off"; \
            echo "error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT & ~E_NOTICE"; \
            echo "[opcache]"; \
            echo "opcache.enable = 1"; \
            echo "opcache.enable_cli = 1"; \
            echo "opcache.revalidate_freq = 30"; \
            echo "opcache.save_comments = 0"; \
            echo "opcache.optimization_level = 1"; \
            echo "APP_ENV = _APP_ENV_"; \
        } | tee -a conf.d/99-overrides.ini \
        # - config site
        && echo "<?php phpinfo();?>" > /var/www/index.php \
        && { \
            echo "#!/bin/sh"; \
            echo "sed -i -e \"s#_APP_ENV_#\${APP_ENV}#\" /etc/php7/php-fpm.d/www.conf"; \
            echo "sed -i -e \"s#_APP_ENV_#\${APP_ENV}#\" /etc/php7/conf.d/99-overrides.ini"; \
            echo "for script in /var/www/docker/init_*.sh ; do"; \
            echo "  if [ -r \$script ] ; then"; \
            echo "    . \$script"; \
            echo "  fi"; \
            echo "done"; \
            echo "nginx -g 'daemon on;'"; \
            echo "php-fpm -R -F"; \
        } | tee /run.sh \
        && chmod 755 /run.sh \
        && echo -e "\033[42;37m Build Completed :).\033[0m\n"

EXPOSE 9501 80
#VOLUME ["/var/www", "/data"]
WORKDIR /var/www

CMD /run.sh
