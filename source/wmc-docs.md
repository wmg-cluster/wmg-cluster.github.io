# WMG集群使用指南

> Version: v1.0  |  Date: 2025-04-24

## 使用须知

### 基本信息

- **集群管理和作业调度系统**目前采用**SLURM**（**S**imple **L**inux **U**tility for **R**esource **M**anagement）开源框架。
- WMG计算集群包括一个**登录节点**和若干**计算节点**组成。
- 集群用户直接访问的是**登录节点**，需要通过 Slurm 作业管理器提供的命令行接口向**计算节点**提交计算任务。具体方法参见[编写/提交任务脚本](#编写/提交任务脚本)。
- 计算集群操作系统为使用 **Linux** 内核的 `Ubuntu 20.04` 操作系统。
- 使用集群需要掌握简单的 **Linux** 命令行操作，网上有很多这方面的教程，也可以使用 GPT 等 LLM 工具帮助生成和编辑目标命令。

### 重要注意事项

- <strong style="color: red;">严禁</strong>在登录节点上直接运行计算任务！！！后果包括但不限于**集群登录节点过载，导致用户集体掉线**。

## 登录集群

### 集群账户

需要使用集群的老师和同学请联系管理员开通账户。每个集群用户使用独立的 Linux 账户登录和提交任务。为保证服务器安全，普通用户不提供 root 权限。

- 登录节点： cluster.weiming.win （用于 SSH 连接）

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

## 代码调试

### 计算资源申请&任务调试

- 用户需要通过 `salloc` 命令申请资源后，使用 `srun` 运行调试任务。
- `salloc` 命令示例如下：
```bash
# 使用 Slurm 分配交互式计算资源：
#   --nodelist=wmc-slave-g12  # [可选] 强制指定节点名称，若省略则由系统自动分配
#   -p gpu3                   # 提交到 gpu3 分区（通常为 GPU 节点专用队列）
#   -N 1                      # 分配 1 个计算节点
#   -c 4                      # 每任务占用 4 个 CPU 核心
#   --mem 30G                 # 分配 30GB 内存
#   --gres gpu:1              # 分配 1 块目标 GPU
salloc --nodelist=wmc-slave-g12 -p gpu3 -N 1 -c 4 --mem 30G --gres gpu:1
```
- 获得计算资源分配后，<strong style="color: red;">务必</strong>使用 `srun` 进行调试，否则任务会运行在登录节点上，可能导致集群崩溃。
```bash
# 调试命令必须以 srun 开头，才能运行在分配的计算节点上
# 未加 srun 仍是在登录节点执行命令
hostname                      
# 使用 srun 在计算节点执行命令
srun hostname
# 查看分配的 gpu 信息
srun nvidia-smi
# 运行 python 脚本
srun python debug.py
```
- **推荐**在已分配的 Slurm 计算节点上，直接启动一个交互式 Bash Shell ，这样在后续调试中可以不用 `srun` 命令，防止不小心忘记。
```bash
# 分配临时节点启动交互式 Bash
srun --pty bash
# 进入交互式环境后可自由执行命令，不需要再使用 srun
nvidia-smi
python debug.py
# 退出交互式环境
exit
```

- 结束调试后，需要释放分配的计算资源，这里的 `exit` 命令和退出交互式环境的命令是分开的。
```bash
# 释放分配的计算资源
exit
```
## CUDA 版本问题

- 集群提供多个 CUDA 版本，存放在 `/app/cuda` 路径下。
 TBD

## TBD
```bash
source ~/miniconda3/bin/activate
# 申请1个节点（wmc-slave-g12），4核CPU，30G内存，1块GPU
salloc --nodelist=wmc-slave-g12 -p gpu3 -N 1 -c 4 --mem 30G --gres gpu:1

# 调试完成后释放资源
exit

srun python your_script.py

srun --pty bash
# 进入交互式环境后可自由执行命令
python your_script.py
```

##  重要注意事项

1. **禁止**在登录节点上直接运行计算任务
2. 必须通过`salloc`申请资源后使用`srun`运行任务

## 计算任务管理

### 编写/提交任务脚本

任务脚本用于描述 Slurm 计算任务。下面是一个简单的例子，这个脚本打印了一行文字、执行了一个 python 脚本、最后又打印了一行文字。

- 文件：test_sbatch.sh

```bash
#!/bin/bash
#SBATCH -p gpu1       # 在 gpu1 分区运行（不写默认为 cpu1）
#SBATCH -N 1          # 只在一个节点上运行任务
#SBATCH -c 1          # 申请 CPU 核心：1个
#SBATCH --mem 500     # 申请内存：500 MB
#SBATCH --gres gpu:1  # 分配1个GPU（纯 CPU 任务不用写）
#SBATCH -o /public/ziyuanfang/aigc_detect/SAFE/my_logs/bsl_v2.1.log # 注意可以修改"slurm"为与任务相关的内容方便以后查询实验结果

echo "job begin"
python3 my_script.py  # my_script.py 是用户上传的 python 程序
echo "job end"
```

这个任务脚本实际上就是个 bash 脚本。Slurm 读取以 `#SBATCH` 起始的行，这些行在 bash 语法里是注释，因此 bash 不会理会。

[集群:任务脚本示例](https://wmg.ustc.edu.cn/wiki/index.php/集群:任务脚本示例) 中提供了更多示例。

写好任务脚本后，使用 `sbatch` 命令向集群提交计算任务。若提交成功，`sbatch` 会输出任务编号，该任务会进入队列，请求的资源可用后就会执行。

```
$ sbatch test_sbatch.sh
Submitted batch job 114
```