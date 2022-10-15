FROM mysql:latest
COPY my.cnf /etc/mysql/conf.d
COPY master.sh /docker-entrypoint-initdb.d/
CMD ["mysqld"]
