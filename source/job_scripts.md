# 集群任务脚本

## 任务信息邮件通知

- TODO

## 多卡任务通信端口

- 多卡任务通常需要指定通信端口，为避免端口冲突，建议在任务脚本通过以下代码动态确定端口。
```bash
# 生成 50000~60000 的随机端口，直到找到可用端口 MASTER_PORT
while :; do
    MASTER_PORT=$(( RANDOM % 10001 + 50000 ))
    if ! ss -tuln | grep -q ":${MASTER_PORT} "; then
        echo "Selected available port: ${MASTER_PORT}"
        break
    fi
done
```