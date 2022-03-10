# 简介

- 本篇是关于`Google Cloud`的一些使用指南，主要是为了解决远程连接的问题。
- 以下不会涉及`Linux`操作系统的应用，内容仅供学术研究使用。

# SSH远程连接

- 使用工具为`Xshell`、`Xftp`，主要目的是为了更改`Google Cloud`远程主机的`root`密码，使得`SSH`工具可连接。

## 1. 切换到root角色

- 使用以下命令切换到`root`用户：

```shell
sudo -i
```

## 2. 修改SSH配置文件

- 修改`/etc/ssh/sshd_config`配置文件，将以下两项默认值改为`yes`：
  - `PermitRootLogin yes`
  - `PasswordAuthentication yes`

```shell
# 默认此项为no，改为yes，运行root角色登录
PermitRootLogin yes
# 默认此项为no，改为yes，开启密码认证
PasswordAuthentication yes
```

## 3. 设定root密码

- 保存配置文件后，修改`root`密码：

```shell
# 修改root密码
passwd root
# 成功后输出以下讯息：
	passwd: all authentication tokens updated successfully.
```

## 4. 重启SSH服务

- `Google Cloud`中找不到重启`SSH`的命令，那么选择重启`VPS`主机：

```shell
# 重启主机
reboot
```

# 总结

- `Google Cloud`和虚拟机不一样的地方在于远程连接上，以上设定之后，就可以使用`Xshell`连接上`VPS`了。