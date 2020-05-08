FROM php:7-fpm

# Update
RUN apt-get update -y \
	&& docker-php-ext-install mysqli pdo pdo_mysql
RUN apt-get install -y net-tools
RUN mkdir /home/cicd-laravel

# Copy config file
ADD ./my-php.ini /usr/local/etc/php/php.ini

# Copy source
ADD ./src /home/cicd-laravel
# Add permission for www-data user and group
RUN chown -R www-data:www-data /home/cicd-laravel
RUN chmod -R 755 /home/cicd-laravel
RUN chmod -R 755 /home/cicd-laravel/storage

# Configure default WORKDIR
WORKDIR /home/cicd-laravel

# Expose port
EXPOSE 9000

