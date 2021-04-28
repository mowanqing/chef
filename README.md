# chef
learning chef

## 文件介绍
demo.tar.gz 里面有一个web部署recipe，一个nginx负载均衡器，一个logrotate的cookbook
docker-compose.yml 快速构建chef学习环境，包括workstation,server,lb,web1,web2

## 使用方法：
进入docker-compose.yml所在目录，执行：
```shell
docker-compose up -d
docker exec -it workstation bash
docker exec -it server bash
docker exec -it lb bash
docker exec -it web1 bash
docker exec -it web2 bash

#在workstation环境下执行以下命令
eval `ssh-agent` && ssh-add ~/.ssh/id_rsa
```
