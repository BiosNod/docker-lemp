############################################################
# Dockerfile to run Composer Containers
# Based on Ubuntu Image
############################################################

# Set the base image to use to Ubuntu
FROM composer

# чтобы не спрашивали что-то вида "Please select the geographic area in which you live. Subsequent..."
ENV DEBIAN_FRONTEND noninteractive
ENV COMPOSER_ALLOW_SUPERUSER 1

# Устанавливаем зависимости уже после старта контейнера, а не в момент сборки по причине того, что volumes
# монтируются только при старте контейнера, а не при его сборке, вот такой своеобразный костыль, после запуска
# контейнер с композером подтянет зависимости и умрёт с кодом 0 (успешно)
CMD composer install --prefer-source --no-interaction