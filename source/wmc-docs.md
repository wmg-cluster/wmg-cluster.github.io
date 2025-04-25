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

- <strong style="color: red;">严禁</strong>在登录节点上直接运行计算任务！！！后果包括但不限于<strong style="color: red;">集群登录节点过载，导致用户集体掉线</strong>。
- <strong style="color: red;">务必</strong>记得使用 `srun` 命令或者通过 `srun --pty bash` 启动交互式 Bash 环境！！！

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

### TODO

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
srun python your_script.py
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

### 查看当前版本

- 可以使用以下命令查看版本，如果驱动支持 CUDA 版本过低，请联系管理员更新 GPU 节点驱动。

```bash
# 驱动支持的最高 CUDA 版本​（不一定是实际安装版本）
nvidia-smi
# 当前实际安装的 CUDA Toolkit 版本
nvcc --version
```

### 更改使用的 CUDA 版本

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

## 计算任务管理

### 编写/提交任务脚本

- 任务脚本用于描述 Slurm 计算任务。下面是一个简单的例子，这个脚本打印了一行文字、执行了一个 python 脚本、最后又打印了一行文字。

- 文件：test_sbatch.sh

```bash
#!/bin/bash
#SBATCH -p gpu1             # 在 gpu1 分区运行（不写默认为 cpu1）
#SBATCH -N 1                # 只在一个节点上运行任务
#SBATCH -c 1                # 申请 CPU 核心：1个
#SBATCH --mem 500           # 申请内存：500 MB
#SBATCH --gres gpu:1        # 分配1个GPU（纯 CPU 任务不用写）
#SBATCH -o test-%j.log      # 输出 log 文件，%j会被自动替换为任务的ID（JobID）​

echo "job begin"
python3 my_script.py  # my_script.py 是用户的 python 程序
echo "job end"
```

- 这个任务脚本实际上就是个 bash 脚本。Slurm 读取以 `#SBATCH` 起始的行，这些行在 bash 语法里是注释，因此 bash 不会理会。[集群任务脚本](./job_scripts.md) 中提供了更多示例。

- 写好任务脚本后，使用 `sbatch` 命令向集群提交计算任务。若提交成功，`sbatch` 会输出任务编号，该任务会进入队列，请求的资源可用后就会执行。

```bash
# 这里也可以用 --nodelist 参数指定节点
sbatch test_sbatch.sh
```

### 监控任务状态

- 用 `squeue` 命令查看当前正在运行或排队等待的计算任务。默认情况下，`squeue` 的输出包含以下列：
```bash
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON) 
```
- 输出解析

| 字段名            | 说明                                                                 |
|-------------------|----------------------------------------------------------------------|
| ​**JOBID**​         | 任务的唯一标识符                                                     |
| ​**PARTITION**​     | 任务所在的队列/分区名称                                              |
| ​**NAME**​          | 任务名称（通常为提交脚本的名称）                                     |
| ​**USER**​          | 提交任务的用户名                                                     |
| ​**ST**​            | 任务状态代码（如 `PD`、`R`、`CG` 等）                               |
| ​**TIME**​          | 任务已运行时间（格式为 `天数-小时:分钟:秒`，如 `2-13:45:30`）        |
| ​**NODES**​         | 任务使用的节点数量                                                   |
| ​**NODELIST(REASON)​**​ | 分配的节点列表（如已运行）或未运行的原因（如 `Resources`、`Priority`） |

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

### TODO

## 便捷脚本工具

### 任务信息邮件通知脚本

- [更多](./handy_script_utilities.md)

