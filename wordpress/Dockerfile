FROM wordpress:php7.4-apache

# Install Xdebug
RUN pecl install xdebug-3.1.5 \
    && docker-php-ext-enable xdebug

# Install WP-CLI
RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar && \
    chmod +x wp-cli.phar && \
    mv wp-cli.phar /usr/local/bin/wp

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set ServerName to suppress Apache warning
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Install db
RUN apt-get update && apt-get -y dist-upgrade
RUN apt-get install -y default-mysql-server
RUN apt-get install -y default-mysql-client
RUN apt-get install -y subversion
RUN apt-get install -y net-tools

RUN service mariadb start && \
    mysql --user="root" --password="" --execute="ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';"

# Install wp for tests
COPY ./tests/install-wp-tests.sh /tmp/tests/install-wp-tests.sh
COPY ./tests/set-php-temp-path.php /tmp/tests/set-php-temp-path.php
COPY ./tests/db.sql /tmp/tests/db.sql

RUN service mariadb start && \
    chmod +x /tmp/tests/install-wp-tests.sh && \
    /tmp/tests/install-wp-tests.sh wordpress_test root root

# Permissions for db
RUN usermod -u 1000 mysql && groupmod -g 1000 mysql

RUN mkdir -p /run/mysqld && \
    chown -R mysql:mysql /run/mysqld && \
    mkdir -p /var/lib/mysql && \
    chown -R mysql:mysql /var/lib/mysql

ENTRYPOINT ["bash", "-c", "mysqld & apache2-foreground"]