# fluentd

## 第一章：fluentd安装方法

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

## 第二章：fluentd事件的生命周期

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

## 第三章：fluentd配置文件语法

我们在之前的文章中介绍过Fluentd事件的生命周期，事件是按照配置文件中的配置在不同插件之间进行传递的，配置文件可以控制事件的执行顺序，对于事件的处理极为重要。

本文描述Fluentd配置文件涉及的一些基本概念。

1. 配置文件的存放位置

   - 如果Fluentd是通过rpm、deb或dmg等安装包来安装的，那配置文件默认为**/etc/td-agent/td-agent.conf**

   - 如果Fluentd是通过ruby gem安装的，可以通过以下命令指定配置文件

      ```
       $ sudo fluentd --setup /etc/fluent
       $ sudo vi /etc/fluent/fluent.conf
      ```
   
   - 如果使用的是docker镜像，配置文件默认为**/fluentd/etc/fluentd.conf**
   
   - 可以通过**FLUENT_CONF变量**来修改使用的配置文件，该变量可在/usr/sbin/td-agent中进行设置
   
   - 也可以通过-c来指定配置文件，之前文章有介绍

2. 配置文件使用的字符集为UTF-8 或 ASCII.
3. 配置指令
   1. Fluentd配置文件主要包含以下配置指令：
      - source：设置输入来源
      - match：设置输出目的
      - filter：事件过滤器，用来控制处理流程
      - system：全局配置
      - label：封装一组过滤器和输出，用以调整事件的内部路由
      - @include：引用其他配置文件

接下来，我们使用这些配置指令，一步一步创建一个示例配置文件。

### source：这是日志的源头:

Fluentd输入源通过选择和配置所需的input插件来指定。

Fluentd的标准input插件包含http和forward。

1. **http**使得Fluentd开启一个http服务，以接收http消息；

2. **forward**使得Fluentd开启一个tcp服务，以接收tcp数据包。

你可以同时使用这两个插件，不限个数。

```shell
# Receive events from 24224/tcp
# This is used by log forwarding and the fluent-cat command
<source>
  @type forward
  port 24224
</source>

# http://this.host:9880/myapp.access?json={"event":"data"}
<source>
  @type http
  port 9880
</source>
```

**source指令包含一个@type参数，用以指定要使用的插件。**

source将事件提交给Fluentd的路由引擎处理。

### 事件组成

事件包含**tag**、**time**和**record**三个属性。

1. tag是以点号（.）分隔的字符串（比如，myapp.access），供Fluentd内部路由使用，建议使用小写的字母、数字、下划线来命名；

2. time由input插件生成，必须是Unix时间格式；

3. record是一个JSON对象。

对于上边这个例子，http插件提交的事件格式如下：

```shell
# generated by http://this.host:9880/myapp.access?json={"event":"data"}
tag: myapp.access
time: (current time)
record: {"event":"data"}
```

### match：告诉Fluentd怎么处理收到的日志事件

match指令查找和设置的tag相匹配的事件，并处理这些事件。

通常的用法就是将事件输出到其他系统中，所以我们可以看到所有的output插件都是在此指令下添加的。

Fluentd的标准output插件包含**file**和**forward**。我们将其添加到示例中看一下。

```shell
# Receive events from 24224/tcp
# This is used by log forwarding and the fluent-cat command
<source>
  @type forward
  port 24224
</source>

# http://this.host:9880/myapp.access?json={"event":"data"}
<source>
  @type http
  port 9880
</source>

# Match events tagged with "myapp.access" and
# store them to /var/log/fluent/access.%Y-%m-%d
# Of course, you can control how you partition your data
# with the time_slice_format option.
<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>
```

每个match指令**必须包含模式匹配字符串**(myapp.access)，以及@type参数。只有和模式匹配字符串相符的tag标记的事件才会被发送到目的地；

和source一样，@type指定了要使用的output插件。

### **filter：事件处理流水线**

filter和match指令语法相同，不同的是，filter可以串联起来形成处理流：

 ```shell
 Input -> filter 1 -> ... -> filter N -> Output
 ```

我们继续修改上边的例子，加入一个标准的record_transformer过滤器。

```shell
# http://this.host:9880/myapp.access?json={"event":"data"}
<source>
  @type http
  port 9880
</source>

<filter myapp.access>
  @type record_transformer
  <record>
    host_param "#{Socket.gethostname}"
  </record>
</filter>

<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>
```

这个片段中，收到的http请求事件*{'event':'data'}*会首先进入record_transformer过滤器，这个过滤器向事件中添加了一个host_param字段，原事件被修改为*{'event':'data', 'host_param':'your_host_name'}*。然后接着输出到file中。

### system全局配置：

Fluentd的全局配置可以在system指令中设置（除此之外，也可以通过命令行设置）。常见的一些全局配置如下：

log_level

suppress_repeated_stacktrace

emit_error_log_interval

suppress_config_dump

without_source

process_name

具体含义可以参见官方文档。向我们的例子中添加一些全局参数：

```shell
<system>
  # equal to -qq option
  log_level error
  # equal to --without-source option
  without_source
  # ...
</system>
```

### label：封装filter和output

把一些filter和output封装成label之后，这个label可以被其他指令直接引用，这样可以降低配置文件的复杂度。

下边是一个示例，注意label是内置的参数，其前边需要加@符号。

 ```shell
 
 <source>
   @type forward
 </source>
 
 <source>
   @type tail
   @label @SYSTEM
 </source>
 
 <filter access.**>
   @type record_transformer
   <record>
     # ...
   </record>
 </filter>
 
 <match **>
   @type elasticsearch
   # ...
 </match>
 
 <label @SYSTEM>
   <filter var.log.middleware.**>
     @type grep
     # ...
   </filter>
   
   <match **>
     @type s3
     # ...
   </match>
 </label>
 ```

在这个配置片段中，有两个input插件：forward和tail。

1. **forward**产生的事件被**顺序路由**到**record_transformer**和**elasticsearch**中

2. **tail**产生的事件则跳转到**label @SYSTEM**指定的**grep**和**s3**中。

### @include：复用其他配置文件

我们可以把一些配置放在单独的文件中，需要用到时通过include指令将其包含到主配置中。

```shell
# Include config files in the ./config.d directory
@include config.d/*.conf
```

include不仅可以指定本地路径（支持通配符），还可以引用通过url指定的配置文件。

```shell
# http
@include http://example.com/fluent.conf
```

下边举一个配置复用的例子。

```shell
# config file
<match pattern>
  @type forward
  # other parameters...
  <buffer>
    @type file
    path /path/to/buffer/forward
    @include /path/to/out_buf_params.conf
  </buffer>
</match>

<match pattern>
  @type elasticsearch
  # other parameters...
  <buffer>
    @type file
    path /path/to/buffer/es
    @include /path/to/out_buf_params.conf
  </buffer>
</match>


# /path/to/out_buf_params.conf
flush_interval 5s
total_limit_size 100m
chunk_limit_size 1m
```

这个例子包含两个输出插件：forward和elasticsearch，都使用了文件作为输出缓存。它们使用的缓存配置是相同的，所以可以把这个配置单独提取为out_buf_params.conf，然后通过include引用到buffer指令中。

### Match匹配规则

如前所述，Fluentd通过tag实现事件路由。除了精确指定tag（如<filter app.log>），还有以下几种方法可以提升匹配效率。

1. 通配符与组合（expansions）
   - 在<match>和<filter>使用的tags中可以指定如下匹配模式：
     -  \*：匹配单个tag段，如a.*匹配a.b，但是不匹配a、也不匹配a.b.c
     - \**：匹配0或多个tag段，如a.**匹配a、a.b和a.b.c
     - {X,Y,Z}：匹配X、Y、Z表示的任一模式，如{a,b}匹配a和b，但不匹配c。这种模式可以和*、**组合使用，如a.{b,c}.*、a.{b,c.**}
     - \#{...}：可以在{}中使用ruby表达式进行匹配
     - 多个模式可以空格分隔的形式组合为一个tag，用以匹配满足任一模式的事件。如<match a b>匹配a和b

### 匹配顺序

Fluentd默认按照tag在配置文件中出现的顺序依次进行匹配。比如

```shell
# ** matches all tags. Bad :(
<match **>
  @type blackhole_plugin
</match>

<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>
```

这个配置片段中，myapp.access永远不会被匹配到。因此，精确匹配模式需要放在宽泛匹配模式之前。

如果包含两个相同的匹配模式，第二个也不再被匹配到。想要通过这种形式实现多路输出的话，可以参考out_copy插件。

如果将使用同一tag的<filter>放在<match>之后，这个filter通常也不会被执行到。

```shell
# You should NOT put this <filter> block after the <match> block below.
# If you do, Fluentd will just emit events without applying the filter.
<filter myapp.access>
  @type record_transformer
  ...
</filter>

<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>
```

### 内嵌ruby表达式（暂不作研究，可自查文档）

### 参数值支持的数据类型

每个插件支持若干参数，每个参数接受不同类型的值。下边是Fluentd支持的参数数据类型。

1. string：字符串，最常用的数据类型，可以为单行不带引号的字符串，或者单引号'或双引号"引用的字符串
2. integer：整型数字
3. float：浮点数
4. size：字节数，可以附加k、m、g、t等单位
5. time：时长，可以附加s、m、h、d等时间单位
6. array：JSON数组
7. hash：JSON数组

### 通用插件参数

系统保留的参数，以@为前缀。

1. @type：插件类型
2. @id：插件id，in_monitor_agent使用此参数
3. @label：标签，封装filter和match
4. @log_level：设置插件使用的日志级别

### 配置检查

配置文件编写完成之后，可以通过以下命令做下检查

```shell
fluentd --dry-run -c fluent.conf

/etc/init.d/td-agent configtest
```

### 配置格式相关事项

1. 双引号字符串和array、hash支持多行

```shell
str_param "foo  # This line is converted to "foo\nbar". NL is kept in the parameter
bar"
array_param [
  "a", "b"
]
hash_param {
  "k":"v",
  "k1":10
}
```

2. 双引号字符串使用\转义

```shell
str_param "foo\nbar" # \n is interpreted as actual LF character
```

## 第四章：Fluentd路由示例

继续了解Fluentd配置之前，我们先通过几个示例来了解Fluentd的路由过程。

1. 简单场景：单输入->过滤器->输出

```shell
<source>
  @type forward
</source>

<filter app.**>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
  </record>
</filter>

<match app.**>
  @type file
  # ...
</match>
```

![image-20210502203006820](https://i.loli.net/2021/05/02/JBAkILmnFGORpZM.png)

forward接收tcp消息，record_transformer给日志增加一个hostname字段，输出到file

2. 两个输入

```shell
<source>
  @type forward
</source>

<source>
  @type tail
  tag system.logs
  # ...
</source>

<filter app.**>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
  </record>
</filter>

<match {app.**,system.logs}>
  @type file
  # ...
</match>
```

较上一个示例，增加了一个tail输入，tail产生的事件直接写文件。

3. 输入->过滤器->带标签的输出

```shell
<source>
  @type forward
</source>

<source>
  @type dstat
  @label @METRICS # dstat events are routed to <label @METRICS>
  # ...
</source>

<filter app.**>
  @type record_transformer
  <record>
    # ...
  </record>
</filter>

<match app.**>
  @type file
  # ...
</match>

<label @METRICS>
  <match **>
    @type elasticsearch
    # ...
  </match>
</label>
```

forward产生的事件处理流程不变，dstat直接跳转至@METRICS指定的label，写入elasticsearch

4. 改写tag重新路由

```shell
<match worker.**>
  @type route
  remove_tag_prefix worker
  add_tag_prefix metrics.event

  <route **>
    copy # For fall-through. Without copy, routing is stopped here. 
  </route>
  <route **>
    copy
    @label @BACKUP
  </route>
</match>

<match metrics.event.**>
  @type stdout
</match>

<label @BACKUP>
  <match metrics.event.**>
    @type file
    path /var/log/fluent/backup
  </match>
</label>
```

route插件将worker标记的事件重新标记为metrics.event，并重新发送事件给路由引擎，事件进入两个处理分支：输出到stdout；写入file

5. 根据record内容重新路由

```shell
<source>
  @type forward
</source>

# event example: app.logs {"message":"[info]: ..."}
<match app.**>
  @type rewrite_tag_filter
  <rule>
    key message
    pattern ^\[(\w+)\]
    tag $1.${tag}
  </rule>
  # you can put more <rule>
</match>

# send mail when receives alert level logs
<match alert.app.**>
  @type mail
  # ...
</match>

# other logs are stored into file
<match *.app.**>
  @type file
  # ...
</match>
```

forward产生的事件由rewrite_tag_filter处理，提取record中的[log_level]，添加到原tag之前，生成新的tag。事件再次进入路由引擎，alert开头的tag标记的事件，通过mail处理；其他类型的事件写入file

6. 重新路由到指定label

```shell
<source>
  @type forward
</source>

<match app.**>
  @type copy
  <store>
    @type forward
    # ...
  </store>
  <store>
    @type relabel
    @label @NOTIFICATION
  </store>
</match>

<label @NOTIFICATION>
  <filter app.**>
    @type grep
    regexp1 message ERROR
  </filter>

  <match app.**>
    @type mail
  </match>
</label>
```

使用relabel插件，直接将事件路由到@NOTIFICATION指定的label处理。relabel不修改事件的tag。

### Fluentd配置：解析（Parse）配置项

Fluentd的某些插件支持<parse>配置项，用来自定义对输入数据的解析方法。

比如，对于一般的应用程序，输入给Fluentd的就是一行行的文本，开发者可以通过配置将文本解析成具有实际意义的JSON对象，方便后续处理。

### parse概览

parse配置项可以使用在<source>、<match>或<filter>中。如果使用的插件支持解析器特性，parse配置项就会生效。

```shell
<source>
  @type tail
  # parameters for input plugin
  <parse>
    # parse section parameters
  </parse>
</source>
```

这里，对于tail的输入，需要由parse指定的解析器来解析。

### 解析器插件类型

<parse>配置项需要通过@type参数来指定解析器的类型。Fluentd内核绑定了很多有用的解析器插件，也可以根据需要安装其他第三方解析器。

```shell
<parse>
  @type apache2
</parse>
```

这里，@type指定使用apache2这个解析器来解析输入日志。

### parse参数说明

- @type
  - Fluentd内置的解析器包含：regexp、apache2、apache_error、nginx、syslog、csv、tsv、ltsv、json、multiline、none

可选参数，这些参数的默认值会随使用的解析器不同而改变，具体使用时可参考相关解析器的使用说明。

- types：用于转换字段的数据类型，支持的数据类型为：string、bool、integer、float、time

   ```shell
   types user_id:integer,paid:bool,paid_usd_amount:float
   ```

- time_key：指定事件time属性使用的字段，若事件不含此字段，将使用当前时间

- keep_time_key：true则保留record中的time字段，默认false
- timeout：设置解析处理超时时间，主要用于检测错误的正则匹配

### 对time的进一步说明

如果把record的某个字段解析为事件的time，则需要说明如何去解析这个“时间字段”。可通过以下参数进行说明。

- time_type：时间字段使用的时间格式，支持float、unixtime和string格式。

```shell
float: seconds from Epoch + nano seconds (e.g. 1510544836.154709804)
unixtime: seconds from Epoch (e.g. 1510544815)
string: use format specified by time_format, local time or time zone
```

- time_format：用以说明time_type为string时的时间格式

- localtime：true则使用local time作为事件的time

- utc：true则使用UTC作为事件的time，和上边的localtime是互斥配置

- timezone：时区格式

### EFK：免费的日志采集与可视化搜索套件

我们收集日志是为了做进一步的分析。收集是第一步，收集到日志后还需要进行存储、索引，以便进行快速查询分析。我们还需要一个友好的查询界面，来方便用户使用日志。

本文介绍一个免费的开源软件组合，正好可以实现上述目的。它们就是Fluentd + Elasticsearch + Kibana，简称EFK。

Fluentd用于采集日志。Elasticsearch是一个开源的搜索引擎，以使用方便而著称。Kibana是一个开源的Web UI，为Elasticsearch提供了一个友好的使用界面。

使用EFK的日志系统，一般采用如下架构：

![image-20210502204528136](https://i.loli.net/2021/05/02/DeXplTdRLzWraQI.png)

![image-20210502204548340](https://i.loli.net/2021/05/02/s3B9aKluJ4f26pq.png)

下边，我们简单介绍一下如何搭建及使用EFK日志采集分析系统。

1. 准备工作

   Elasticsearch依赖Java，请安装Java 8（或更高版本）

2. 安装Elasticsearch

   可以直接从Elasticsearch官网上下载安装包，并解压。需要注意的是，elasticsearch不可以使用root身份用户，得创建单独的运行用户，并赋予其elasticsearch目录权限。

   我们这里通过yum的方式在CentOS6.5 x86_64系统上使用rpm包进行安装。

   - 创建并编辑/etc/yum.repos.d/elasticsearch.repo

     ```shell
     
     [elasticsearch]
     name=Elasticsearch repository for 7.x packages
     baseurl=https://artifacts.elastic.co/packages/7.x/yum
     gpgcheck=1
     gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
     enabled=0
     autorefresh=1
     type=rpm-md
     ```

   - 通过yum进行安装

     ```shell
     sudo yum install --enablerepo=elasticsearch elasticsearch
     ```

   - 启动elasticsearch

     ```shell
     sudo chkconfig --add elasticsearch
     sudo service elasticsearch start
     ```

3. 安装Kibana

   可以直接从Kibana官网上下载安装包，并解压。和elasticsearch一样，也需要为kibana创建单独的运行用户并赋予相关权限，才可以运行kibana。

   我们这里通过yum的方式在CentOS6.5 x86_64系统上使用rpm包进行安装。

   - 创建并编辑/etc/yum.repos.d/kibana.repo

     ```shell
     [kibana-7.x]
     name=Kibana repository for 7.x packages
     baseurl=https://artifacts.elastic.co/packages/7.x/yum
     gpgcheck=1
     gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
     enabled=1
     autorefresh=1
     type=rpm-md
     ```

   - 通过yum进行安装

      ```shell
      sudo yum install kibana
      ```

   - 启动Kibana

     ```shell
     sudo chkconfig --add kibana
     sudo service kibana start
     ```

4. 安装Fluentd

   - 请参考之前的文章来安装td-agent

   - 为Fluentd安装elasticsearch插件

     ```shell
     sudo /usr/sbin/td-agent-gem install fluent-plugin-elasticsearch --no-document
     ```

   - 在td-agent.conf中使用elasticsearch

     ```shell
     
     # get logs from syslog
     <source>
       @type syslog
       port 42185
       tag syslog
     </source>
     
     # get logs from fluent-logger, fluent-cat or other fluentd instances
     <source>
       @type forward
     </source>
     
     <match syslog.**>
       @type elasticsearch
       logstash_format true
       <buffer>
         flush_interval 10s # for testing
       </buffer>
     </match>
     ```

     这里syslog会经由Fluentd写入elasticsearch，并在elasticsearch中以logstash-%Y.%m.%d的命名方式创建索引。

   - 启动td-agent

     ```shell
     sudo /etc/init.d/td-agent start
     ```

5. 配置rsyslogd

   - 最后一步，我们需要在/etc/rsyslog.conf中添加如下配置，这样系统日志就可以从rsyslogd转发到Fluentd（端口为42185），并由Fluentd转发到elasticsearch中。

     ```shell
     *.* @127.0.0.1:42185
     ```

   - 重启rsyslogd：

     ```shell
     sudo /etc/init.d/rsyslog restart
     ```

6. 存储并搜索日志

   一旦Fluentd接收到了rsyslogd发送来的日志，并将这些日志写入到elasticsearch，我们就可以通过kibana进行可视化的数据查询了。

   打开kibana（http://localhost:5601），创建索引：

![image-20210502205501442](https://i.loli.net/2021/05/02/ilo2VmD1xu45ytc.png)

设置索引模式为logstash-*，选择@timestamp作为Time Filter字段名称。

然后就可以在“Discover”面板看到rsyslogd发送过来的日志了。

##  第五章：Fluentd配置：缓存（Buffer）配置项

Fluentd的output插件支持<buffer>配置项，用以缓存日志事件，提高系统性能。可在此配置项中设置buffer插件的相关参数。

### Buffer配置项概览

由于是output缓存，<buffer>需要在<match>中进行配置。

```shell

<match tag.*>
  @type file

  # ... parameters for output plugin

  <buffer>
    # buffer section parameters ...
  </buffer>

  # <buffer> section can be configured just once
</match>
```

### buffer插件类型

使用@type参数在<buffer>中指定缓存插件类型：

```shell
<buffer>
  @type file
</buffer>
```

Fluentd内核绑定了两种缓存插件：memory和file。也可以根据需要安装其他插件。

-  memory

   - 内存缓冲区插件提供了一个快速的缓冲区实现。它使用内存来存储缓冲区块。当Fluentd关闭时，无法快速写入的缓冲日志将被删除。

@type可以省略。省略则使用output插件默认的缓存插件（如果有的话），否则就使用memory。

对于大多数场景，建议使用file buffer插件，可以更多的持久化事件，避免memory的易失性。

### Chunk键值配置

output插件将收集到的事件组织为buffer chunk进行管理。可在<buffer>中配置chunk的键值，来决定如何收集日志事件。

```shell
<buffer ARGUMENT_CHUNK_KEYS>
  # ...
</buffer>
```

chunk的键值形式为逗号分隔的字符串，或者留空。

- 默认chunk键

  若不设置<buffer>的chunk键，并且所在output插件也没有指定默认的chunk键，output插件将会把所有收匹配到的事件写入同一个chunk，直到写满才进行flush。

```shell
<match tag.**>
  # ...
  <buffer>
    # ...
  </buffer>
</match>

# No chunk keys: All events will be appended into the same chunk.

11:59:30 web.access {"key1":"yay","key2":100}  --|
                                                 |
12:00:01 web.access {"key1":"foo","key2":200}  --|---> CHUNK_A
                                                 |
12:00:25 ssh.login  {"key1":"yay","key2":100}  --|
```

- chunk为tag

  若使用tag作为chunk的键值，output插件将会为每类tag维护一个chunk，相同tag的事件被写入同一个chunk。

  ```shell
  <match tag.**>
    # ...
    <buffer tag>
      # ...
    </buffer>
  </match>
  
  # Tag chunk key: events will be separated per tags
  
  11:59:30 web.access {"key1":"yay","key2":100}  --|
                                                   |---> CHUNK_A
  12:00:01 web.access {"key1":"foo","key2":200}  --|
  
  12:00:25 ssh.login  {"key1":"yay","key2":100}  ------> CHUNK_B
  ```

- chunk为time

  若time作为chunk的键值（此时必须设置buffer的time_key参数），output插件将按照time_key的设定将事件写入每个time_key_chunk。

  Time key的计算公式为：time(unix time) / timekey(seconds)。例如：

  ```shell
  timekey 60: ["12:00:00", ..., "12:00:59"], ["12:01:00", ..., "12:01:59"], ...
  timekey 180: ["12:00:00", ..., "12:02:59"], ["12:03:00", ..., "12:05:59"], ...
  timekey 3600: ["12:00:00", ..., "12:59:59"], ["13:00:00", ..., "13:59:59"], ...
  ```

  事件会按照计算出的时间范围写入不同的chunk，并将在chunk过期后进行flush。

  ```shell
  <match tag.**>
    # ...
    <buffer time>
      timekey      1h # chunks per hours ("3600" also available)
      timekey_wait 5m # 5mins delay for flush ("300" also available)
    </buffer>
  </match>
  
  # Time chunk key: events will be separated for hours (by timekey 3600)
  
  11:59:30 web.access {"key1":"yay","key2":100}  ------> CHUNK_A
  
  12:00:01 web.access {"key1":"foo","key2":200}  --|
                                                   |---> CHUNK_B
  12:00:25 ssh.login  {"key1":"yay","key2":100}  --|
  ```

  timekey_wait参数用来设定chunk的输出延迟时间，事件在chunk中停留“延迟时间”后被flush。默认值为600（10分钟）。

  ```shell
   timekey: 3600
   -------------------------------------------------------
   time range for chunk | timekey_wait | actual flush time
    12:00:00 - 12:59:59 |           0s |          13:00:00
    12:00:00 - 12:59:59 |     60s (1m) |          13:01:00
    12:00:00 - 12:59:59 |   600s (10m) |          13:10:00
  ```

  上边表格展示了每小时的缓存事件在不同的timekey_wait设置情况下实际的flush时间。

- 使用事件record的字段作为chunk键

  除去tag和time，chunk的键值还可以设置为事件record的字段。output插件将会根据这个字段的值来将事件写入不同的chunk。

  ```shell
  
  <match tag.**>
    # ...
    <buffer key1>
      # ...
    </buffer>
  </match>
  
  # Chunk keys: events will be separated by values of "key1"
  
  11:59:30 web.access {"key1":"yay","key2":100}  --|---> CHUNK_A
                                                   |
  12:00:01 web.access {"key1":"foo","key2":200}  -)|(--> CHUNK_B
                                                   |
  12:00:25 ssh.login  {"key1":"yay","key2":100}  --|
  ```

  这里，事件将会按照key1的不同取值写入相应的chunk中。

  chunk键值支持嵌套的record字段。可以参照插件的record_accessor语法来访问嵌套字段。例如：

  ```shell
  <match tag.**>
    # ...
    <buffer $.nest.field> # access record['nest']['field']
      # ...
    </buffer>
  </match>
  ```

- 键值组合

  可以使用多个chunk键来配置缓存方式。

   ```shell
   # <buffer tag,time>
   
   11:58:01 ssh.login  {"key1":"yay","key2":100}  ------> CHUNK_A
   
   11:59:13 web.access {"key1":"yay","key2":100}  --|
                                                    |---> CHUNK_B
   11:59:30 web.access {"key1":"yay","key2":100}  --|
   
   12:00:01 web.access {"key1":"foo","key2":200}  ------> CHUNK_C
   
   12:00:25 ssh.login  {"key1":"yay","key2":100}  ------> CHUNK_D
   ```

  这里，将事件按照tag+time的方式缓存到不同的chunk中。

  当然，chunk键值组合不宜过多，否则会降低I/O性能，也会消耗系统资源。

- 空键

  chunk的键值可以设置为[]，用以禁用output插件默认的chunk键配置。

  ```shell
  <match tag.**>
    # ...
    <buffer []>
      # ...
    </buffer>
  </match>
  ```

### 占位符

Fluentd配置文件支持占位符变量，这些变量会在运行中被替换为实际的值。比如out_file插件支持在path参数中设置占位符。

```shell
# chunk_key: tag
# ${tag} will be replaced with actual tag string
<match log.*>
  @type file
  path  /data/${tag}/access.log  #=> "/data/log.map/access.log"
  <buffer tag>
    # ...
  </buffer>
</match>
```

### buffer参数

除了上边chunk键值的配置，buffer还支持以下参数。

- 部分参数
  - chunk_limit_size：每个chunk的最大空间。memory默认为8MB，file默认为256MB。
  - chunk_limit_records：chunk可存储的最大事件数
  - total_limit_size：buffer插件可用的最大空间。memory默认为512MB，file默认为64GB。超出此值，后续操作会失败，数据会丢失！
  - chunk_full_threshold：chunk写满阈值，默认为0.95。当chunk实际占用存储超过此百分比后，事件会被flush。
  - compress：数据压缩方式（text或gzip），默认text表示不压缩。若设为gzip，Fluentd会将事件压缩后才写入chunk，在flush到output之前会自动解压。

- flush参数

  用以配置flush方式，以优化性能（包括时延和吞吐量）

  - flush_at_shutdown：程序退出时进行flush
  - flush_mode：flush模式。lazy，每timekey一次；interval，根据flush_interval的设置进行间隔flush；immediate，事件写入chunk后就flush。default，chunk键为time时等同lazy，其他等同interval。
  - flush_interval：默认每60s flush一次。
  - flush_thread_count：执行flush的线程数，默认1.
  - flush_thread_interval：flush线程等待间隔。
  - overflow_action：buffer队列满时执行的操作。throw_exception，抛异常；block，阻塞input插件；drop_oldest_chunk，丢弃oldestchunk。

- 重试参数
  - retry_timeout：flush失败后最大重试时长，默认72h。
  - retry_forever：是否一直重试
  - retry_max_times：最大重试次数
  - retry_wait：重试等待时长

## 第六章：使用Fluentd+MongoDB采集Apache日志

我们之前介绍了EFK日志采集分析套件，今天再介绍一个组合：Fluentd+MongoDB，用以实时收集半结构化数据。

### 背景知识

日志接入Fluentd后，会以json的格式在Fluentd内部进行路由。这就决定了Fluentd处理日志的方式是非常灵活的，它将日志视为半结构化数据，可以方便地修改其结构。

相应地，日志的最终存储数据库也应该擅长处理这样的半结构或者非结构化数据。这样整个系统搭配起来才更协调和高效。

而MongoDB恰好也是以类json的方式来处理内部数据的，非常适合作为Fluentd的目标存储。

### 实现机制

我们通常以下列架构来组合Fluentd+MongoDB这对CP。

![image-20210503203235678](https://i.loli.net/2021/05/03/x3UIHl27rRjmETk.png)

在这个组合中，Fluentd的职责为：

1. 持续“tail”Apache访问日志
2. 将Apache日志文本解析为有意义的字段（如ip、path等），并缓存
3. 定期将缓存的日志写入MongoDB

### 安装部署

1. 安装Apache、MongoDB
2. 安装Fluentd
3. 在Fluentd中安装MongoDB插件（最新版Fluentd已内置）

```shell
fluent-gem install fluent-plugin-mongo
```

### 配置说明

1. 首先配置输入端

```shell
<source>
  @type tail
  path /var/log/apache2/access_log
  pos_file /var/log/td-agent/apache2.access_log.pos
  <parse>
    @type apache2
  </parse>
  tag mongo.apache.access
</source>
```

使用tail来追踪Apache的日志文件access_log，使用Fluentd内置的Apache日志解析器apache2来解析日志。日志事件tag为mongo.apache.access。

2. 再配置输出端

```shell
<match mongo.**>
  # plugin type
  @type mongo

  # mongodb db + collection
  database apache
  collection access

  # mongodb host + port
  host localhost
  port 27017

  # interval
  <buffer>
    flush_interval 10s
  </buffer>

  # make sure to include the time key
  <inject>
    time_key time
  </inject>
</match>
```

<match>匹配所有mongo开头的tag，使用out_mongo作为输出插件。依次配置日志存储在MongoDB中的数据库和集合、MongoDB地址和端口。设置flush间隔为10秒，每10秒将缓存的日志写入MongoDB。

### 测试验证

确保各服务正常运行。

我们通过ping Apache来制造一些测试数据。

```shell
$ ab -n 100 -c 10 http://localhost/
```

然后，在MongoDB中就可以看到这些日志了。

```shell

$ mongo
> use apache
> db["access"].findOne();
{ "_id" : ObjectId("4ed1ed3a340765ce73000001"), "host" : "127.0.0.1", "user" : "-", "method" : "GET", "path" : "/", "code" : "200", "size" : "44", "time" : ISODate("2011-11-27T07:56:27Z") }
{ "_id" : ObjectId("4ed1ed3a340765ce73000002"), "host" : "127.0.0.1", "user" : "-", "method" : "GET", "path" : "/", "code" : "200", "size" : "44", "time" : ISODate("2011-11-27T07:56:34Z") }
{ "_id" : ObjectId("4ed1ed3a340765ce73000003"), "host" : "127.0.0.1", "user" : "-", "method" : "GET", "path" : "/", "code" : "200", "size" : "44", "time" : ISODate("2011-11-27T07:56:34Z") }
```

## Fluentd部署：日志

Fluentd是用来处理其他系统产生的日志的，它本身也会产生一些运行时日志。我们一起来了解一下Fluentd本身的日志机制。

Fluentd包含两个日志层：全局日志和插件级日志。每个层次的日志都可以进行单独配置。

### 日志级别

Fluentd的日志包含6个级别：fatal、error、warn、info、debug和trace。级别依次递增，高级别的日志包含低级别的日志。默认为info，所以默认情况下，日志中包含info、warn、error、fatal这4个级别的日志。

### 全局日志

Fluentd内核使用全局日志配置，若插件没有单独设置自己的日志配置项，插件也共用全局日志配置项。可通过命令行或配置文件进行设置。

1. 命令行

   -v、-vv用于增加日志级别，-q、-qq用于降低日志级别。

```shell
$ fluentd -v  ... # debug level
$ fluentd -vv ... # trace level
```

```shell
$ fluentd -q  ... # warn level
$ fluentd -qq ... # error leve
```

使用命令行可以在不改变配置文件的情况下调整日志级别，方便调试。

2. 配置文件

   也可以在配置文件中设置<system>的log_level来配置全局日志级别。

```shell
<system>
  # equal to -qq option
  log_level error
</system>
```

3. 插件日志

   可通过@log_level对每个插件单独设置日志级别，这个级别将覆盖全局日志级别。

```shell
<source>
  @type tail
  @log_level debug
  path /var/log/data.log
  ...
</source>
<source>
  @type http
  @log_level fatal
</source>
```

上边这个片段中，我们对两个不同的输入源分别设置了各自的日志级别。

4. 日志格式

   如今天第一篇文章中所述，Fluentd的日志支持text和json两种格式，默认使用text，可在<system>中进行设定。

```shell
<system>
  <log>
    format json
    time_format %Y-%m-%d
  </log>
</system>
```

若使用json格式，

```shell
2017-07-27 06:44:54 +0900 [info]: #0 fluentd worker is now running worker=0
```

这条日志将会转化为如下输出：

```shell
{"time":"2017-07-27","level":"info","message":"fluentd worker is now running worker=0","worker_id":0}
```

5. 将日志写入文件

   Fluentd默认将其日志输出到stdout，可通过-o将日志输出到文件中。

```shell
$ fluentd -o /path/to/log_file
```

若将日志写入文件，默认情况下Fluentd不会进行日志轮转，即会向指定的文件中不断写入日志，这可能会导致日志文件过大。可通过命令行参数开启日志轮转功能。

1. --log-rotate-age AGE

   这里AGE为整数或字符串，需要和下边的rotate-size配合使用。

   整数表示轮转文件个数；

   字符串表示轮转频率，可为daily、weekly或monthly。

2. -log-rotate-size BYTES

   BYTES为轮转文件的大小，达到此字节数即开始写入新的文件。

   当rotate-age值为整数时，通过此配置项控制日志的轮转。

```shell
$ fluentd -c fluent.conf --log-rotate-age 5 --log-rotate-size 104857600
```

6. 捕获Fluentd日志

   Fluentd自身日志也可以被采集。

   Fluentd使用fluent作为自身日志的tag，我们可以通过<label @FLUENT_LOG>来处理Fluentd自身的日志。

```shell
# Add hostname for identifying the server
<label @FLUENT_LOG>
  <filter fluent.*>
    @type record_transformer
    <record>
      host "#{Socket.gethostname}"
    </record>
  </filter>

  <match fluent.*>
    @type monitoring_plugin
    # parameters...
  </match>
<label>
```

这样做的一个用处是用来监控Fluentd运行情况。

## Fluentd部署：高可用配置

对于高访问量的web站点或者服务，我们可以采用Fluentd的高可用配置模式。

### **消息分发语义**

Fluentd设计初衷主要是用作事件日志分发系统的。这类系统支持几种不同的分发模式：

1. 至多一次。消息被立即发送，若传输成功，该消息不会再被发送。发送失败，则会导致消息丢失。现实环境下会有很多情况导致发送失败，比如网络暂时不可用。
2. 至少一次。消息至少会被发送一次，若发送失败，消息会被重发。这保证了消息不会被丢失，但可能导致接收端收到重复的消息。
3. 精确只发一次。消息刚好发送一次，能确保送达且不会重复。这是大家所期望的分发模式。实现此模式可能需要采用同步化的日志处理方式，当达到发送瓶颈时，告知业务层已无法接收更多的日志。

为了在不影响业务性能的情况下收集大量的日志，日志层必须以异步的方式运行。因此，Fluentd只提供了前两种传输模式。



### **网络拓扑**

为使得Fluentd具备高可用性，典型的部署架构需要包含两种不同角色的Fluentd模块：转发器（forwarder）和聚合器（aggregator）。其拓扑结构如下图所示

![image-20210503204657122](https://i.loli.net/2021/05/03/UZyQJ3xhFi2PtGR.png)

转发器部署在业务节点，用于收集业务方产生的本地日志事件，并将事件发送至聚合器。

聚合器持续地从转发器接收日志，对日志进行缓存，并定期上传日志到下一个处理方（典型的就是存储）。

聚合器采用主备模式。如上图，192.168.0.1为主，192.168.0.2为备。

### **转发器配置**

转发器的典型配置如下所示：

```shell
# TCP input
<source>
  @type forward
  port 24224
</source>

# HTTP input
<source>
  @type http
  port 8888
</source>

# Log Forwarding
<match mytag.**>
  @type forward

  # primary host
  <server>
    host 192.168.0.1
    port 24224
  </server>
  # use secondary host
  <server>
    host 192.168.0.2
    port 24224
    standby
  </server>

  # use longer flush_interval to reduce CPU usage.
  # note that this is a trade-off against latency.
  <buffer>
    flush_interval 60s
  </buffer>
</match>
```

这里有两个输入源，使用forward插件将日志事件发送到两个聚合器server中，其中通过standby指定192.168.0.2为备用聚合器。若两个聚合器节点都不可用，日志将会缓存在转发器节点。

### 聚合器配置

聚合器的典型配置如下所示：

```shell
# Input
<source>
  @type forward
  port 24224
</source>

# Output
<match mytag.**>
  ...
</match>
```

这个比较简单，使用forward插件作为输入源。日志会在本地缓存，并通过重传机制确保能送达目的地。

### 失败场景提示

1. 转发失败

   转发器收到应用层的日志事件后，先将事件写入本地磁盘缓存（由buffer_path指定）。每个flush_interval到来时，缓存事件被转发至聚合器。

   转发器进程若发生崩溃，进程重启后会自动重发已缓存的日志；转发器和聚合器网络若发生故障，转发器也会对日志进行重传。这在一定程度上保证了转发器的健壮性。

   但仍有一些情况可导致数据丢失：

2. 1. 转发器收到业务层日志，在将日志写入缓存之前发生崩溃
   2. 磁盘损坏

3. 聚合失败

   聚合器采用和转发器相同的失败处理机制，失败场景类似。

### 错误排查

采用此架构进行部署时，有时候会遇到“no nodes are available”的错误提示。这可能是节点间网络不通导致的。需要注意的是，节点之间通过24224端口传输数据，既使用TCP，也会使用UDP。

可通过以下命令进行检查：

```shell
$ telnet host 24224
$ nmap -p 24224 -sU host
```

