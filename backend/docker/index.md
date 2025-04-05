# docker

## 下载安装 docker

### CentOS（dnf/yum）

在下载 docker 的时候，会发现有很多的组件，这些组件是做什么的？

1. docker-ce
   - Docker CE 是 Docker 的开源版本（社区维护）。
   - 安装后会有 dockerd 守护进程 Docker Daemon (dockerd)，负责管理容器、镜像、网络、卷等。
   - 它会启动后台服务 systemctl start docker。
2. docker-ce-cli
   - 提供 docker 命令工具（比如 docker run, docker build, docker ps 等）。
   - 与 docker-ce 配合使用：CLI 发命令给守护进程 dockerd。
   - 如果只安装 docker-ce 而没有 CLI，你将无法用命令行操作 Docker。
3. containerd
   - Docker 默认的容器运行时。
   - 用来真正创建、运行、暂停、停止容器。
   - Docker 只是上层控制器，它底层调用的是 containerd。
   - 现在很多项目（如 Kubernetes）也可以直接使用 containerd 来运行容器，不依赖 dockerd。
   - 作用类似于：
     - Docker → 控制平台
     - containerd → 工人，实际干活
4. docker-compose-plugin
   - Docker Compose 的插件。
   - 它允许你通过命令行管理 Docker Compose 的容器、镜像、网络等。
5. docker-buildx-plugin
   - Docker Buildx 的插件。
   - 它允许你通过命令行管理 Docker Buildx 的容器、镜像、网络等。

```bash
你输入 docker 命令, 如 docker run hello-world（docker-ce-cli）
        ↓
Docker 守护进程 dockerd（docker-ce）
        ↓
调用 containerd 运行容器（containerd.io）
```

正式安装流程：

```bash
# 安装dnf-plugins-core，它是dnf的插件，用于管理yum源
sudo dnf -y install dnf-plugins-core

# 添加yum源
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo # 添加docker的yum源
sudo dnf config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 添加阿里云的yum源

# 安装 docker 的各个组件
sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-buildx-plugin

# 设置开机自启动，且同时立刻启动docker
sudo systemctl enable --now docker
# 上面的命令等价于
sudo systemctl enable docker
sudo systemctl start docker
```

[ERROR1：安装成功了，但服务没启动](#error1)

## 配置 docker 镜像加速

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": [
        "https://xxx",
    ]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 常见镜像加速地址

[Docker/DockerHub 国内镜像源/加速列表（3 月 16 日更新-长期维护）](https://zhuanlan.zhihu.com/p/24461370776)

```bash
# 阿里云
https://<你的ID>.mirror.aliyuncs.com

# 腾讯云
https://mirror.ccs.tencentyun.com

# 华为云
https://h.docker.huaweicloud.com

# 七牛云
https://docker.nq.com

# 中科大
https://docker.mirrors.ustc.edu.cn

# 网易
https://hub-mirror.c.163.com

# 其他
https://docker.1ms.run
https://docker.xuanyuan.me
```

## 拉取镜像

```bash
# 拉取镜像
docker pull hello-world

# 运行镜像
docker run hello-world

# 查看镜像
docker images
```

## docker 的常见 ERROR

### ERROR1

安装成功了，但服务没启动

```bash
[ecs-user@iZ0jl0uxv85b2iysieugfnZ ~]$ docker run hello-world
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
See 'docker run --help'.

# Docker 安装成功了，但服务没启动
sudo systemctl status docker # 查看docker状态
sudo systemctl start docker # 启动docker

sudo systemctl enable docker # 另外可以设置开机自动启动 docker
```

### ERROR2

docker 运行权限问题

```bash
[ecs-user@iZ0jl0uxv85b2iysieugfnZ ~]$ docker run hello-world
docker: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Head "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'.

# 以普通用户（ecs-user）运行 docker，但该用户 没有权限访问 Docker 的 socket 文件（即 /var/run/docker.sock）
# 默认只有 root 和 docker 用户组成员才能访问 Docker 守护进程
# 查看 docker.sock 文件权限
[ecs-user@iZ0jle5t8rkuvgnsh9rhkaZ ~]$ ls -l /var/run/docker.sock
srw-rw---- 1 root docker 0 Apr  5 12:33 /var/run/docker.sock

# 添加当前用户到docker组
sudo usermod -aG docker $USER # 添加当前用户到docker组

# 查看当前用户所属组
groups $USER

# 退出机器
exit

# 重新登录机器
ssh aliyun
```
