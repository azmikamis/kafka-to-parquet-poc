# kafka-to-parquet-poc

## Install Confluent
```
sudo apt-get install apt-transport-https -y
wget -qO - https://packages.confluent.io/deb/5.1/archive.key | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://packages.confluent.io/deb/5.1 stable main"
sudo apt-get update && sudo apt-get install confluent-community-2.11 -y
```
## Some setup
```
mkdir ${HOME}/logs
mkdir ${HOME}/confluent/
export LOG_DIR=${HOME}/logs
export CONFLUENT_CURRENT=${HOME}/confluent/
```
## Modify schema-registry.properties
```
sudo vim /etc/schema-registry/schema-registry.properties
kafkastore.bootstrap.servers as PLAINTEXT://localhost:9092
```
## Get HDFS URL
```
hdfs getconf -confKey fs.defaultFS
```
## Modify kafka-connect-hdfs properties
```
sudo vim /etc/kafka-connect-hdfs/quickstart-hdfs.properties
hdfs.url=<value-from-above-step>
format.class=io.confluent.connect.hdfs.parquet.ParquetFormat
```
## Start Confluent schema-registry
```
confluent start schema-registry
```
## Start standalone Connect
```
connect-standalone /etc/schema-registry/connect-avro-standalone.properties \
  /etc/kafka-connect-hdfs/quickstart-hdfs.properties
```
## Start producer and write some values
```
kafka-avro-console-producer \
  --broker-list localhost:9092 \
  --topic test_hdfs \
  --property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"f1","type":"string"}]}'
{"f1":"value1"}
{"f1":"value2"}
{"f1":"value3"}
```
## Check HDFS
```
hadoop fs -ls /topics/test_hdfs/partition=0
```
## Check file
```
hadoop jar parquet-tools-1.9.0.jar cat --json /topics/test_hdfs/partition=0/test_hdfs+0+0000000000+0000000002.parquet
```
# Guide
- https://docs.confluent.io/current/connect/kafka-connect-hdfs/index.html
# Reference issues
- https://github.com/confluentinc/schema-registry/issues/705
- https://github.com/confluentinc/schema-registry/issues/765
- https://github.com/confluentinc/kafka-connect-hdfs/issues/159
- https://stackoverflow.com/questions/49255878/kafka-conenct-hdfs-sink-saving-data-in-parquet-format

