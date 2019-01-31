# 适用于kafka_2.11-1.1.1版本
## 第1步  
将kafka_client_jaas.conf/kafka_server_jaas.conf/kafka_zoo_jaas.conf三个文件放入kafka的config文件夹中，文件中配置用户，admin用户必须配置。  
* kafka_client_jaas.conf内容如下
```
KafkaClient {  
org.apache.kafka.common.security.plain.PlainLoginModule required  
    username="admin"  
    password="admin";  
};
```
* kafka_server_jaas.conf内容如下
```
KafkaServer {
	org.apache.kafka.common.security.plain.PlainLoginModule required
		username="admin"
		password="admin"
		user_admin="admin"
		user_test="test#2018";
};
KafkaClient {
	org.apache.kafka.common.security.plain.PlainLoginModule required
		username="admin"
		password="admin";
};

Client {
	org.apache.kafka.common.security.plain.PlainLoginModule required
		username="admin"
		password="admin";
};
```
* kafka_zoo_jaas.conf内容如下
```
ZKServer{
	org.apache.kafka.common.security.plain.PlainLoginModule required
		username="admin"
		password="admin"
		user_admin="admin";
};
```
## 第2步
修改kafka的bin文件夹中的zookeeper-server-start.sh,  
添加： 
```
export KAFKA_OPTS=" -Djava.security.auth.login.config=/data/application/kafka_2.11-1.1.1/config/kafka_zoo_jaas.conf  -Dzookeeper.sasl.serverconfig=ZKServer"
```
## 第3步
修改kafka的bin文件夹中的kafka-server-start.sh，  
添加： 
```
export KAFKA_OPTS=" -Djava.security.auth.login.config=/data/application/kafka_2.11-1.1.1/config/kafka_server_jaas.conf"
```
## 第4步
修改kafka的bin文件夹中的kafka-console-producer.sh  
添加：
```
export KAFKA_OPTS=" -Djava.security.auth.login.config=/data/application/kafka_2.11-1.1.1/config/kafka_client_jaas.conf"
```
## 第5步
修改kafka的bin文件夹中的kafka-console-consumer.sh  
添加：
```
export KAFKA_OPTS=" -Djava.security.auth.login.config=/data/application/kafka_2.11-1.1.1/config/kafka_client_jaas.conf"
```
## 第6步
修改kafka的config文件夹中的consumer.properties  
添加：  
```
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
```
## 第7步
修改kafka的config文件夹中的producer.properties  
添加：  
```
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
```
## 第8步
修改kafka的config文件夹中的zookeeper.properties  
添加：  
```
authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider
requireClientAuthScheme=sasl
jaasLoginRenew=3600000
```
## 第9步
修改kafka的config文件夹中的server.properties  
修改：  
```
listeners=SASL_PLAINTEXT://192.168.1.115:9092
```  
添加：  
```
#使用的认证协议
security.inter.broker.protocol=SASL_PLAINTEXT
#SASL机制
sasl.enabled.mechanisms=PLAIN
sasl.mechanism.inter.broker.protocol=PLAIN
#完成身份验证的类
authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
#如果没有找到ACL（访问控制列表）配置，则允许任何操作。
#allow.everyone.if.no.acl.found=true
super.users=User:admin

delete.topic.enable=true
auto.create.topics.enable=false
```
## 第10步
启动zookeeper服务  
执行  
`sh bin/zookeeper-server-start.sh config/zookeeper.properties`
## 第11步
启动kafka服务  
执行  
`sh bin/kafka-server-start.sh config/server.properties`
## 第12步
查看topic列表  
执行  
`sh bin/kafka-topics.sh --list --zookeeper localhost:2181`
## 第13步
创建新的topic  
执行  
`sh bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 16 --topic test`
## 第14步
给admin用户授权  
执行  
`sh bin/kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2181 --add --allow-principal User:admin --group=* --topic=*`
## 第15步
给用户test授予某个topic的读写的权限  
执行  
`sh bin/kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2181 --add --allow-principal User:test --operationRead --operationWrite --topic test --group=*`  
说明：  
控制读写：--operationRead--operationWrite  
控制消费组：不控制组 --group=*，指定消费组 --grouptest-comsumer-group

## 第16步
移除权限  
执行  
`sh bin/kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2181 --remove --allow-principal User:test --allow-host 192.168.1.101 --operationRead --operationWrite --topictest`
## 第17步
列出topic为test的所有权限账户  
执行  
`sh bin/kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2181 --list --topic test`
## 第18步
测试启动消费者  
执行  
`sh bin/kafka-console-consumer.sh --bootstrap-server 192.168.1.115:9092 --topic test --from-beginning --consumer.config config/consumer.properties`
## 第19步
测试启动生产者  
执行  
`sh bin/kafka-console-producer.sh --broker-list 192.168.1.115:9092 --topic test --producer.config config/producer.properties`
