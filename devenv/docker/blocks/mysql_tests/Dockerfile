ARG mysql_version=9.0.1
FROM mysql:${mysql_version}
ADD setup.sql /docker-entrypoint-initdb.d
RUN chown -R mysql:mysql /docker-entrypoint-initdb.d/
CMD ["mysqld"]
