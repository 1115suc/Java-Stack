# Docker镜像源

```shell
mkdir -p /etc/docker
```

```shell
vim /etc/docker/daemon.json  
```

```shell
{
  "registry-mirrors": [
    "https://docker.1ms.run",  
    "https://dockerproxy.com", 
    "https://hub.rat.dev"      
  ]
}
```

```shell
sudo systemctl daemon-reload
```

```shell
sudo systemctl restart docker
```

```shell
docker info
```