FROM php:5.6-fpm

LABEL maintainer Kyben <kc.deguia@gmail.com>

#(c) Other commands credit to thier respective composer


ENV PHALCON_VERSION=3.1.2
RUN set -xe \

# cphalcon
    && curl -LO https://github.com/phalcon/cphalcon/archive/v${PHALCON_VERSION}.tar.gz \
    && tar xzf v${PHALCON_VERSION}.tar.gz \
    && cd cphalcon-${PHALCON_VERSION}/build \
    && ./install \
    && echo "extension=phalcon.so" > /usr/local/etc/php/conf.d/phalcon.ini \
    && cd ../.. \
    && rm -rf v${PHALCON_VERSION}.tar.gz cphalcon-${PHALCON_VERSION} \
    && curl -LO https://github.com/phalcon/phalcon-devtools/archive/v${PHALCON_VERSION}.tar.gz \
    && tar xzf v${PHALCON_VERSION}.tar.gz \
    && mv phalcon-devtools-${PHALCON_VERSION} /usr/local/phalcon-devtools \
    && ln -s /usr/local/phalcon-devtools/phalcon.php /usr/local/bin/phalcon \
    && rm -f v${PHALCON_VERSION}.tar.gz

# composer
ENV COMPOSER_VERSION=1.4.1
RUN curl -L https://getcomposer.org/installer -o composer-setup.php \
    && php composer-setup.php --version=${COMPOSER_VERSION} --install-dir=/usr/local/bin --filename=composer \
    && rm -f composer-setup.php

# Install Git, Node, Npm, Bower, wget, build-essential, tcl8.5
RUN apt-get update && \
	  apt-get install -y git nodejs-legacy npm wget build-essential tcl8.5 && \
	  npm install -g bower && \
    rm -rf /var/lib/apt/lists/*

# PHP ROBO TASK RUNNER
Run wget http://robo.li/robo.phar && chmod +x robo.phar && mv robo.phar /usr/bin/robo 

# php modules
RUN apt-get update -y \
    && apt-get install -y libxml2-dev zlib1g-dev unzip --no-install-recommends --no-install-suggests \
    && apt-get clean -y \
    && pecl install redis xdebug \
    && docker-php-ext-enable redis xdebug \
    && docker-php-ext-install opcache pdo_mysql mysqli soap zip

WORKDIR /app

RUN usermod -u 1000 www-data

ENV NGINX_VERSION 1.13.0-1~jessie

RUN set -e; \
	NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62; \
	found=''; \
	for server in \
		ha.pool.sks-keyservers.net \
		hkp://keyserver.ubuntu.com:80 \
		hkp://p80.pool.sks-keyservers.net:80 \
		pgp.mit.edu \
	; do \
		echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
		apt-key adv --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; \
	done; \
	test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; \
	exit 0

RUN echo "deb http://nginx.org/packages/mainline/debian/ jessie nginx" >> /etc/apt/sources.list \
	&& apt-get update \
	&& apt-get install --no-install-recommends --no-install-suggests -y \
						ca-certificates \
						nginx=${NGINX_VERSION} \
						gettext-base \
                        supervisor \
	&& rm -rf /var/lib/apt/lists/*

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log && ln -sf /dev/stderr /var/log/nginx/error.log

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

COPY nginx.conf /etc/nginx/nginx.conf

#Add timezone
COPY /phpini/timezone.ini /usr/local/etc/php/conf.d

EXPOSE 80 443

CMD ["/usr/bin/supervisord"]
