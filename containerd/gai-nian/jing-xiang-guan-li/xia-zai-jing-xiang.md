# 下载镜像

> containerd支持oci标准的镜像，所以可以直接使用docker官方或dockerfile构建的镜像
>
> 说明： 这里ctr命令pull镜像时，不能直接把镜像名字写成<mark style="color:purple;">**`nginx:alpine`**</mark>

```
# ctr images pull --all-platforms docker.io/library/nginx:alpine

docker.io/library/nginx:alpine:                                                   resolved       |++++++++++++++++++++++++++++++++++++++|
docker.io/library/nginx:alpine:                                                   resolved       |++++++++++++++++++++++++++++++++++++++|
index-sha256:da9c94bec1da829ebd52431a84502ec471c8e548ffb2cedbf36260fd9bd1d4d3:    done           |++++++++++++++++++++++++++++++++++++++|
manifest-sha256:050385609d832fae11b007fbbfba77d0bba12bf72bc0dca0ac03e09b1998580f: done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:f2303c6c88653b9a6739d50f611c170b9d97d161c6432409c680f6b46a5f112f:    done           |++++++++++++++++++++++++++++++++++++++|
config-sha256:bef258acf10dc257d641c47c3a600c92f87be4b4ce4a5e4752b3eade7533dcd9:   done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:59bf1c3509f33515622619af21ed55bbe26d24913cedbca106468a5fb37a50c3:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:8d6ba530f6489d12676d7f61628427d067243ba4a3a512c3e28813b977cb3b0e:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:5288d7ad7a7f84bdd19c1e8f0abb8684b5338f3da86fe9ae1d7f0e9bc2de6595:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:39e51c61c033442d00c40a30b2a9ed01f40205875fbd8664c50b4dc3e99ad5cf:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:ee6f71c6f4a82b2afd01f92bdf6be0079364d03020e8a2c569062e1c06d3822b:    done           |++++++++++++++++++++++++++++++++++++++|
elapsed: 11.0s                                                                    total:  8.7 Mi (809.5 KiB/s)                                    
unpacking linux/amd64 sha256:da9c94bec1da829ebd52431a84502ec471c8e548ffb2cedbf36260fd9bd1d4d3...
done: 1.860946163s
```

```
查看已下载容器镜像
# ctr images ls
REF                            TYPE                                                      DIGEST                                                                  SIZE    PLATFORMS                                                                                LABELS
docker.io/library/nginx:alpine application/vnd.docker.distribution.manifest.list.v2+json sha256:da9c94bec1da829ebd52431a84502ec471c8e548ffb2cedbf36260fd9bd1d4d3 9.7 MiB linux/386,linux/amd64,linux/arm/v6,linux/arm/v7,linux/arm64/v8,linux/ppc64le,linux/s390x -
```

```bash
指定平台下载容器镜像
# ctr images pull --platform linux/amd64 docker.io/library/nginx:alpine
```
