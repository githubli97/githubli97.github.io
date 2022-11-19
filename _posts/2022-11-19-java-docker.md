---
layout:       post
title:        "docker "
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - OS
---

# 使用常见问题
## 容器内时区问题
### docker run
```
docker run -v /etc/localtime:/etc/localtime:ro xxxx
```
### docker-compose
```
  volume:
    - /etc/localtime:/etc/localtime:ro
```
## 清理Docker系统中的无用数据
```
docker system prune --volumes -f
```

# 安装docker
1. 卸载旧的版本
 ```
    sudo yum remove docker \
          docker-client \
          docker-client-latest \
          docker-common \
          docker-latest \
          docker-latest-logrotate \
          docker-logrotate \
          docker-engine
```
2. 设置仓库
```
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
3. 设置阿里安装源
```
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
4. 安装 Docker Engine-Community
```
 sudo yum install docker-ce-20.10.3-3.el7 docker-ce-cli-3:20.10.3-3.el7 containerd.io -y
```
5. 启动docker
```
sudo systemctl start docker
```
6. 设置镜像加速
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2tgmzyyl.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

# 安装docker-compose
- 下载二进制文件
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

```

- 修改可执行文件
```
sudo chmod +x /usr/local/bin/docker-compose
```

# 安装docker-machine
- centos安装
```
curl -L https://github.com/docker/machine/releases/download/v0.16.2/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine &&
    chmod +x /tmp/docker-machine &&
    sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
```

# 安装docker swarm
1. 初始化集群环境
- 使用内网ip即可
```
    docker swarm init --advertise-addr 10.31.149.124
```
2. 宿主机相同网络下的机器执行token即可加入swarm
    1. 初始化网络生成从节点token, manager
    ```
    docker swarm join-token manager
    ```
    2. 初始化网络生成从节点token, worker
    ```
    docker swarm join-token worker
    ```
3. 同网络环境加入swarm，直接输入token即可
4. 加入swarm后，可使用
```
docker node ls
```
查看节点情况

5. 正常情况下至少两主一从
