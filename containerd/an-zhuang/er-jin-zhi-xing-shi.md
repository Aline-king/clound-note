# 二进制形式

Containerd有两种安装包：

* 第一种是`containerd-xxx`,这种包用于单机测试没问题，不包含runC，需要提前安装。
* 第二种是`cri-containerd-cni-xxxx`，包含runc和k8s里的所需要的相关文件。k8s集群里需要用到此包。虽然包含runC，但是依赖系统中的seccomp（安全计算模式，是一种限制容器调用系统资源的模式。）

## 获取安装包

[https://github.com/containerd/containerd/releases](https://github.com/containerd/containerd/releases)

```
下载Containerd安装包
# wget https://github.com/containerd/containerd/releases/download/v1.6.0/cri-containerd-cni-1.6.0-linux-amd64.tar.gz
```

## 安装并测试可用性

### **安装containerd**

```bash
查看已获取的安装包
# ls
cri-containerd-cni-1.6.0-linux-amd64.tar.gz
```

```bash
解压已下载的软件包
# tar xf cri-containerd-cni-1.6.0-linux-amd64.tar.gz
```

```bash
查看解压后目录
# ls
etc opt  usr 
```

```bash
查看etc目录，主要为containerd服务管理配置文件及cni虚拟网卡配置文件
# ls etc
cni  crictl.yaml  systemd
# ls etc/systemd/
system
# ls etc/systemd/system/
containerd.service
​
​
查看opt目录，主要为gce环境中使用containerd配置文件及cni插件
# ls opt
cni  containerd
# ls opt/containerd/
cluster
# ls opt/containerd/cluster/
gce  version
# ls opt/containerd/cluster/gce
cloud-init  cni.template  configure.sh  env
​
查看usr目录，主要为containerd运行时文件，包含runc
# ls usr
local
# ls usr/local/
bin  sbin
# ls usr/local/bin
containerd  containerd-shim  containerd-shim-runc-v1  containerd-shim-runc-v2  containerd-stress  crictl  critest  ctd-decoder  ctr
# ls usr/local/sbin
runc
```

### **查看containerd安装位置**

```bash
查看containerd.service文件，了解containerd文件安装位置
# cat etc/systemd/system/containerd.service


# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
​
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target
​
[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd 查看此位置,把containerd二进制文件放置于此处即可完成安装。
​
Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999
​
[Install]
WantedBy=multi-user.target
```

### **复制containerd运行时文件至系统**

```bash
查看宿主机/usr/local/bin目录，里面没有任何内容。
# ls /usr/local/bin/
​
查看解压后usr/local/bin目录，里面包含containerd运行时文件
# ls usr/
local
# ls usr/local/
bin  sbin
# ls usr/local/bin/
containerd  containerd-shim  containerd-shim-runc-v1  containerd-shim-runc-v2  containerd-stress  crictl  critest  ctd-decoder  ctr
​
复制containerd文件至/usr/local/bin目录中，本次可仅复制containerd一个文件也可复制全部文件。
# cp usr/local/bin/containerd /usr/local/bin/
# ls /usr/local/bin/
containerd
```

### **添加containerd.service文件至系统**

```bash
查看解压后的etc/system目录
# ls etc
cni  crictl.yaml  systemd
​
# ls etc/systemd/
system
​
# ls etc/systemd/system/
containerd.service
​
复制containerd服务管理配置文件至/usr/lib/systemd/system/目录中
# cp etc/systemd/system/containerd.service /usr/lib/systemd/system/containerd.service
​
查看复制后结果
# ls /usr/lib/systemd/system/containerd.service
/usr/lib/systemd/system/containerd.service
```

### **查看containerd使用帮助**

```
# containerd --help
NAME:
   containerd -
                    __        _                     __
  _________  ____  / /_____ _(_)___  ___  _________/ /
 / ___/ __ \/ __ \/ __/ __ `/ / __ \/ _ \/ ___/ __  /
/ /__/ /_/ / / / / /_/ /_/ / / / / /  __/ /  / /_/ /
\___/\____/_/ /_/\__/\__,_/_/_/ /_/\___/_/   \__,_/
​
high performance container runtime
​
​
USAGE:
   containerd [global options] command [command options] [arguments...]
​
VERSION:
   v1.6.0
​
DESCRIPTION:
​
containerd is a high performance container runtime whose daemon can be started
by using this command. If none of the *config*, *publish*, or *help* commands
are specified, the default action of the **containerd** command is to start the
containerd daemon in the foreground.
​
​
A default configuration is used if no TOML configuration is specified or located
at the default file location. The *containerd config* command can be used to
generate the default configuration for containerd. The output of that command
can be used and modified as necessary as a custom configuration.
​
COMMANDS:
   config    information on the containerd config
   publish   binary to publish events to containerd
   oci-hook  provides a base for OCI runtime hooks to allow arguments to be injected.
   help, h   Shows a list of commands or help for one command
​
GLOBAL OPTIONS:
   --config value, -c value     path to the configuration file (default: "/etc/containerd/config.toml")
   --log-level value, -l value  set the logging level [trace, debug, info, warn, error, fatal, panic]
   --address value, -a value    address for containerd's GRPC server
   --root value                 containerd root directory
   --state value                containerd state directory
   --help, -h                   show help
   --version, -v                print the version
```

### **生成containerd模块配置文件**

#### **生成默认模块配置文件**

Containerd 的默认配置文件为 `/etc/containerd/config.toml`，可以使用`containerd config default > /etc/containerd/config.toml`命令创建一份模块配置文件

```
创建配置文件目录
# mkdir /etc/containerd
```

```
生成配置文件
# containerd config default > /etc/containerd/config.toml
```

```bash
查看配置文件
# cat /etc/containerd/config.toml
disabled_plugins = []
imports = []
oom_score = 0
plugin_dir = ""
required_plugins = []
root = "/var/lib/containerd"
state = "/run/containerd"
temp = ""
version = 2
​
[cgroup]
  path = ""
​
[debug]
  address = ""
  format = ""
  gid = 0
  level = ""
  uid = 0
​
[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0
​
[metrics]
  address = ""
  grpc_histogram = false
​
[plugins]
​
  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_delay = "0s"
    startup_delay = "100ms"
​
  [plugins."io.containerd.grpc.v1.cri"]
    device_ownership_from_security_context = false
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    enable_selinux = false
    enable_tls_streaming = false
    enable_unprivileged_icmp = false
    enable_unprivileged_ports = false
    ignore_image_defined_volumes = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "k8s.gcr.io/pause:3.6"  由于网络原因，此处被替换
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemd_cgroup = false
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""
​
    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
      ip_pref = ""
      max_conf_num = 1
​
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      ignore_rdt_not_enabled_errors = false
      no_pivot = false
      snapshotter = "overlayfs"
​
      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
​
        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]
​
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
​
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
​
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = false
​
      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
​
        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]
​
    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"
​
    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""
​
      [plugins."io.containerd.grpc.v1.cri".registry.auths]
​
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
​
      [plugins."io.containerd.grpc.v1.cri".registry.headers]
​
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
​
    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""
​
  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"
​
  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"
​
  [plugins."io.containerd.internal.v1.tracing"]
    sampling_ratio = 1.0
    service_name = "containerd"
​
  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"
​
  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false
​
  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "containerd-shim"
    shim_debug = false
​
  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
    sched_core = false
​
  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]
​
  [plugins."io.containerd.service.v1.tasks-service"]
    rdt_config_file = ""
​
  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""
​
  [plugins."io.containerd.snapshotter.v1.btrfs"]
    root_path = ""
​
  [plugins."io.containerd.snapshotter.v1.devmapper"]
    async_remove = false
    base_image_size = ""
    discard_blocks = false
    fs_options = ""
    fs_type = ""
    pool_name = ""
    root_path = ""
​
  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""
​
  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    root_path = ""
    upperdir_label = false
​
  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""
​
  [plugins."io.containerd.tracing.processor.v1.otlp"]
    endpoint = ""
    insecure = false
    protocol = ""
​
[proxy_plugins]
​
[stream_processors]
​
  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar"
​
  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar+gzip"
​
[timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"
​
[ttrpc]
  address = ""
  gid = 0
  uid = 0
```

#### **替换默认配置文件**

但上述配置文件后期改动的地方较多，这里直接换成可单机使用也可k8s环境使用的配置文件并配置好镜像加速器。

```bash
# vim /etc/containerd/config.toml
​
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
        [plugins.cri.registry.mirrors."harbor.kubemsb.com"] 此处添加了本地容器镜像仓库 Harbor,做为本地容器镜像仓库。
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

### **启动containerd服务并设置开机自启动**

```
# systemctl enable containerd
Created symlink from /etc/systemd/system/multi-user.target.wants/containerd.service to /usr/lib/systemd/system/containerd.service.
# systemctl start containerd
```

```
# systemctl status containerd
● containerd.service - containerd container runtime
   Loaded: loaded (/usr/lib/systemd/system/containerd.service; enabled; vendor preset: disabled)
   Active: active (running) since 五 2022-02-18 13:02:37 CST; 7s ago
     Docs: https://containerd.io
  Process: 60383 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
 Main PID: 60384 (containerd)
    Tasks: 8
   Memory: 20.0M
   CGroup: /system.slice/containerd.service
           └─60384 /usr/local/bin/containerd
           ......
```

### **复制ctr命令至系统**

```
# ls usr/local/bin/
containerd  containerd-shim  containerd-shim-runc-v1  containerd-shim-runc-v2  containerd-stress  crictl  critest  ctd-decoder  ctr
# cp usr/local/bin/ctr /usr/bin/
```

### **查看已安装containerd服务版本**

```
# ctr version
Client:
  Version:  v1.6.0
  Revision: 39259a8f35919a0d02c9ecc2871ddd6ccf6a7c6e
  Go version: go1.17.2
​
Server:
  Version:  v1.6.0
  Revision: 39259a8f35919a0d02c9ecc2871ddd6ccf6a7c6e
  UUID: c1972cbe-884a-41b0-867f-f8a58c168e6d
```

### **安装runC**

> 由于二进制包中提供的runC默认需要系统中安装seccomp支持，需要单独安装，且不同版本runC对seccomp版本要求一致，所以建议单独下载runC 二进制包进行安装，里面包含了seccomp模块支持。

#### **获取runC**

![image-20220218215925093](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220218215925093.png?lastModify=1717923303)

![image-20220218215958943](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220218215958943.png?lastModify=1717923303)

![image-20220218220023181](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220218220023181.png?lastModify=1717923303)

![image-20220218220124773](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220218220124773.png?lastModify=1717923303)

```
使用wget下载
# wget https://github.com/opencontainers/runc/releases/download/v1.1.0/runc.amd64
```

#### **安装runC并验证安装结果**

```
查看已下载文件 
# ls
runc.amd64
```

```
安装runC
# mv runc.amd64 /usr/sbin/runc
```

```
为runC添加可执行权限
# chmod +x /usr/sbin/runc
```

```
使用runc命令验证是否安装成功
# runc -v
runc version 1.1.0
commit: v1.1.0-0-g067aaf85
spec: 1.0.2-dev
go: go1.17.6
libseccomp: 2.5.3
```

\
