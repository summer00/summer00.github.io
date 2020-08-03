# 接收指定offset的消息

实现`ConsumerSeekAware`

```java
void registerSeekCallback(ConsumerSeekCallback callback);

void onPartitionsAssigned(Map<TopicPartition, Long> assignments, ConsumerSeekCallback callback);

void onIdleContainer(Map<TopicPartition, Long> assignments, ConsumerSeekCallback callback);
```