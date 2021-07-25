---
title: "Docker部署并破解Jira、Confluence及插件"
date: 2021-07-25
tags: ["",""]
categories: ["",""]
description: ""
summary: ""
draft: false
---

> JIRA是Atlassian公司出品的项目与事务跟踪工具，被广泛应用于缺陷跟踪、客户服务、需求收集、流程审批、任务跟踪、项目跟踪和敏捷管理等工作领域。
>
> Confluence是一个专业的企业知识管理与协同软件，也可以用于构建企业wiki。使用简单，但它强大的编辑和站点管理特征能够帮助团队成员之间共享信息、文档协作、集体讨论，信息推送

# 数据库设置

首先是不建议将数据库部署在Docker容器，建议使用云数据库或者物理机部署。下文以Mysql5.7为例

## confluence

### 文档

[Confluence Data Center and Server documentation](https://confluence.atlassian.com/doc/confluence-data-center-and-server-documentation-135922.html)

[Database Configuration](https://confluence.atlassian.com/doc/database-configuration-159764.html)

### 数据库设置

选择安装的Confluence版本，阅读[Database Setup For MySQL](https://confluence.atlassian.com/doc/database-setup-for-mysql-128747.html)后，修改[Mysql配置文件](https://dev.mysql.com/doc/refman/5.7/en/option-files.html)

以Confluence 7.12为例

```
[mysqld]
...
character-set-server=utf8mb4 
collation-server=utf8mb4_bin
default-storage-engine=INNODB
max_allowed_packet=256M 
innodb_log_file_size=2GB
transaction-isolation=READ-COMMITTED
binlog_format=row
log-bin-trust-function-creators = 1
// 如果为Mysql5.7，关闭derived_merge能优化仪表板加载缓慢
optimizer_switch = derived_merge=off
...
```

如果`sql_mode = NO_AUTO_VALUE_ON_ZERO`，请删除此选项

### 创建数据库&用户

Confluence数据库

```mysql
CREATE DATABASE <database-name> CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
```

Confluence数据库用户

```mysql
GRANT ALL PRIVILEGES ON <database-name>.* TO '<confluenceuser>'@'localhost' IDENTIFIED BY '<password>';
```

如果 Confluence 与数据库不在同一台服务器上运行（或者是Docker用户），请用 Confluence 服务器的主机名或 IP 地址替换 localhost（也可以使用`%`，表示允许所有host）。

## Jira

### 文档

[Jira Software Data Center and Server documentation](https://confluence.atlassian.com/jirasoftwareserver)

[Connecting Jira applications to a database](https://confluence.atlassian.com/adminjiraserver/connecting-jira-applications-to-a-database-938846850.html)

### 数据库设置

选择安装的Jira版本，阅读[Connecting Jira applications to MySQL 5.7](https://confluence.atlassian.com/adminjiraserver/connecting-jira-applications-to-mysql-5-7-966063305.html)后，修改[Mysql配置文件](https://dev.mysql.com/doc/refman/5.7/en/option-files.html)

以Jira 8.18为例

```mysql
[mysqld]
...
character-set-server=utf8mb4 
collation-server=utf8mb4_bin
default-storage-engine=INNODB
innodb_file_format=Barracuda
nodb_default_row_format=DYNAMIC
innodb_large_prefix=ON
innodb_log_file_size=2G
...
```

如果`sql_mode = NO_AUTO_VALUE_ON_ZERO`，请删除此选项

### 创建数据库&用户

同上Confluence创建数据库&用户，不再赘述

# Docker Compose

### 文档

[atlassian/confluence-server](https://hub.docker.com/r/atlassian/confluence-server)

[atlassian/jira-software](https://hub.docker.com/r/atlassian/jira-software)

[atlassian-agent](https://gitee.com/pengzhile/atlassian-agent)

### docker-compose.yml

已上传Github：[aladdinding/Confluence-and-Jira](https://github.com/aladdinding/Confluence-and-Jira)

```yaml
version: '3'
services:
    confluence:
        image: "atlassian/confluence-server"
        volumes: 
            - ./atlassian-agent.jar:/tmp/atlassian-agent.jar
            - /data/your-confluence-home:/var/atlassian/application-data/confluence
        environment: 
            JAVA_OPTS: "-javaagent:/tmp/atlassian-agent.jar ${JAVA_OPTS}"
        ports: 
            - "8090:8090"
        restart: always

    jira:
        image: "atlassian/jira-software"
        volumes: 
            - ./atlassian-agent.jar:/tmp/atlassian-agent.jar
            - /data/your-jira-home:/var/atlassian/application-data/jira
        environment: 
            JAVA_OPTS: "-javaagent:/tmp/atlassian-agent.jar ${JAVA_OPTS}"
        ports: 
            - "8080:8080"
        restart: always
```

# 初始化



# 插件破解

# 配置Confluence与Jira用户数据对接

# 异常记录
