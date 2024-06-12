# 指令

* 构建类指令 - 用于构建image
  * 其指定的操作不会在运行image的容器上执行（FROM、MAINTAINER、RUN、ENV、ADD、COPY）
* 设置类指令 - 用于设置image的属性
  * 其指定的操作将在运行image的容器中执行（CMD、ENTRYPOINT、USER 、EXPOSE、VOLUME、WORKDIR、ONBUILD）

通过`man dockerfile`可以查看到详细的说明,这里简单的翻译并列出常用的指令

<table data-header-hidden><thead><tr><th width="147"></th><th></th></tr></thead><tbody><tr><td>FROM</td><td>构建新镜像基于的基础镜像</td></tr><tr><td>LABEL</td><td>标签</td></tr><tr><td>RUN</td><td>构建镜像时运行的Shell命令</td></tr><tr><td>COPY</td><td>拷贝文件或目录到镜像中</td></tr><tr><td>ADD</td><td>解压压缩包并拷贝</td></tr><tr><td>ENV</td><td>设置环境变量</td></tr><tr><td>USER</td><td>为RUN、CMD和ENTRYPOINT执行命令指定运行用户</td></tr><tr><td>EXPOSE</td><td>声明容器运行的服务端口</td></tr><tr><td>WORKDIR</td><td>为RUN、CMD、ENTRYPOINT、COPY和ADD设置工作目录</td></tr><tr><td>CMD</td><td>运行容器时默认执行，如果有多个CMD指令，最后一个生效</td></tr></tbody></table>
