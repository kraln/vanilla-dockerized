################################################################################
# Base image
################################################################################

FROM nginx:1.9

################################################################################
# Environment variables
################################################################################

ARG VANILLA_VERSION
ARG VANILLA_URL

################################################################################
# Build instructions
################################################################################

USER root

# Remove default nginx configs.
RUN rm -f /etc/nginx/conf.d/*

# Copy nginx configs to container.
COPY config/nginx.conf /etc/nginx/
COPY config/server.conf /etc/nginx/conf.d/

# Install Supervisor and copy config.
RUN apt-get update && apt-get install -y supervisor
COPY config/supervisor.conf /etc/supervisor/conf.d/

# Install git.
RUN apt-get install -y git-core

# Install PHP5 FPM and copy configs.
RUN apt-get install -y php5-fpm
COPY config/php.ini /etc/php5/fpm/conf.d/
COPY config/php5-fpm.conf /etc/php5/fpm/pool.d/

# Ensure that PHP5 FPM is run as root.
RUN sed -i "s/user = www-data/user = root/" /etc/php5/fpm/pool.d/www.conf
RUN sed -i "s/group = www-data/group = root/" /etc/php5/fpm/pool.d/www.conf

# Install PHP modules.
RUN apt-get install -y \
  php5-curl \
  php5-gd \
  php5-memcached \
  php5-mysql


# Install PHP Composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php -r "if (hash_file('SHA384', 'composer-setup.php') === '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
RUN php composer-setup.php --install-dir=/usr/bin
RUN php -r "unlink('composer-setup.php');"

WORKDIR /var/www

# Install Vanilla.
RUN git clone -b $VANILLA_VERSION --depth 1 $VANILLA_URL .

# Set correct folder permissions.
RUN chmod 777 conf
RUN chmod 777 cache

# Copy applications, plugins, themes, and locales to Vanilla.
COPY applications/ applications/
COPY plugins/ plugins/
COPY themes/ themes/
COPY locales/ locales/

# Make sure the PHP Composer runs
RUN php /usr/bin/composer.phar install

################################################################################
# Volumes
################################################################################

# Expose writeable folders as volumes.
VOLUME "/var/www/conf/"
VOLUME "/var/www/uploads/"

# Expose logging volumes.
VOLUME "/var/log/nginx/"
VOLUME "/var/log/php5-fpm/"

################################################################################
# Entrypoint
################################################################################

ENTRYPOINT ["/usr/bin/supervisord"]
