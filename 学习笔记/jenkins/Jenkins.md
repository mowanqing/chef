# Jenkins

## Jenkins简介

### 持续集成：

最简单的形式就是一个能监控你版本控制系统变化的工具，无论任何时候，只要检测到由变化，这个工具就会自动编译和测试你的应用程序，如果出现问题，马上通知开发人员，以便他们可以立即着手解决这个问题。

- 密切监视代码库的健康，自动监控代码质量和代码覆盖率度量，帮助降低技术债务和减少维护成本
- 结合自动化的端到端的验收测试，持续集成也可以作为一种沟通工具，清晰地发布和展现总体开发工作的当前状态。通过构建自动化部署过程，持续集成能大大简化和加速你的交付过程，自动化和一键部署应用程序的最新版本。
- 本质上，持续集成是通过提供更快的反馈来降低风险的。它被设计成用来更快地识别和修复集成以及回归相关的问题。目的是更平滑，更快的交付和更少的bug。通过更快地为技术和非技术成员提供项目状态的可视化，持续集成打开并改善了团队间的沟通渠道，同时鼓励他们协作解决问题和过程改进。通过部署过程自动化，持续集成可以帮助你更快，更可靠，更轻松地把软件交付到测试人员和最终用户受伤。

从手动到自动化构建

1. 无构建服务器
2. 夜间构建
3. 夜间构建加自动化测试
4. 加入度量指标
5. 更认真对待测试
6. 自动化验收测试和自动化部署
7. 持续部署

## 安装

### 准备工作

第一次使用 Jenkins，您需要：

- 机器要求：
  - 256 MB 内存，建议大于 512 MB
  - 10 GB 的硬盘空间（用于 Jenkins 和 Docker 镜像）
- 需要安装以下软件：
  - Java 8 ( JRE 或者 JDK 都可以)
  - [Docker](https://www.docker.com/) （导航到网站顶部的Get Docker链接以访问适合您平台的Docker下载）

### 下载并运行 Jenkins

1. [下载 Jenkins](http://mirrors.jenkins.io/war-stable/latest/jenkins.war).
2. 打开终端进入到下载目录.
3. 运行命令 `java -jar jenkins.war --httpPort=8080`.
4. 打开浏览器进入链接 `http://localhost:8080`.
5. 按照说明完成安装.

安装完成后，您可以开始使用 Jenkins！

启动一个空容器（target）

```shell
docker run --sysctl net.ipv6.conf.all.disable_ipv6=1 --privileged --name jenkins -d --hostname jenkins centos/systemd

yum -y install initscripts   openssh-clients  epel-release

wget https://download.oracle.com/otn/java/jdk/8u291-b10/d7fc238d0cbf4b0dac67be84580cfb4b/jre-8u291-linux-x64.rpm?AuthParam=1619878863_587f24918903c8d31bf61c0f1143b196  -O jre-8u291-linux-x64.rpm .

rpm -ivh jre-8u291-linux-x64.rpm
java -version

yum install git-core
git --version

sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
sudo yum install jenkins

docker run  --privileged --name jenkins-server  -d  --hostname jenkins-server  -p 8080:8080 riku2020/jenkins:v1

docker run  --privileged --name target  -d  --hostname target  -p 8080:8080 riku2020/target:v2
```

官方方法：

```shell
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
sudo yum install jenkins java-1.8.0-openjdk-devel
sudo systemctl daemon-reload
```

