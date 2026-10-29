FROM alpine:3.14

COPY /config/install-php-extensions /usr/local/bin/install-php-extensions
COPY /config/docker-php-ext-enable /usr/local/bin/docker-php-ext-enable
RUN chmod +x /usr/local/bin/install-php-extensions /usr/local/bin/docker-php-ext-enable \
    && ln -sf /usr/local/bin/install-php-extensions /usr/local/bin/docker-php-ext-install

COPY /config/boostrap.sh \
    /uploads/s6-overlay-amd64.tar.gz \
    /config/www.conf \
    /config/nginx.conf \
    /config/start-nginx.sh \
    /config/start-php-fpm.sh \
    tmp/
    
WORKDIR /tmp/ 
RUN sh boostrap.sh

ENTRYPOINT ["/init"]
CMD []

#Copy FPM service script3.12
COPY /config/start-php-fpm.sh /etc/services.d/php-fpm8/run
RUN chmod 755 /etc/services.d/php-fpm8/run

#Copy NGINX service script
COPY /config/start-nginx.sh /etc/services.d/nginx/run
RUN chmod 755 /etc/services.d/nginx/run

COPY /config/php.ini /etc/php8/

WORKDIR /var/www/html 