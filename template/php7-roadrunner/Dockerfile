FROM openfaas/of-watchdog:0.8.1 as watchdog
FROM spiralscout/roadrunner:1.8.3 as roadrunner
FROM php:7.4.10-cli-alpine

# Install Composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
  && php -r "if (hash_file('SHA384', 'composer-setup.php') === rtrim(file_get_contents('https://composer.github.io/installer.sig'))) { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
  && php composer-setup.php \
  && php -r "unlink('composer-setup.php');" \
  && mv composer.phar /usr/local/bin/composer

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

COPY --from=roadrunner /usr/bin/rr /usr/bin/rr
RUN chmod +x /usr/bin/rr

# Add non root user
RUN addgroup app && adduser --system app
RUN chown app /home/app

USER app
WORKDIR /home/app/

#install function code
USER root


COPY rr.yaml .
COPY ./function  function
RUN chown -R app:app ./
# Install php extensions
RUN cd /home/app/function && ls && sh php-extension.sh && chown -R app /home/app
RUN cd /home/app/function && composer install

USER app

ENV fprocess="/usr/bin/rr serve -d -c rr.yaml"
ENV upstream_url="http://127.0.0.1:5070"
ENV mode="http"
HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
