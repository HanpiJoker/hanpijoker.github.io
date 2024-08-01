---
layout: post
author: HanpiJoker
title: Create A New Development Environment
---

## Create A New Development Environment

## 安装基础软件

### 通过包管理器安装的基础软件

```shell
sudo apt install git zsh python3 python3-pip python3-venv curl xclip ripgrep \
    fd-find libevent-dev flex bison flex-doc bison-doc xclip

# install github cli
(type -p wget >/dev/null || (sudo apt update && sudo apt-get install wget -y)) \
&& sudo mkdir -p -m 755 /etc/apt/keyrings \
&& wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
&& sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y

```

### 通过链接下载的安装包

- [neovim](https://github.com/neovim/neovim/releases)

- [nerdfonts](https://github.com/ryanoasis/nerd-fonts/releases)

> 安装 FiraCode FiraMono SourceCodePro

- [wps fonts](https://github.com/xChen16/wps-fonts)

- [wps](https://www.wps.cn/)

- [neovide](https://github.com/neovide/neovide/releases)
```

## 软件配置

### zsh & oh-my-zsh & power10k

1. 安装 oh-my-zsh

```shell
wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh
ZSH=$HOME/.config/oh-my-zsh sh install.sh
```

2. 安装 powerlevel10k

```shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# Set ZSH_THEME="powerlevel10k/powerlevel10k" in ~/.zshrc
```

3. 安装 zoxide

```shell
curl -sSfL https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | sh
```
> Add this to the end of your config file (usually ~/.zshrc):
> 
> ```shell
> eval "$(zoxide init zsh)"
> ```
> For completions to work, the above line must be added after compinit is called. You may have to rebuild your completions
> cache by running rm ~/.zcompdump*; compinit. 

4. 安装插件

```
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md#oh-my-zsh)

> ```shell
> git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
> ```

- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)

> ```shell
> git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
> ```

### 利用 Gogh 美化终端界面

```shell
sudo apt-get install dconf-cli uuid-runtime
bash -c "$(wget -qO- https://git.io/vQgMr)"
```

### neovim 的配置

#### 1. 安装 npm & nodejs

```shell
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash - 

sudo apt-get install -y nodejs
```

#### 2. 安装 latest luarocks

```shell
# https://luarocks.github.io/luarocks/releases/
wget https://luarocks.github.io/luarocks/releases/luarocks-3.11.1.tar.gz
./configure --with-lua-include=/usr/include/lua-5.3
make
sudo make install
```

#### 3. 下载配置文件

```shell
git clone -b lua https://github.com/hanpijoker/nvim ~/.config/nvim

sudo npm install -g tree-sitter-cli

sudo npm install -g neovim

sudo python3 -m pip install --upgrade pynvim

# After neovim Lazy install
# need use Mason Install `stylua` and `selena` `shellcheck` `shfmt`
```

### llvm install

```shell
echo "deb http://apt.llvm.org/focal/ llvm-toolchain-focal-18 main
deb-src http://apt.llvm.org/focal/ llvm-toolchain-focal-18 main" | sudo tee -a  /etc/apt/sources.list.d/llvm-toolchain.list

wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | sudo tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc

sudo apt update
sudo apt install clang-18 lldb-18 lld-18 libllvm-18-ocaml-dev libllvm18 llvm-18 llvm-18-dev llvm-18-doc llvm-18-examples \
    llvm-18-runtime clang-tools-18 clang-18-doc libclang-common-18-dev libclang-18-dev libclang1-18 clang-format-18 python3-clang-18\
    clangd-18 clang-tidy-18 libclang-rt-18-dev
```

### fzf install

```shell
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.config/fzf
~/.config/fzf/install --xdg --all
```

### tmux install

```shell
# https://github.com/tmux/tmux/releases
```

### sogoupinyin install

[搜狗拼音安装指导](https://shurufa.sogou.com/linux/guide)

```shell
sudo apt remove ibus --purge
sudo apt install fctix
# download sogou pinyin deb
sudo cp /usr/share/applications/fcitx.desktop /etc/xdg/autostart/
sudo apt install libqt5qml5 libqt5quick5 libqt5quickwidgets5 qml-module-qtquick2 libgsettings-qt1
sudo reboot
```
