---
title: RabbitMQ可靠消息最终一致性
date: 2022-10-31 20:29:02
tags: 
- RabbitMQ
index_img: 
banner_img: /img/article4.webp
categories:
- 微服务
- Spring Cloud Alibaba
- RabbitMQ
comment: waline
---

# 1. 实现分布式事务

在实际系统的开发过程中，可能服务间的调用是异步的。也就是说，一个服务发送一个消息给 MQ，即消息中间件，比如RocketMQ、RabbitMQ、Kafka、ActiveMQ 等等。

然后，另外一个服务从 MQ 消费到一条消息后进行处理。这就成了基于 MQ 的异步调用了。那么针对这种基于 MQ 的异步调用，如何保证各个服务间的分布式事务呢？也就是说，我希望的是基于MQ 实现异步调用的多个服务的业务逻辑，要么一起成功，要么一起失败。这个时候，就要用上可靠消息最终一致性方案，来实现分布式事务。

# 2. 可靠消息最终一致性

可靠消息最终一致性方案是指当事务发起方执行完成本地事务后并发出一条消息，事务参与方(消息消费者)一定能够接收消息并处理事务成功，此方案强调的是只要消息发给事务参与方最终事务要达到一致。

事务发起方（消息生产方）将消息发给消息中间件，事务参与方从消息中间件接收消息，事务发起方和消息中间件之间，事务参与方（消息消费方）和消息中间件之间都是通过网络通信，由于网络通信的不确定性会导致分布式事务问题。

# 3. 要解决的问题

### 3.1. 上游服务把信息成功发送

**本地事务与消息发送的原子性问题**：事务发起方在本地事务执行成功后消息必须发出去，否则就回滚事务。即实现本地事务和消息发送的原子性，要么都成功，要么都失败。

### 3.2. 下游服务成把消息成功消费

**事务参与方接收消息的可靠性**：事务参与方必须能够从消息队列接收到消息。

### 3.3. 对消息做幂等

**消息重复消费的问题**：由于MQ到消费者之间网络传输的存在，若某一个消费节点响应超时但是消费成功，此时消息中间件会重复投递此消息，就导致了消息的重复消费。

# 4. 解决方案

<div>
  <img src="https://img-blog.csdnimg.cn/20210128223058373.png">
</div>

### 4.1. 问题一：上游服务把消息成功发送

本地消息表：[该方案](http://maofg.site/arch/2018/12/19/arch-ds-async-mq.html)最初是eBay提出的，在系统A处理任务完成后，在本地记录待发送信息。一个`定时任务`不断检查，是否发送成功，如果发送成功，将记录状态修改。

### 4.2. 问题二：下游服务成把消息成功消费

消息持久化：可保证消息中间件宕机后消息不丢失

手动ack：保证消息投递失败时消息的重新投递

### 4.3. 问题三：对消息做幂等

消息去重表：任务B处理消息前，先查询该消息是否被消费，如果没消费，处理任务B成功，记录消息。如果消息已经被消费，直接返回应答成功

推荐阅读：

[基于RabbitMQ的分布式事务最终一致性解决方案](https://juejin.cn/post/6953986394549305374)

[分布式事务：可靠消息最终一致性方案 ](https://www.cnblogs.com/Libbo/p/16035775.html)

[RabbitMQ消息可靠性投递及分布式事务最终一致性实现](https://blog.csdn.net/forever_and_ever/article/details/98073430)

# 5. RabbitMQ实现可靠消息最终一致性

通过商品信息后台上架添加业务场景实现案例

### 5.1 需求说明：

**shop_item：**

​       1、保存商品信息

​       2、保存本地消息记录

​       3、quartz定时向MQ Server发送添加消息

​       4、在MQ Server响应返回后更新本地消息为已成功发送状态

**shop_search：**

​       1、接收消息，同步到Elasticsearch索引库

​       2、向MQ Server发送应答状态

​       3、对同步索引库方法做幂等

### 5.2 Mysql数据库创建表

数据库中新增local_message表，本地消息记录表(初始化状态为发送失败)，用于保证上游服务把消息成功发送。

```mysql
DROP TABLE IF EXISTS `local_message`;
CREATE TABLE `local_message` (
  `tx_no` varchar(255) NOT NULL, # 主键
  `item_id` bigint DEFAULT NULL, # 商品id
  `state` int(11) DEFAULT NULL,  # 0失败，1成功
  PRIMARY KEY (`tx_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

在数据库中新增msg_distinct，去重表，用于商品信息幂等控制。

```mysql
DROP TABLE IF EXISTS `msg_distinct`;
CREATE TABLE `msg_distinct` (
  `tx_no` varchar(255) NOT NULL,      # 主键
  `create_time` datetime DEFAULT NULL,# 创建时间
  PRIMARY KEY (`tx_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

### 5.3 shop_item

#### application.yml

```yaml
spring:
  rabbitmq:
    host: 10.211.55.18
    port: 5672
    username: admin
    password: 1111
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual #前两种消息模型不自动ack
      direct:
        acknowledge-mode: manual #后三种消息模型不自动ack
    publisher-returns: true #开启消息退回回调
    publisher-confirm-type: correlated #开启消息确认回调
```

#### MQSender

```java
package com.example.mq;

import com.example.mapper.LocalMessageMapper;
import com.example.pojo.LocalMessage;
import com.example.utils.JsonUtils;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.core.RabbitTemplate.ConfirmCallback;
import org.springframework.amqp.rabbit.core.RabbitTemplate.ReturnCallback;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class ItemMQSender implements ReturnCallback, ConfirmCallback{

    @Autowired
    private LocalMessageMapper localMessageMapper;

    @Autowired
    private AmqpTemplate amqpTemplate;

    public void sendMsg(LocalMessage localMessage) {
        RabbitTemplate rabbitTemplate = (RabbitTemplate) this.amqpTemplate;
        //调用当前类的回调逻辑
        rabbitTemplate.setConfirmCallback(this);//确认回调
        rabbitTemplate.setReturnCallback(this);//失败回退

        //用于确认之后更改本地消息状态或删除本地消息--本地消息id
        CorrelationData correlationData = new CorrelationData(localMessage.getTxNo());
        rabbitTemplate.convertAndSend("index_exchange","item.add",                JsonUtils.objectToJson(localMessage),correlationData);
    }

    /**
     * 失败回调
     */
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        System.out.println("return--message:" + new String(message.getBody())  + ",exchange:" + exchange + ",routingKey:" + routingKey);
    }
    /**
     * 确认回调，响应业务之后---修改local_message的status
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if (ack) {
            // 消息发送成功,更新本地消息为已成功发送状态或者直接删除该本地消息记录
            String txNo = correlationData.getId();
            LocalMessage localMessage = new LocalMessage();
            localMessage.setTxNo(txNo);
            localMessage.setState(1);
            localMessageMapper.updateByPrimaryKeySelective(localMessage);
        }
    }
}
```

#### service

```java
    /**
    	* 系统a业务
    	*/
    @Override
    public void insertTbItem(TbItem tbItem, String desc, String itemParams) {
        //保存商品信息
				
        //保存本地消息记录
        LocalMessage localMessage = new LocalMessage();
        // 设置主键
        localMessage.setTxNo(UUID.randomUUID().toString());
        localMessage.setItemId(itemId);
        // 设置上游服务消息状态
        localMessage.setState(0);
        localMessageMapper.insertSelective(localMessage);
    }
```

```java
package com.example.service;

import com.example.mapper.LocalMessageMapper;
import com.example.pojo.LocalMessage;
import com.example.pojo.LocalMessageExample;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
@Service
@Transactional
public class LocalMessageServiceImpl implements LocalMessageService{

    @Autowired
    private LocalMessageMapper localMessageMapper;

    @Override
    public List<LocalMessage> selectlocalMessageByStatus() {
        LocalMessageExample localMessageExample = new LocalMessageExample();
        LocalMessageExample.Criteria criteria = localMessageExample.createCriteria();
        criteria.andStateEqualTo(0);
        return localMessageMapper.selectByExample(localMessageExample);
    }
}
```

#### quartz-job

```java
package com.example.quartz;

import com.example.mq.ItemMQSender;
import com.example.pojo.LocalMessage;
import com.example.service.LocalMessageService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.List;

@Component
public class ItemQuartz {

    @Autowired
    private LocalMessageService localMessageService;

    @Autowired
    private ItemMQSender itemMQSender;

    /**
     * 关闭超时订单
     * 检查本地消息表
     */
    public void scanLocalMessage(){
        System.out.println("执行扫描本地消息表的任务：" + new Date());
        // 发送消息：扫描local_message中status是0的记录，并发送消息
        List<LocalMessage> localMessageList = 
            		localMessageService.selectlocalMessageByStatus();
        for (int i = 0; i < localMessageList.size(); i++) {
            LocalMessage localMessage = localMessageList.get(i);
            itemMQSender.sendMsg(localMessage);
        }
    }
}
```

#### config

```java
package com.example.config;

import com.example.quartz.ItemQuartz;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.quartz.CronTriggerFactoryBean;
import org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean;
import org.springframework.scheduling.quartz.SchedulerFactoryBean;

@Configuration
public class QuartzConfig {

    //job：做什么事
    @Bean
    public MethodInvokingJobDetailFactoryBean 
           			methodInvokingJobDetailFactoryBean(ItemQuartz itemQuartz) {
        MethodInvokingJobDetailFactoryBean JobDetailFactoryBean = 
            							new MethodInvokingJobDetailFactoryBean();
        JobDetailFactoryBean.setTargetObject(itemQuartz);
        JobDetailFactoryBean.setTargetMethod("scanLocalMessage");
        return JobDetailFactoryBean;
    }

    //trigger：什么时候做
    @Bean//trigger（job）
    public CronTriggerFactoryBean cronTriggerFactoryBean(
            				MethodInvokingJobDetailFactoryBean JobDetailFactoryBean) {
        CronTriggerFactoryBean triggerFactoryBean = new CronTriggerFactoryBean();
        triggerFactoryBean.setCronExpression("*/1 * * * * ?");
        triggerFactoryBean.setJobDetail(JobDetailFactoryBean.getObject());
        return triggerFactoryBean;
    }

    //scheduled：什么时候做什么事
    @Bean
    public SchedulerFactoryBean schedulerFactoryBean(
        								CronTriggerFactoryBean triggerFactoryBean) {
        SchedulerFactoryBean schedulerFactoryBean = new SchedulerFactoryBean();
        schedulerFactoryBean.setTriggers(triggerFactoryBean.getObject());
        return schedulerFactoryBean;
    }
}
```

### 5.4 shop_search

#### application.yml

```yaml
spring:
  rabbitmq:
    host: 10.211.55.18
    port: 5672
    username: admin
    password: 1111
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual #前两种消息模型不自动ack
      direct:
        acknowledge-mode: manual #后三种消息模型不自动ack
```

#### service

##### SearchItemServiceImpl

```java
    @Override
    public void insertDocument(Long itemId) throws IOException {
        // 1、根据商品id查询商品信息。
        SearchItem searchItem = searchItemMapper.getItemById(itemId);

        //2、添加商品到索引库
        IndexRequest indexRequest = new IndexRequest(ES_INDEX_NAME, ES_TYPE_NAME);
        indexRequest.source(JsonUtils.objectToJson(searchItem), XContentType.JSON);
        restHighLevelClient.index(indexRequest);
    }
```

##### MsgDistinctServiceImpl

```java
package com.example.service;

import com.example.mapper.MsgDistinctMapper;
import com.example.pojo.MsgDistinct;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.Date;

@Service
@Transactional
public class MsgDistinctServiceImpl implements MsgDistinctService {

    @Autowired
    private MsgDistinctMapper msgDistinctMapper;

    @Override
    public MsgDistinct selectMsgDistinctByTxNo(String txNo) {
        return msgDistinctMapper.selectByPrimaryKey(txNo);
    }

    @Override
    public void insertMsgDistinct(String txNo) {
        MsgDistinct msgDistinct = new MsgDistinct();
        msgDistinct.setTxNo(txNo);
        msgDistinct.setCreateTime(new Date());
        msgDistinctMapper.insertSelective(msgDistinct);
    }
}
```

#### listener

```java
package com.example.listener;

import com.example.pojo.LocalMessage;
import com.example.pojo.MsgDistinct;
import com.example.service.MsgDistinctService;
import com.example.service.SearchItemService;
import com.example.utils.JsonUtils;
import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class SearchMQListener {

    @Autowired
    private MsgDistinctService msgDistinctService;
    @Autowired
    private SearchItemService searchItemService;

    /**
     * 监听者接收消息三要素：
     *  1、queue 
     *  2、exchange 默认持久化
     *  3、routing key
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value="index_queue",durable = "true"),
            exchange = @Exchange(value="index_exchange",type= ExchangeTypes.TOPIC),
            key= {"item.*"}
    ))
    public void listen(String msg, Channel channel, Message message)
            throws IOException {
        System.out.println("接收到消息：" + msg);
        LocalMessage localMessage = JsonUtils.jsonToPojo(msg, LocalMessage.class);

        //进行幂等判断，防止ack应为网络问题没有送达，导致扣减库存业务重复执行
        MsgDistinct msgDistinct =
                msgDistinctService.selectMsgDistinctByTxNo(localMessage.getTxNo());
        if(msgDistinct==null){
            searchItemService.insertDocument(localMessage.getItemId());
            //记录成功执行过的事务
            msgDistinctService.insertMsgDistinct(localMessage.getTxNo());
        }else{
            System.out.println("=======幂等生效：事务"+msgDistinct.getTxNo()
                    +" 已成功执行===========");
        }
     channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
    }
}
```

### 测试

1. 关闭RabbitMQ，添加商品，再开启RabbitMQ，观察local_message表的status
2. 关闭下游服务的手动ack，观察消息是否具有幂等
3. 测试正常添加商品，观察索引库是否同步



<div>
<hr>
<script src="https://unpkg.com/@waline/client@v2/dist/waline.js"></script> 
<link
  rel="stylesheet"
  href="https://unpkg.com/@waline/client@v2/dist/waline.css"
/>
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>