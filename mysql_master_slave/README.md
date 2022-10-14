# docker_master_slave_sample
## my.cnf配置说明

```
[mysqld]
## 设置server_id，同一局域网中需要唯一
server-id=100
## 端口
port=3306
## 加密方法
default-authentication-plugin=mysql_native_password
## 最大连接数
max_connections=800
## 对于同一主机，如果有超出该参数值个数的中断错误连接，则该主机将被禁止连接
max_connect_errors=1000
## 开启binlog							
log_bin=ON
## 慢查询
slow_query_log=1
## 慢查询日志
slow_query_log_file=/var/lib/mysql/slow.log
## 错误日志路径
log_error=/var/lib/mysql/mysql.err

## slave从m复制的数据会写入log-bin日志文件里
log-slave-updates=ON
##  保证事务安全完整性的
enforce-gtid-consistency=ON
## slave进行同步复制的时候，无须找到binlog日志和POS点
gtid_mode=ON

## 不记录information_schema的二进制日志
binlog-ignore-db=information_schema
## 不记录mysql
binlog-ignore-db=mysql
## 不记录sys
binlog-ignore-db=sys
## 不记录performance_schema
binlog-ignore-db=performance_schema

## STRICT_TRANS_TABLES:如果一个值不能插入到一个事务表中,则中断当前的操作,对非事务表不做限制
## NO_ZERO_IN_DATE:不允许日期和月份为零
## NO_ZERO_DATE:不允许插入零日期,插入零日期会抛出错误而不是警告
## ERROR_FOR_DIVISION_BY_ZERO:在INSERT或UPDATE过程中,如果数据被零除,则产生错误而非警告
## NO_ENGINE_SUBSTITUTION:如果需要的存储引擎被禁用或未编译,那么抛出错误
sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

## docker-compose配置说明

```
version: '3'
services:
  mysql_master:
    container_name: mysql_master
    environment:
      - MYSQL_ROOT_PASSWORD=qwer1234
      - MYSQL_DATABASE=test_db
      - MYSQL_USER=test
      - MYSQL_PASSWORD=123456
      - MASTER_SYNC_USER=sync_admin
      - MASTER_SYNC_PASSWORD=sync_admin
      - ADMIN_USER=root
      - ADMIN_PASSWORD=qwer1234
      - TZ=Asia/Shanghai
    image: mysql
    build:
      context: ./mysql_master
    ports:
      - 4306:3306
    volumes:
      - ./mysql_master/data:/var/lib/mysql
    networks:
      shardingSphere:
        ipv4_address: 192.168.150.10
    restart: always

  mysql_slave1:
    container_name: mysql_slave1
    image: mysql
    build:
      context: ./mysql_slave/slave1
    environment:
      - MYSQL_ROOT_PASSWORD=qwer1234
      - MYSQL_DATABASE=test_db
      - MYSQL_USER=test
      - MYSQL_PASSWORD=123456
      - MASTER_SYNC_USER=sync_admin
      - MASTER_SYNC_PASSWORD=sync_admin
      - ADMIN_USER=root
      - ADMIN_PASSWORD=qwer1234
      - MASTER_HOST=192.168.150.10
      - MASTER_PORT=4306
      - TZ=Asia/Shanghai
    ports:
      - 5306:3306
    volumes:
      - "./mysql_slave/slave1/data:/var/lib/mysql"
    restart: always
    networks:
      shardingSphere:
        ipv4_address: 192.168.150.11
    depends_on:
      - mysql_master

  mysql_slave2:
    container_name: mysql_slave2
    image: mysql
    build:
      context: ./mysql_slave/slave2
    environment:
      - MYSQL_ROOT_PASSWORD=qwer1234
      - MYSQL_DATABASE=test_db
      - MYSQL_USER=test
      - MYSQL_PASSWORD=123456
      - MASTER_SYNC_USER=sync_admin
      - MASTER_SYNC_PASSWORD=sync_admin
      - ADMIN_USER=root
      - ADMIN_PASSWORD=qwer1234
      - MASTER_HOST=192.168.150.10
      - MASTER_PORT=4306
      - TZ=Asia/Shanghai
    ports:
      - 5307:3306
    volumes:
      - "./mysql_slave/slave2/data:/var/lib/mysql"
    restart: always
    networks:
      shardingSphere:
        ipv4_address: 192.168.150.12
    depends_on:
      - mysql_master

  mysql_slave3:
    container_name: mysql_slave3
    image: mysql
    build:
      context: ./mysql_slave/slave3
    environment:
      - MYSQL_ROOT_PASSWORD=qwer1234
      - MYSQL_DATABASE=test_db
      - MYSQL_USER=test
      - MYSQL_PASSWORD=123456
      - MASTER_SYNC_USER=sync_admin
      - MASTER_SYNC_PASSWORD=sync_admin
      - ADMIN_USER=root
      - ADMIN_PASSWORD=qwer1234
      - MASTER_HOST=192.168.150.10
      - MASTER_PORT=4306
      - TZ=Asia/Shanghai
    ports:
      - 5308:3306
    volumes:
      - "./mysql_slave/slave3/data:/var/lib/mysql"
    restart: always
    networks:
      shardingSphere:
        ipv4_address: 192.168.150.13
    depends_on:
      - mysql_master
networks:
  shardingSphere:
    ipam:
      driver: default
      config:
        - subnet: "192.168.150.0/24"
```

> 提示
>
> 由于每个slave都有脚本，且通过`environment`的`MASTER_HOST`和`MASTER_PORT`指定从库，所以需要指定IP
> 所以使用`docker-compose`创建`networks`和`subnet`，再只当主库和从库的`ipv4_address`
> 在生产环境中，需根据自己的业务需求去配置`networks`
