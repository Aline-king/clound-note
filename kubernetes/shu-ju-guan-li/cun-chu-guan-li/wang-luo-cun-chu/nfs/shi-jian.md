# 实践



{% tabs %}
{% tab title="格式化" %}
将10.0.0.18作为NFS服务环境，加载一块磁盘并进行格式化

<pre class="language-bash"><code class="lang-bash"><strong>[root@kubernetes-ha1 ~]# fdisk -l | grep 'sdb'
</strong>磁盘 /dev/sdb：21.5 GB, 21474836480 字节，41943040 个扇区
</code></pre>

{% code title="磁盘格式化" %}
```bash
[root@kubernetes-ha1 ~]# mkfs.ext4 /dev/sdb
mke2fs 1.42.9 (28-Dec-2013)
[root@kubernetes-ha1 ~]# blkid | grep sdb
/dev/sdb: UUID="b01ca48c-f8a6-4817-a9b4-7bb7a53c03c2" TYPE="ext4"
```
{% endcode %}


{% endtab %}

{% tab title="部署" %}
{% code title="创建数据目录" %}
```bash
[root@kubernetes-ha1 ~]# mkdir /superopsmsb/nfs-data -p
```
{% endcode %}

{% code title="部署nfs服务" %}
```
[root@kubernetes-ha1 ~]# yum install -y rpcbind nfs-utils
```
{% endcode %}

{% hint style="danger" %}
<mark style="color:red;">注意：</mark>&#x20;

<mark style="color:yellow;">**\***</mark>：允许所有连接，该值可以是一个网段|IP|域名的形式&#x20;

<mark style="color:purple;">rw</mark>：读写共享权限&#x20;

<mark style="color:green;">no\_root\_squash</mark>：root登录nfs资源的时候，以nobody身份，但不压缩其权限&#x20;

sync：所有操作数据会同时写入硬盘和内存
{% endhint %}

{% code title="配置共享目录" overflow="wrap" %}
```bash
echo '/superopsmsb/nfs-data *(rw,no_root_squash,sync)' > /etc/exports
```
{% endcode %}
{% endtab %}

{% tab title="Untitled" %}
```
配置生效
[root@kubernetes-ha1 ~]# exportfs -r
```
{% endtab %}

{% tab title="重启确认效果" %}
22

{% code title="重启服务" %}
```bash
# systemctl start rpcbind.service
# systemctl start nfs.service
# systemctl enable rpcbind.service nfs.service
```
{% endcode %}

{% code title="确认效果" %}
```bash
# showmount -e localhost
Export list for localhost:
/superopsmsb/nfs-data *
```
{% endcode %}
{% endtab %}

{% tab title="所有的k8s节点部署nfs" %}
```bash
# for i in {12..17};do ssh root@10.0.0.$i "yum install nfs-utils -y";done
```



{% code title="挂载测试" %}
```bash
# mount -t nfs 10.0.0.18:/superopsmsb/nfs-data /tmp
# mount | grep nfs
10.0.0.18:/superopsmsb/nfs-data on /tmp type nfs4 (rw,relatime,vers=4.1,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.0.0.12,local_lock=none,addr=10.0.0.18)
# umount /tmp
```
{% endcode %}


{% endtab %}
{% endtabs %}

