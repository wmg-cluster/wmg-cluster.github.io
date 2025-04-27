# 便捷脚本工具

## TODO

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
