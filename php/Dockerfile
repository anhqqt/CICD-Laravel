FROM php:7-fpm

# Update
RUN apt-get update -y && apt-get install -y libmcrypt-dev mysql-client \
	&& docker-php-ext-install mysqli pdo pdo_mysql pdo_pgsql
RUN apt-get install -y net-tools
RUN mkdir /home/cicd-laravel

# Copy config file
ADD ./my-php.ini /usr/local/etc/php/php.ini

# Copy source
ADD ../src /home/cicd-laravel
# Add permission for www-data user and group
RUN chown -R www-data:www-data /home/cicd-laravel

# Configure default WORKDIR
# WORKDIR /var/www/blog

# Expose port
EXPOSE 9000

