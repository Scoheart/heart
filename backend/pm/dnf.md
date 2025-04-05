# dnf

## 使用

```bash
# 添加源
sudo dnf config-manager --add-repo <repo-url>

# example
sudo dnf config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

```bash
# 查看一个源中所有包
sudo dnf repoquery --repoid=<repo-id>

# example
sudo dnf repoquery --repoid=docker-ce-stable 
```

```bash
# 下载包
sudo dnf install <package-name>

# example
sudo dnf install docker-ce
```