FROM mysql:latest
COPY my.cnf /etc/mysql/conf.d
COPY slave.sh /docker-entrypoint-initdb.d/
CMD ["mysqld"]

