# docker

安装：

## windos：

暂定

## mac：

暂定

## linux：

### centos

#### 先决条件：

- 要安装Docker Engine，你需要一个CentOS 7或8的维护版本。归档版本不受支持，也没有经过测试。

- centos-extras存储库必须启用。该存储库在默认情况下是启用的，但如果已经禁用了它，则需要重新启用它。
- 推荐使用overlay2存储驱动程序。

#### 卸载旧版本

/var/lib/docker/的内容(包括图像、容器、卷和网络)会被保留。

```shell
 $ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### 安装方法

1. Install using the repository

   - SET UP THE REPOSITORY

     - 安装yum-utils包(它提供yum-config-manager实用程序)并设置稳定存储库。

       ```shell
        sudo yum install -y yum-utils
        sudo yum-config-manager \
           --add-repo \
           https://download.docker.com/linux/centos/docker-ce.repo
       ```

2. INSTALL DOCKER ENGINE

   ```shell
   sudo yum install docker-ce docker-ce-cli containerd.io
   ```

   

3. Start Docker

   ```shell
   # 将用户添加到docker组中
   id riku
   sudo gpasswd -a riku docker
   id riku
   ```

4. docker run hello-world

   ```shell
   docker run hello-world
   ```

   如果出现以下问题：

   - ![image-20210412103934472](https://i.loli.net/2021/04/12/6I3vLKelH1qkdAw.png)

   - **原因**：docker在本地没有找到hello-world镜像，也没有从docker仓库中拉取镜像，出项这个问题的原因：是因为docker服务器再国外，我们在国内 无法正常拉取镜像，所以就需要我们为docker设置国内阿里云的镜像加速器；

     ```shell
     #需要修改配置文件/etc/docker/daemon.json  如下
     { 
     "registry-mirrors": ["https://alzgoonw.mirror.aliyuncs.com"] 
     }
     ```

     ![image-20210412104607528](https://i.loli.net/2021/04/12/hu6mKQsZHBc1ylE.png)

### jenkins安装

1. install jenkins with docker

   - docker pull jenkins/jenkins

   ```shell
   #建立一个桥接网络
   docker network create jenkins
   
   #构建一个可以在容器中运行的docker
   docker run --name jenkins-docker --rm --detach \
     --privileged --network jenkins --network-alias docker \
     --env DOCKER_TLS_CERTDIR=/certs \
     --volume jenkins-docker-certs:/certs/client \
     --volume jenkins-data:/var/jenkins_home \
     --publish 2376:2376 docker:dind --storage-driver overlay2
     
   #create Dockerfile
   FROM jenkins/jenkins
   USER root
   RUN apt-get update && apt-get install -y apt-transport-https \
          ca-certificates curl gnupg2 \
          software-properties-common
   RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
   RUN apt-key fingerprint 0EBFCD88
   RUN add-apt-repository \
          "deb [arch=amd64] https://download.docker.com/linux/debian \
          $(lsb_release -cs) stable"
   RUN apt-get update && apt-get install -y docker-ce-cli
   USER jenkins
   RUN jenkins-plugin-cli --plugins "blueocean:1.24.5 docker-workflow:1.26"
   
   #docker build image
   docker build -t myjenkins-blueocean:1.1 .
   
   #构建一个jenkins容器
   docker run --name jenkins-blueocean --rm --detach \
     --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
     --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
     --publish 8080:8080 --publish 50000:50000 \
     --volume jenkins-data:/var/jenkins_home \
     --volume jenkins-docker-certs:/certs/client:ro \
     myjenkins-blueocean:1.1
     
   #查看密码
   sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword
   ```

### Gitlab安装

#### 安装前准备

```shell
 vi ~/.bashrc
 export GITLAB_HOME=/srv/gitlab
 source ~/.bashrc
```

#### 启动容器

```shell
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  gitlab/gitlab-ee:latest
```

#### 拉取仓库

```shell
git clone http://gitlab.riku.com:8081/riku/devops.git

git@gitlab.riku.com:riku/blog.git
```

正确git配置文件：

```shell
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[remote "origin"]
        url = http://gitlab.riku.com:8081/riku/devops.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
```

