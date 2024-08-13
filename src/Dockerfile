FROM php:7.4-apache

RUN docker-php-ext-install mysqli

COPY . /var/www/html/

ENV DB_HOST=mysql-service \
    DB_USER=ecomuser \
    DB_PASSWORD=ecompassword \
    DB_NAME=ecomdb

RUN echo "ServerName 172.20.1.101:3306" >> /etc/apache2/apache2.conf

# Add this line for debugging
RUN echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php

EXPOSE 80

# Add this to print environment variables when the container starts
## CMD echo "DB_HOST: $DB_HOST" && \
    ## echo "DB_USER: $DB_USER" && \
    # echo "DB_PASSWORD: $DB_PASSWORD" && \
    # echo "DB_NAME: $DB_NAME" && \
    # apache2-foreground