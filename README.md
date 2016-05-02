# dcos-iot-demo
Demonstrates how to configure a full stack geo-enabled Internet of Things (IoT) solution using <a href="https://mesosphere.com/">Mesosphere's</a> open sourced <a href="https://dcos.io/">Data Center Operating System (DC/OS)</a>, <a href="https://www.docker.com/">Docker</a>, <a href="http://kafka.apache.org/">Kafka</a>, <a href="http://spark.apache.org/">Spark</a>, <a href="https://www.elastic.co/products/elasticsearch">Elasticsearch</a>, and the <a href="https://www.playframework.com/">Play Framework</a>.

## Working locally (no DC/OS):
It is useful to do development and verification locally prior to installing applications onto DC/OS.  This section will walk you through the process of setting up, building, and running individual apps locally prior to installing them onto DC/OS.
### To setup locally:
<pre>
(1) TODO: Setup SBT
(2) TODO: Setup Kafka
(3) TODO: Spark
    ~$ export SPARK_HOME=~/spark-1.6.1-bin-hadoop2.6
</pre>

### To build locally:
<pre>
(1) Build event source:
    eventsource$ sbt assembly
(2) Build Spark analytic tasks:
    esrispatiotemporalanalytics$ sbt assembly
    TODO: add other analytics ...
</pre>

### To run locally:
<pre>
(1) Start Zookeeper:
    kafka_2.11-0.9.0.1$ ./bin/zookeeper-server-start.sh config/zookeeper.properties
(2) Start Kafka:
    kafka_2.11-0.9.0.1$ ./bin/kafka-server-start.sh config/server.properties
(3) Run Source:
    eventsource$ java -jar target/scala-2.11/eventsource-assembly-1.0.jar localhost:9092 source01 4 1000 true
    note: you can verify events are being sent by running a command line Kafka Consumer utility to listen to the topic:
    kafka_2.11-0.9.0.1$ ./bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic source01
(4) Run Spark analytic tasks:
    esrispatiotemporalanalytics$ $SPARK_HOME/bin/spark-submit --class "org.cam.geo.analytics.esri.EsriSpatiotemporalAnalyticTask"
                                     --master local[2] target/scala-2.10/esrispatiotemporalanalytics-assembly-1.0.jar
                                     localhost:9092 source01 source01-consumer-id false
</pre>

## Working on DC/OS:
### To configure a DC/OS
<pre>
(1) TODO: Create a DC/OS cluster on Azure or Amazon
(2) TODO: Install packages on DC/OS:
    In DC/OS dashboard, go to 'Universe' and install the Marathon, Chronos, Kafka & Spark packages.
(3) TODO: Configure the DC/OS-CLI
    ~$ dcos config set core.dcos_url <your DC/OS url>
</pre>

### to run on DC/OS:
<pre>
(1) Configure Kafka topic:
    ~$ dcos kafka broker list
    ~$ dcos kafka topic list
    ~$ dcos kafka topic create source01 --partitions=3 --replication=1
    ~$ dcos kafka topic list
(2) Build, create/upload docker image:
    eventsource$ docker-machine start default
    eventsource$ eval "$(docker-machine env default)"
    eventsource$ docker build -t amollenkopf/event-source .
    eventsource$ docker login
    eventsource$ docker push amollenkopf/event-source
(3) Add Marathon app:
    eventsource$ dcos marathon app add eventsource-docker.json
(4) Verify events are being sent by running command line Kafka Consumer utilities to listen to the topic:
    azureuser@dcos-master-3F983CB-0:~$ wget http://mirror.reverse.net/pub/apache/kafka/0.9.0.1/kafka_2.10-0.9.0.1.tgz
    azureuser@dcos-master-3F983CB-0:~$ tar -xvf kafka_2.10-0.9.0.1.tgz
    azureuser@dcos-master-3F983CB-0:~$ cd kafka_2.10-0.9.0.1
    azureuser@dcos-master-3F983CB-0:~/kafka_2.10-0.9.0.1$ ./bin/kafka-console-consumer.sh --zookeeper master.mesos:2181/kafka --topic source01
</pre>
