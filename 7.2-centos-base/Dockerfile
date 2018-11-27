# ------------------------------------------------------------------------------------
# @link https://hub.docker.com/_/php/         php image
# @link https://github.com/docker-library/php php dockerfiles
# ------------------------------------------------------------------------------------
# @build-example docker build . -f Dockerfile -t passkey/php:7.2-centos-base
#                docker build --build-arg app_env=test . -f Dockerfile -t passkey/php:7.2-centos-base
# ------------------------------------------------------------------------------------
# @run-container docker run -d -it --name tbase passkey/php:7.2-centos-base
#                docker exec -it tbase /bin/sh
# ------------------------------------------------------------------------------------

FROM centos:7.5.1804
LABEL maintainer="gengwh <8505569@qq.com>" version="1.0"

##
# ---------- env settings ----------
##

# --build-arg timezone=Asia/Shanghai
ARG timezone
# prod grey test dev
ARG app_env=prod
# php version
ARG php_version=7.2.12

ENV APP_ENV=${app_env:-"prod"} \
    TIMEZONE=${timezone:-"Asia/Shanghai"} \
    PHP_VERSION=${php_version:-"7.2.12"}

##
# ---------- building ----------
##

RUN yum makecache \
    && yum -y update \
    && yum install -y \
    autoconf \
    gcc \
    glibc-headers \
    gcc-c++ \
    tar \
    bison \
    re2c \
    libcurl \
    curl-devel \
    bzip2 \
    bzip2-devel \
    openssl \
    openssl-devel \
    libjpeg \
    libjpeg-devel \
    libpng \
    libpng-devel \
    freetype \
    freetype-devel \
    libxml2 \
    libxml2-devel \
    libicu-devel \    
    # install php7 and some extensions
    && mkdir -p /etc/php7/conf.d \
    && cd /tmp \
    && curl -SL "http://jp2.php.net/distributions/php-${PHP_VERSION}.tar.gz" -o php-${PHP_VERSION}.tar.gz \
    && tar xzf php-${PHP_VERSION}.tar.gz \
    && cd /tmp/php-${PHP_VERSION} \
    && ./configure \
        --prefix=/usr/local/php --sysconfdir=/etc/php7 --with-config-file-path=/etc/php7 --with-config-file-scan-dir=/etc/php7/conf.d --with-mysqli --with-pdo-mysql --with-iconv-dir --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir --enable-simplexml --enable-xml --disable-rpath --enable-bcmath --enable-soap --enable-zip --with-curl --enable-fpm --with-fpm-user=root --with-fpm-group=root --enable-mbstring --enable-sockets --with-gd --with-openssl --with-mhash --enable-pcntl --enable-session --enable-opcache --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --with-xmlrpc --with-bz2 --enable-intl --enable-ftp --enable-dom --with-gettext --enable-calendar --enable-mysqlnd  --with-mysql-sock=/run/mysqld/mysqld.sock --enable-sysvmsg --enable-sysvshm --enable-phar --without-pear --without-readline \	
    && make -j `grep 'processor' /proc/cpuinfo | wc -l` \
    && make install \  
    && ln -s /usr/local/php/bin/php /usr/bin/php \
    && curl -SL "http://pear.php.net/go-pear.phar" -o go-pear.php \
    && chmod +x go-pear.php \
    && php go-pear.php \
    && ln -s /usr/local/php/bin/pecl /usr/bin/pecl \
    && rm -rf /tmp/* \
    && yum clean all \
    && rm -rf /var/lib/apt/lists/* /var/cache/yum \
    #  ---------- some config,clear work ----------
    && cd /etc/php7 \
    # - config PHP
    && { \
        echo "upload_max_filesize=100M"; \
        echo "post_max_size=108M"; \
        echo "memory_limit=1024M"; \
        echo "date.timezone=${TIMEZONE}"; \
    } | tee conf.d/99-overrides.ini \
    # - config timezone
    && rm -rf /etc/localtime \    
    && ln -sf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo -e "\033[42;37m Build Completed :).\033[0m\n"

WORKDIR /var/www
