# fluentd

## fluentd安装方法

### 安装前

在安装 Fluentd 之前，请确保您的环境已经正确设置，以避免以后出现任何不一致的情况。

下面是一些建议:

- 设立 NTP
- 增加文件描述符的最大数量
- 网络核心参数的优化

### 设立 NTP

强烈建议您在节点上设置 NTP 守护进程(例如 chrony、 ntpd 等) ，以获得准确的当前时间戳。这对于所有生产级日志记录服务都是至关重要的。

对于 Amazon Web Services 用户，我们建议使用 AWS-hosted NTP 服务器。

### 增加文件描述符的最大数量

增加文件描述符的最大数量。您可以使用 ulimit-n 命令检查现有的配置:

```
$ ulimit -n65535
```

如果你的控制台显示1024，这是不够的。请在/etc/security/limits.conf 文件中添加以下行，然后重新启动您的机器:

```
root soft nofile 65536root hard nofile 65536* soft nofile 65536* hard nofile 65536
```

如果在 systemd 下运行 fluentd，还可以使用 LimitNOFILE = 65536选项。而且，如果您使用的是 td-agent 包，这个值是默认设置的。

### 网络核心参数的优化

对于有很多 Fluentd 实例的高负载环境，将以下配置添加到/etc/sysctl.conf 文件中:

```
net.core.somaxconn = 1024net.core.netdev_max_backlog = 5000net.core.rmem_max = 16777216net.core.wmem_max = 16777216net.ipv4.tcp_wmem = 4096 12582912 16777216net.ipv4.tcp_rmem = 4096 12582912 16777216net.ipv4.tcp_max_syn_backlog = 8096net.ipv4.tcp_slow_start_after_idle = 0net.ipv4.tcp_tw_reuse = 1net.ipv4.ip_local_port_range = 10240 65535
```

使用 sysctl-p 命令或重新启动节点以使更改生效。

## 开始安装(Red Hat Linux)

Fluentd是用C+Ruby来开发的，考虑到很多开发者并不熟悉Ruby，官方体贴地提供了稳定的发布版本，这就是td-agent。我们安装后看到的可执行文件就叫这个名字，而不是叫作fluentd。

### 安装前

按先决条件正确配置操作系统

启动容器

```shell
docker run --sysctl net.ipv6.conf.all.disable_ipv6=1 --privileged --name fluentd -d --hostname fluentd centos/systemd
#改时区utc改为cst
timedatectl set-timezone Asia/Shanghai
```



### 从rpm Repository安装

强烈建议在节点上设置 ntpd，以防止日志中出现无效的时间戳。请参阅安装前指南。

注意: 如果你的操作系统不被支持，那么考虑安装 gem。

### 红帽/CentOS

```shell
# td-agent 4
$ curl -L https://toolbelt.treasuredata.com/sh/install-redhat-td-agent4.sh | sh

# td-agent 3
$ curl -L https://toolbelt.treasuredata.com/sh/install-redhat-td-agent3.sh | sh
```

执行此脚本将自动在计算机上安装 td-agent。这个 shell 脚本在/etc/yum.repos.d/td.repo 注册一个新的 rpm 存储库，并安装 td-agent。

我们在脚本中使用 $releasever 作为存储库路径，而 $releasever 应该是主要版本，如“7”。如果您的环境使用其他格式，比如“7.2”，那么仅将其更改为主版本，或者手动设置 TD 存储库。

### 第二步: 启动守护进程

Td-agent 提供两个脚本:

#### 系统

使用/usr/lib/systemd/system/td-agent 脚本启动、停止或重新启动代理:

```shell
$ sudo systemctl start td-agent.service
$ sudo systemctl status td-agent.service
● td-agent.service - td-agent: Fluentd based data collector for Treasure Data
   Loaded: loaded (/lib/systemd/system/td-agent.service; disabled; vendor preset: enabled)
   Active: active (running) since Thu 2017-12-07 15:12:27 PST; 6min ago
     Docs: https://docs.treasuredata.com/articles/td-agent
  Process: 53192 ExecStart = /opt/td-agent/embedded/bin/fluentd --log /var/log/td-agent/td-agent.log --daemon /var/run/td-agent/td-agent.pid (code = exited, statu
 Main PID: 53198 (fluentd)
   CGroup: /system.slice/td-agent.service
           ├─53198 /opt/td-agent/embedded/bin/ruby /opt/td-agent/embedded/bin/fluentd --log /var/log/td-agent/td-agent.log --daemon /var/run/td-agent/td-agent
           └─53203 /opt/td-agent/embedded/bin/ruby -Eascii-8bit:ascii-8bit /opt/td-agent/embedded/bin/fluentd --log /var/log/td-agent/td-agent.log --daemon /v

Dec 07 15:12:27 ubuntu systemd[1]: Starting td-agent: Fluentd based data collector for Treasure Data...
Dec 07 15:12:27 ubuntu systemd[1]: Started td-agent: Fluentd based data collector for Treasure Data.
```

要自定义 systemd 行为，请将 td-agent. 服务放入/etc/systemd/system 中。

注意: 在 td-agent 4中，路径是不同的，即/opt/td-agent/bin 而不是/opt/td-agent/embedded/bin。

### init.d

这是针对 CentOS 6，非基于系统的系统。

使用/etc/init.d/td-agent 脚本启动、停止或重新启动代理:

```shell
$ sudo /etc/init.d/td-agent start
Starting td-agent: [  OK  ]
$ sudo /etc/init.d/td-agent status
td-agent (pid  21678) is running...
```

支持以下命令:

```shell
$ sudo /etc/init.d/td-agent start
$ sudo /etc/init.d/td-agent stop
$ sudo /etc/init.d/td-agent restart
$ sudo /etc/init.d/td-agent status
```

请确保您的配置文件路径是:

```shell
/etc/td-agent/td-agent.conf
```

### 第三步: 通过 HTTP 发布示例日志

默认配置(/etc/td-agent/td-agent。是在 HTTP 端点接收日志并将它们路由到 stdout。有关 td-agent 日志，请参见/var/log/td-agent/td-agent。原木。

你可以使用 curl 命令发布日志记录样例:

```shell
$ curl -X POST -d 'json={"json":"message"}' http://localhost:8888/debug.test
$ tail -n 1 /var/log/td-agent/td-agent.log
2018-01-01 17:51:47 -0700 debug.test: {"json":"message"}
```

## fluentd事件的生命周期

**什么是事件？**

事件（Event）是Fluentd内部处理流程使用的数据结构，日志记录一旦进入Fluentd便被封装成一个event。Event由三部分组成：**tag、time、record**。

- tag标识事件的来源，或者说类型，用于内部消息路由，即后续交由哪个插件处理；

- time是事件的发生时间；

- record为日志的实际内容，这是一个JSON对象。

Input插件负责将源数据封装为event，比如in_tail插件从文本中生成event。对于下边这行文本：

```shell
   192.168.0.1 - - [28/Feb/2013:12:00:00 +0900] "GET / HTTP/1.1" 200 777
```

将会产生下边的event对象：

```shell
tag: apache.access    #根据插件的tag参数来设置

time: 1362020400      # 28/Feb/2013:12:00:00 +0900

record: {"user":"-","method":"GET","code":200,"size":777,"host":"192.168.0.1","path":"/"}
#根据in_tail插件中的parse项来决定如何解析单行日志记录，并生成相应的JSON对象
```

### 配置文件

下边我们通过一个具体的配置来讲解事件的处理过程。

rpm,deb,dmg

```shell
/etc/td-agent/td-agent.conf
```

ruby gem：创建配置文件

```shell
$ sudo fluentd --setup /etc/fluent
$ sudo vi /etc/fluent/fluent.conf
```

docker

对于 Docker 容器，配置文件的默认位置是/fluentd/etc/fluent.conf。若要从 Docker 外部挂载配置文件，请使用 bind-mount。

```shell
docker run -ti --rm -v /path/to/dir:/fluentd/etc fluentd -c /fluentd/etc/<conf-file>
```

`FLUENT_CONF` Environment Variable

您可以通过 FLUENT _ conf 更改默认的配置文件位置。例如,/etc/td-agent/td-agent. conf 是通过 td-agent 脚本中的 FLUENT _ conf 指定的。

### 配置文件由以下指令组成:

**`source`** directives determine the input sources

- **source指令确定输入源**

**`match`** directives determine the output destinations

- **match指令决定输出目的地**

**`filter`** directives determine the event processing pipelines

- **filter指令确定事件处理管道**

**`system`** directives set system-wide configuration

- **system指令设置全系统配置**

**`label`** directives group the output and filter for internal routing

- **Label 指令对输出和内部路由过滤器进行分组**

**`@include`** directives include other files

- **@include指令包括其他文件**

### 处理过程

 本例使用一个很基础的配置片段来描述各插件是如何关联到一起的，它包括了如何定义输入源（或者说监听器），以及如何设置通用的匹配规则将event路由到输出端。

我们使用in_http和out_stdout这两个插件来描述event的循环过程。

```shell
vi /etc/td-agent/td-agent.conf

<source>
  @type http
  port 8888
  bind 0.0.0.0
</source>
```

上边的配置使用in_http插件定义了一个HTTP服务器，监听端口为8888。

然后我们再定义一个匹配（Match）规则，event路由引擎会根据这个规则将http请求派发到输出端。这里的输出端是stdout，仅仅将http请求打印到屏幕上。

```shell
<match test.cycle>
  @type stdout 
</match>
#这里改完配置文件后需要重启服务
systemctl restart td-agent
```

 Match的作用是设置一个匹配规则test.cycle，对于每个进入Fluentd的event，如果其tag值和test.cycle相等（或者说匹配，因为match可以使用通配符。这里的tag是由in_http插件生成的。），那么这个event就会进入此match定义的output插件，本例中的output插件就是out_stdout。

至此，我们定义了三个基本项：Input、Match和Output，虽然仅仅使用两个配置段。这就是一个可以使用的采集配置了，可以通过以下命令进行测试：

```shell
curl -i -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.cycle
```

你会看到如下输出：

```shell
HTTP/1.1 200 OK
Content-Type: text/plain
Connection: Keep-Alive
Content-Length: 0
```

在/var/log/td-agent.log中会有如下输出：

```shell
2020-03-05 14:06:24.144168913 +0800 test.cycle: {"action":"login","user":2}
```

![image-20210501211401886](https://i.loli.net/2021/05/01/CyQltqfKjNXzxHk.png)

### 下边我们开始了解一下事件是如何被处理和改变的。

当你准备好一个采集配置后，Fluentd就生成了用以处理输入数据的各种规则。日志事件会历经一系列的处理流程，从而决定了事件的循环周期。

### **过滤器（Filters）**

- 过滤器用于对事件进行筛选，决定是否接收或者丢弃事件。我们可以在上边的示例中增加一个过滤器。

```shell
<source>
  @type http
  port 8888
  bind 0.0.0.0
</source>

<filter test.cycle>
  @type grep
  <exclude>
    key action
    pattern ^logout$
  </exclude>
</filter>

<match test.cycle>
  @type stdout
</match>
```

添加过滤器之后，事件在路由到match之前必须经过过滤器的处理。过滤器根据事件的类型和过滤规则来决定是否接受此事件。

我们示例中使用的是grep过滤器，这个过滤器对test.cycle这类事件进行过滤，会排除http请求中action值为logout的事件。

所以，如果尝试发送下边的请求，在td-agent.log中是看不到任何输出的。

```shell
curl -i -X POST -d 'json={"action":"logout","user":2}' http://localhost:8888/test.cycle
```

从示例中可以看到，事件是**根据配置顺序自上而下来被处理的**。

我们可以根据需要配置任意多个过滤器，这样一来，配置文件会变得很长很复杂。Fluentd提供了标签来解决此问题。

### 标签（Labels）

标签的作用是用来定义一组配置项，这组配置项可以被其他配置项引用，从而实现事件路由跳转。类似编程语言中的goto的功能。

还是上边的示例，我们定义一个标签来看一下效果。

```shell
<source>
  @type http
  bind 0.0.0.0
  port 8888
  @label @STAGING
</source>

<filter test.cycle>
  @type grep
  <exclude>
    key action
    pattern ^login$
  </exclude>
</filter>

<label @STAGING>
  <filter test.cycle>
    @type grep
    <exclude>
      key action
      pattern ^logout$
    </exclude>
  </filter>

  <match test.cycle>
    @type stdout
  </match>
</label>
```

这个STARTING标签将之前的filter和match封装到了一起，然后在source中进行了引用。如此一来，事件由input插件生成后将会跳过那个独立的filter，直接进入STARTING定义的处理流程中。

这种效果可以让我们实现一些特定的处理逻辑，让事件快速到达指定目的地。

### 缓存（Buffers）

我们看到了事件从input产生，经由filter筛选，最后到达output的过程。在上边的示例中，我们使用的是stdout插件直接输出到控制台，并没有经过缓存。

实际应用中，一般会先把数据进行缓存，达到一定条件后再flush到目标存储中。这样可以提升系统可靠性，对于稳定系统吞吐量也很重要。可在后续文章中共同了解更多关于缓存插件的知识。

总的来说，事件会在各插件之间接续流转，直到到达output，结束整个生命周期。如下图：

![image-20210501212421265](https://i.loli.net/2021/05/01/dnbLtpSOarEBVqh.png)

