FROM ubuntu:16.04

RUN apt-get update && apt-get install -y \
    cron \
    rsyslog

COPY ./cron /etc/cron.d/sample

CMD service rsyslog start && service cron start && tail -f /var/log/syslog