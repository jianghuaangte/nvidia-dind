# nvidia-dind
在宿主机上运行 docker in docker 并在dind中运行需要GPU的容器

## 宿主机设置
一、安装显卡驱动
以英伟达显卡为例：
```shell
# 查看是否安装驱动
nvidia-smi
```

二、安装 NVIDIA Container Toolkit  
- 安装它可以让 docker 容器调用宿主机上的GPU
- 具体安装方法不再写，注意配置/etc/docker/daemon.json
```shell
# 自动配置 注意=docker 应该与实际的显卡调用匹配
sudo nvidia-ctk runtime configure --runtime=docker
```

三、运行带vidia的dind容器
```shell
docker run -d \
  --name nvdia-dind \
  --restart unless-stopped \
  --privileged \
  --gpus all \
  ghcr.1ms.run/jianghuaangte/nvidia-dind:latest
```

一例：
```json
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "args": []
    }
  }
}
```


四、dind中运行需要GPU驱动的容器
以 docker-obsidian 为例
- 没有显示器的服务器还要根据[docker-obsidian](https://github.com/linuxserver/docker-obsidian) 设置内核


```yml
version: '3'
services:
  sealskin:
    image: lscr.io/linuxserver/sealskin:latest
    container_name: sealskin
    networks:
      sealskin:
        ipv4_address: 172.38.0.17
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - "HOST_URL=172.17.0.4" #optional
    volumes:
      - ./config:/config
      - ./storage:/storage
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 8443:8443
      - 8000:8000 #optional
    deploy:  
      resources:  
        reservations:  
          devices:  
            - driver: nvidia  
              count: all  
              capabilities: [gpu]
    restart: unless-stopped

networks:
  sealskin:
    driver: bridge
    ipam:
      config:
        - subnet: 172.38.0.0/16
          gateway: 172.38.0.1   # 设置网关地址
```

## FRP
设置监听本地的127.17.0.4


## 查看是否GPU驱动obsidian
查看日志
```shell
docker logs obsidian
```
