---
title: "我的第一篇博客"
description: "这是我用 Astro + Markdown 搭建的第一篇博客文章。"
pubDate: 2026-07-08
tags: ["Astro", "博客", "Markdown"]
draft: false
---


# Bash 终端配置文档

本文档对 Bash 进行配置，目标让 Bash 获得 zsh 的体验：

- 使用 **Bash** 作为默认 shell
- 使用 **ble.sh** 实现自动补全、历史建议、语法高亮
- 使用 **zoxide** 实现 `z` 目录跳转
- 使用 **starship** 实现 prompt 美化
- 使用 **bash-completion** 增强 Tab 补全

---

## 1. 安装 bash-completion

```bash
sudo apt update
sudo apt install -y bash-completion
```

`.bashrc` 中配置：

```bash
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
```

---

## 2. 安装 ble.sh

### 2.1 安装

```bash
sudo apt update
sudo apt install -y git make gawk

git clone --recursive https://github.com/akinomyoga/ble.sh.git
cd ble.sh
make
make INSDIR="$HOME/.local/share/blesh" install
```

安装后目录类似：

```bash
ls ~/.local/share/blesh/
```

可看到：

```text
ble.sh  cache.d  contrib  doc  lib  licenses  run
```

### 2.2 bashrc 配置

`.bashrc` 最顶部加入：

```bash
# Write the following line near the top of .bashrc
[[ $- != *i* ]] && return
source -- "$HOME/.local/share/blesh/ble.sh" --attach=none --rcfile "$HOME/.blerc"
```

`.bashrc` 最底部加入：

```bash
# Write the following line at the bottom of .bashrc
[[ ! ${BLE_VERSION-} ]] || ble-attach
```

### 2.3 ~/.blerc 配置

创建文件：

```bash
code ~/.blerc
```

粘贴以下内容：

``` text
# =========================
# ble.sh auto-complete style
# =========================

# 开启自动补全 / 历史建议
bleopt complete_auto_complete=1
bleopt complete_auto_history=1

# 自动补全延迟，单位 ms；越大越不打扰
bleopt complete_auto_delay=150

# 错误命令：只显示红色字体，不要红色背景
ble-face -s syntax_error fg=red

# 参数错误：也只显示红色字体，不要背景
ble-face -s argument_error fg=red

# 自动补全建议：灰色、无背景、无下划线，接近 zsh autosuggestions
ble-face -s auto_complete fg=240

# 关闭 ble.sh 的退出状态提示，例如 [ble: exit 127]
bleopt exec_errexit_mark=
bleopt exec_exit_mark=

# 可选：关闭其他 ble 标记，例如 [ble: ...]
bleopt edit_marker=
bleopt edit_marker_error=


# =========================
# ble.sh syntax highlight style
# =========================

# 命令统一绿色：cd/source/conda/ros2/git/code 等
ble-face -s syntax_command fg=green
ble-face -s command_builtin fg=green
ble-face -s command_builtin_dot fg=green
ble-face -s command_file fg=green
ble-face -s command_function fg=green
ble-face -s command_alias fg=green
ble-face -s command_keyword fg=green

# 错误命令：只红色字体，不要红色背景
ble-face -s syntax_error fg=red

# 路径统一白色：/opt/ros/...、~/work/...、文件/目录参数
ble-face -s filename_directory fg=white
ble-face -s filename_directory_sticky fg=white
ble-face -s filename_link fg=white
ble-face -s filename_orphan fg=white
ble-face -s filename_executable fg=white
ble-face -s filename_other fg=white
ble-face -s filename_url fg=white
ble-face -s filename_ls_colors fg=white

# 选项参数颜色，例如 -s、--help、--build-type
ble-face -s argument_option fg=teal

```

---

## 3. 安装 zoxide

zoxide 用于实现类似 zsh 中 `z` 的目录记忆跳转。

### 3.1 使用 cargo 安装

安装 cargo：

```bash
sudo apt update
sudo apt install -y cargo

```

安装 zoxide：

```bash
# zoxide 版本根据当前 Rust 版本选择合适进行安装，此处 0.9.7 仅是示例
cargo install zoxide --version 0.9.7 --locked
```

### 3.2 bashrc 配置

`.bashrc` 中加入：

```bash
export PATH="$HOME/.cargo/bin:$PATH"
command -v zoxide >/dev/null 2>&1 && eval "$(zoxide init bash)"
```

检查：

```bash
which zoxide
zoxide --version
```

### 3.3 使用方式

直接跳转：

```bash
z robot
z sdk
z daq
z humanoid
```

交互式选择跳转：

```bash
zi robot
zi sdk
zi daq
zi humanoid
```

查看已记录目录：

```bash
zoxide query -l
zoxide query -l robot
```


## 4. 安装 starship

starship 用于美化 Bash prompt。

### 4.1 不建议 cargo 安装最新版

执行：

```bash
cargo install starship --locked
```


### 4.2 使用官方预编译安装脚本

显式指定平台：

```bash
mkdir -p ~/.local/bin

curl -sS https://starship.rs/install.sh | sh -s -- \
  --bin-dir "$HOME/.local/bin" \
  --yes \
  --platform unknown-linux-musl \
  --arch x86_64
```

检查：

```bash
which starship
starship --version
```

### 4.3 bashrc 配置

`.bashrc` 中加入：

```bash
command -v starship >/dev/null 2>&1 && eval "$(starship init bash)"
```

### 4.4 配置 starship

配置文件：

```bash
code ~/.config/starship.toml
```

我的自定义配置：

```toml
format = "$conda$username$directory$git_branch$character"
add_newline = false

[conda]
format = '\([$environment]($style)\) '
style = 'white'
ignore_base = false

[username]
show_always = true
format = '[$user]($style):'
style_user = 'bold white'
style_root = 'bold red'

[directory]
format = '[$path/]($style) '
style = 'bold blue'
truncation_length = 1
truncate_to_repo = false
home_symbol = '~'

[git_branch]
format = '\([$branch]($style)\) '
style = 'bold yellow'

[character]
success_symbol = '[\$](white)'
error_symbol = '[\$](white)'
```


