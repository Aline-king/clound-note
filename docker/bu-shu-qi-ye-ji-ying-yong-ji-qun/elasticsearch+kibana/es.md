# ES

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
# docker pull elasticsearch:7.17.0
```

```
# mkdir -p /opt/es/config
# mkdir -p /opt/es/data
```

```
# echo "http.host: 0.0.0.0" >> /opt/es/config/elasticsearch.yml
```

```
# chmod -R 777 /opt/es/
```

```
# docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /opt/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /opt/es/data:/usr/share/elasticsearch/data \
-v /opt/es/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.17.0
```

```
# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                                                                  NAMES
e1c306e6e5a3   elasticsearch:7.17.0   "/bin/tini -- /usr/l…"   22 seconds ago   Up 20 seconds   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300->9300/tcp, :::9300->9300/tcp   elasticsearch
```

![image-20220212224446838](file:///G:/game/04\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Docker/04\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Docker/05\_Docker%E5%AE%B9%E5%99%A8%E5%8C%96%E9%83%A8%E7%BD%B2%E4%BC%81%E4%B8%9A%E7%BA%A7%E5%BA%94%E7%94%A8%E9%9B%86%E7%BE%A4/01\_%E7%AC%94%E8%AE%B0/Docker%E5%AE%B9%E5%99%A8%E5%8C%96%E9%83%A8%E7%BD%B2%E4%BC%81%E4%B8%9A%E7%BA%A7%E5%BA%94%E7%94%A8%E9%9B%86%E7%BE%A4.assets/image-20220212224446838.png?lastModify=1718205836)

\
