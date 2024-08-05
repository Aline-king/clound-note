# 快照管理

RBD支持image快照技术，借助于快照可以保留image的状态历史。

Ceph还支持快照“分层”机制，从而可实现快速克隆VM映像。

## 常见命令

<table><thead><tr><th width="104"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td>创建快照</td><td>rbd snap create [--pool ] --image &#x3C;image> --snap &#x3C;snap><br>或者 rbd snap create [&#x3C;pool-name>/]&#x3C;image-name>@&#x3C;snapshot-name><br><mark style="background-color:red;">在创建映像快照之前应停止image上的IO操作，且image上存在文件系统时，还要确保其处于一致状态；</mark></td><td></td></tr><tr><td>列出快照</td><td>rbd snap ls [--pool &#x3C;pool>] --image &#x3C;image> ...</td><td></td></tr><tr><td>回滚快照</td><td>rbd snap rollback [--pool &#x3C;pool>] --image &#x3C;image> --snap &#x3C;snap> ...<br><mark style="background-color:red;">意味着会使用快照中的数据重写当前版本的image，回滚所需的时间将随映像大小的增加而延长</mark></td><td></td></tr><tr><td>限制快照数量</td><td>快照数量过多，必然会导致image上的原有数据第一次修改时的IO压力恶化<br>rbd snap limit set [--pool &#x3C;pool>] [--image &#x3C;image>] ...</td><td></td></tr><tr><td>解除限制</td><td><p></p><pre><code>rbd snap limit clear [--pool &#x3C;pool>] [--image &#x3C;image>]
</code></pre></td><td></td></tr><tr><td>删除快照</td><td><pre class="language-bash"><code class="lang-bash">rbd snap rm [--pool &#x3C;pool>] [--image &#x3C;image>] [--snap &#x3C;snap>] [--no-progress]
[--force]
</code></pre><p>Ceph OSD会以异步方式删除数据，因此删除快照并不能立即释放磁盘空间;</p></td><td></td></tr><tr><td>清理快照</td><td><pre><code>rbd snap purge [--pool &#x3C;pool>] --image &#x3C;image> [--no-progress]
</code></pre><p>删除一个image的所有快照，可以使用rbd snap purge命令</p></td><td></td></tr></tbody></table>

## 实践



{% tabs %}
{% tab title="客户端准备rbd文件" %}

{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}

