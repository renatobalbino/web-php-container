FROM alpine:3.9
ADD https://repos.php.earth/alpine/phpearth.rsa.pub /etc/apk/keys/phpearth.rsa.pub

COPY /config/boostrap.sh \
    /uploads/s6-overlay-amd64.tar.gz \
    /uploads/instantclient-basic-linux.x64-11.2.0.4.0.zip \
    /uploads/instantclient-sdk-linux.x64-11.2.0.4.0.zip \
    /config/www.conf \
    /config/nginx.conf \
    /config/start-nginx.sh \
    /config/start-php-fpm7.sh \
    tmp/ 
    
WORKDIR /tmp/ 
RUN sh boostrap.sh

ENTRYPOINT ["/init"]
CMD []

#Copy FPM service script
COPY /config/start-php-fpm7.sh /etc/services.d/php-fpm7/run
RUN chmod 755 /etc/services.d/php-fpm7/run

#Copy NGINX service script
COPY /config/start-nginx.sh /etc/services.d/nginx/run
RUN chmod 755 /etc/services.d/nginx/run

WORKDIR /var/www/html 