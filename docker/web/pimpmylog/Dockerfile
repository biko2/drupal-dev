FROM debian:stretch

MAINTAINER juanjo <juanjo.lopez@gmail.com>

#
# Step 1: Installation
#

# Set frontend. We'll clean this later on!
ENV DEBIAN_FRONTEND noninteractive

# Locale
ENV LOCALE es_ES.UTF-8

# Set repositories
RUN \
  echo "deb http://ftp.de.debian.org/debian/ stretch main non-free contrib\n" > /etc/apt/sources.list && \
  echo "deb-src http://ftp.de.debian.org/debian/ stretch main non-free contrib\n" >> /etc/apt/sources.list && \
  echo "deb http://security.debian.org/ stretch/updates main contrib non-free\n" >> /etc/apt/sources.list && \
  echo "deb-src http://security.debian.org/ stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
  # Update repositories cache and distribution
  apt-get -qq update && apt-get -qqy upgrade

# Install some basic tools needed for deployment
RUN apt-get -yqq install \
  apt-utils \
  build-essential \
  debconf-utils \
  debconf \
  mysql-client \
  locales \
  curl \
  wget \
  unzip \
  patch \
  rsync \
  vim \
  openssh-client \
  git \
  bash-completion \
  php-ssh2 \
  locales

# Install locale
RUN \
  sed -i -e "s/# $LOCALE/$LOCALE/" /etc/locale.gen && \
  echo "LANG=$LOCALE">/etc/default/locale && \
  dpkg-reconfigure --frontend=noninteractive locales && \
  update-locale LANG=$LOCALE

# Configure Sury sources
# @see https://www.noobunbox.net/serveur/auto-hebergement/installer-php-7-1-sous-debian-et-ubuntu
RUN \
  apt-get -yqq install apt-transport-https lsb-release ca-certificates && \
  wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg && \
  echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list && \
  # Update repositories cache and distribution
  apt-get -qq update && apt-get -qqy upgrade


# Install PHP7 with Xdebug (dev environment)
RUN apt-get -yqq install \
  php7.2 		\
  php7.2-curl 		\
  php7.2-dev 		\
  php7.2-gd 		\
  php7.2-imagick 		\
  php7.2-dom		\
  php7.2-intl 		\
  php7.2-json 		\
  php7.2-soap 		\
  php7.2-mbstring	\
  php7.2-bcmath     \
#  php7.2-mcrypt 	\
  php7.2-mysql		\
  php7.2-ssh2		\
  php7.2-xml		\
  php7.2-zip		\
  php7.2-apcu		\
  php7.2-opcache	\
  php7.2-memcached 	\
  php7.2-redis		\
  php7.2-xdebug		\
  libapache2-mod-php7.2

# PHP Timezone
ENV TZ=Europe/Madrid
RUN \
  echo $TZ | tee /etc/timezone && \
  dpkg-reconfigure --frontend noninteractive tzdata && \
  echo "date.timezone = \"$TZ\";" > /etc/php/7.2/apache2/conf.d/timezone.ini && \
  echo "date.timezone = \"$TZ\";" > /etc/php/7.2/cli/conf.d/timezone.ini

# Install manually xhprof
RUN \
  cd /tmp && \
  wget https://github.com/Yaoguais/phpng-xhprof/archive/master.zip && \
  unzip master.zip && \
  cd phpng-xhprof-master && \
  phpize7.2 && \
  ./configure --with-php-config=/usr/bin/php-config7.2 && \
  make && make install && \
  mv /usr/lib/php/20170718/phpng_xhprof.so /usr/lib/php/20170718/xhprof.so  && \
  echo "extension=xhprof.so" > /etc/php/7.2/mods-available/xhprof.ini && \
  echo "xhprof.output_dir=/var/www/xhprof" >> /etc/php/7.2/mods-available/xhprof.ini

# Install SMTP.
RUN apt-get install -y ssmtp && \
  echo "FromLineOverride=YES" >> /etc/ssmtp/ssmtp.conf

# Install Apache web server.
RUN apt-get -yqq install apache2

#
# Step 2: Configuration
#

# Disable by default xdebug. Use docker-compose to add file.
RUN phpdismod xdebug

# Remove all sites enabled
RUN rm /etc/apache2/sites-enabled/*

# Configure needed apache modules and disable default site
RUN a2enmod		\
  access_compat		\
  actions		\
  alias			\
  auth_basic		\
  authn_core		\
  authn_file		\
  authz_core		\
  authz_groupfile	\
  authz_host 		\
  authz_user		\
  autoindex		\
  dir			\
  env 			\
  expires 		\
  filter 		\
  headers		\
  mime 			\
  negotiation 		\
  php7.2 		\
  mpm_prefork 		\
  reqtimeout 		\
  rewrite 		\
  setenvif 		\
  status 		\
  ssl \
  && a2dismod cgi

# Install composer (latest version)
RUN curl -sS https://getcomposer.org/installer | php && \
  mv composer.phar /usr/local/bin/composer

# alias to disable xdebug when running composer command
# https://getcomposer.org/doc/articles/troubleshooting.md#xdebug-impact-on-composer
RUN mkdir -p /usr/local/etc/php/conf.d.disabled/
RUN echo 'function composer() { COMPOSER="$(which composer)" || { echo "Could not find composer in path" >&2 ; return 1 ; } && sudo mv /usr/local/etc/php/conf.d/xdebug.ini /usr/local/etc/php/conf.d.disabled/ ; $COMPOSER "$@" ; STATUS=$? ; sudo mv /usr/local/etc/php/conf.d.disabled/xdebug.ini /usr/local/etc/php/conf.d/ ; return $STATUS ; }' >> /etc/bash.bashrc

# prestissimo to speed up composer
RUN composer global require "hirak/prestissimo:^0.3"

### Install DRUSH (latest stable) ###
# Run this in your terminal to get the latest DRUSH project version:
# RUN curl http://files.drush.org/drush.phar -L -o drush.phar && \
  # Accessing from anywhere on your system:
  # mv drush.phar /usr/local/bin/drush && \
  # Apply executable permissions on the downloaded file:
  # chmod +x /usr/local/bin/drush && \
  # Copy configuration files to user home directory:
  # drush init -y

# RUN composer global require drush/drush && \
#  ln -s /root/.composer/vendor/bin/drush /usr/local/bin && \
#  drush init -y


### Install DRUPAL CONSOLE (latest version) ###
# Run this in your terminal to get the latest project version:
# RUN curl https://drupalconsole.com/installer -L -o drupal.phar && \
  # Accessing from anywhere on your system:
  # mv drupal.phar /usr/local/bin/drupal && \
  # Apply executable permissions on the downloaded file:
  # chmod +x /usr/local/bin/drupal && \
  # Copy configuration files to user home directory:
  # drupal init --override

# Bash setup.
RUN echo ". /usr/share/bash-completion/bash_completion" >> ~/.bashrc && echo "alias ll='ls -lahs'" >> ~/.bashrc

#
# Step 3: Clean the system
#

# Cleanup some things.
RUN apt-get -q autoclean && \
  rm -rf /var/lib/apt/lists/*

#
# Step 4: Run
#

# Create 'me' user like local machime user.
RUN useradd me && usermod -G www-data -a me

# Add www-data to adm group
RUN usermod -a -G adm www-data

# Working dir
WORKDIR /var/www

# Volume for Apache2 data
VOLUME /var/www

COPY scripts/apache2-foreground /usr/bin/

EXPOSE 80 443

CMD ["apache2-foreground"]
