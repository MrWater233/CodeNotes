# 一.安装oh my zsh

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

# 二.安装额外字体库

```shell
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```

# 三.配置终端配色方案

```shell
cd ~/Desktop/OpenSource

git clone https://github.com/altercation/solarized

open ./solarized/osx-terminal.app-colors-solarized
```

双击`Solarized Dark ansi.terminal`

进入终端-偏好设置-描述文件

设置并使用Solarized Dark ansi主题，设置字体为`Meslo LG S Regular for Powerline`

# 三.安装主题

```shell
vim ./zshrc
```

找到下面配置并修改主题为`agnoster`

```shell
ZSH_THEME="agnoster"
```

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201121120459.png)

# 四.安装命令高亮插件

```shell
cd ~/.oh-my-zsh/custom/plugins/

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git

vi ~/.zshrc
```

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201121120600.png)

