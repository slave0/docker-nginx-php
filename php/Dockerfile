FROM php:8-fpm

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"\
    && php composer-setup.php\
    && php -r "unlink('composer-setup.php');"\
    && mv composer.phar /usr/local/bin/composer

# Install dependencies
RUN apt-get update \
    && apt-get install -y \
    supervisor \
    cron \
    zlib1g-dev \
    libzip-dev \
    procps \
    && docker-php-ext-install \
    zip \
    pdo \
    pdo_mysql \
    mysqli \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update \
    && apt-get install -y libfreetype6-dev libjpeg62-turbo-dev libpng-dev \
    && docker-php-ext-configure gd \ 
    && docker-php-ext-install -j$(nproc) gd 

RUN apt-get update && apt-get install -y libc-client-dev libkrb5-dev && rm -r /var/lib/apt/lists/*
RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap

RUN docker-php-ext-install sockets

RUN docker-php-ext-install \
    pdo \
    pdo_mysql \
    mysqli \
    calendar \
    bcmath \
    exif \
    opcache

# CRON
# Install cron and supervisor
RUN apt-get update && apt-get -y install cron \
    supervisor


# Add crontab file in the cron directory
ADD crontab /etc/cron.d/hello-cron

# Give execution rights on the cron job
RUN chmod 0644 /etc/cron.d/hello-cron

# Apply cron job
RUN crontab /etc/cron.d/hello-cron

# Create the log file to be able to run tail
RUN touch /var/log/cron.log

RUN mkdir -p /var/log/supervisor

RUN echo "memory_limit = 1G" >> /usr/local/etc/php/conf.d/settings.ini

RUN echo "upload_max_filesize = 100M" >> /usr/local/etc/php/conf.d/settings.ini
RUN echo "output_buffering = 4096" >> /usr/local/etc/php/conf.d/settings.ini
RUN echo "session.sid_length = 26" >> /usr/local/etc/php/conf.d/settings.ini
RUN echo "session.sid_bits_per_character = 5" >> /usr/local/etc/php/conf.d/settings.ini

RUN echo "* soft  nofile  200000" >> /etc/security/limits.conf
RUN echo "* hard  nofile  200000" >> /etc/security/limits.conf
RUN echo "www-data  soft  nofile 200000" >> /etc/security/limits.conf
RUN echo "www-data  hard  nofile 200000" >> /etc/security/limits.conf
RUN echo "root soft nofile 200000" >> /etc/security/limits.conf
RUN echo "root hard nofile 200000" >> /etc/security/limits.conf
RUN echo "nginx soft nofile 200000" >> /etc/security/limits.conf
RUN echo "nginx hard nofile 200000" >> /etc/security/limits.conf

RUN echo "fs.file-max = 200000" >> /etc/sysctl.conf

EXPOSE 9000

WORKDIR /var/www/

CMD /usr/bin/supervisord -c /etc/supervisor/supervisord.conf

