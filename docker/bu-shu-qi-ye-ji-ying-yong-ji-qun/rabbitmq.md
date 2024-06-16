# RabbitMQ

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 8.2 部署RabbitMQ

> 部署带管理控制台的RabbitMQ

{% hint style="info" %}
端口说明：

&#x20;Erlang发现&集群端口    4369, 25672&#x20;

AMQP端口                        5672, 5671&#x20;

web管理后台端口            15672&#x20;

STOMP协议端口              61613, 61614&#x20;

MQTT协议端口                1883, 8883&#x20;
{% endhint %}

```
# docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 -v /opt/rabbitmq:/var/lib/rabbitmq rabbitmq:management
```

```
# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS         PORTS                                                                                                                                                                                                                                             NAMES
97d28093faa4   rabbitmq:management   "docker-entrypoint.s…"   11 seconds ago   Up 6 seconds   0.0.0.0:4369->4369/tcp, :::4369->4369/tcp, 0.0.0.0:5671-5672->5671-5672/tcp, :::5671-5672->5671-5672/tcp, 0.0.0.0:15671-15672->15671-15672/tcp, :::15671-15672->15671-15672/tcp, 0.0.0.0:25672->25672/tcp, :::25672->25672/tcp, 15691-15692/tcp   rabbitmq
```



\
