cat > Dockerfile << 'EOF'
# Hvilket image er vores start images
FROM php:8.2-fpm

# Kommandoer der skal køres i Image, for instalation af NginX
RUN apt-get update -y \
    && apt-get install -y nginx

# Sætter en Enviroment variable (ligsom Path på Windows)
# PHP_CPPFLAGS are used by the docker-php-ext-* scripts
ENV PHP_CPPFLAGS="$PHP_CPPFLAGS -std=c++11"

# Kør flere kommandoer der skal bruges af php eller NginX
RUN docker-php-ext-install pdo_mysql \
    && docker-php-ext-install opcache \
    && apt-get install libicu-dev -y \
    && docker-php-ext-configure intl \
    && docker-php-ext-install intl \
    && apt-get remove libicu-dev icu-devtools -y

#Lav filen /usr/local/etc/php/conf.d/php-opocache-cfg.ini
RUN { \
    echo 'opcache.memory_consumption=128'; \
    echo 'opcache.interned_strings_buffer=8'; \
    echo 'opcache.max_accelerated_files=4000'; \
    echo 'opcache.revalidate_freq=2'; \
    echo 'opcache.fast_shutdown=1'; \
    echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/php-opocache-cfg.ini

# Kopier NginX konfig filen.
COPY nginx-site.conf /etc/nginx/sites-enabled/default

# Kopier Start Filen til Image
COPY entrypoint.sh /etc/entrypoint.sh

# Gør entrypoint.sh executable
RUN chmod +x /etc/entrypoint.sh

# Det er her alt hvad vi laver med COPY osv. bliver placeret
WORKDIR /var/www/mysite

# kopier vores rå site ind i image, og sætter sikkerhed
COPY --chown=www-data:www-data --chmod=777 ./Site/* /var/www/mysite

# Åben for port 80
EXPOSE 80

# dette udføres hver gang Container startes
ENTRYPOINT ["/etc/entrypoint.sh"]
EOF
