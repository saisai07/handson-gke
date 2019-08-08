FROM php:7.2-fpm

WORKDIR app
COPY index.php index.php
COPY config/personal.ini /etc/php.d/personal.ini
COPY --from=vendor /app/vendor/ /var/www/html/vendor/
COPY --from=frontend /app/public/js/ /var/www/html/public/js/
COPY --from=frontend /app/public/css/ /var/www/html/public/css/
COPY --from=frontend /app/mix-manifest.json /var/www/html/mix-manifest.json
