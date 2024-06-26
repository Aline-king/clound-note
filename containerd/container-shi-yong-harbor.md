# container使用harbor

## Harbor主机名解析

> 在所有安装containerd宿主机上添加此配置信息。

<pre class="language-bash"><code class="lang-bash"># vim /etc/hosts
# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.165 harbor.kubemsb.com
<strong>// 192.168.10.165是harbor的IP
</strong>harbor.kubemsb.com建议用FQDN形式，如果用类似harbor这种短名，后面下载镜像会出问题
</code></pre>

## 修改Containerd配置文件

```bash
此配置文件已提前替换过，仅修改本地容器镜像仓库地址即可。
# vim /etc/containerd/config.toml
# cat /etc/containerd/config.toml
root = "/var/lib/containerd"
state = "/run/containerd"
oom_score = -999
​
[grpc]
  address = "/run/containerd/containerd.sock"
  uid = 0
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
​
[debug]
  address = ""
  uid = 0
  gid = 0
  level = ""
​
[metrics]
  address = ""
  grpc_histogram = false
​
[cgroup]
  path = ""
​
[plugins]
  [plugins.cgroups]
    no_prometheus = false
  [plugins.cri]
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    enable_selinux = false
    sandbox_image = "easzlab/pause-amd64:3.2"
    stats_collect_period = 10
    systemd_cgroup = false
    enable_tls_streaming = false
    max_container_log_line_size = 16384
    [plugins.cri.containerd]
      snapshotter = "overlayfs"
      no_pivot = false
      [plugins.cri.containerd.default_runtime]
        runtime_type = "io.containerd.runtime.v1.linux"
        runtime_engine = ""
        runtime_root = ""
      [plugins.cri.containerd.untrusted_workload_runtime]
        runtime_type = ""
        runtime_engine = ""
        runtime_root = ""
    [plugins.cri.cni]
      bin_dir = "/opt/kube/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = "/etc/cni/net.d/10-default.conf"
    [plugins.cri.registry]
      [plugins.cri.registry.mirrors]
        [plugins.cri.registry.mirrors."docker.io"]
          endpoint = [
            "https://docker.mirrors.ustc.edu.cn",
            "http://hub-mirror.c.163.com"
          ]
        [plugins.cri.registry.mirrors."gcr.io"]
          endpoint = [
            "https://gcr.mirrors.ustc.edu.cn"
          ]
        [plugins.cri.registry.mirrors."k8s.gcr.io"]
          endpoint = [
            "https://gcr.mirrors.ustc.edu.cn/google-containers/"
          ]
        [plugins.cri.registry.mirrors."quay.io"]
          endpoint = [
            "https://quay.mirrors.ustc.edu.cn"
          ]
        [plugins.cri.registry.mirrors."harbor.kubemsb.com"]   在此处添加,在镜像加速器下面添加这一段
          endpoint = [
            "http://harbor.kubemsb.com"
          ]
    [plugins.cri.x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""
  [plugins.diff-service]
    default = ["walking"]
  [plugins.linux]
    shim = "containerd-shim"
    runtime = "runc"
    runtime_root = ""
    no_shim = false
    shim_debug = false
  [plugins.opt]
    path = "/opt/containerd"
  [plugins.restart]
    interval = "10s"
  [plugins.scheduler]
    pause_threshold = 0.02
    deletion_threshold = 0
    mutation_threshold = 100
    schedule_delay = "0s"
    startup_delay = "100ms"
```

```
重启containerd，以便于重新加载配置文件。
# systemctl restart containerd
```

## ctr下载镜像

> <mark style="color:yellow;">**说明:**</mark>
>
> \--platform linux/amd64 指定系统平台，也可以使用--all-platforms指定所有平台镜像。

```bash
下载容器镜像
# ctr images pull --platform linux/amd64 docker.io/library/nginx:lates
docker.io/library/nginx:latest:                                                   resolved       |++++++++++++++++++++++++++++++++++++++|
index-sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767:    done           |++++++++++++++++++++++++++++++++++++++|
manifest-sha256:bb129a712c2431ecce4af8dde831e980373b26368233ef0f3b2bae9e9ec515ee: done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:b559bad762bec166fd028483dd2a03f086d363ee827d8c98b7268112c508665a:    done           |++++++++++++++++++++++++++++++++++++++|
config-sha256:c316d5a335a5cf324b0dc83b3da82d7608724769f6454f6d9a621f3ec2534a5a:   done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:5eb5b503b37671af16371272f9c5313a3e82f1d0756e14506704489ad9900803:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:1ae07ab881bd848493ad54c2ba32017f94d1d8dbfd0ba41b618f17e80f834a0f:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:78091884b7bea0fa918527207924e9993bcc21bf7f1c9687da40042ceca31ac9:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:091c283c6a66ad0edd2ab84cb10edacc00a1a7bc5277f5365c0d5c5457a75aff:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:55de5851019b8f65ed6e28120c6300e35e556689d021e4b3411c7f4e90a9704b:    done           |++++++++++++++++++++++++++++++++++++++|
elapsed: 20.0s                                                                    total:  53.2 M (2.7 MiB/s)
unpacking linux/amd64 sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767...
done: 3.028652226s
```

```
查看已下载容器镜像
# ctr images ls
REF                              TYPE                                                      DIGEST                                                                  SIZE      PLATFORMS                                                                                                                          LABELS
​
docker.io/library/nginx:latest   application/vnd.docker.distribution.manifest.list.v2+json sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767 54.1 MiB  linux/386,linux/amd64,linux/arm/v5,linux/arm/v7,linux/arm64/v8,linux/mips64le,linux/ppc64le,linux/s390x                            -
```

## ctr上传镜像

> <mark style="color:yellow;">**说明：**</mark>
>
> 先tag再push
>
> 因为我们harbor是http协议，不是https协议，所以需要加上`--plain-http`
>
> `--user admin:Harbor12345`指定harbor的用户名与密码

```bash
重新生成新的tag
# ctr images tag docker.io/library/nginx:latest harbor.kubemsb.com/library/nginx:latest
harbor.kubemsb.com/library/nginx:latest

查看已生成容器镜像
# ctr images ls
REF                                     TYPE                                                      DIGEST                                                                  SIZE      PLATFORMS                                                                                                                          LABELS
docker.io/library/nginx:latest          application/vnd.docker.distribution.manifest.list.v2+json sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767 54.1 MiB  linux/386,linux/amd64,linux/arm/v5,linux/arm/v7,linux/arm64/v8,linux/mips64le,linux/ppc64le,linux/s390x                            -
harbor.kubemsb.com/library/nginx:latest application/vnd.docker.distribution.manifest.list.v2+json sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767 54.1 MiB  linux/386,linux/amd64,linux/arm/v5,linux/arm/v7,linux/arm64/v8,linux/mips64le,linux/ppc64le,linux/s390x                            -

推送容器镜像至Harbor
# ctr images push --platform linux/amd64 --plain-http -u admin:Harbor12345 harbor.kubemsb.com/library/nginx:latest

推送容器镜像至Harbor
# ctr images push --platform linux/amd64 --plain-http -u admin:Harbor12345 harbor.kubemsb.com/library/nginx:latest
manifest-sha256:0fd68ec4b64b8dbb2bef1f1a5de9d47b658afd3635dc9c45bf0cbeac46e72101: done           |++++++++++++++++++++++++++++++++++++++|
config-sha256:dd025cdfe837e1c6395365870a491cf16bae668218edb07d85c626928a60e478:   done           |++++++++++++++++++++++++++++++++++++++|
elapsed: 0.5 s                                         
```



## 下载已上传容器镜像

```
ctr images pull --plain-http harbor.kubemsb.com/library/nginx:latest
```
