---
title: "zsh,oh-my-zsh安装"
date: 2021-04-24T16:18:28+08:00
draft: false
toc: false
tags: ["shell"]
---

~~~shell
# 安装zsh
sudo apt-get install zsh

# 将zsh切换成默认shell
chsh -s /bin/zsh

# 安装 oh-my-zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Add powerlevel10k to the list of Oh My Zsh themes.
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
# Replace ZSH_THEME="powerlevel9k/powerlevel9k" with ZSH_THEME="powerlevel10k/powerlevel10k".
sed -i.bak 's/powerlevel9k/powerlevel10k/g' ~/.zshrc


# 安装插件
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

vim ~/.zshrc
# 修改文件内容
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
# 保存,执行文件
source ~/.zshrc
~~~