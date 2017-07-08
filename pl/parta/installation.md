# 安装软件

Part A 使用 `Standard ML` 语言，采用 Standard ML of New Jersey compiler (SML/NJ) + Emacs 组合工具。

## 安装 Emacs

从 [这里](https://emacsformacosx.com/) 下载 Emacs Mac 版安装，需要 24.x 以上版本。

## Emacs 快捷键

![Emacs](./images/emacs.png)

* 光标是矩形块，称之为点。
* `buffer`：逻辑概念，比如当打开一个文件时，该文件被装载到 `buffer` 中；但 `buffer` 并非一定和文件相关。
* `mode`：每个 `buffer` 都在特定模式下编辑，不同语言有不同模式，最基础的是 `Fundamental` 模式。

Emacs 快捷键经常使用 `Control` `Meta` 键，分别用 `C-x` `M-x` 标记（注意，现在电脑很少有 `Meta` 键，在 windows 上用 `Alt` 代替，在 mac 上用 `Option` 代替）。当命令输入错误时，使用 `C-g` 或 `Esc` 取消当前命令。

基本命令：

* `C-x C-c` 退出 Emacs
* `C-g` 取消当前动作
* `C-x C-f` 打开文件
* `C-x C-s` 保存文件
* `C-x C-w` 写文件（另存为）

获取帮助：

* `C-h`
* `C-h b` 获取当前模式下的快捷键绑定

## 安装 SML/NJ

在 [这里](http://smlnj.cs.uchicago.edu/dist/working/110.81/index.html) 下载安装包，使用 `.𝚙𝚔𝚐` 版本，不要用 `.dmg`，安装路径选择默认。

编辑 `~/.bash_profile` 文件，添加 `export PATH=$PATH:/usr/local/smlnj/bin`，并导入配置 `source .bash_profile`。

在终端输入 `𝚜𝚖𝚕` 验证是否安装成功，使用 `Ctrl D` 退出 `sml`。
