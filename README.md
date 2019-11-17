# Spark writer
```bash
docker exec -it adhoc /bin/bash

# Orders
/spark/bin/spark-submit \
--packages org.apache.hadoop:hadoop-aws:2.7.7 \
--conf spark.driver.extraJavaOptions='-Dcom.amazonaws.services.s3.enableV4' \
--conf spark.executor.extraJavaOptions='-Dcom.amazonaws.services.s3.enableV4' \
--conf spark.hadoop.fs.s3a.endpoint=s3.ap-south-1.amazonaws.com \
--conf spark.hadoop.fs.s3a.access.key=<ACCESS_KEY> \
--conf spark.hadoop.fs.s3a.secret.key=<SECRET_KEY> \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
/var/demo/config/spark-packages/hudi-utilities-bundle-0.5.0-incubating.jar \
--storage-type COPY_ON_WRITE \
--source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
--source-ordering-field order_date \
--target-base-path s3a://atlan-dhs/lake/orders \
--target-table orders \
--props /var/demo/config/deltastreamer/orders-kafka-source.properties \
--schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
--continuous
--filter-dupes

# Customers
/spark/bin/spark-submit \
--packages org.apache.hadoop:hadoop-aws:2.7.7 \
--conf spark.driver.extraJavaOptions='-Dcom.amazonaws.services.s3.enableV4' \
--conf spark.executor.extraJavaOptions='-Dcom.amazonaws.services.s3.enableV4' \
--conf spark.hadoop.fs.s3a.endpoint=s3.ap-south-1.amazonaws.com \
--conf spark.hadoop.fs.s3a.access.key=<ACCESS_KEY> \
--conf spark.hadoop.fs.s3a.secret.key=<SECRET_KEY> \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
/var/demo/config/spark-packages/hudi-utilities-bundle-0.5.0-incubating.jar \
--storage-type COPY_ON_WRITE \
--source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
--source-ordering-field partition_date \
--target-base-path s3a://atlan-dhs/lake/customers \
--target-table customers \
--props /var/demo/config/deltastreamer/customers-kafka-source.properties \
--schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider

```


# Spark queries
```bash
docker exec -it adhoc /bin/bash

/spark/bin/spark-shell \
--packages org.apache.hadoop:hadoop-aws:2.7.7 \
--conf spark.driver.extraJavaOptions='-Dcom.amazonaws.services.s3.enableV4' \
--conf spark.executor.extraJavaOptions='-Dcom.amazonaws.services.s3.enableV4' \
--conf spark.hadoop.fs.s3a.endpoint=s3.ap-south-1.amazonaws.com \
--conf spark.hadoop.fs.s3a.access.key=<ACCESS_KEY> \
--conf spark.hadoop.fs.s3a.secret.key=<SECRET_KEY> \
--conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer' \
--jars /var/demo/config/spark-packages/hudi-spark-bundle-0.5.0-incubating.jar
```

```scalaval ordersDF = spark.
    read.
    format("org.apache.hudi").
    load("s3a://atlan-dhs/lake/orders/*/*/*/*")


ordersDF.show()

ordersDF.registerTempTable("orders")
spark.sql("select count(*) from orders").show()

val customersDF = spark.
    read.
    format("org.apache.hudi").
    load("s3a://atlan-dhs/lake/customers/*/*/*/*")

customersDF.show()

customersDF.registerTempTable("customers")
spark.sql("select count(*) from customers").show()


// Analysis
spark.sql("select contact_title, count(*) as c from customers group by contact_title order by c desc").show()

// Get num of orders placed by customer designation
spark.sql("select contact_title, count(*) as count from customers c inner join orders o on c.customer_id=o.customer_id group by contact_title order by count desc").show()
```


# Postgres setup
```bash
docker exec -it postgres /bin/bash

psql -U postgres
create database dhs;
\q

psql -U postgres -d dhs
```

```sql
CREATE TABLE customers (
    customer_id varchar(255) primary key,
    company_name varchar(255),
    contact_name varchar(255),
    contact_title varchar(255),
    address varchar(255),
    city varchar(255),
    region varchar(255),
    postal_code varchar(255),
    country varchar(255),
    phone varchar(255),
    fax varchar(255),
    partition_date varchar(255)
);

COPY customers(customer_id,company_name,contact_name,contact_title,address,city,region,postal_code,country,phone,fax,partition_date) 
FROM '/var/demo/config/deltastreamer/customers.csv' DELIMITER ',' CSV HEADER;
```
