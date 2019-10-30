ARG MYSQL_VERSION=5.6
FROM mysql:${MYSQL_VERSION}

LABEL maintainer="opso <945206413@qq.com>"

ARG TZ=UTC
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && chown -R mysql:root /var/lib/mysql/

COPY my.cnf /etc/mysql/conf.d/my.cnf

CMD ["mysqld"]

EXPOSE 3306