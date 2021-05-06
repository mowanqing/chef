# How to Parse Syslog Messages

## 简介

Syslog是一种流行的协议，它实际上运行在每个服务器上。用于收集各种日志。`syslog`的问题是服务有广泛的日志格式，没有一个解析器可以有效地解析所有`syslog`消息。

在本教程中，我们将展示如何使用Fluentd健壮地过滤和解析不同的`syslog`消息。

### 先决条件

- 对Fluentd有基本的了解
- rsyslogd正在运行的实例

在本指南中，我们假设您正在Ubuntu上运行td-agent

### 设置rsyslogd

打开/etc/rsyslogd.conf文件，并添加以下内容:

```shell
*.* @127.0.0.1:5140
```

然后重启rsyslogd服务。

```shell
$ sudo systemctl restart syslog
```

这告诉rsyslogd将日志转发到Fluentd将侦听的端口5140。

### 设置Fluentd

在本节中，我们将逐步发展Fluentd配置。

#### 步骤1:监听syslog消息

首先，让我们将Fluentd配置为侦听syslog消息。

打开/etc/td-agent/td-agent.conf，配置如下:

```shell
<source>
  @type syslog
  port 5140
  tag system
</source>

<match system.**>
  @type stdout
</match>
```

这是最基本的设置:它侦听所有syslog消息并将它们输出到标准输出。

现在请重启td-agent:

```shell
$ sudo systemctl restart td-agent
```

让我们来确认一下数据:

```shell
$ less /var/log/td-agent/td-agent.log
```

#### 步骤2:从sudo中提取syslog消息

现在，让我们看看这样的sudo消息:

```shell
2018-09-27 16:00:01.000000000 +0900 system.authpriv.info: {"host":"localhost",
"ident":"sudo","message":"pam_unix(sudo:session): session opened for user root by admin(uid=0)"}
```

出于安全原因，有必要了解哪个用户使用sudo执行了什么。为此，我们需要解析消息字段。换句话说，我们需要从sudo中提取syslog消息并以不同的方式处理它们。

为此，我们可以使用grep过滤器插件。它检查事件的字段，并基于正则表达式模式对它们进行筛选。在下面的示例中，Fluentd过滤出来自sudo并包含命令数据的事件:

```shell
<source>
  @type syslog
  port 42185
  tag system
</source>

<filter system.**>
  @type grep
  <regexp>
    key ident
    pattern /^sudo$/
  </regexp>
  <regexp>
    key message
    pattern /COMMAND/
  </regexp>
</filter>

<match system.**>
  @type stdout
</match>
```

#### 步骤3:从消息中提取信息

现在让我们从syslog消息中提取一些信息。为此，我们使用另一个名为**filter-parser**的插件。使用这个插件，你可以用正则表达式解析字段的内容。

下面是最终的配置:

```shell
<source>
  @type syslog
  port 5140
  tag system
</source>

<filter system.**>
  @type grep
  <regexp>
    key ident
    pattern /^sudo$/
  </regexp>
  <regexp>
    key message
    pattern /COMMAND/
  </regexp>
</filter>

<filter system.**>
  @type parser
  key_name message
  <parse>
    @type regexp
    expression /USER=(?<sudoer>[^ ]+); COMMAND=(?<command>.*)$/
  </parse>
</filter>

<match system.**>
  @type stdout
</match>
```

然后重启td-agent:

```shell
$ sudo systemctl restart td-agent
```

让我们用sudo执行一些注释:

```shell
$ sudo cat /var/log/auth.log
```

现在，你应该在/var/log/td-agent/td-agent.log中有这样一行:

```shell
2018-09-27 16:00:01.000000000 +0900 system.authpriv.notice: {"sudoer":"root","command":"/bin/cat"}
```

### 结论

Fluentd使接收`syslog`事件变得容易。您可以立即将数据发送到MongoDB和Elasticsearch等输出系统，但也可以在将处理过的数据传递到输出目的地之前在Fluentd中进行过滤和进一步解析。