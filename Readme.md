# ELMEC Document

wanglinqiang 2025/1/25

## Lists

## Git
### 查看所有的配置
你可以通过以下命令查看所有的配置以及它们所在的文件：
```sh
$ git config --list --show-origin
```

### 用户信息
安装完 Git 之后，要做的第一件事就是设置你的用户名和邮件地址。 这一点很重要，因为每一个 Git 提交都会使用这些信息，它们会写入到你的每一次提交中，不可更改：
```sh
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```
再次强调，如果使用了 --global 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情， Git 都会使用那些信息。 当你想针对特定项目使用不同的用户名称与邮件地址时，可以在那个项目目录下运行没有 --global 选项的命令来配置。

很多 GUI 工具都会在第一次运行时帮助你配置这些信息。

## SSH设置

### 生成 SSH 密钥对

打开终端或命令提示符，并输入以下命令以生成 SSH 密钥对:
```sh
ssh-keygen -t rsa -b 4096 -C "wanglinqiang@gmail.com"
```

### 将 SSH 公钥添加到 GitHub

1. 复制 SSH 公钥内容
公钥文件`C:\Users\Administrator\.ssh/id_rsa.pub`

2. 粘贴 SSH 公钥内容
登录到 GitHub，进入 Settings -> SSH and GPG keys，点击 New SSH key，粘贴公钥内容并保存。

### SSH 连接测试
```sh
ssh -T git@github.com
```

### 打开VSCODE 用 SSH 方式克隆仓库