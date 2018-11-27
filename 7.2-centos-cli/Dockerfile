# ------------------------------------------------------------------------------------
# @link https://hub.docker.com/_/php/         php image
# @link https://github.com/docker-library/php php dockerfiles
# ------------------------------------------------------------------------------------
# @build-example docker build . -f Dockerfile -t passkey/php:7.2-centos-cli
# ------------------------------------------------------------------------------------
# @run-container docker run -d -it --name tcli passkey/php:7.2-centos-cli
#                docker exec -it tcli /bin/sh
# ------------------------------------------------------------------------------------

FROM passkey/php:7.2-centos-base
LABEL maintainer="gengwh <8505569@qq.com>" version="1.0"

##
# ---------- env settings ----------
##

ENV SWOOLE_VERSION=4.2.7 \
    MONGO_VERSION=1.5.3 \
    REDIS_VERSION=4.2.0

##
# install php extensions
##

RUN cd /tmp \
        && curl -SL "http://pecl.php.net/get/swoole-${SWOOLE_VERSION}.tgz" -o swoole.tgz \
        && curl -SL "http://pecl.php.net/get/mongodb-${MONGO_VERSION}.tgz" -o mongodb.tgz \
        && curl -SL "http://pecl.php.net/get/redis-${REDIS_VERSION}.tgz" -o redis.tgz \
        && ls -alh \
        # for swoole extension libaio linux-headers
        && yum install -y libaio inotify-tools \
        # php extension: mongodb
        && pecl install mongodb.tgz \
        && echo "extension=mongodb.so" > /etc/php7/conf.d/20_mongodb.ini \
        # php extension: redis
        && pecl install redis.tgz \
        && echo "extension=redis.so" > /etc/php7/conf.d/20_redis.ini \
        # php extension: swoole
        && cd /tmp \
        && mkdir -p swoole \
        && tar -xf swoole.tgz -C swoole --strip-components=1 \
        && rm swoole.tgz \
        && ( \
            cd swoole \
            && /usr/local/php/bin/phpize \
            && ./configure --with-php-config=/usr/local/php/bin/php-config --enable-mysqlnd --enable-openssl \
            && make -j$(nproc) && make install \
        ) \
        && rm -r swoole \
        && echo "extension=swoole.so" > /etc/php7/conf.d/20_swoole.ini \
        && php -v \
        # ---------- clear works ----------
        && yum clean all \
        && rm -rf /var/lib/apt/lists/* /var/cache/yum /tmp/* \
        && echo -e "\033[42;37m Build Completed :).\033[0m\n"

EXPOSE 9501
WORKDIR /var/www
