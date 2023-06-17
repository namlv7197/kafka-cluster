# Create Kafka cluster on AWS
Create Kafka cluster with 3 nodes on AWS
## Create Zookeeper server
### Create Zookeeper VPC
- IPv4 CIDR: 10.10.1.0/24

### Create Zookeeper Subnet
- Subnet name: kafka-zookeeper
- IPv4 CIDR block: 10.10.1.0/28 (addresses in range 0 -> 31

Notes: Rememeber to create Internet gateway named "zookeeper-igw" for Zookeeper VPC and attach it. Then add "zookeeper-igw" to Zookeeper Subnet Route tables.
### Launch Zookeeper for Kafka cluster
Solarized dark             |  Solarized Ocean
:-------------------------:|:-------------------------:
![](https://github.com/namlv7197/kafka-cluster/blob/main/kafka-zookeeper-ami.png)  |  ![](https://github.com/namlv7197/kafka-cluster/blob/main/kafka-zookeeper-networks.png)
## Create Kafka cluster
