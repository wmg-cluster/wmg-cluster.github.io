# 集群使用补充说明

## 1. 存储类型

集群用户的个人目录为 `/public/<username>`, 默认存储池为 `base_pool`。

全部ceph存储池：
*   `base_pool` (HDD, 无数据保护)
*   `metapool` (SSD, 2x冗余，仅存放元数据)
*   `ssd_store_pool` (SSD, 无数据保护)
*   `ecpool` (HDD, 3+1纠错)

可以通过 `setfattr` 设定空目录的存储池 (详见: [CephFS File Layouts](https://docs.ceph.com/en/latest/cephfs/file-layouts/))

例如:
```bash
mkdir my_precious_data
setfattr -n ceph.dir.layout.pool -v ecpool my_precious_data
```
`my_precious_data` 目录下的文件会存储在 `ecpool`。

## 2. 使用 module 加载软件环境

集群中一部分预装的软件，或者环境配置比较复杂、或者安装在非标准目录、或者安装了多个版本相互冲突，因此默认没有加载。需要通过 `module` 命令手动加载相应的软件模块。

例如，加载 Matlab:
```bash
$ matlab                            # 未加载 Matlab 模块
-bash: matlab: command not found

$ module avail                      # 查看系统中所有软件模块
------------------------- /public/.shared/modulefiles --------------------------
  caffe/1.0    matlab/R2017a (D)    matlab/R2018b

$ module load matlab/R2017a        # 加载 Matlab R2017a 模块
$ matlab -nosplash -nodesktop
MATLAB is selecting SOFTWARE OPENGL rendering.
......

$ module unload matlab/R2017a      # 卸载模块
```
加载 CUDA 12.4:
```bash
$ module load cuda/12.4
```

在批处理脚本中使用包括 Matlab 在内的未默认加载的软件时，也需要先加载对应模块。

## 3. 容器

~~容器, 由于集群使用的nft作为防火墙，存在与docker的兼容性问题 ([Docker and nftables, is there a way linux?](https://www.reddit.com/r/docker/comments/1fcvvfc/docker_and_nftables_is_there_a_way_linux/)), 因此仅在g20装有docker,如果需要container，可以使用singularity~~

**注意**: Singularity 长期未更新，现在已经几乎不可用。