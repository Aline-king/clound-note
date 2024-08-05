# OSS（对象存储）

## 简介

对象存储(Object Storage) 是无层次结构的数据存储方法，通常用于云计算环境中。

不同于其他数据存储方法，基于对象的存储<mark style="color:blue;">**不使用目录树**</mark>&#x20;

数据作为单独的对象进行存储 数据并不放置在目录层次结构中，而是存在于平面地址空间内的同一级别&#x20;

应用通过唯一地址来识别每个单独的数据对象&#x20;

每个对象可包含有助于检索的元数据&#x20;

专为使用API在应用级别（而非用户级别）进行访问而设计

### 对象

对象是**对象存储系统中数据存储**的基本单位，每个Object是数据和数据属性集的综合体，数据属性可以根据应用的需求进行设置，包括数据分布、服务质量等 每个对象自我维护其属性，从而简化了存储系统的管理任务 对象的大小可以不同，甚至可以包含整个数据结构，如文件、数据库表项等

适用场景

对象存储系统一般是一类智能设备，它具有自己的存储介质、处理器、内存以及网络系统等，负责管理本地的对象，是对象存储系统的核心.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### 术语解释

一般说来，一个对象存储系统的核心资源类型应该包括<mark style="color:purple;">**用户**</mark>（User）、<mark style="color:purple;">**存储桶**</mark>（ bucket）和<mark style="color:purple;">**对象**</mark>（object）&#x20;

它们之间的关系是：&#x20;

* User将Object存储到存储系统上的Bucket&#x20;
* 存储桶属于某个用户并可以容纳对象，一个存储桶用于存储多个对象&#x20;
* 同一个用户可以拥有多个存储桶，不同用户允许使用相同名称的bucket

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### 常见方案

<table data-header-hidden><thead><tr><th width="117">方案</th><th>描述</th></tr></thead><tbody><tr><td>Amazon S3</td><td><p>提供了user、bucket和object分别表示用户、存储桶和对象，</p><p>其中bucket隶属于user，因此user名称即可做为bucket的名称空间，不同用户允许使用相同名称的bucket</p></td></tr><tr><td>OpenStack Swift</td><td>提供了user、container和object分别对应于用户、存储桶和对象，不过它还额外为user提供了父级组件account，用于表示一个项目或租户，因此一个account中可包含一到多个user，它们可共享使用同一组container，并为container提供名称空间</td></tr><tr><td>RadosGW</td><td><p>提供了user、subuser、bucket和object，其中的user对应于S3的user，而subuser则对应于Swift的user，不过user和subuser都不支持为bucket提供名称空间，因此 ，不同用户的存储桶也不允许同名；不过，自Jewel版本起，RadosGW引入了tenant（租户）用于为user和bucket提供名称空间，但它是个可选组件 </p><p>Jewel版本之前，radosgw的所有user位于同一名称空间，它要求所有user的ID必须惟一，并且即便是不同user的bucket也不允许使用相同的bucket ID</p></td></tr></tbody></table>
