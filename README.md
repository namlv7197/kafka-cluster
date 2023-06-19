# Create Kafka cluster on AWS
Create Kafka cluster with 3 nodes on AWS (Kafka cluster terminated for lack of money)

Author: Le Viet Nam

Note: This is a simple Kafka cluster configuration

## Create Zookeeper server
### Create Zookeeper VPC
- IPv4 CIDR: 10.10.1.0/24

### Create Zookeeper Subnet
- Subnet name: kafka-zookeeper
- IPv4 CIDR block: 10.10.1.0/28 (addresses in range 0 -> 31)

Notes: Rememeber to create Internet gateway named "zookeeper-igw" for Zookeeper VPC and attach it. Then add "zookeeper-igw" to Zookeeper Subnet Route tables.
### Launch Zookeeper instance for Kafka cluster
Amazon Machine Image             |  Network settings
:-------------------------:|:-------------------------:
![](https://github.com/namlv7197/kafka-cluster/blob/main/kafka-zookeeper-ami.png)  |  ![](https://github.com/namlv7197/kafka-cluster/blob/main/kafka-zookeeper-networks.png)
Assumption:
- Public IPv4 address: 54.254.152.12
- Private IPv4 addresses: 10.10.1.4
### Configure Zookeeper instance
Modify /etc/ssh/sshd_config
```
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
sudo systemctl restart sshd
```
Add password for root user
```
sudo passwd root
```
Download [Zookeeper](https://dlcdn.apache.org/zookeeper/zookeeper-3.8.1/apache-zookeeper-3.8.1-bin.tar.gz) and extract it
```
cd /home/ubuntu/
wget https://dlcdn.apache.org/zookeeper/zookeeper-3.8.1/apache-zookeeper-3.8.1-bin.tar.gz
tar -xvf apache-zookeeper-3.8.1-bin.tar.gz
mv apache-zookeeper-3.8.1-bin zookeeper
```
Replace zoo_sample.cfg to zoo.cfg
```
cd /home/ubuntu/zookeeper
mv conf/zoo_sample.cfg conf/zoo.cfg
```
Install Java 8
```
sudo apt-get update
sudo apt install openjdk-8-jdk -y
```
## Create Kafka cluster
### Create Kafka VPC
- IPv4 CIDR: 10.10.2.0/24

### Create Kafka Subnet
- Subnet name: kafka-zookeeper
- IPv4 CIDR block: 10.10.2.0/28 (addresses in range 0 -> 31)

Notes: Rememeber to create Internet gateway named "kafka-igw" for Kafka VPC and attach it. Then add "kafka-igw" to Zookeeper Subnet Route tables.

### Launch Kafka broker 1 instance
Amazon Machine Image             |  Network settings
:-------------------------:|:-------------------------:
![](https://github.com/namlv7197/kafka-cluster/blob/main/kafka-broker-1-ami.png)  |  ![](https://github.com/namlv7197/kafka-cluster/blob/main/kafka-broker-1-networks.png)
Assumption:
- Public IPv4 address: 54.179.7.184
- Private IPv4 addresses: 10.10.2.4
### Configure Kafka broker instance
Modify /etc/ssh/sshd_config
```
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
sudo systemctl restart sshd
```
Add password for root user
```
sudo passwd root
```
Download [Kafka](https://dlcdn.apache.org/kafka/3.5.0/kafka_2.13-3.5.0.tgz) and extract it.
```
cd /home/ubuntu/
wget https://dlcdn.apache.org/kafka/3.5.0/kafka_2.13-3.5.0.tgz
tar -xvf kafka_2.13-3.5.0.tgz
mv kafka_2.13-3.5.0 kafka
```
Modify config/server.properties
```
cd /home/ubuntu/kafka
sed -i 's/zookeeper.connect=localhost:2181/zookeeper.connect=10.10.1.4:2181/g' config/server.properties
sed -i '$alisteners=INTERNAL://0.0.0.0:19092,EXTERNAL://0.0.0.0:9092' config/server.properties
sed -i '$alistener.security.protocol.map=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT' config/server.properties
sed -i '$aadvertised.listeners=INTERNAL://10.10.2.4:19092,EXTERNAL://54.179.7.184:9092' config/server.properties
sed -i '$ainter.broker.listener.name=INTERNAL' config/server.properties
```
Install Java 8
```
sudo apt-get update
sudo apt install openjdk-8-jdk -y
```
### Start Zookeeper server and Kafka server
- Create Peering connections between Kafka VPC and Zookeeper VPC named "kafka-zookeeper"
- Add "kafka-zookeeper" to Kakfa Route tables with destination 10.10.1.0/24
- Add "kafka-zookeeper" to Zookeeper Route tables with destination 10.10.2.0/24
- Add inbound rules to kafka-broker-1 security group
  - All TCP with source 10.10.1.0/28
  - All ICMP - IPv4 with source 10.10.1.0/28
- Add inbound rules to kafka-zookeeper security group
  - All TCP with source 10.10.2.0/28
  - All ICMP - IPv4 with source 10.10.2.0/28
#### Start Zookeeper at kafka-zookeeper instance
Create kafka_zookeeper.service [Example](https://github.com/namlv7197/kafka-cluster/blob/main/kafka_zookeeper.service)
```
sudo nano /etc/systemd/system/kafka_zookeeper.service
sudo systemctl enable kafka_zookeeper.service
sudo systemctl start kafka_zookeeper.service
```
#### Start Kafka at kafka-broker-1 instance
Create kafka.service [Example](https://github.com/namlv7197/kafka-cluster/blob/main/kafka.service)
```
sudo nano /etc/systemd/system/kafka.service
sudo systemctl enable kafka.service
sudo systemctl start kafka.service
```

Create kafka-broker-2 and kafka-broker 3 instances and update ```num.paritions=3``` and ```offsets.topic.replication.factor=3``` per each broker
- kafka-broker-2:
  - Public IPv4 address: 54.151.183.113
  - Private IPv4 addresses: 10.10.2.11
- kafka-broker-3:
  - Public IPv4 address: 54.254.228.131
  - Private IPv4 addresses: 10.10.2.7

## Producer
```
from confluent_kafka import Producer
import json

p = Producer({'bootstrap.servers': '54.179.7.184:9092,54.151.183.113:9092,54.254.228.131:9092'})

def delivery_report(err, msg):
    """ Called once for each message produced to indicate delivery result.
        Triggered by poll() or flush(). """
    if err is not None:
        print('Message delivery failed: {}'.format(err))
    else:
        print('Message delivered to topic: {} at partition: {}'.format(msg.topic(), msg.partition()))

data=json.dumps({'sentence':'This is the sentence'})
    
p.produce('test-topic', value=data, callback=delivery_report)

p.flush()
```

## Consumer
```
from confluent_kafka import Consumer
import json

c = Consumer({
    'bootstrap.servers': '54.179.7.184:9092,54.151.183.113:9092,54.254.228.131:9092',
    'group.id': 'test',
    'auto.offset.reset': 'earliest'
})

c.subscribe(['test-topic'])

while True:
    msg = c.poll(1.0)

    if msg is None:
        continue
    if msg.error():
        print("Consumer error: {}".format(msg.error()))
        continue
    msg=msg.value().decode('utf-8')
    msg=json.loads(msg)
    print('Received message: {}'.format(msg))

c.close()
```
## Todo
- Add security configuration
