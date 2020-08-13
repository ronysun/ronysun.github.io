---
layout: post
title: "testlink1.9.16的搭建"
categories:
  - 测试
---
## 摘要

在centos7.3上搭建testlink1.9.16，要求环境：

- apache(httpd)
- php version > 5.6
- mysql(mariadb) > 5.6：mysql-5.5版本在创建数据库时会报错  

centos7.3上php和mysql都需要升级
升级完成后，按照testlink的安装文档:

1. 进行logs和upload_area文件夹的创建和权限修改
2. 在浏览器上点击http://($ip)/testlink/install/index.php，进行安装操作



## Docker部署testlink

[官方连接](https://hub.docker.com/r/bitnami/testlink/)

1. 由于testlink服务与mariadb服务是两个docker容器部署因此需要建立一个网络

```bash
docker network create testlink-tier
```

1. Create a volume for MariaDB persistence and create a MariaDB container

```bash
$ docker volume create --name mariadb_data
$ docker run -d --name mariadb \
 -e ALLOW_EMPTY_PASSWORD=yes \
 -e MARIADB_USER=bn_testlink \
 -e MARIADB_DATABASE=bitnami_testlink \
 --net testlink-tier \
 --volume mariadb_data:/bitnami \
 bitnami/mariadb:latest
```

1. Create volumes for Testlink persistence and launch the container

```bash
$ docker volume create --name testlink_data
$ docker run -d --name testlink -p 8082:80 -p 443:443 \
 -e ALLOW_EMPTY_PASSWORD=yes \
 -e TESTLINK_DATABASE_USER=bn_testlink \
 -e TESTLINK_DATABASE_NAME=bitnami_testlink \
 --net testlink-tier \
 --volume testlink_data:/bitnami \
 bitnami/testlink:1.9.19
```

## 数据库备份/恢复

```bash
docker exec -i 51415d2861be mysqldump -ubn_testlink bitnami_testlink|gzip > /opt/DATA/tlbackup/$the_date'-tlbk.sql.gz'
docker exec -i 51415d2861be mysql -ubn_testlink bitnami_testlink < 2020-02-13-tlbk.sql
```

## Testlink Upgrade

以docker部署方式，升级版本1.9.16 to 1.9.19为例：

1. 首先需要备份数据库
2. Stop docker容器：docker stop testlink
3. rename docker容器，作为备份：docker rename testlink testlink_bak
4. 备份testlink数据文件夹：rsync -a /path/to/testlink-persistence /path/to/testlink-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
5. 使用新版本docker image，并使用原有容器的volume，数据库信息等参数，run一个新的容器
6. 如有必要升级数据库
7. 可能需要运行，但我没用到。Copy your old config_db.inc.php and custom_config.inc.php over to the new directory.

## 升级testlink数据库

```bash
MariaDB [(none)]> use bitnami_testlink_test；
source /opt/bitnami/testlink_install/sql/alter_tables/1.9.17/mysql/DB.1.9.17/step1/db_schema_update.sql
source /opt/bitnami/testlink_install/sql/alter_tables/1.9.17/mysql/DB.1.9.17/stepZ/z_final_step.sql
source /opt/bitnami/testlink_install/sql/alter_tables/1.9.18/mysql/DB.1.9.18/step1/db_schema_update.sql
source /opt/bitnami/testlink_install/sql/alter_tables/1.9.18/mysql/DB.1.9.18/stepZ/z_final_step.sql
source /opt/bitnami/testlink_install/sql/alter_tables/1.9.19/mysql/DB.1.9.19/step1/db_schema_update.sql
source /opt/bitnami/testlink_install/sql/alter_tables/1.9.19/mysql/DB.1.9.19/stepZ/z_final_step.sql

```
