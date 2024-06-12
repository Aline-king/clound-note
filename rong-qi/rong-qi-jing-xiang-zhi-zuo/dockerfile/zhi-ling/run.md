# RUN

RUN指令用于在**构建**镜像中执行命令，有以下两种格式:

{% hint style="info" %}
**注意:**&#x20;

按优化的角度来讲:当有多条要执行的命令,不要使用多条RUN,尽量使用&&符号与\符号连接成一行。因为多条RUN命令会让镜像建立多层(总之就是会变得臃肿了😃)。
{% endhint %}



{% tabs %}
{% tab title="shell格式" %}
> 格式：RUN <命令>&#x20;
>
> 例：RUN echo 'kubemsb' > /var/www/html/index.html
{% endtab %}

{% tab title="exec格式" %}
> 格式：RUN \["可执行文件", "参数1", "参数2"]&#x20;
>
> 例：RUN \["/bin/bash", "-c", "echo kubemsb > /var/www/html/index.html"]
{% endtab %}
{% endtabs %}

## 事例

{% hint style="danger" %}
```bash
RUN yum install httpd httpd-devel -y
RUN echo test > /var/www/html/index.html
```
{% endhint %}

可以改为

{% hint style="success" %}
```
RUN yum install httpd httpd-devel -y && echo test > /var/www/html/index.html
```
{% endhint %}

{% hint style="success" %}
```bash
RUN yum install httpd httpd-devel -y  \
    && echo test > /var/www/html/index.html
```
{% endhint %}

\
