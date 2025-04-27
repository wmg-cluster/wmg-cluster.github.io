# 集群任务脚本

## 任务信息邮件通知

- 为了方便监控任务运行状况，可以添加一个自动发送邮件的 python 脚本。

```python
import smtplib
import argparse
from email.mime.text import MIMEText

def parse_args():
    parser = argparse.ArgumentParser(
        description="Provide message to send."
    )
    parser.add_argument(
        '-t', '--task', 
        type=str, 
        default='train',
        help="The name of submitted task",
    )
    return parser.parse_args()


if __name__ == '__main__':
    args = parse_args()
    # print(args.task)
    msg_from=""                                  #填入发送方邮箱
    smtp=""                                      #填入发送方邮箱 smtp 授权码
    to=""                                        #填入接收方邮箱
    subject="CLUSTER-INFO"                       #电子邮件的主题
    content=f"task {args.task} completed"        #电子邮件的内容

    #构造邮件（注意大写字母）
    msg=MIMEText(content)
    msg["Subject"]=subject
    msg["From"]=msg_from
    msg["To"]=to

    #发送邮件
    try:
        ss=smtplib.SMTP_SSL("smtp.163.com",465)     #使用163邮箱时发送方邮箱smtp安全协议
        # ss=smtplib.SMTP_SSL("smtp.qq.com",465)    #使用qq邮箱时的协议设置
        ss.login(msg_from, smtp)                    #登录                 
        ss.sendmail(msg_from,to,msg.as_string())    #发送
        print("email sent successfully")
    except Exception as e:
        print("sending failed: ",e) 
```
- 注意有一点是需要用户自行获得邮箱的 smtp 码才能够使用，下面是两种常用的邮箱 smtp 码获取方式：
  - **qq邮箱**：进入邮箱网页版界面后，点击“设置”，再点击“账号”，在 “POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务” 中管理或者生成 smtp 授权码即可。
  - **163邮箱**：进入邮箱网页版界面后，点击“设置”，再点击“POP3/SMTP/IMAP”中进行配置即可。

- 最后在 bash 脚本中添加这样一行代码即可使用。
```bash
# your_task_name 替换成你的任务名称
srun python your_mail_script.py -t your_task_name
```

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