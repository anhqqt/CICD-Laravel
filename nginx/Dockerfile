FROM nginx:latest

# Update
RUN apt-get update -y
# Install Net tool
RUN apt-get install net-tools -y
# Create new web dir
RUN mkdir /var/www/blog
# Set permission for user www-data and group www-data
RUN chown -R www-data:www-data /var/www/blog


# Config for web server
# Important: Co-operate between nginx and php-fpm
ADD ./my-default.conf /etc/nginx/conf.d/default.conf

# Copy source to image
ADD ../src /var/www/blog

# Choose default working dir
# WORKDIR /var/www/blog

# Expose port 80 and 443
EXPOSE 80 443
