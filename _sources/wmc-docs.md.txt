# WMG集群使用指南

> **Version:** v1.0  |  **Date:** 2025-04-24

## 使用须知

### 基本信息

- **集群管理和作业调度系统**目前采用**SLURM**（**S**imple **L**inux **U**tility for **R**esource **M**anagement）开源框架。
- WMG计算集群包括一个**登录节点**和若干**计算节点**组成。
- 集群用户直接访问的是**登录节点**，需要通过 Slurm 作业管理器提供的命令行接口向**计算节点**提交计算任务。具体方法参见[编写/提交任务脚本](#hyperlink1)。
- 使用集群需要掌握简单的 **Linux** 命令行操作，网上有很多这方面的教程，也可以使用 GPT 等 LLM 工具帮助生成和编辑目标命令。

### 重要注意事项

- <strong style="color: red;">严禁</strong>在登录节点上直接运行计算任务！！！后果包括但不限于<strong style="color: red;">集群登录节点过载，导致用户集体掉线</strong>。
- <span style="color:red; font-weight:bold">务必</span>记得使用 <code>srun [选项] 可执行程序 [程序参数]</code> 命令提交交互式作业，或者通过 <code>srun --pty bash</code> 启动交互式 Bash 环境再运行可执行程序！！！否则任务会在登录节点上运行！！！
- <strong style="color: red;">注意</strong> 节点计算资源有限，请估计或测试好任务需要的 CPU/memory 资源，谨慎填写任务脚本，避免出现有 GPU 但是其他资源不够导致他人任务长时间排队的情况。

## 集群节点清单

- 集群目前由登录节点、存储节点和计算节点组成，登录节点只负责 SLURM 控制/登录功能，<strong style="color: red;">严禁</strong>在登录节点上运行计算任务。

### 节点硬件配置

| 节点名            | 型号              | 厂商    | CPU 配置            | GPU 配置    |
|-------------------|------------------|---------|---------------------|------------|
| wmc-slave-g6      | SYS-4029GP-TRT   |   AMAX  | Xeon Gold 5120 28核 | 1080Ti * 8 |
| wmc-slave-g7      | SYS-4029GP-TRT   |   AMAX  | Xeon Gold 5120 28核 | 1080Ti * 8 |
| wmc-slave-g8      |                  | 思腾合力 |   28    | 2080Ti * 8 |
| wmc-slave-g9      |                  | 思腾合力 |   28    |  2080Ti * 8 |
| wmc-slave-g10     | SYS-4029GP-TRT   |         | Xeon Gold 5120 28核 | 2080Ti * 8 |
| wmc-slave-g11     | | |   28    | 2080Ti * 8 |
| wmc-slave-g12     | | |   28    | 2080Ti * 8 |
| wmc-slave-g13     | | |   28核    | 2080Ti * 8 |
| wmc-slave-g14     | | |   32    | 2080Ti * 10|
| wmc-slave-g15     | | |   32    | 2080Ti * 10|
| wmc-slave-g16     | SYS-4029GP-TRT2  |     AMAX | Xeon Gold 5220R 48核 | 3090 * 7 |
| wmc-slave-g17     | SYS-4029GP-TRT2  |     AMAX | Xeon Gold 5220R 48核 | 3090 * 6 |
| wmc-slave-g18     | | 思腾合力 |   48    | A6000 * 10 |
| wmc-slave-g19     | | 思腾合力 |   48    | A6000 * 10 |
| wmc-slave-g20     | | 思腾合力 |   48    | A6000 * 10 |
| wmc-argon         | | |                        |         | |
| wmc-helium        | | |                        |         | |
| wmc-krypton       | | 思腾合力 |                |         | |

### 节点软件配置

| 节点名               | 操作系统            | GPU 驱动版本   | 负责功能         |
|----------------------|--------------------|---------------|-----------------|
| wmc-slave-g6         | Ubuntu 20.04.6 LTS | 535.183.01    | SLURM 控制/登录  |
| wmc-slave-g7         | Ubuntu 18.04.4 LTS | 495.44        | 计算             |
| wmc-slave-g8         | Ubuntu 20.04.6 LTS | 535.230.02    | 计算             |
| wmc-slave-g9         | Ubuntu 20.04.6 LTS | 535.230.02    | 计算             |
| wmc-slave-g10        | Ubuntu 20.04.6 LTS | 570.169       | 计算             |
| wmc-slave-g11        | Ubuntu 18.04.4 LTS |               | 计算             |
| wmc-slave-g12        | Ubuntu 18.04.4 LTS |               | 计算             |
| wmc-slave-g13        |                    |               | 退役             |
| wmc-slave-g14        | Ubuntu 18.04.4 LTS |               | 计算             |
| wmc-slave-g15        | Ubuntu 18.04.4 LTS |               | 计算             |
| wmc-slave-g16        | Ubuntu 18.04.5 LTS | 550.54.14     | 计算             |
| wmc-slave-g17        | Ubuntu 18.04.6 LTS | 550.54.14     | 计算             |
| wmc-slave-g18        | Ubuntu 18.04.6 LTS |               | 计算             |
| wmc-slave-g19        | Ubuntu 20.04.6 LTS |               | 计算             |
| wmc-slave-g20        | Ubuntu 18.04.6 LTS |               | 计算             |
| wmc-argon            | Debian 10 (buster) |               | 存储             |
| wmc-helium           | Ubuntu 20.04.2 LTS |               | 存储             |
| wmc-krypton          | Ubuntu 20.04.2 LTS |               | 存储             |

### 计算资源分区

- 以下是计算资源的分区情况，在申请调试节点和编写任务脚本时需要指定目标分区。如果是 GPU 分区，默认会分配对应物理节点的 CPU 资源，例如当分配的节点为 wmc-slave-g8 时，会默认分配 wmc-slave-g8 节点的 CPU 资源。目前 wmc-slave-g[18-20] 的 CPU 不参与分区，在申请这些节点的 GPU 任务时默认分配。

| PARTITION	| NODELIST                |
|-----------|-------------------------|
| cpu3	    | wmc-slave-g[7-12]       |
| cpu4	    | wmc-slave-g[14-15]      |
| cpu5	    | wmc-slave-g[16-17]      |
| gpu3	    | wmc-slave-g[8-12,14-15] |
| gpu4	    | wmc-slave-g[16-17]      |
| gpu5	    | wmc-slave-g[18-20]      |

## 登录集群

### 集群账户

- 需要使用集群的老师和同学请联系管理员开通账户。每个集群用户使用独立的 Linux 账户登录和提交任务。为保证服务器安全，普通用户不提供 root 权限。

- 默认登录节点：wmc-slave-g6（用于 SSH 连接）

### 新账户首次登录

- 首次登录时需要修改初始密码（注意输入一遍初始密码两遍新密码）：

```bash
You must change your password now and login again!
Current Password: (输入初始密码)
New password: (输入新密码)
Retype new password: (再次输入新密码)
```

- 修改完成后重新登录，成功后即可得到命令提示符：

```bash
Welcome! This is the Computer Cluster of Weiming Lab.
```

### 通过 SSH 连接登录节点

- 用户在登录节点上使用 Slurm 命令行提交计算任务，另外也可以进行一些简单的数据处理操作。

- 推荐使用 VS Code 插件 Remote-SSH 进行连接。config 文件示例（将信息提示替换为对应内容即可使用）：

```
Host wmg-cluster
    HostName ClusterAddress
    User YourAccount
    # 可以选择生成公私密钥对，将公钥添加到集群，如果使用密码登录可以将这条注释掉
    IdentityFile /path/to/your/private/key
```

- Windows/Linux 公私密钥对生成及使用请自行查找相关资料或借助 GPT 生成相关命令。

## 查看集群/节点状态

### 查看集群状态

- 在登陆节点上，使用 Slurm 提供的命令可以查看集群的各项状态。
- `sinfo` 命令可以看到哪些分区/计算节点是空闲的（idle）、分配了一部分资源（mix）或者完全占满了（alloc），输出示例如下：
```
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
cpu3         up   infinite      1  drain wmc-slave-g7
cpu3         up   infinite      5   idle wmc-slave-g[8-12]
cpu4         up   infinite      1    mix wmc-slave-g14
cpu4         up   infinite      1   idle wmc-slave-g15
cpu5         up   infinite      1    mix wmc-slave-g16
cpu5         up   infinite      1   idle wmc-slave-g17
gpu3         up   infinite      1    mix wmc-slave-g14
gpu3         up   infinite      6   idle wmc-slave-g[8-12,15]
gpu4         up   infinite      1    mix wmc-slave-g16
gpu4         up   infinite      1   idle wmc-slave-g17
gpu5         up   infinite      3    mix wmc-slave-g[18-20]
```
- 通过 `-o` 参数自定义格式可以查看更多信息，例如：
```bash
# 显示每个节点的 CPU 和 GPU 总数
sinfo -o "%20n %10c %25G"
# 查看内存占用情况
sinfo -o "%20n %10m %10e %10a"
```

### 查看节点状态

- `scontrol` 命令则可展示更详细的信息。`scontrol show node 具体节点` 查询节点信息、`scontrol show partition` 查询分区信息。
- 例如 `scontrol show node wmc-slave-g20`，得到输出如下：
```
NodeName=wmc-slave-g20 Arch=x86_64 CoresPerSocket=24
   CPUAlloc=7 CPUTot=48 CPULoad=15.36
   AvailableFeatures=(null)
   ActiveFeatures=(null)
   Gres=gpu:10(S:0)
   NodeAddr=wmc-slave-g20 NodeHostName=wmc-slave-g20 Version=20.02.3
   OS=Linux 5.4.0-89-generic #100~18.04.1-Ubuntu SMP Wed Sep 29 10:59:42 UTC 2021
   RealMemory=248000 AllocMem=214688 FreeMem=59620 Sockets=2 Boards=1
   State=MIXED ThreadsPerCore=1 TmpDisk=0 Weight=1 Owner=N/A MCS_label=N/A
   Partitions=gpu5
   BootTime=2024-08-26T11:16:42 SlurmdStartTime=2024-08-26T11:17:20
   CfgTRES=cpu=48,mem=248000M,billing=48
   AllocTRES=cpu=7,mem=214688M
   CapWatts=n/a
   CurrentWatts=0 AveWatts=0
   ExtSensorsJoules=n/s ExtSensorsWatts=0 ExtSensorsTemp=n/s
```
- `wmc-mon` 工具可以查看特定计算节点上的 CPU、内存以及 GPU 使用信息。
- `wmc-mon cpu wmc-slave-g20` : 查看 g20 节点上的所有进程信息（主要是 CPU、内存，输出同 top 命令）
- `wmc-mon gpu wmc-slave-g20` : 查看 g20 节点上GPU 的使用情况（输出同 nvidia-smi 命令）
- 如果需要查看各个节点的任务信息，参见[监控任务状态](#hyperlink2)。

## 调试节点申请&远程 debug

### 计算资源申请&使用

- 用户需要通过 `salloc` 命令申请资源后，使用 `srun` 运行调试任务。
- `salloc` 命令示例如下：
```bash
# 使用 Slurm 分配交互式计算资源：
#   --nodelist=wmc-slave-g12  # [可选] 强制指定节点名称，若省略则由系统自动分配
#   -p gpu3                   # 提交到 gpu3 分区（也可选择 cpu 分区）
#   -N 1                      # 分配 1 个计算节点
#   -c 4                      # 分配 4 个 CPU 核心
#   --mem 30G                 # 分配 30GB 内存
#   --gres gpu:1              # [可选] 分配 1 块目标 GPU，纯 CPU 任务可不填写
salloc --nodelist=wmc-slave-g12 -p gpu3 -N 1 -c 4 --mem 30G --gres gpu:1
```
- 获得计算资源分配后，<strong style="color: red;">务必</strong>使用 `srun [选项] 可执行程序 [程序参数]` 进行交互式作业调试，否则<strong style="color: red;">任务会运行在登录节点上，可能导致集群崩溃</strong>。
```bash
# 调试命令必须以 srun 开头，才能运行在分配的计算节点上
# 未加 srun 仍是在登录节点执行命令
hostname                      
# 使用 srun 在计算节点执行命令
srun hostname
# 查看分配的 gpu 信息
srun nvidia-smi
# 运行 python 脚本
srun python your_script.py
```
- **推荐**在已分配的 Slurm 计算节点上，直接启动一个交互式 Bash Shell ，这样在后续调试中可以不用 `srun` 命令，防止不小心忘记。
```bash
# 在分配临时节点上启动交互式 Bash
srun --pty bash
# 进入交互式环境后可自由执行命令，不需要再使用 srun
nvidia-smi
python debug.py
# 退出交互式环境
exit
```

- 结束调试后，<strong style="color: red;">务必</strong>记得释放分配的计算资源，这里的 `exit` 命令和退出交互式环境的命令是分开的。
```bash
# 释放分配的计算资源
exit
```

### 在集群上使用 VS Code 远程 debug
- 首先，需要安装 VS Code 的 Remote-SSH 插件。
- `pip install ptvsd`，将以下代码加到需要调试的代码前面：
```python
import ptvsd
ptvsd.enable_attach(address = ('0.0.0.0', 3000))
ptvsd.wait_for_attach()
```
- 在服务器上申请资源，`srun` 运行 python 文件。
- 在 VS Code 左侧点击“运行和调试”的图标（或 `Ctrl+Shift+D` ），点击“运行和调试”按钮（没有的话点击 Python Debugger 旁边的齿轮），调试器选择 Python Debugger ，调试配置选择“远程附加”， 主机名填 `wmc-slave-gX` （X为申请的调试节点），端口号填 3000 ，之后选择调试。此时，在代码上可以直接添加断点，中间变量可以在左侧实时看到。这个过程实际上是在创建一个 json 文件作为远程调试的配置。
- 在调试完之后，<strong style="color: red;">务必</strong>记得释放申请的资源！

## 环境搭建

### python 环境管理

- 推荐使用 Miniconda 进行 python 环境管理。
- 鉴于某些已知原因，建议 conda 和 pip 设置国内源。下面是简单示例：
```bash
# 查看所有配置的软件源和优先级
conda config --get channels
pip config list -v
# 添加软件源（更多源请自行查找）
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### Hugging Face 加载模型

- 对于从 huggingface 上 load 模型，推荐两种方式：
  1. 在 bash 脚本里，运行 python 文件之前先通过命令 `export HF_ENDPOINT="https://hf-mirror.com"` 修改环境变量。
  2. 直接本地手动下载后上传到集群。

## CUDA 版本问题

### 查看当前版本

- 可以使用以下命令查看版本，如果驱动支持 CUDA 版本过低，请联系管理员更新 GPU 节点驱动。

```bash
# 驱动支持的最高 CUDA 版本（不一定是实际安装版本）
nvidia-smi
# 当前实际安装的 CUDA Toolkit 版本
nvcc --version
```
- 某些节点运行 `nvcc --verison` 会出现 `Command 'nvcc' not found` 的情况，可以尝试手动指定CUDA版本，具体操作见下一小节。

### 指定/更改使用的 CUDA 版本

- 集群提供多个 CUDA 版本，存放在 `/app/cuda` 路径下，用户可以查看并选择合适的版本。
- 用户可以通过修改环境变量指定 CUDA 版本：

```bash
# 临时修改环境变量（仅当前终端生效）
export CUDA_HOME=/app/cuda/cuda-11.8
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
# 验证环境变量
echo $CUDA_HOME      # 应输出 /app/cuda/cuda-11.8
which nvcc           # 应显示 /app/cuda/cuda-11.8/bin/nvcc
nvcc --version       # 确认版本是否正确
```
- 在任务脚本中同样可以用上述命令指定任务运行的 CUDA 版本。

## 计算任务管理

> 本章仅介绍基础的任务脚本编写、提交、监控和取消，更多详细内容以及 Q&A 参见[集群任务脚本](./job_scripts.md)和[常见问题](./FAQs.md)。

### 编写/提交任务脚本 <a id="hyperlink1"></a>

- 任务脚本用于描述 Slurm 计算任务。下面是一个简单的例子，这个脚本打印了一行文字、执行了一个 python 脚本、最后又打印了一行文字。

- 文件：test_sbatch.sh

```bash
#!/bin/bash
#SBATCH -p gpu1             # 在 gpu1 分区运行（不写默认为 cpu1）
#SBATCH -N 1                # 只在一个节点上运行任务
#SBATCH -c 1                # 申请 CPU 核心：1 个
#SBATCH --mem 500           # 申请内存：500 MB
#SBATCH --gres gpu:1        # 分配 1 个 GPU（纯 CPU 任务不用写）
#SBATCH -o test-%j.log      # 输出 log 文件，%j会被自动替换为任务的ID（JobID）

echo "job begin"
python3 my_script.py  # my_script.py 是用户的 python 程序
echo "job end"
```

- 这个任务脚本实际上就是个 bash 脚本。Slurm 读取以 `#SBATCH` 起始的行，这些行在 bash 语法里是注释，因此 bash 不会理会。[集群任务脚本](./job_scripts.md) 中提供了更多示例。

- 调试时可以通过以下命令进行试运行：
```bash
# 在调试节点运行名为 test_sbatch.sh 的 Bash 脚本
srun bash test_sbatch.sh
```

- 写好任务脚本后，使用 `sbatch` 命令向集群提交计算任务。若提交成功，`sbatch` 会输出任务编号，该任务会进入队列，请求的资源可用后就会执行。

```bash
# 这里也可以用 --nodelist 参数指定节点
sbatch test_sbatch.sh
```

- <strong style="color: red;">注意</strong> 节点计算资源有限，请估计或测试好任务需要的 CPU/memory 资源，谨慎填写任务脚本，避免出现有 GPU 但是其他资源不够导致他人任务长时间排队的情况。

### 监控任务状态 <a id="hyperlink2"></a>

- 用 `squeue` 命令查看当前正在运行或排队等待的计算任务。默认情况下，`squeue` 的输出包含以下列：

```bash
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON) 
```

- `squeue` 命令也支持通过 `-o` 参数自定义格式，示例如下。每个 %.Nx 表示一个字段，其中： N 是字段的固定宽度（字符数），不足时填充空格，超过时截断。 x 是字段类型标识符。
  
```bash
squeue -o "%.10i %.9P %.50j %.14u %.12T %.14M %.18R %.6C %.14b %.12m"
# 输出格式示例
     JOBID PARTITION                                               NAME           USER        STATE           TIME   NODELIST(REASON)   CPUS  TRES_PER_NODE   MIN_MEMORY
    252990      gpu5                                            Test.sh    xxxxxxxxxxx      PENDING           0:00        (Resources)      4          gpu:1          40G
    252996      gpu5                                      sbatch_0_0.sh   xxxxxxxxxxxx      PENDING           0:00         (Priority)      4          gpu:1          10G
    252498      gpu5                                      train_cond.sh  xxxxxxxxxxxxx      RUNNING     4-01:53:14      wmc-slave-g19     16          gpu:4          32G
    252596      cpu3                                            run2.sh          xxxxx      RUNNING     3-14:47:09       wmc-slave-g9     12            N/A          20G
```

- 输出解析

| 字段名            | 说明                                                                 |
|-------------------|----------------------------------------------------------------------|
| **JOBID**         | 任务的唯一标识符                                                     |
| **PARTITION**     | 任务所在的队列/分区名称                                              |
| **NAME**          | 任务名称（通常为提交脚本的名称）                                     |
| **USER**          | 提交任务的用户名                                                     |
| **ST(STATE)**     | 任务状态代码（如 `PD`、`R`、`CG` 等）                               |
| **TIME**          | 任务已运行时间（格式为 `天数-小时:分钟:秒`，如 `2-13:45:30`）        |
| **NODES**         | 任务使用的节点数量                                                   |
| **NODELIST(REASON)** | 分配的节点列表（如已运行）或未运行的原因（如 `Resources`、`Priority`） |
| **CPUS**          |​	任务分配的 CPU 核心数量                                            |
| **TRES_PER_NODE** | 每节点可跟踪资源（Trackable RESources per Node），通常为任务分配的 GPU 数目|
| **MIN_MEMORY**    | 任务请求的最小内存量                                                 |

- 常见任务状态

| 代码 | 全称      | 说明                          |
|------|-----------|-------------------------------|
| `PD` | PENDING   | 作业正在排队等待资源         |
| `R`  | RUNNING   | 作业正在运行中               |
| `CG` | COMPLETING| 作业正在收尾（即将完成）      |
| `CD` | COMPLETED | 作业已成功完成               |
| `F`  | FAILED    | 作业因错误失败               |
| `TO` | TIMEOUT   | 作业因超时被终止             |

- 任务执行完毕后会很快从队列中移除，这时 `squeue` 就看不到任务信息了。这时可以通过 `sacct` 查阅任务的审计信息。

```bash
# 根据需要选择参数
sacct -j <jobid> -u <username>
```

### 取消任务

- 使用 `scancel` 取消正在等待或者执行中的任务。任务将会从任务队列中消失。

```bash
scancel <JOB_ID>
```

## 文件传输&管理

>  VS Code 支持 Windows 图形化界面的文件拖曳操作，但不显示传输速度和进度等相关信息，因此我们在这里提供一些其他的文件传输&管理工具。

### 使用 SCP 进行文件传输

- SCP​（Secure Copy Protocol）是一个基于 ​SSH（Secure Shell）​​的安全文件传输协议，用于在本地主机和远程主机之间（或两台远程主机之间）​加密传输文件或目录。
- 正常 Linux 系统都原生支持 SCP 协议；Windows 系统则取决于当前是否已安装 OpenSSH 客户端，而 Windows 10 在 2018 年 4 月更新（版本 1803）中默认安装了 OpenSSH 客户端，只要你的电脑不是古董，大概率可以直接使用。

- 用法示例
```bash
# ​从本地复制到远程
scp /local/path/file.txt username@remote_host:/remote/path/
# 复制整个目录（递归）
scp -r /local/dir username@remote_host:/remote/path/
# 从远程复制到本地
scp username@remote_host:/remote/path/file.txt /local/path/
```

- Linux 和 OpenSSH 客户端还支持 SFTP（SSH File Transfer Protocol）协议，提供多功能文件管理，支持断点续传，但比 SCP 稍慢一些，有需要的用户可以自行查阅相关使用方法。
- 如需图形化操作，推荐使用 ​WinSCP​（同时支持 SCP 和 SFTP ）。

## 常用命令

```bash
# 申请调试节点
salloc --nodelist=<NODE_NAME> -p <PARTITION> -N <NODE_COUNT> -c <CPU_PER_NODE> --mem <MEMORY_SIZE> --gres gpu:<GPU_COUNT>
# 查看任务信息
squeue -o "%.10i %.9P %.50j %.14u %.12T %.14M %.18R %.6C %.14b %.12m"
# 取消任务
scancel <JOB_ID>
```

## 更多

### 导航

- [集群任务脚本](./job_scripts.md)
- [常见问题](./FAQs.md)
- [软件安装](./useful_softwares.md)

