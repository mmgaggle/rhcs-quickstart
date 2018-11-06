# Background

Presto is an open source SQL engine that was initially designed by [Facebook](https://www.facebook.com/notes/facebook-engineering/presto-interacting-with-petabytes-of-data-at-facebook/10151786197628920/) to increase the volume of queries against their 300 petabyte data warehouse, and get the results of those queries faster. Since it's release, Preso has seen adoption and contribution from tech heavyweights like [Netflix](https://medium.com/netflix-techblog/using-presto-in-our-big-data-platform-on-aws-938035909fd4). [Amazon Athena](https://aws.amazon.com/athena/) is an interactive query service designed to process data in Amazon S3, which is built on Presto. Enterprise versions of Presto are available from [Starburst](https://www.starburstdata.com/).

# Object Storage



This guide details how Presto can be used to query data stored as objects using the S3 compatable API as provided by [Red Hat Ceph Storage](https://www.redhat.com/en/technologies/storage/ceph).

[Starburst](https://www.starburstdata.com/)

# Requirements

* [Hive Metastore](https://cwiki.apache.org/confluence/display/Hive/Design#Design-Metastore)
* [Hive Connector](https://prestodb.io/docs/current/connector/hive.html) (Included in Presto)

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
