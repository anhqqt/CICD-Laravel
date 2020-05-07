FROM nginx:latest

# Update
RUN apt-get update -y
# Install Net tool
RUN apt-get install net-tools -y
#
RUN chown -R www-data:www-data /var/www/blog


# Config for web server
# Important: Co-operate between nginx and php-fpm
ADD ./my-default.conf /etc/nginx/conf.d/default.conf

# Copy source to image
ADD ./src /var/www/blog

# Expose port 80 and 443
EXPOSE 80 443
