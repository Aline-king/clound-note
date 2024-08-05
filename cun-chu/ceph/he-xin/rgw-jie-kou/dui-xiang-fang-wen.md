# 对象访问

## 简介

作为OSS对象存储，我们可以在互联网上以http(s):// 的方式来访问对象文件，基本格式如下： 对于存储在rgw01.superopsmsb.com上的S3 API对象存储系统上bucket存储桶中的名为images/1.jpg 的对象，可通过 http://bucket.rgw01.superopsmsb.com/images/1.jpg对其进行访问。

默认情况下，这种情况是拒绝的

这里拒绝的原因就是 AccessDenied，是需要我们通过专用的访问策略方式来进行功能实现。

```
[root@stor04 ~]# s3cmd ls s3://images/linux/issue
2062-09-27 00:49           23  s3://images/linux/issue
[root@stor04 ~]# curl http://images.stor04.superopsmsb.com:7480/linux/issue
<?xml version="1.0" encoding="UTF-8"?><Error><Code>AccessDenied</Code><BucketName>images</BucketName><RequestId>tx00000000000000000012f-0063324d15-63ad-default</RequestId><HostId>63ad-default-default</HostId></Error> 
```

## 访问对象存储注意事项

1. 资源对象的访问方式 http 还是 https，依赖于rgw的基本配置&#x20;
2. 资源对象的访问控制 通过定制策略的方式来实现&#x20;
3. 资源对象的跨域问题 通过定制cors的方式来实现&#x20;
4. 资源对象在浏览器端的缓存机制 rgw的基本配置定制

> 参考资料
>
> https://docs.ceph.com/en/latest/radosgw/bucketpolicy/ https://docs.ceph.com/en/latest/radosgw/s3/python/



查看默认策略

```
[root@stor04 /data/conf]# s3cmd info s3://swift-super
s3://swift-super/ (bucket):
   Location:  default
   Payer:     BucketOwner
   Expiration Rule: none
   Policy:    none
   CORS:      none
   ACL:       S3 Testing user: FULL_CONTROL
```

**简单实践**

定制策略

> 参考资料
>
> [https://docs.ceph.com/docs/master/radosgw/bucketpolicy/](https://docs.ceph.com/docs/master/radosgw/bucketpolicy/)

```
定制访问策略文件 cat /data/conf/policy.json
{
        "Statement": [{
                "Effect": "Allow",
                "Principal": "*",
                "Action": ["s3:GetObject"],
                "Resource": "*"
        }]
}
属性解析：
    Resource参数匹配某一种文件：Resource:["*.jpg","*.png"]
    Action可以设置较多参数，
```

```
应用访问策略
[root@stor04 /data/conf]# s3cmd setpolicy policy.json s3://images --acl-public
s3://images/: Policy updated
​
确认效果
[root@stor04 /data/conf]# s3cmd info s3://images
s3://images/ (bucket):
   Location:  default
   Payer:     BucketOwner
   Expiration Rule: none
   Policy:    {
        "Statement": [{
                "Effect": "Allow",
                "Principal": "*",
                "Action": ["s3:GetObject"],
                "Resource": "*"
        }]
}
​
   CORS:      none
   ACL:       S3 Testing user: FULL_CONTROL
```

设置访问限制

```
定制cors的策略文件 /data/conf/rules.xml 
<CORSConfiguration>
  <CORSRule> 
    <ID>Allow everything</ID>  
    <AllowedOrigin>*</AllowedOrigin>  
    <AllowedMethod>GET</AllowedMethod>  
    <AllowedMethod>HEAD</AllowedMethod>  
    <AllowedMethod>PUT</AllowedMethod>  
    <AllowedMethod>POST</AllowedMethod>  
    <AllowedMethod>DELETE</AllowedMethod>  
    <AllowedHeader>*</AllowedHeader>  
    <MaxAgeSeconds>30</MaxAgeSeconds>
  </CORSRule>
</CORSConfiguration>
```

```
应用浏览器跨域设置文件
[root@stor04 /data/conf]# s3cmd setcors rules.xml s3://images
查看效果
[root@stor04 /data/conf]# s3cmd info s3://images
s3://images/ (bucket):
   Location:  default
   Payer:     BucketOwner
   Expiration Rule: none
   Policy:    {
        ...
}
​
   CORS:      <CORSConfiguration xmlns=...</CORSConfiguration>
   ACL:       S3 Testing user: FULL_CONTROL
```

访问效果

```
[root@stor04 /data/conf]# curl http://images.stor04.superopsmsb.com:7480/linux/issue
\S
Kernel \r on an \m
```

脚本测试

```
定制访问测试脚本 /data/scripts/tests4.py
import boto.s3.connection
​
access_key = 'BX2IDCO5ADZ8IWZ4AVX7'
secret_key = 'wKXAiBBspTx9kl1NpcAi4eOCHxvQD7ORHnrmvBlf'
conn = boto.connect_s3(
        aws_access_key_id=access_key,
        aws_secret_access_key=secret_key,
        host='10.0.0.16', port=7480,
        is_secure=False, calling_format=boto.s3.connection.OrdinaryCallingFormat(),
       )
def get_object_url(bucket_name, object_name):
    try:
        bucket = conn.get_bucket(bucket_name)
        plans_key = bucket.get_key(object_name)
        plans_url = plans_key.generate_url(0, query_auth=False,
                                           force_http=False)
        return plans_url
    except Exception as e:
        print("get {} error:{}".format(object_name, e))
        return False
print(get_object_url("images", "linux/issue"))
```

```
获取资源对象的url效果
[root@stor04 /data/scripts]# python tests4.py
http://10.0.0.16:7480/images/linux/issue?x-amz-meta-s3cmd-attrs=atime%3A1664200700/ctime%3A1654585109/gid%3A0/gname%3Aroot/md5%3Af078fe086dfc22f64b5dca2e1b95de2c/mode%3A33188/mtime%3A1603464839/uid%3A0/uname%3Aroot
访问url的效果
[root@stor04 /data/scripts]# curl http://10.0.0.16:7480/images/linux/issue?x-amz-meta-s3cmd-attrs=atime%3A1664200700/ctime%3A1654585109/gid%3A0/gname%3Aroot/md5%3Af078fe086dfc22f64b5dca2e1b95de2c/mode%3A33188/mtime%3A1603464839/uid%3A0/uname%3Aroot
\S
Kernel \r on an \m
```
