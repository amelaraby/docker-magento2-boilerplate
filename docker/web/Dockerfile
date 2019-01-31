FROM php:7.2-apache

LABEL maintainer="araby2305@gmail.com"

ENV MAGENTO_VERSION 2.3.0
ENV INSTALL_DIR /var/www/html
ENV COMPOSER_HOME /var/www/.composer

RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer
COPY ./config/auth.json $COMPOSER_HOME

RUN requirements="libjpeg-dev libmcrypt-dev libmcrypt4 libcurl3-dev libfreetype6 libfreetype6-dev libicu-dev libxslt1-dev libxslt1.1 cron unzip nano git" \
    && apt-get update \
    && apt-get install -y $requirements \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) iconv gd pdo_mysql mbstring zip intl xsl soap bcmath \
    && apt-get purge --auto-remove

RUN yes '' | pecl install mcrypt-1.0.1 && \
    docker-php-ext-enable mcrypt

RUN yes '' | pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_autostart=1" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_connect_back=0" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.idekey=PHPSTORM" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.max_nesting_level=1000" >> /usr/local/etc/php/conf.d/xdebug.ini

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN a2enmod rewrite
RUN echo "memory_limit=2048M" > /usr/local/etc/php/conf.d/memory-limit.ini

RUN chsh -s /bin/bash www-data

RUN chown -R www-data:www-data /var/www
# RUN su www-data -c "composer create-project --repository=https://repo.magento.com/ magento/project-community-edition $INSTALL_DIR $MAGENTO_VERSION"

# RUN cd $INSTALL_DIR \
#     && find . -type d -exec chmod 770 {} \; \
#     && find . -type f -exec chmod 660 {} \; \
#     && chmod u+x bin/magento

COPY ./bin/download-magento /usr/local/bin/download-magento
RUN chmod +x /usr/local/bin/download-magento

COPY ./bin/install-magento /usr/local/bin/install-magento
RUN chmod +x /usr/local/bin/install-magento

COPY ./bin/install-sampledata /usr/local/bin/install-sampledata
RUN chmod +x /usr/local/bin/install-sampledata

# Add cron job
ADD ./config/crontab /etc/cron.d/magento2-cron
RUN chmod 0644 /etc/cron.d/magento2-cron \
    && crontab -u www-data /etc/cron.d/magento2-cron

CMD ["apache2-foreground"]