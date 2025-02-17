# Docker 代理脚本

这个脚本用于优化 Docker 镜像的拉取速度。通过转发本地代理，解决 Docker Hub 访问速度慢的问题。

## 系统要求

- Linux 或 macOS 操作系统
- 已安装 socat
- 本地已配置代理服务(可以访问 Docker Hub)
- sudo/root 权限

## 功能特点

- 自动配置 Docker Hub 相关域名的本地代理
- 支持自定义代理端口
- 支持 Linux 和 macOS 系统
- 支持 SSH 隧道转发
- 一键开启/关闭代理
- 自动备份和恢复 hosts 文件

## 使用方法

### 第一步：建立 SSH 隧道转发

在使用脚本之前，需要先建立 SSH 隧道，将本地代理端口转发到远程服务器：

```bash
# 格式：ssh -R [远程主机端口]:[本地主机]:[本地端口] [远程主机]
ssh -R 7890:localhost:7890 user@remote-server

# 例如：将本地 7890 端口转发到远程服务器的 7890 端口
ssh -R 7890:localhost:7890 root@example.com
```

注意事项：
- 确保 SSH 配置允许远程端口转发
- 可能需要在远程服务器的 sshd_config 中启用 `GatewayPorts yes`
- 建议使用 SSH 密钥认证并配置 ~/.ssh/config 简化连接
- ssh扩展文章 
  - [SSH原理与运用（二）：远程操作与端口转发](https://www.ruanyifeng.com/blog/2011/12/ssh_port_forwarding.html)
  - [SSH Tunneling: Examples, Command, Server Config](https://www.ssh.com/academy/ssh/tunneling-example)

### 第二步：在远程服务器上运行代理脚本

#### 方式一：直接运行（推荐）
```bash
curl -sSL https://github.com/cnm4ak/shell-docker-pull/raw/main/docker-proxy.sh | sudo bash
```

停止代理：
```bash
curl -sSL https://github.com/cnm4ak/shell-docker-pull/raw/main/docker-proxy.sh | sudo bash -s stop
```

#### 方式二：下载后运行

1. 下载脚本并添加执行权限：
```bash
curl -O https://github.com/cnm4ak/shell-docker-pull/raw/main/docker-proxy.sh
chmod +x docker-proxy.sh
```

2. 启动代理（默认使用7890端口）：
```bash
sudo ./docker-proxy.sh
```

3. 使用自定义代理端口启动：
```bash
sudo ./docker-proxy.sh -p 1087
```

4. 停止代理：
```bash
sudo ./docker-proxy.sh stop
```

## 安装依赖

在使用脚本之前，请确保已安装 socat：

### macOS
```bash
brew install socat
```

### Ubuntu/Debian
```bash
sudo apt-get install socat
```

### CentOS/RHEL
```bash
sudo yum install socat
```

## 参数说明

- `-p <端口号>`: 指定代理端口号（默认：7890）
- `stop`: 停止代理服务

## 工作原理

1. 通过 SSH 隧道将本地代理端口转发到远程服务器
2. 脚本通过修改 hosts 文件将 Docker Hub 相关域名指向本地
3. 使用 socat 建立本地端口转发
4. 通过本地代理服务转发 Docker Hub 的请求

## 注意事项

1. 确保先建立 SSH 隧道再运行代理脚本
2. 脚本需要以 root 权限运行
3. 确保本地代理服务正常运行
4. 使用前请确保指定的代理端口正确
5. 脚本会自动备份 hosts 文件，停止代理时会自动恢复
6. 使用 SSH 隧道时注意网络稳定性和安全性

## 故障排除

1. 如果代理启动后仍然无法拉取镜像：
   - 检查本地代理服务是否正常运行
   - 确认代理端口是否正确
   - 检查网络连接是否正常
   - 验证 SSH 隧道是否正常工作


## 项目背景

### 问题场景
在运维环境中，我们经常遇到这样的情况：
1. 在内网环境或云服务器
2. Docker Hub 访问速度极慢或经常超时
3. Docker 服务正在运行重要的生产容器
4. 不能重启 Docker 服务或修改 Docker 的全局配置

### 解决思路
1. 不修改 Docker 守护进程配置（避免重启 Docker）
2. 利用 hosts 文件和本地端口转发（无需修改 Docker 配置）
3. 通过 SSH 隧道转发将本地代理能力扩展到远程服务器
4. 实现对 Docker Hub 请求的透明代理


## 许可证 

本项目采用 Apache 许可证，详细内容请参见 [LICENSE](https://github.com/cnm4ak/shell-docker-pull/blob/main/LICENSE) 文件。

## 贡献

欢迎提交 Issue 和 Pull Request！