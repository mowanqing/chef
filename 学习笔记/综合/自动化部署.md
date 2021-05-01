#  自动化部署

## 持续交付

概念: 交付 + 版本控制+ 持续集成工具+部署工具 = 持续交付

## GitLab

Gitlab是一个开源分布式版本控制系统

开发语言:Ruby

功能:管理项目源代码,版本控制,代码复用与查找

Github分布式在线代码托管仓库,个人版本暴露在公网上,不安全,企业版要收费

Gitlab分布式在线代码仓库托管软件,社区免费版+服务器可以搭建私人代码仓库

优势:

- 开源免费,时和中小型公司将代码放置在该系统中,要进行二次开发,只需升级到企业版,现行的代码仓库可以无缝衔接
- 差异化的版本管理,离线同步以及强大的分支管理功能
- 便捷的GUI操作界面一级强大的账户权限管理功能
- 集成度很高,能够集成绝大多数的开发工具
- 支持内置HA,保证在高并发下仍旧实现高可用性

### 主要服务构成

- Nginx静态Web服务器(处理https请求)
- Gitlab-workhorse轻量级的反向代理服务器(处理一些较大的文件上传下载,以及经常使用的git push 命令行操作)
- Gitlab-shell用于处理Git命令和修改authorized keys列表
- Logrotate 日志文件管理工具(日志的切割,打包操作)
- Posstgresql 数据库(保存所有数据信息)
- Redis缓存服务器(缓存数据库信息,加快前台的访问速度,以及数据的交互读写)

### Gitlab的工作流程

- 创建并克隆项目
- 创建项目某Feature分支
- 编写代码并提交至该分支
- 推送该项目分支至远程Gitlab服务器
- 进行代码检查并提交Master分支合并申请
- 项目领导审查代码并确认合并申请

### Gitlab安装配置管理

- 利用VitualBox创建测试服务器(centos7)

- 安装Gitlab前系统预配置准备

  1. 关闭firewalled防火墙

     `# systemctl stop firewalld`

     `# systemctl disable firewalld`(禁用开机启动)

  2. 关闭SELINUX并重启系统(强制访问安全策略)

     `# vi /etc/sysconfig/selinux`

     ...

     SELINUX=disabled

     ...

     `# reboot`

  3. 安装Omnibus Gitlab-ce package

     1. 安装Gitlab组件

        `yum install curl policycoreutils openssh-server openssh-clients postfixs`

     2. 配置YUM仓库

        `# curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | bash`

     3. 启动postfix邮件服务

        `# systemctl start postfix && systemctl enable postfix`

     4. 安装Gitlab-ce社区版本

        `# yum install -y gitlab-ce`

     5. Omnibus Gitlab等相关配置初始化并完成安装

        1. 证书创建与配置加载

           1. `# openssl genrsa -out "/etc/gitlab/ssl/gitlab.example.com.key" 2048`(密钥)

           2. `# openssl req -new -key "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.csr"`(根据密钥创建的证书)

              ```
              Country Name (2 letter code) [XX]:cn
              State or Province Name (full name) []:js
              Locality Name (eg, city) [Default City]:sz
              Organization Name (eg, company) [Default Company Ltd]:
              Organizational Unit Name (eg, section) []:
              Common Name (eg, your name or your server's hostname) []:gitlab.example.com
              Email Address []:riku7032@163.com
              ```

           3. `openssl x509 -req -days 365 -in "/etc/gitlab/ssl/gitlab.example.com.csr" -signkey "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.crt"`(签署crt证书)

           4. `openssl dhparam -out /etc/gitlab/ssl/dhparams.pem 2048`(创建pem证书)

           5. 修改gitlab配置

              ```
              vim /etc/gitlab/gitlab.rb
              external_url 'https://gitlab.example.com'
              nginx['redirect_http_to_https'] = true
              # nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.example.com.crt"
              # nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.example.com.key"
              # nginx['ssl_dhparam'] = /etc/gitlab/ssl/dhparams.pem # Path to dhparams.pem, eg. /etc/gitlab/ssl/dhparams.pem
              ```

        2. ```
           #执行gitlab-ctl reconfigure前先运行以下命令,docker需要
           /opt/gitlab/embedded/bin/runsvdir-start &
           #初始化完成后
           vim /var/opt/gitlab/nginx/conf/gitlab-http.conf
           rewrite ^(.*)$ https://$host$1 permanent;
           ```

        3. Nginx SSL代理服务配置

        4. 初始化Gitlab相关服务并完成安装

           ```
           在宿主机更改 vim /etc/hosts文件
           将docker容器的ip地址加入到文件中并换成gitlab.example.com
           创建仓库
           mkdir repo 
           cd repo 
           git -c http.sslverify=false clone https://gitlab.example.com/root/test-repo.git
           vim test.py
           git add .
           git commit -m "first commit"
           git -c http.sslverify=false push origin master
           ```

### docker centos容器配置

```
docker run -d -p 5022:22 --name centos --privileged=true centos:7 /usr/sbin/init
docker exec -it centos /bin/bash
```

### 应用

- Gitlab后台管理
- 开发是角的Gitlab
- 运维视角的Gitlab(监控cpu,内存,硬盘等系统健康,保证在高并发下的高可用,权限分配问题)
- Gitlab不同角色使用实例

## Ansible

### 定义

ansible是一个开源部署工具

开发语言为Python

特点:SSH协议通讯,全平台,无需编译,模块化部署管理

作用:推送Playbook进行远程节点快速部署

### Ansibe与Chef,Saltstack的不同

- Chef
  - Ruby语言编写,C/S架构,配置需要Git依赖,Recipe脚本编写规范,需要编程经验
- Saltstack
  - Python语言编写,C/S架构,模块化配置管理,YAML脚本编写规范,适合大规模集群部署
- Ansible
  - Python语言编写,无Client,模块化配置管理,Playbook脚本编写规范,易于上手,适合中小规模快速部署.

### Ansible的优势和应用场景

1. 轻量级无客户端(Agentless)(不需要Client,减轻部署成本,使用SSH作为替代)
2. 开源免费,学习成本低,快速上手
3. 使用Playbook作为核心配置架构,统一的脚本格式批量化部署
4. 完善的模块化扩展,支持目前主流的开发场景
5. 强大的稳定性和兼容性
6. 活跃的官方社区问题讨论,方便Trubleshooting与DEBUG问题

### Ansible配合Virtualenv安装配置

使用python自带的Virtualenv去隔离python3.6和ansibe2.5,保证这个语言环境只提供给ansible

- Ansible的两种安装模式(Centos7)

  1. Yum包管理安装

     `# yum -y install ansible`(该ansible成为系统下全局的工具)

  2. Git源代码安装(推荐)

     `# git clone https://github.com/ansible/ansible.git`

     配合virtualenv实现ansible在独立的python环境下运行

- Ansible2.5+python3.6安装步骤(Centos7)

  1. 预先安装Python3.6版本

     ```
      wget http://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz
      tar xf Python-3.6.5.tar.xz
      ./configure --prefix=/usr/local --with-ensurepip=install --enable-shared LDFLAGS="-wl,-rpath /usr/local/lib"
      # --prefix=/usr/local 安装在该目录下
      # --with-ensurepip=install 安装包管理工具
      # --enable-shared LDFLAGS="-wl,-rpath /usr/local/lib" 用来匹配当前系统参数的值
     make && make altinstall
     ln -s /usr/local/bin/pip3.6 /usr/local/bin/pip
     ```

  2. 安装Virtualenv

     `# pip install virtualenv`

  3. 创建Ansible账户并安装python3.6版本Virtualenv实例

     `# useradd deploy && su - deploy`

     `# virtualenv -p /usr/local/bin/python3 .py3-a2.5-env`

  4. Git源代码安装ansible2.5

     `yum -y install git nss curl`

     `# cd /home/deploy/.py3-a2.5-env`

     `# git clone https://github.com/ansible/ansible.git`

     `# cd ansible && git checkout stable-2.5`

  5. 加载python3.6 virtualenv环境(每次加载时一定要做)

     `# source /home/deploy/.py3-a2.5-env/bin/activate`

  6. 安装ansible依赖包

     `# pip install paramiko PyYAML jinja2`

  7. 在python3.6虚拟环境下加载ansible2.5

     `# mv ansible/ .py3-a2.5-env/`

     `# git checkout stable-2.5`

     `# source /home/deploy/.py3-a2.5-env/ansible/hacking/env-setup -q`

  8. 验证ansible2.5

     `# ansible --version`

### Ansible playbooks入门和编写规范

playbooks基础的任务文件格式为YAML,playbooks像一个总的乐谱,每一个YAML文件称做为一个playbook乐章,在这个playbook下去编写一个或多个task做为这个乐章的音符,通过ansible的相关命令去play演奏这个乐谱,就可以预先将我们写好的任务,按照特定的编排,部署到远程服务器当中.

### Playbooks框架与格式

#### TEST Playbooks

文件结构:

inventory/ (存放一个或多个Server详细清单目录,用来保存目标部署主机的相关域名或者IP地址,以及该主机的变量参数,根据dev,uat单元测试环境,production环境,用来给server清单命名,对应清单下的主机地址具体部署到哪个环境当中)

 testenv (具体清单与变量声明文件,将testenv下的主机地址部署到test环境当中)

roles/ (保存详细的roles任务列表,它下面可以存放一个或多个role,通常或命名为具体的APP,或者项目名称)

 testbox/ (testbox详细任务,做为我们的项目名称)

 tasks/(用来保存我们最终的taskbox任务乐章文件)

 main.yml(testbox主任务文件)

deploy.yml (Playbooks任务入口文件,它将调度我们roles下需要去部署的项目,以及该项目下的所有任务,最终将该任务部署在我们的inventory/ 下定义的目标主机中)

- 详细目录testenv

  ```
  [testservers] #server组列表,可以保存一个或者多个目标主机的域名或者ip地址
  test.example.com # 目标部署服务器的主机名(域名)
  
  [testservers:vars] # server组列表参数标签,用来定义该组下的远程主机用到的所有的key/value参数键值对作为我们server组的变量声明
  server_name=test.example.com
  user=root
  output=/root/test.txt
  ```

- 主任务文件main.yml(用来保存特定role下面需要执行的具体任务乐章,这里乐章里面会保存一个或多个task作为我们的音符)

  ```
  - name:Print server name and user to remote testbox # 任务名称定义task名称
    shell:"echo 'Currently {{user}} is logining {{server_name}}' > {{output}}" #具体要执行的任务,通常调用ansible内建模块去编排我们的任务逻辑
  ```

- 任务入口文件deploy.yml(直接和playbook命令直接对话,它将playbooks下的所有编排内容展示给我们的ansible命令进行最终的play演奏,最后部署到对应的目标主机当中)

  ```
  - hosts:"testservers" # server列表 对应server组列表参数标签,用来调用这个标签下的定义的目标主机(test.example.com),告诉playbook命令,需要部署的主机是谁
    gather_facts:true # 获取server基本信息,用来获取目标主机下的一些基本信息
    remote_user:root # 目标服务器系统用户指定 告诉playbook命令,使用目标主机下的root用户,进行所有的文件系统操作
    roles: # 进入roles/testbox任务目录 告诉playbook命令,进入这个目录去执行任务
    	- testbox
  ```

#### SSH免密码密钥认证

- Ansible服务器端创建SSH本地密钥

  `# ssh-keygen -t rsa`

- Ansible服务器端建立与目标部署机器的密钥认证

  `# ssh-copy-id -i /home/deploy/.ssh/id_rsa.pub root@test.example.com`

#### 执行Playbooks

- 部署到testenv环境

  `# ansible-playbook -i inventory/testenv ./deploy.yml`

#### 实战编写playbooks

```
vim /etc/hosts # 将目标主机的主机名以及ip地址写入

mkdir test_playbooks
├── deploy.yml
├── inventory
│   └── testenv
└── roles
    └── testbox
        └── tasks
            └── main.yml
#deploy.yml:
- hosts: "testservers"
  gather_facts: true
  remote_user: root
  roles:
    - testbox
#testenv:
[testservers]
test.example.com:5023

[testservers:vars]
server_name=test.example.com
user=root
output=/root/test.txt

# main.yml:
- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }}' > {{ output }}"
```

### Ansible Playbooks常用模块

ansible的模块是由ansible对特定部署操作脚本的打包封装后的成品,我们可以利用该成品直接去编写playbooks,大大简化里对部署脚本编写的逻辑,方便后期维护管理

#### file模块

- 在目标主机创建文件或目录,并赋予其系统权限

```
通过file模块执行的一个task任务
- name: create a file
  file: 'path=/root/foo.txt state=touch mode=0755 owner=foo group=foo'
```

#### copy模块

- 实现Ansible服务端到目标主机的文件传送

```
- name: copy a file
  copy: 'remote_src=no src=roles/testbox/files/foo.sh dest=/root/foo.sh mode=0644 force=yes'
# remote_src=no声明我们是需要去将源ansible主机端的文件传送到我们的目标主机当中
```

#### Stat模块

- 获取远程文件状态信息(并将其信息保存在一个环境变量下供随后使用)

```
- name: check if foo.sh exists
  stat: 'path=/root/foo.sh'
  register: script_stat 
 # register将stat获取到的文件信息传送给script_stat这个变量
```

#### Debug模块

- 打印语句到Ansible执行输出

```
- debug: msg=foo.sh exists
  when: script_stat.stat.exists
# when调用之前使用script_stat信息来进行判断
```

#### Command/Shell模块

- 用来执行Linux目标主机命令行(shell可以使用/bin/bash,所以可以使用管道符,重定向符,command不行)

```
- name: run the script
  command: 'sh /root/foo.sh'

- name: run the script
  shell: "echo 'test' > /root/test.txt"
```

#### Template模块

- 实现Ansible服务端到目标主机的jinja2模板传送(利用jinja的语法格式编写一个app配置文件,例如nginx配置文件,在里面添加一些变量参数,然后通过定义ansible的变量参数,将我们的模板传送到目标主机的app配置文件目录,从而实现我们对不同环境的app的配置参数管理)

```
- name: write the nginx config file
  template: src=roles/testbox/templates/nginx.conf.j2 dest=/etc/nginx/nginx.conf
```

#### Packaging模块

- 调用目标主机系统包管理工具(yum,apt)进行安装(是一个广义的模块集yum,apt是子模块)

```
- name: ensure nginx is at the latest version
  yum: pkg=nginx state=latest
- name: ensure nginx is at the latest version
  apt: pkg=nginx state=latest
```

#### Service模块

- 管理目标主机init系统服务(调用service systemctl去管理系统服务资源的启动,停止状态检查)

```
- name: start nginx service
  service: name=nginx state=started
```

### 实战

```
ansible主机 root@192.168.122.128 -p 5022
ssh root@test.example.com -p 5023
test主机 root@192.168.122.128 -p 5023
# 远程文件创建
- name: create a file
  file: 'path=/root/foo.txt state=touch mode=0755 owner=foo group=foo'
# 本地到远程文件传送
- name: copy a file
  copy: 'remote_src=no src=roles/testbox/files/foo.sh dest=/root/foo.sh mode=0644 force=yes'
# 获取文件状态
- name: check if foo.sh exists
  stat: 'path=/root/foo.sh'
  register: script_stat 
# 查看文件是否存在
- debug: msg="foo.sh exists"
  when: script_stat.stat.exists
# 远程sh执行文件
- name: run the script
  shell: "echo 'test' > /root/test.txt"
# 本地到远程模板传送
- name: write the nginx config file
  template: src=roles/testbox/templates/nginx.conf.j2 dest=/etc/nginx/nginx.conf
# 安装nginx
- name: ensure nginx is at the latest version
  yum: pkg=nginx state=latest
# 启动nginx
- name: start nginx service
  service: name=nginx state=started
```

安装nginx

```
source .py3-a2.5-env/bin/activate
source .py3-a2.5-env/ansible/hacking/env-setup -q
ansible-playbook --version
ssh root@test.example.com -p 5023
useradd foo
useradd deploy
mkdir /etc/nginx
# 安装一个nginx的yum源,保证我们随后可以利用相应的ansible模块在我们的目标主机上安装我们的nginx安装包
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
# 编写主任务清单
vim roles/testbox/tasks/main.yml
# 执行playbook
ansible-playbook -i inventory/testenv ./deploy.yml
ssh root@test.example.com -p 5023 ls -l /root/foo.txt
```

## JenKins

- JenKins是一个开源持续集成工具(使用java开发提供了软件开发的集成服务,支持很多主流软件的配置管理工具与其配合实现软件配置管理,持续集成功能)
- Java
- 功能:提供了软件开发的持续集成服务
- 特点:支持主流软件配置管理,配合实现软件配置管理,持续集成功能

> 传统运维的工作像一个黑盒,每天写的脚本,执行的命令数以万计,如何去管理,如何去审计,如何在数以万计的服务器中和同事配合在一起协同完成某一项工作,则是我们需要去考虑的问题,如果仅限于command,如果遇到突发情况,将疲于奔命,虽然可以知道在哪里写的脚本有什么用,在哪里执行的命令,实现了什么功能,但没有办法有一套完整的系统去管理,和让别人去了解你日常工作的内容,对于别人来说,我的工作就是一个黑盒,jenkins彻底打开了这个黑盒,让开发人员和运维人员协同工作在一个白盒中,它在我们的运维当中,起到承上启下的作用,它的前台界面,让我们直观的收集到我们执行的脚本的相关信息,而且能作为一个pipeline,将我们的开发周期中,各个环节利用stage组合到一起,无论选择的是sona,代码质量检测工具,还是mavan或者ant,编译打包工具,还是gitlab,github,版本控制系统,ansibe,saltstack,产品部署工具,jenkins都可以将这些工具串连成一个完整的框架,最终实现一个高效的,可扩展的全自动部署流程平台

### Jenkins的优势和应用场景

1. 主流的运维开发平台,兼容所有主流开发环境(它可以将我们平时开发,测试,部署,基础运维浓缩到我们每一个脚本,也就是任务当中,配合其强大的兼容性,匹配我们大中小的开发环境,无论使用centos,redhat,debian,ubuntu,windows,mac,docker都可以搭建jenkins平台,保证我们不同的开发环境都能在jenkins上运转)
2. 插件市场可与海量业内主流开发工具实现集成(方便将不同类型的数据在开发工具间调用处理,比如调用gitlab或者github的插件去与其进行git交互,实现clone,pull,push等操作,可以利用sona插件,给jenkins传入代码仓库地址,在sona系统下实现静态代码扫描,最终生成的报告可以检测出代码的语法是否不规范,是有bug,缺陷等,来提高代码质量,利用maven插件去通过传入对应的参数,对代码进行编译,测试,打包,并最终上传到我们的代码仓库)
3. job为配置单位与日志管理,使运维和开发人员能协同工作(打开开发和运维之间的墙壁,所有人都可以通过对某一个job任务的操作配置,得到自己想要获取的信息,作为开发或者测试人员,可以无需关注去如何搭建这个平台,以及job部署配置如何实现,只需将项目所需的参数传给jenkins下对应你的项目的具体job,jenkins就会帮助你完成所有的部署,这样就能将开发与测试人员将更多的时间关注到项目代码的实现与项目测试当中,无需花费额外的时间到部署工作当中,做为运维人员就可以去关注具体的基础平台搭建维护工作,通过监控jenkins系统相关指标,保证jenkins在一个健康状态下运转,以及关注项目部署配置工作中出现的权限,参数配置,工具集成调用等问题,从而无需关注代码成面的问题,最终让我们各司其职,共同在这个平台下,完成我们项目开发周期的所有工作)
4. 权限管理划分不同job不同角色(jenkins严谨的权限管理功能充分的划分了每个人在开发周期内,对每一个job的不同角色,我们可以通过设定不同用户在登录系统后具有不同的权限,例如开发与测试人员只能右job任务,build,以及查看日志的权限,从而对代码进行日常测试部署等操作,运维人员在开发人员的基础上,具有对job任务的写入权限,从而进行日常的任务编写,从而保证大家不会越权去操作别人的任务,提高项目的安全性)
5. jenkins强大的负载均衡功能,保证我们项目的可靠性(可以让job游走在我们的jenkins集群当中,保证具体我们开发过程中的可靠性,jenkins不仅仅是一个独立的系统,它可以在自己创建后作为一个master节点,然后衍生出若干个*slave*结点,从而组合在一起成为一个jenkins集群,这个集群的优势就在于我们可以将我们的job任务随机或者手动指定到任意master或者slave节点上去执行,最终实现我们强大的负载均衡功能,保证我们项目的可靠性,jenkins的多语言兼容性和强大的插件安装平台,保证它几乎可以适合我们所有的主流软件测试环境与生产环境,无论作为一个开发人员想部署一个java,python,php,ruby等环境,还是作为一个测试人员去集成sonar,caslink,去测试代码或者生成测试报表,jenkins都会给我们不同相关开发人员或者测试人员提供一个较为开放的平台去完成我们的工作,不需要我们在像以前传统通过command line命令行去跑我们的程序,我们可以直接在jenkins上完成所有集成部署工作,实现全平台,全语言,全环境,无缝连接的应用场景)

### JenKins安装配置管理