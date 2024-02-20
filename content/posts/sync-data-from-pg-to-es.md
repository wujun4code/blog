+++
authors = ["Jun Wu"]
title = "Sync data from postgres to elasticsearch"
date = "2024-01-23"
description = "Guide to debezium and confluentinc connect"
tags = [
    "kafka",
    "debezium",
    "confluentinc",
    "postgres",
    "elasticsearch"
]
categories = [
    "beckend",
    "database",
]
series = ["tech"]
+++

# What you will Learn after reading this

1. Understanding of debezium and the renowned confluentinc
2. Implementation of data synchronization between Postgres and ElasticSearch
3. Deployment of debezium and confluentinc's connect on Kubernetes

<!-- more -->

## Synchronizing Postgres Data to BI Database/ElasticSearch

This requirement is usually to address read-write separation:

1. Complex data analysis is handed over to professional third-party BI data analysis agencies. You need to remap your business data to a new data structure and provide it to BI data service providers.
2. To support full-text search functionality, the first idea is to synchronize everything from the relational database to ElasticSearch. Then, write search interfaces based on business needs. This interface should not directly query the original relational database but should call ElasticSearch's SDK or REST API to meet full-text search requirements.

Therefore, we can understand that with the growth of business volume and perhaps the urgency of requirements, someone or a community will write such a component.

## debezium - Pushing Data Changes from MySql/Postgres/SQL Server/MongoDB to Kafka

To achieve data synchronization, the normal engineering approach is to consider using scheduled tasks, scanning every minute. However, if you are an old-fashioned single-point service, using a single server for a factory, running code from 20 years ago, this timer might not be easy to write. It might deal with tens of millions of data, and the timer task might accidentally fail, and you have to design fault tolerance, retry, and rollback well. This is terrifying.

Perhaps you thought, I can use a timer to push data changes to a queue in message format. But firstly, the health check of this timer task needs to be done well, and you can't run without knowing it, and then after restarting, this job must have the ability to "resume running from breakpoint." You need to figure out a framework to help you implement this. Just thinking about it gives you a headache.

Therefore, even if you think about the queue idea, you need to distinguish between a good and not quite good or unreasonable idea.

Here, let's take MongoDB as an example. MongoDB has been called for this feature by the community for a long time, so it provided a powerful ChangeStream feature after version 3.6. Check it out -> [MongoDB ChangeStreams](https://www.mongodb.com/docs/manual/changeStreams/)

Also, there's a tutorial -> [MongoDB ChangeStreams Tutorial](https://www.mongodb.com/basics/change-streams)

ChangeStream provides a Push-based approach:

> Records similar to messages are generated for operations such as Create/Update/Delete of data, and are pushed to subscribers.

In reality, MongoDB has implemented an internal message queue to push data changes to subscribers.

### debezium - Standardizing Data Changes of Various Databases into Unified Message Format and Pushing to Kafka


First, download the sample code -> [Postgres to ElasticSearch Sync](https://github.com/strapi-extensions/postgres-sync-elasticsearch)

```shell
$ git clone git@github.com:strapi-extensions/postgres-sync-elasticsearch.git
```
Or download the docker-compose.yml file to your local -> [docker-compose.yml](https://raw.githubusercontent.com/strapi-extensions/postgres-sync-elasticsearch/main/docker-compose.yml)

Note: If you want to start Postgres, make sure you have a file locally, refer to ->[Postgres Configuration](https://github.com/strapi-extensions/postgres-sync-elasticsearch/blob/main/postgres/extra.conf)

Let's start all components.

```shell
docker compose up -d --build
```

### Postgres - Set User Permissions and Create Sample Table

```shell
docker ps

# Identify the container ID of Postgres
docker exec -i -t CONTAINER_ID /bin/bash
```

Inside the Postgres container, execute:

```shell
PGPASSWORD=admin123abc psql -U postgres demo
psql (16.1)
Type "help" for help.

> ALTER ROLE "pg-connector" SUPERUSER CREATEDB NOCREATEROLE INHERIT LOGIN REPLICATION NOBYPASSRLS;
> CREATE TABLE public.todos (
	id serial4 PRIMARY KEY NOT NULL,
	title varchar(255) NULL,
	completed bool NULL,
	due timestamp(6) NULL
);
```

### pg-connect - Install pg-sink-connector

Using the open REST API provided by debezium-connect, a new connector can be added, which will start scanning modifications from Postgres and send messages to Kafka.

```shell
curl --location 'http://localhost:8082/connectors' \
--header 'Content-Type: application/json' \
--data '{
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "database.dbname": "demo",
        "database.hostname": "postgresql",
        "database.password": "pg123abc",
        "database.port": "5432",
        "database.user": "pg-connector",
        "name": "pg-sink-connector",
        "table.include.list": "public.todos",
        "message.key.columns": "my_database.users:id",
        "schema.include.list": "public",
        "tasks.max": "1",
        "topic.prefix": "demo",
        "plugin.name": "pgoutput",
        "transforms":"unwrap",
        "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
        "transforms.unwrap.drop.tombstones": "false",
        "transforms.unwrap.delete.handling.mode": "rewrite",
        "transforms.unwrap.add.fields": "table,ts_ms"
    },
    "name": "pg-sink-connector"
}'
```

### Postgres - back to Postgres and Insert a Record

```shell
INSERT INTO public.todos (title, completed, due)
VALUES ('Fix 502 bug', false, '2024-02-15 16:00:00.000');
```

### Kafka - Check if the Message has been Received

Open your browser to http://localhost:8080/topics/demo.public.todos?p=-1&s=50&o=-1#messages

![image](https://github.com/strapi-extensions/postgres-sync-elasticsearch/assets/5119542/47899738-ddbe-4e73-ad37-dbf52c4205b8)

The first step is complete, we can see the message flow of data modifications has entered Kafka. Next, we need to complete the second half of the work, ensuring that Kafka's data lands in Elasticsearch.

### kafka-connect-elasticsearch - Landing Kafka Messages to Elasticsearch

confluentinc-es-connect also provides a REST API to add es-sink-connector.

```shell
curl --location 'http://localhost:8084/connectors' \
--header 'Content-Type: application/json' \
--data '{
    "name": "es-sink-connector",
    "config": {
        "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
        "tasks.max": "1",
        "topics": "demo.public.todos",
        "connection.url": "http://elasticsearch:9200",
        "key.ignore": "false",
        "transforms": "extractKey",
        "transforms.extractKey.type": "org.apache.kafka.connect.transforms.ExtractField$Key",
        "transforms.extractKey.field": "id"
    }
}'
```

## Verify and test

We use a chrome elasticsearch plug-in -> https://github.com/cars10/elasticvue.

![image](https://github.com/strapi-extensions/postgres-sync-elasticsearch/assets/5119542/6b57e212-7d30-4072-b5dc-b25e439fbafb)


### Test 1 - Add a new todo and see if there is a new record in es

```sql
INSERT INTO public.todos (title, completed, due)
VALUES ('Publish new release', false, '2024-02-15 16:00:00.000');
```


### Test 2 - Update a todo to see if es is modified and not added

```sql
UPDATE public.todos SET completed = false, title = 'Create bug issue edited'
WHERE id = 2;
```

We can see that the record with id = 2 has indeed only modified the content in the doc and has not added anything new:

![image](https://github.com/strapi-extensions/postgres-sync-elasticsearch/assets/5119542/96d66f71-8511-4ef4-9a94-950a06622fa3)


verified.


## Summarize

Synchronization of data sources has always been a necessity for various complex systems. Reading and writing are separated, and reading is also classified. Searching, querying lists, and obtaining individual items are all different. Sometimes, if the data sources can be diversified and there is a high-performance system to ensure data consistency It will greatly increase the robustness and scalability of the system, so it is still very important to master this skill. In addition, this article is only a stand-alone case. To truly move to cluster + distribution, you need to use other components.