---
layout: post
author: HanpiJoker
title: Create A New Development Environment
---

## Create A New Development Environment


## 包管理器安装的基础软件

```shell
sudo apt install git zsh python3 python3-pip python3-venv curl xclip ripgrep \
    fd-find libevent-dev flex bison flex-doc bison-doc xclip bat delta

# install github cli
(type -p wget >/dev/null || (sudo apt update && sudo apt-get install wget -y)) \
&& sudo mkdir -p -m 755 /etc/apt/keyrings \
&& wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
&& sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y

```

## 安装包

- [neovim](https://github.com/neovim/neovim/releases)

- [nerdfonts](https://github.com/ryanoasis/nerd-fonts/releases)

> 安装 FiraCode FiraMono SourceCodePro

- [wps fonts](https://github.com/xChen16/wps-fonts)

- [wps](https://www.wps.cn/)

- [neovide](https://github.com/neovide/neovide/releases)

## zsh & oh-my-zsh & power10k

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

## 利用 Gogh 美化终端界面

```shell
sudo apt-get install dconf-cli uuid-runtime
bash -c "$(wget -qO- https://git.io/vQgMr)"
```

## neovim 的配置

### 1. 安装 npm & nodejs

```shell
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash - 

sudo apt-get install -y nodejs
```

### 2. 安装 latest luarocks

```shell
# https://luarocks.github.io/luarocks/releases/
wget https://luarocks.github.io/luarocks/releases/luarocks-3.11.1.tar.gz
./configure --with-lua-include=/usr/include/lua-5.3
make
sudo make install
```

### 3. 下载配置文件

```shell
git clone -b lua https://github.com/hanpijoker/nvim ~/.config/nvim

sudo npm install -g tree-sitter-cli

sudo npm install -g neovim

sudo python3 -m pip install --upgrade pynvim

# After neovim Lazy install
# need use Mason Install `stylua` and `selena` `shellcheck` `shfmt`
```

## llvm install

```shell
echo "deb http://apt.llvm.org/focal/ llvm-toolchain-focal-19 main
deb-src http://apt.llvm.org/focal/ llvm-toolchain-focal-19 main" | sudo tee -a  /etc/apt/sources.list.d/llvm-toolchain.list

wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | sudo tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc

sudo apt update
sudo apt install clang-19 lldb-19 lld-19 libllvm-19-ocaml-dev libllvm19 llvm-19 llvm-19-dev llvm-19-doc llvm-19-examples \
    llvm-19-runtime clang-tools-19 clang-19-doc libclang-common-19-dev libclang-19-dev libclang1-19 clang-format-19 python3-clang-19\
    clangd-19 clang-tidy-19 libclang-rt-19-dev

```

脚本用于批量配置 clang 和 llvm 的配置链接
```shell
#!/usr/bin/env bash

## update-alternatives for clang & llvm

#     --slave /usr/bin/$1 $1 /usr/bin/$1-\${version} \\

function register_clang_version {
    local version=$1
    local priority=$2

    update-alternatives \
        --install /usr/bin/llvm-config       llvm-config      /usr/bin/llvm-config-${version} ${priority} \
        --slave   /usr/bin/llvm-ar           llvm-ar          /usr/bin/llvm-ar-${version} \
        --slave   /usr/bin/llvm-as           llvm-as          /usr/bin/llvm-as-${version} \
        --slave   /usr/bin/llvm-bcanalyzer   llvm-bcanalyzer  /usr/bin/llvm-bcanalyzer-${version} \
        --slave   /usr/bin/llvm-cov          llvm-cov         /usr/bin/llvm-cov-${version} \
        --slave   /usr/bin/llvm-diff         llvm-diff        /usr/bin/llvm-diff-${version} \
        --slave   /usr/bin/llvm-dis          llvm-dis         /usr/bin/llvm-dis-${version} \
        --slave   /usr/bin/llvm-dwarfdump    llvm-dwarfdump   /usr/bin/llvm-dwarfdump-${version} \
        --slave   /usr/bin/llvm-extract      llvm-extract     /usr/bin/llvm-extract-${version} \
        --slave   /usr/bin/llvm-link         llvm-link        /usr/bin/llvm-link-${version} \
        --slave   /usr/bin/llvm-mc           llvm-mc          /usr/bin/llvm-mc-${version} \
        --slave   /usr/bin/llvm-mcmarkup     llvm-mcmarkup    /usr/bin/llvm-mcmarkup-${version} \
        --slave   /usr/bin/llvm-nm           llvm-nm          /usr/bin/llvm-nm-${version} \
        --slave   /usr/bin/llvm-objdump      llvm-objdump     /usr/bin/llvm-objdump-${version} \
        --slave   /usr/bin/llvm-ranlib       llvm-ranlib      /usr/bin/llvm-ranlib-${version} \
        --slave   /usr/bin/llvm-readobj      llvm-readobj     /usr/bin/llvm-readobj-${version} \
        --slave   /usr/bin/llvm-rtdyld       llvm-rtdyld      /usr/bin/llvm-rtdyld-${version} \
        --slave   /usr/bin/llvm-size         llvm-size        /usr/bin/llvm-size-${version} \
        --slave   /usr/bin/llvm-stress       llvm-stress      /usr/bin/llvm-stress-${version} \
        --slave   /usr/bin/llvm-symbolizer   llvm-symbolizer  /usr/bin/llvm-symbolizer-${version} \
        --slave   /usr/bin/llvm-tblgen       llvm-tblgen      /usr/bin/llvm-tblgen-${version}

    update-alternatives \
        --install /usr/bin/clang                 clang                 /usr/bin/clang-${version} ${priority} \
        --slave   /usr/bin/clang++               clang++               /usr/bin/clang++-${version}  \
        --slave   /usr/bin/asan_symbolize        asan_symbolize        /usr/bin/asan_symbolize-${version} \
        --slave   /usr/bin/c-index-test          c-index-test          /usr/bin/c-index-test-${version} \
        --slave   /usr/bin/clang-check           clang-check           /usr/bin/clang-check-${version} \
        --slave   /usr/bin/clang-cl              clang-cl              /usr/bin/clang-cl-${version} \
        --slave   /usr/bin/clang-cpp             clang-cpp             /usr/bin/clang-cpp-${version} \
        --slave   /usr/bin/clang-format          clang-format          /usr/bin/clang-format-${version} \
        --slave   /usr/bin/clang-format-diff     clang-format-diff     /usr/bin/clang-format-diff-${version} \
        --slave   /usr/bin/clang-import-test     clang-import-test     /usr/bin/clang-import-test-${version} \
        --slave   /usr/bin/clang-include-fixer   clang-include-fixer   /usr/bin/clang-include-fixer-${version} \
        --slave   /usr/bin/clang-offload-bundler clang-offload-bundler /usr/bin/clang-offload-bundler-${version} \
        --slave   /usr/bin/clang-query           clang-query           /usr/bin/clang-query-${version} \
        --slave   /usr/bin/clang-rename          clang-rename          /usr/bin/clang-rename-${version} \
        --slave   /usr/bin/clang-reorder-fields  clang-reorder-fields  /usr/bin/clang-reorder-fields-${version} \
        --slave   /usr/bin/clang-tidy            clang-tidy            /usr/bin/clang-tidy-${version} \
        --slave   /usr/bin/lldb                  lldb                  /usr/bin/lldb-${version} \
        --slave   /usr/bin/lldb-server           lldb-server           /usr/bin/lldb-server-${version}
}

register_clang_version $1 $2

```

## fzf install

```shell
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.config/fzf
~/.config/fzf/install --xdg --all
```

## tmux install

```shell
# https://github.com/tmux/tmux/releases

./configure
make && sudo make install

git clone https://github.com/HanpiJoker/tmux.git ~/.config/tmux

pushd > $HOME/.config
ln -sf ./tmux/tmux-powerline
popd
```

## sogoupinyin install

[搜狗拼音安装指导](https://shurufa.sogou.com/linux/guide)

```shell
sudo apt remove ibus --purge
sudo apt install fctix
# download sogou pinyin deb
sudo cp /usr/share/applications/fcitx.desktop /etc/xdg/autostart/
sudo apt install libqt5qml5 libqt5quick5 libqt5quickwidgets5 qml-module-qtquick2 libgsettings-qt1
sudo reboot
```
