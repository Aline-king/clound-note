# 镜像优化

## **减少镜像分层**

Dockerfile中包含多种指令，如果涉及到部署最多使用的算是RUN命令了，使用RUN命令时，不建议每次安装都使用一条单独的RUN命令，可以把能够合并安装指令合并为一条，这样就可以减少镜像分层。

{% hint style="danger" %}
```
FROM centos:latest
MAINTAINER www.kubemsb.com
RUN yum install epel-release -y 
RUN yum install -y gcc gcc-c++ make -y
RUN wget http://docs.php.net/distributions/php-5.6.36.tar.gz
RUN tar zxf php-5.6.36.tar.gz
RUN cd php-5.6.36
RUN ./configure --prefix=/usr/local/php 
RUN make -j 4 
RUN make install
EXPOSE 9000
CMD ["php-fpm"]
```
{% endhint %}

**优化内容如下：**

{% hint style="success" %}
```docker
FROM centos:latest
MAINTAINER www.kubemsb.com
RUN yum install epel-release -y && \
    yum install -y gcc gcc-c++ make
​
RUN wget http://docs.php.net/distributions/php-5.6.36.tar.gz && \
    tar zxf php-5.6.36.tar.gz && \
    cd php-5.6.36 && \
    ./configure --prefix=/usr/local/php && \
    make -j 4 && make install
EXPOSE 9000
CMD ["php-fpm"]
```
{% endhint %}

**4.4.6.2 清理无用数据**

* 一次RUN形成新的一层，如果没有在同一层删除，无论文件是否最后删除，都会带到下一层，所以要在每一层清理对应的残留数据，减小镜像大小。
* 把生成容器镜像过程中部署的应用软件包做删除处理

```
FROM centos:latest
MAINTAINER www.kubemsb.com
RUN yum install epel-release -y && \
    yum install -y gcc gcc-c++ make gd-devel libxml2-devel \
    libcurl-devel libjpeg-devel libpng-devel openssl-devel \
    libmcrypt-devel libxslt-devel libtidy-devel autoconf \
    iproute net-tools telnet wget curl && \
    yum clean all && \
    rm -rf /var/cache/yum/*
​
RUN wget http://docs.php.net/distributions/php-5.6.36.tar.gz && \
    tar zxf php-5.6.36.tar.gz && \
    cd php-5.6.36 && \
    ./configure --prefix=/usr/local/php \
    make -j 4 && make install && \
    cd / && rm -rf php*
```

**4.4.6.3 多阶段构建镜像**

项目容器镜像有两种，

一种直接把项目代码复制到容器镜像中，下次使用容器镜像时即可直接启动；

另一种把需要对项目源码进行编译，再复制到容器镜像中使用。

不论是哪种方法都会让制作镜像复杂了些，并也会让容器镜像比较大，建议采用分阶段构建镜像的方法实现。

{% hint style="info" %}
第一个 FROM 后边多了个 AS 关键字，可以给这个阶段起个名字&#x20;

第二个 FROM 使用上面构建的 Tomcat 镜像，COPY 关键字增加了 —from 参数，用于拷贝某个阶段的文件到当前阶段。
{% endhint %}

<pre data-line-numbers><code>$ git clone https://github.com/kubemsb/tomcat-java-demo
$ cd tomcat-java-demo
$ vi Dockerfile
FROM maven AS <a data-footnote-ref href="#user-content-fn-1">build</a>
ADD ./pom.xml pom.xml
ADD ./src src/
RUN mvn clean package
​
FROM kubemsb/tomcat
RUN rm -rf /usr/local/tomcat/webapps/ROOT
COPY --from=build target/*.war /usr/local/tomcat/webapps/ROOT.war
​
$ docker build -t demo:v1 .
$ docker container run -d -v demo:v1
</code></pre>

\


[^1]: 
