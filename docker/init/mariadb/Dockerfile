FROM mariadb:latest AS mariadb

RUN addgroup --gid 1000 appuser
RUN adduser --uid 1000 --gid 1000 --disabled-password appuser
RUN adduser mysql appuser

RUN apt-get -y autoremove \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

USER appuser

WORKDIR /var/lib/mysql
