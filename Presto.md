# Background

Presto is a [Apache licensed](https://github.com/prestodb/presto/blob/master/LICENSE), distributed ANSI SQL query engine initially developed by [Facebook](https://www.facebook.com/notes/facebook-engineering/presto-interacting-with-petabytes-of-data-at-facebook/10151786197628920/). Facebook developed Presto to decrease query latency and in turn increase query throughput against their 300 petabyte data warehouse. Since it's initial release, Presto has seen adoption and contributions from the likes of Airbnb, [Netflix](https://medium.com/netflix-techblog/using-presto-in-our-big-data-platform-on-aws-938035909fd4), [Uber](https://eng.uber.com/presto/), and Walmart. [Presto Enterprise](https://www.starburstdata.com/presto-enterprise/) is a commercially supported distribution of Presto made available by [Starburst](https://www.starburstdata.com/).

# Object Storage

Presto's connector API allows queries across a variety of data sources, including object storage. Object storage is often the most economical way of storing large volumes of data, which makes it attractive for large data warehouses. A common architectural pattern in the public cloud is to use Presto to analyze data stored in Amazon S3. This pattern is so prevalent that it has been expressed as a service in the form of [Amazon Athena](https://aws.amazon.com/athena/).

There is a growing number of organizations who are interested in extending this architectural pattern, or even developing an equivalent service, on their own infrastructure. The overarching goal is t provide data engineers and data analysts with a similar experience, regardless of where the data exists. This guide details how to configure Presto to query data stored as objects using the S3 compatable API as provided by [Red Hat Ceph Storage](https://www.redhat.com/en/technologies/storage/ceph).

# Hive Connector

The [Hive connector](https://prestodb.io/docs/current/connector/hive.html) makes it possible for Presto to query data sets cataloged by a [Hive Metastore](https://cwiki.apache.org/confluence/display/Hive/Design#Design-Metastore). This includes Hive external tables which reference a path to a S3 bucket or prefix.


Hive databases, tables, or partitions can include a LOCATION that maps to a S3 bucket or S3 bucket prefix. As an example, an entire database could be created through the hive cli such that all of its tables exist within the bucket "example", under the prefix "mydb":

```
create database mydb location 's3a://example/mydb'
```

# Hive Metastore





# Presto Configuration

In addition to this quickstart guide, please refer to the Presto upstream documentation for the [Hive Connector](https://prestodb.io/docs/current/connector/hive.html) for comprehensive coverage of all the supported properties. Configuration properties for Presto should be written to a properties file under *etc/catalog/hive.properties*.

In the configuration we will describe, Presto will source table schema from the Hive metastore. First, we detail the connector to be used by Presto, and in this case it will be ```hive-hadoop```. The value for the ```hive.metastore.uri``` configuration parameter should be the URI of the Hive metastore thrift service. If you have any other relevant configuration files from an adjacent Hadoop deployment, you can provide them as a list to Presto and they will be sourced during the coordinator / worker startup sequence.


```
connector.name=hive-hadoop2
hive.metastore.uri=thrift://THRIFT_URI:9083
hive.config.resources=/path/to/core-site.xml,/path/to/hdfs-site.xml
```



```
hive.s3.endpoint=s3.example.com
hive.s3.ssl.enabled=true
hive.s3.aws-access-key=AWS_ACCESS_KEY_ID
hive.s3.aws-secret-key=AWS_SECRET_ACCESS_KEY
hive.s3.use-instance-credentials=false
hive.s3.path-style-access=true
```

```
hive.s3.staging-directory={{ presto_staging_dir }}
hive.s3.sse.enabled=false
hive.s3.max-connections=100
hive.s3.connect-timeout=2m

hive.s3.max-client-retries=20
hive.s3.max-error-retries=20

```

# Performance Considerations

The Presto query planner can optimize queries using any table statistics that may have been calculated and stored in the Hive metastore. If you plan on running more than a few queries against a given table, it's recommended to [calculate table statistics with Hive](https://cwiki.apache.org/confluence/display/Hive/StatsDev).
