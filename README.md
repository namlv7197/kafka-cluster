# Create Kafka cluster on AWS
Create Kafka cluster with 3 nodes on AWS
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
sed -i '$aserver.1=10.10.4.6:2888:3888' conf/zoo.cfg
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

### Launch Kafka instance

