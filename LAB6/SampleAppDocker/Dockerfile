FROM php:7.3.6-apache-stretch

COPY index.php /srv/app/index.php
COPY vhost.conf /etc/apache2/sites-available/000-default.conf
RUN docker-php-ext-install mysqli

RUN chown -R www-data:www-data /srv/app && a2enmod rewrite