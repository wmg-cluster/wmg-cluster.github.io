# 便捷脚本工具

## 任务信息邮件通知脚本
```
import smtplib
from email.mime.text import MIMEText
import argparse
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
    msg_from=""  #填入发送方邮箱
    smtp=""       #填入发送方邮箱smtp授权码
    to=""        #填入接收方邮箱
    subject="CLUSTER-INFO"    #电子邮件的主题
    content=f"task {args.task} completed"        #电子邮件的内容

    #构造邮件（注意大写字母）
    msg=MIMEText(content)
    msg["Subject"]=subject
    msg["From"]=msg_from
    msg["To"]=to

    #发送邮件
    try:
        ss=smtplib.SMTP_SSL("smtp.163.com",465)   #使用163邮箱时发送方邮箱smtp安全协议
        # ss=smtplib.SMTP_SSL("smtp.qq.com",465) #使用qq邮箱时的协议设置
        ss.login(msg_from, smtp)                   #登录                 
        ss.sendmail(msg_from,to,msg.as_string()) #发送
        print("email sent successfully")
    except Exception as e:
        print("sending failed: ",e) 
```
- 注意有一点是需要用户自行获得邮箱的smtp码才能够使用，介绍两种常用的邮箱smtp码获取方式
- <strong>qq邮箱</strong>：进入邮箱网页版界面后，点击“设置”，再点击“账号”，在 “POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务” 中管理或者生成smtp授权码即可。
- <strong>163邮箱</strong>：进入邮箱网页版界面后，点击“设置”，再点击“POP3/SMTP/IMAP”中进行配置即可。

- 最后在bash脚本中添加这样一行代码即可使用，`srun python your_mail_script.py -t your_task_name`


## 安装 Linuxbrew (Homebrew)

> 未完成编辑

- 由于用户没有 `sudo` 权限，这里给出不需要权限的 Linuxbrew 安装方法。
```bash
# export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
# export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
# export HOMEBREW_INSTALL_FROM_API=1
# export HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"
# export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"
# export HOMEBREW_PIP_INDEX_URL="https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple"
cd ~
git clone --depth=1 https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/install.git brew-install
# 创建一个文件夹作为安装的位置
mkdir yourbrew
```
- 由于没有 `sudo` 权限，用户对默认安装路径 `/home/linuxbrew/.linuxbrew` 没有写权限，需要注释掉 `~/brew-install/install.sh` 中设置 `HOMEBREW_PREFIX` 的命令：
```sh
else
  UNAME_MACHINE="$(uname -m)"

  # On Linux, this script installs to /home/linuxbrew/.linuxbrew only
  # HOMEBREW_PREFIX="/home/linuxbrew/.linuxbrew"  # 像这样注释掉这一行
  HOMEBREW_REPOSITORY="${HOMEBREW_PREFIX}/Homebrew"
  HOMEBREW_CACHE="${HOME}/.cache/Homebrew"
```
- 执行安装：
```bash
# 安装路径改为新创建的文件夹
export HOMEBREW_PREFIX=/public/yourname/yourbrew
# 安装
/bin/bash brew-install/install.sh
rm -rf brew-install

# TODO
~/.linuxbrew/bin/brew shellenv
test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"
```

- TODO
