---
title: 'Disruptor'
layout: post
tags:
  - disruptor
category: 框架
---
disruptor并发工具使用记录。

<!--more-->



# 一、mave

```xml
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.4.4</version>
</dependency>
```

# 二、构建队列并使用

## DisruptorUtil

```java
@Component
@Slf4j
public class DisruptorUtil {

    private final Integer RING_BUFFER_SIZE = 1024 * 32;

    /**
     * 创建disruptor
     *
     * @return
     */
    public Disruptor<MessageDto> createDisruptor(String beanName, Integer qps) {
        //单线程模式，获取额外的性能
        Disruptor<MessageDto> disruptor = new Disruptor<MessageDto>(MessageDto::new, RING_BUFFER_SIZE, Executors.defaultThreadFactory(), ProducerType.SINGLE, new YieldingWaitStrategy());
        //创建默认异常
        disruptor.setDefaultExceptionHandler(new DefaultExceptionHandler());
        //设置事件业务处理器---消费者
        WorkHandler[] handlers = new WorkHandler[qps];
        for (int i = 0; i < qps; i++) {
            handlers[i] = SpringUtil.getBean(beanName,WorkHandler.class);
        }
        disruptor.handleEventsWithWorkerPool(handlers);
        disruptor.start();
        return disruptor;
    }

    /**
     * 发送message
     *
     * @param ringBuffer
     * @param msg
     */
    public void publish(RingBuffer<MessageDto> ringBuffer, MessageDto msg) {
        long sequence = ringBuffer.next();
        try {
            MessageDto dto = ringBuffer.get(sequence);
            BeanUtils.copyProperties(msg, dto);
        } catch (Exception e) {
            log.error("failed to add event to MessageVoRingBuffer for : e = {},{}", e, e.getMessage());
        } finally {
            ringBuffer.publish(sequence);
        }
    }
}
```

## DTO

```JAVA
@Data
@ToString
public class MessageDto implements Serializable {
    
}
```

## Handler

```java
@Slf4j
public class DefaultExceptionHandler implements ExceptionHandler<MessageDto> {
    @Override
    public void handleEventException(Throwable ex, long sequence, MessageDto event) {
        log.error("handleEventException {}",ex);
    }

    @Override
    public void handleOnStartException(Throwable ex) {
        log.error("handleOnStartException {}",ex);
    }

    @Override
    public void handleOnShutdownException(Throwable ex) {
        log.error("handleOnShutdownException {}",ex);
    }
}
```

```java
public interface DemoHandler extends WorkHandler<MessageDto> {

    /**
     * 默认执行器
     *
     * @param msg
     * @throws Exception
     */
    @Override
    default void onEvent(MessageDto msg) throws Exception {
        
       
    }
}
```

