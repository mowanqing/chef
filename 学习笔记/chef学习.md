## chef语法和案例

chef使用的领域专用语言（DSL）是ruby的一个子集。

```ruby
#使用chef的“用户”资源来创建一个用户账户。
#资源是用来定义你的基础架构特定部分的组件

#创建一个名为alice，用户ID为503的用户账户
user 'alice' do
    uid '503'
end

# 语法
resource 'NAME' do
    parameter1 value1
    parameter2 value2
end
```

### 说明：

- 第一个部分指定要使用的资源（template,package或service）,然后跟着该资源的名字属性。
  - package ---> 需要安装/管理的包名
    - version参数定义要安装的程序包版本
  - template ---> 使用该模板最终渲染的文件在目标机器上的路径
    - source参数定义源模板的位置
  - service ---> 需要管理的应用/服务名
    - action参数定义要执行的动作

- do和end构成了一个代码块，在此代码块中可以声明资源的参数和他们的值。

```ruby
#这里的resource相当于
resource = Resource.new('NAME')
resource.parameter1 = value1
resource.parameter2 = value2
resource.run！
```

```ruby
#比如：
template '/etc/resolv.conf' do
    source 'my_resolv.conf.erb'
    owner 'root'
    group 'root'
	mode '0644'
end

package 'ntp' do
    action :upgrade
end

service 'apache2' do
	restart_command '/etc/init.d/apache2 restart'
end
```

### 多阶段执行模式

```ruby
#允许chef代码中包含控制逻辑和循环
['apple','banana','orange'].each do |type|
    file "/tmp/#{type}" do
        content "#{type} is delicious!"
    end
end
```

1. 第一阶段执行时被计算和存储

   ```ruby
   free_menory=node['memory']['total']
   file '/tmp/free' do
       contents "#{free_memory} bytes free on #{Time.now}"
   end
   ```

   

2. 第二阶段执行时chef执行的资源实际如下所示

   ```ruby
   file '/tmp/free' do
       contents "12345689 bytes free on 2021-05-01 22:37:25 -0400"
   end
   ```

### Resource

```ruby
# bash 使用bash解释器执行多行bash脚本：
bash 'echo "hello"'

#在chef内安装一个ryby程序
chef_gem 'httparty'

#cron，创建或管理cron任务，用来在指定的时间间隔运行指定的命令
#每周重启电脑
cron 'weekly_restart' do
    weekday '1'
    minute '0'
    hour '0'
    command 'sudo reboot'
end

#deploy_revision
#控制和管理应用程序部署，部署在代码版本控制工具中的代码
#从版本控制工具中复制和同步代码
deploy_revision '/opt/my_app' do
    repo 'git://github.com/username/app.git'
end

#directory
#管理目录或目录树，处理权限和所有者
# 用递归来确保/opt/my/deep/directory目录树中的每层目录都存在
directory '/opt/my/deep/directory' do
    owner 'root'
    group 'root'
    mode   '0644'
    recursive true
end

#execute
#执行任何单行的命令
#将内容写至文件
execute 'write status' do
    command 'echo "delicious" > /tmp/bacon'
end

#file
#管理已经存在（但不受chef管理）的文件
# 删除/tmp/bacon文件
file '/tmp/bacon' do
    action :delete
end

#gem_package
#在chef外安装一个ruby程序（gem），比如在目标机器的系统中安装一个应用程序或工具：
#安装bundler来管理应用程序依赖
gem_package 'bundler'

#group
#创建或管理一个包含本地用户账户的本地组：
#创建bacon组
group 'bacon'

#link
#创建和管理符号和硬链接
# link /tmp/bacon to /tmp/delicious
link '/tmp/bacon' do
    to '/tmp/delicious'
end

#mount
#挂载或卸载文件系统：
#挂载/dev/sda8
mount '/dev/sda8'

#package
#用操作系统提供的程序安装管理器安装一个程序包
#安装apache2程序包
package 'apache2'

#remote_file
#从一个远程位置（比如网站）传输一个文件：
#下载一个远程文件到/tmp/bacon
remote_file '/tmp/bacon' do
    source 'http://bacon.org/bits.tar.gz'
end

#service
#启动，停止或重启一个服务
#重启apache2服务
service 'apache2' do
    action :restart
end

#template
# 管理以纯文本为内容的嵌入式Ruby(ERB)模板
# bits.erb模板渲染/tmp/bacon文件
template '/tmp/bacon' do
    source 'bits.erb'
end

user
#创建或管理本地用户账户：
#创建bacon用户
user 'bacon'


```

### 第一个chef配方单

```ruby
file 'hello.txt' do
    content 'Welcome to chef'
end
#运行以下命令执行
chef-apply hello.rb
more hello.txt
```

### 用配方单指定理想配置

只需要告诉Chef你想要的配置是什么，而不是如何到达它。

chef在做任何事情之前，它用配方单中指定的资源和属性来回答“我在意什么”这个问题，然后Chef根据其管理的机器现在的状态，决定如何将其转换到理想的配置。

![image-20210501135958072](https://i.loli.net/2021/05/01/owxmS3rljMqDd6s.png)

```ruby
file "#{ENV['HOME']}/stone.txt" do
    content 'Written in stone'
end

file "#{ENV['HOME']}/stone.txt" do
    action :delete
end
```

 #### chef概念

配方单（recipe）

- 一系列用Ruby领域专用语言（DSL）来些的描述理想配置的指令

资源（resource）

- 对于Chef所管理的东西（比如文件）的一个跨平台的抽象表达。资源是chef代码的组成部分。通过在配方单中使用不同的资源来告诉Chef你的理想配置。

属性（attribute）

- 传递给资源的参数



