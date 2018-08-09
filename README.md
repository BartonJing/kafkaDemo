环境搭建

zookeeper-3.4.10：

    1.配置zookeeper  的配置文件  conf/zoo.cfg
        dataDir=D:/bigdata/zookeeper-3.4.10/data
        其他参数默认
    2.启动zookeeper   
        bin/zkServer.cmd
        
kafka_2.11-2.0.0:

    1.进入kafka配置文件所在目录，D:\bigdata\kafka_2.11-0.9.0.1\config
    2.编辑文件"server.properties"，找到并编辑：
        log.dirs=/tmp/kafka-logs  to  log.dirs=D:/bigdata/kafka_2.11-0.9.0.1/kafka-logs 或者 D:\\bigdata\\kafka_2.11-0.9.0.1\\kafka-logs
        同样注意：路径要么是"/"分割，要么是转义字符"\\"，这样会生成正确的路径(层级，子目录)。错误路径情况可自行尝试，文件夹名为这种形式：bigdatakafka_2.11-0.9.0.1kafka-logs
    3.在server.properties文件中，zookeeper.connect=localhost:2181代表kafka所连接的zookeeper所在的服务器IP以及端口，可根据需要更改。本文在同一台机器上使用，故不用修改。
    4.kafka会按照默认配置，在9092端口上运行，并连接zookeeper的默认端口2181
    5.修改 config下的log4j.properties
        log4j.appender.kafkaAppender.File=D:\\bigdata\\kafka-log\\server.log  日志地址
        log4j.logger.kafka.controller=INFO, controllerAppender                日志级别
    6.启动kafka 进入bin目录
        ./windows/kafka-server-start.bat ../config./server.properties
    7.创建主题  进入bin目录下
        ./windows/kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test0811
    8.创建生产者(producer)和消费者(consumer)
        ./windows/kafka-console-producer.bat --broker-list localhost:9092 --topic test0811 生产者
        #./windows/kafka-console-consumer.bat --zookeeper localhost:2181 --topic test0811   消费者   2.11版本不合适
        ./windows/kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test0811 --from-beginning  消费者 此命令可以使用
    
    

客户端Demo:


生产者：


    public class ProducerDemo {
        public static void main(String [] args){
            Properties props = new Properties();
            props.put("bootstrap.servers", "localhost:9092");
            //props.put("client.id", "DemoProducer");
            props.put("acks", "all");
            props.put("retries", 0);
            props.put("batch.size", 16384);
            props.put("linger.ms", 1);
            props.put("buffer.memory", 33554432);
            props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
            props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
     
            Producer<String, String> producer = new KafkaProducer<String, String>(props);
            for (int i = 0; i < 100; i++)
                producer.send(new ProducerRecord<String, String>("test0811", Integer.toString(i), Integer.toString(i)));
     
            producer.close();
     
        }
    }
消费者：




    public class ConsumerDemo {
        public static void main(String [] args){
            Properties props = new Properties();
            props.put("bootstrap.servers", "localhost:9092");
            props.put("group.id", "test-consumer-group");//consumer.properties 中的 group.id
            props.put("enable.auto.commit", "true");
            props.put("auto.commit.interval.ms", "1000");
            props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
            props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
            KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
            consumer.subscribe(Arrays.asList("test0811"));//设置主题，可多个
            while (true) {
                System.out.println("********");
                ConsumerRecords<String, String> records = consumer.poll(1000);
                for (ConsumerRecord<String, String> record : records)
                    System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
            }
        }
    }
