title: RocketMQ快速入门
date: 2015-06-03 08:48:23
categories: rocketmq
tags: [rocketmq,mq]
---


## RocketMQ 是什么?

1. 是一个队列模型的消息中间件，具有高性能、高可靠、高实时、分布式特点。
2. Producer、Consumer、队列都可以分布式。
3. Producer 向一些队列轮流发送消息，队列集合称为Topic，Consumer 如果做广播消费，则一个consumer
实例消费这个Topic 对应的所有队列，如果做集群消费，则多个Consumer 实例平均消费这个topic 对应的队列集合。
4. 能够保证严格的消息顺序
5. 提供丰富的消息拉取模式
6. 高效的订阅者水平扩展能力
7. 实时的消息订阅机制
8. 亿级消息堆积能力
9. 较少的依赖

以上为RocketMQ的主要特征，也说明了RocketMQ是什么。


## RocketMQ下载地址
去github https://github.com/alibaba/RocketMQ 下载clone,当前使用版本V3.2.6。


## RocketMQ安装
运行install.bat，将target下的alibaba-rocketmq-3.2.6-alibaba-rocketmq.tar.gz 移动到指定的安装目录并解压。

## RocketMQ运行
进入bin目录下  
依次执行  
	
	$ ./mqnamesrv
	The Name Server boot success.

	$ ./mqbroker -n "192.168.133.1:9876"
	The broker[kirs-PC, 192.168.133.1:10911] boot success. and name server is 192.168.133.1:9876

NameSrv和Broker成功运行起来了。

## 示例运行

**生产者**

	public class Producer {

    public static void main(String[] args) throws MQClientException,
            InterruptedException {
        /**
         * 一个应用创建一个Producer，由应用来维护此对象，可以设置为全局对象或者单例<br>
         * 注意：ProducerGroupName需要由应用来保证唯一<br>
         * ProducerGroup这个概念发送普通的消息时，作用不大，但是发送分布式事务消息时，比较关键，
         * 因为服务器会回查这个Group下的任意一个Producer
         */
        final DefaultMQProducer producer = new DefaultMQProducer("ProducerGroupName");
        producer.setNamesrvAddr("192.168.133.1:9876");
        producer.setInstanceName("Producer");

        /**
         * Producer对象在使用之前必须要调用start初始化，初始化一次即可<br>
         * 注意：切记不可以在每次发送消息时，都调用start方法
         */
        producer.start();

        /**
         * 下面这段代码表明一个Producer对象可以发送多个topic，多个tag的消息。
         * 注意：send方法是同步调用，只要不抛异常就标识成功。但是发送成功也可会有多种状态，<br>
         * 例如消息写入Master成功，但是Slave不成功，这种情况消息属于成功，但是对于个别应用如果对消息可靠性要求极高，<br>
         * 需要对这种情况做处理。另外，消息可能会存在发送失败的情况，失败重试由应用来处理。
         */
        for (int i = 0; i < 100000; i++) {
            try {
                {
                    Message msg = new Message("TopicTest1", "TagA", "OrderID001", ("Hello MetaQA").getBytes());
                    SendResult sendResult = producer.send(msg);
                    System.out.println(sendResult);
                }

                {
                    Message msg = new Message("TopicTest2", "TagB", "OrderID0034", ("Hello MetaQB").getBytes());
                    SendResult sendResult = producer.send(msg);
                    System.out.println(sendResult);
                }

                {
                    Message msg = new Message("TopicTest3", "TagC", "OrderID061", ("Hello MetaQC").getBytes());
                    SendResult sendResult = producer.send(msg);
                    System.out.println(sendResult);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
            TimeUnit.MILLISECONDS.sleep(10);
        }

        /**
         * 应用退出时，要调用shutdown来清理资源，关闭网络连接，从RocketMQ服务器上注销自己
         * 注意：我们建议应用在JBOSS、Tomcat等容器的退出钩子里调用shutdown方法
         */
	//    producer.shutdown();
	        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
	            public void run() {
	                producer.shutdown();
	            }
	        }));
	        System.exit(0);
	    }
	}


**消费者**

	public class Consumer {

    /**
     * 当前例子是PushConsumer用法，使用方式给用户感觉是消息从RocketMQ服务器推到了应用客户端。<br>
     * 但是实际PushConsumer内部是使用长轮询Pull方式从RocketMQ服务器拉消息，然后再回调用户Listener方法<br>
     */
    public static void main(String[] args) throws InterruptedException, MQClientException {
        /**
         * 一个应用创建一个Consumer，由应用来维护此对象，可以设置为全局对象或者单例<br>
         * 注意：ConsumerGroupName需要由应用来保证唯一
         */
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ConsumerGroupName");
        consumer.setNamesrvAddr("192.168.133.1:9876");
        consumer.setInstanceName("Consume");

        /**
         * 订阅指定topic下tags分别等于TagA或TagC或TagD
         */
        consumer.subscribe("TopicTest1", "TagA || TagC || TagD");
        /**
         * 订阅指定topic下所有消息<br>
         * 注意：一个consumer对象可以订阅多个topic
         */
        consumer.subscribe("TopicTest2", "*");

        consumer.registerMessageListener(new MessageListenerConcurrently() {

            public ConsumeConcurrentlyStatus consumeMessage(
                    List<MessageExt> msgs, ConsumeConcurrentlyContext context) {

                System.out.println(Thread.currentThread().getName() + " Receive New Messages: " + msgs.size());

                MessageExt msg = msgs.get(0);
                if (msg.getTopic().equals("TopicTest1")) {
                    //执行TopicTest1的消费逻辑
                    if (msg.getTags() != null && msg.getTags().equals("TagA")) {
                        //执行TagA的消费
                        System.out.println(new String(msg.getBody()));
                    } else if (msg.getTags() != null
                            && msg.getTags().equals("TagC")) {
                        //执行TagC的消费
                        System.out.println(new String(msg.getBody()));
                    } else if (msg.getTags() != null
                            && msg.getTags().equals("TagD")) {
                        //执行TagD的消费
                        System.out.println(new String(msg.getBody()));
                    }
                } else if (msg.getTopic().equals("TopicTest2")) {
                    System.out.println(new String(msg.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        /**
         * Consumer对象在使用之前必须要调用start初始化，初始化一次即可<br>
         */
        consumer.start();

        System.out.println("ConsumerStarted.");
    }
	}

