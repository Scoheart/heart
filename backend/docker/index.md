# docker

## 下载安装 docker

### CentOS（dnf/yum）

在下载 docker 的时候，会发现有很多的组件，这些组件是做什么的？

- docker-ce：docker 社区版
- docker-ce-cli：docker 命令行工具
- containerd：容器运行时
- docker-compose-plugin：docker compose 插件
- docker-buildx-plugin：docker buildx 插件

```bash
# 添加源
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装docker
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
