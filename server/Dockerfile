FROM centos:centos7

LABEL maintainer="Linnk"

# User and Enviroments
RUN groupadd incremen && useradd -g incremen incremen
ENV PACKAGE_URL https://repo.mysql.com/yum/mysql-5.7-community/docker/x86_64/mysql-community-server-minimal-5.7.18-1.el7.x86_64.rpm

# Installing utilities
RUN yum install -y \
	unzip \
	epel-release \
	bind-utils \
	yum-utils \
	wkhtmltopdf \
	gearmand \
	gettext \
	which \
	nano

# Installing MySQL server
RUN rpmkeys --import http://repo.mysql.com/RPM-GPG-KEY-mysql \
  && yum install -y $PACKAGE_URL \
  && yum install -y libpwquality \
  && rm -rf /var/cache/yum/*

VOLUME /var/lib/mysql

# Installing NodeJS & NPM
RUN yum install -y nodejs npm
RUN npm install -g pm2 grunt-cli n
RUN n 6.9.3

# Installing PHP 7.1
RUN yum install -y \
	https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm \
	http://rpms.remirepo.net/enterprise/remi-release-7.rpm

RUN	yum-config-manager --enable remi-php71
RUN yum install -y php \
	php-pecl-redis \
	php-fpm \
	php-gd \
	php-pecl-gearman \
	php-mbstring \
	php-pecl-yaml \
	php-mcrypt \
	php-mysql \
	php-pdo \
	php-opcache \
	php-pecl-xdebug \
	php-xml

RUN sed -ie "s/;date.timezone =/date.timezone = America\/Mexico_City/" /etc/php.ini && \
	sed -ie "s/upload_max_filesize = 2M/upload_max_filesize = 5M/" /etc/php.ini && \
	sed -ie "s/group = apache/group = incremen/" /etc/php-fpm.d/www.conf && \
	sed -ie "s/user = apache/user = incremen/" /etc/php-fpm.d/www.conf

# Installing Redis server
RUN yum install -y redis

# Installing NGINX server
RUN yum install -y nginx

# Installing Composer
ENV PATH "/composer/vendor/bin:$PATH"
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer
ENV COMPOSER_VERSION 1.4.2

RUN curl -s -f -L -o /tmp/installer.php https://raw.githubusercontent.com/composer/getcomposer.org/da290238de6d63faace0343efbdd5aa9354332c5/web/installer \
 && php -r " \
    \$signature = '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410'; \
    \$hash = hash('SHA384', file_get_contents('/tmp/installer.php')); \
    if (!hash_equals(\$signature, \$hash)) { \
        unlink('/tmp/installer.php'); \
        echo 'Integrity check failed, installer is either corrupt or worse.' . PHP_EOL; \
        exit(1); \
    }" \
 && php /tmp/installer.php --no-ansi --install-dir=/usr/bin --filename=composer --version=${COMPOSER_VERSION} \
 && rm /tmp/installer.php \
 && composer --ansi --version --no-interaction

# Clearing
RUN rm -rf /var/cache/yum/*

# Setting develop enviroment
ENV ENV_NODE development
ENV DEBUG ""

RUN mkdir -p /usr/src/app

# Entrypoint
RUN mkdir /docker-entrypoint-initdb.d
ADD mysql-entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

# Final configs
EXPOSE 3306 33060
COPY process.yml /process.yml

CMD ["pm2-docker", "process.yml"]
