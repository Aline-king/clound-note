# 镜像核心原理

## 镜像文件是基于分层机制实现的

<figure><img src="../../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

<img src="../../../.gitbook/assets/file.excalidraw (1).svg" alt="" class="gitbook-drawing">

在 bootfs之上是rootfs(root file system)，它包含了不同Linux发行版(Ubuntu|Centos)文件系统中的标准目录和文件，比如/dev,/proc,/bin,/etc等。

它由内核挂载为只读模式，而后通过"联合挂载"在其基础上挂载一个"可写层"

<figure><img src="../../../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>

## UnionFS 联合文件系统

<figure><img src="../../../.gitbook/assets/image (20) (1).png" alt=""><figcaption></figcaption></figure>

Union文件系统是一种分层、轻量级并且高性能的文件系统，

它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将存在不同物理位置的不同目录挂载到同一个虚拟文件系统下，从外面看起来，只能看到一个包含所有底层的文件和目录的文件系统。

UnionFS可以把只读和可读写文件系统合并在一起，具有写时复制功能，允许只读文件系统的修改可以保存到可写文件系统当中。



Union文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

Docker镜像使用分层最大的好处就是 共享资源。许多镜像基于相同的base镜像构建而来，宿主机只需在磁盘上保存一份base镜像，同时内存中也只需加载一份base镜像，就可以为所有容器服务了，而且镜像的每一层都可以被共享。

当用docker run启动这个容器时，实际上在镜像的顶部添加了一个新的可写层。这个可写层也叫容器层。容器启动后，其内的应用所有对容器的改动，文件的增删改操作都只会发生在容器层中，根据容器镜像的写时拷贝(Copy-on-Write)技术，某个容器对基础镜像的修改会被限制在单个容器内,对容器层下面的所有只读镜像层没有影响。



**容器层的细节说明**

镜像层数量可能会很多，所有镜像层会联合在一起组成一个统一的文件系统。如果不同层中有一个相同路径的文件，比如 /a，上层的 /a 会覆盖下层的 /a，也就是说用户只能访问到上层中的文件 /a。在容器层中，用户看到的是一个叠加之后的文件系统。

<table data-header-hidden><thead><tr><th width="148"></th><th></th></tr></thead><tbody><tr><td><strong>文件操作</strong></td><td><strong>说明</strong></td></tr><tr><td><strong>添加文件</strong></td><td>在容器中创建文件时，新文件被添加到容器层中。</td></tr><tr><td><strong>读取文件</strong></td><td>在容器中读取某个文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后打开并读入内存。</td></tr><tr><td><strong>修改文件</strong></td><td>在容器中修改已存在的文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后修改之。</td></tr><tr><td><strong>删除文件</strong></td><td>在容器中删除文件时，Docker 也是从上往下依次在镜像层中查找此文件。找到后，会在容器层中记录下此删除操作。（只是记录删除操作）</td></tr></tbody></table>
