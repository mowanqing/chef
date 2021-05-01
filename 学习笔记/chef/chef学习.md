## jchef语法和案例

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

## 第六章：用Chef客户端管理节点

chef-workstation：

- 用来写chef代码的电脑，宿主机上安装了workstation包

chef-client（node）：

- 受chef管理的机器，当一个机器执行chef配方单并因此保证机器是在理想的配置下时，该机器成为node（chef不仅可以管理一般意义上的电脑或服务器，也可以管理基础架构中的其他组件，比如交换机，路由器，或存储设备等）

### 第一次运行chef客户端

```ruby
echo 'log "Hello,this is an important message."' > hello.rb

chef-client --local-mode  hello.rb --log_level info --logfile <LOGLOCATION>选项
```

### chef客户端的三种模式

- 本地模式
  - 内存中模拟一个完整的chef服务器
- 客户端模式
  - 生产模式中使用，chef-client被管理
- Solo模式
  - 在本地运行但chef功能有限，不支持回写（服务器数据写在本地的过程）

### 命令行工具Ohai

```shell
ohai | more
```

ohai收集电脑当前状态的信息，包括网络配置，cpu状态，操作系统类型和版本，内存使用量

输出格式时json，chef-client从ohai读取json输出，然后将信息转化到node对象中，chef代码中我们可以访问到node对象。

通过以下属性，可以在代码里引用节点的ip地址，属性时chef管理的一个变量。

```node['ipaddress']```

node是另外一个在chef代码中可以使用的属性，它包含在节点上运行ohai输出的所有信息。和ENV相似。

可以嵌套：

```node['virtualization']['system']```

也可以：

```node[:vituralization][:system]```

```node.vituralization.system```

### 访问节点信息

chef-client使用ohai来收集节点的许多信息。这样chef才可以智能地判断如何将节点转换至配方单中指定的理想的配置。chef将这些信息作为节点属性来让你可以在代码中访问。属性是chef维护的一个变量。

```shell
cat << EOF > info.rb
> log "IP Address: #{node['node['ipaddress']']}"
> log "MAC Address: #{node['macaddress']}"
> EOF

chef-client --local-mode info.rb --log_level info
```

任何被chef管理的实体必须安装chef客户端。使用chef-client工具来执行chef运行。chef靠这样运行来管理节点。

chef-client在chef运行中读取配方单。配方单中通过 **resource**指定**理想的配置**，chef决定具体执行的步骤来将系统转兑换为理想配置。chef能偶合理，智能地分析节点的配置因为它通过+ohai+收集大量的关于节点状态的信息并储存在node（节点）属性中。

## 第七章：攥写和使用菜谱

菜谱（cookbook）是使用chef进行基础架构管理的基础组件。可以将其想象为一些配方单的集合。

**每个菜谱表示配置一个单位的基础架构**（比如网页服务器，数据库或应用程序）所需的指令集合。

**配方单**则是这整个过程中包含代码的一小部分。菜谱包含一个或多个配方单，也包含其他用来支持配方单运行的组件，比如存档，图像或程序库。此外，菜谱中存有**配置信息**，针对平台的实现以及需要用chef管理基础架构的资源的声明。

### 第一个菜谱：每日消息

```shell
chef generate cookbook motd
cd motd
chef generate file motd
vi motd # 写一些东西上去

##knife
knife cookbook create motd --cookbook-path .
cd motd
vi motd
```

配方单都位于cookbook中的recipes/子目录中

```ruby
#编辑 motd/recipes/default.rb
cookbook_file "/etc/motd" do
    source "motd"
    mode "0644"
end

cookbook_file 将菜谱中files/子目录下的文件传输到chef管理的节点中
"/etc/motd"是name属性，在此资源中name定义拷贝到节点的目标文件路径。
source定义此菜谱中files/子目录中源文件的名字
```

### 第一次运行chef

使用收敛converge来表示通过运行chef_client把菜谱部署到节点并通过执行一个运行清单来将节点转化成理想的状态的过程。称为收敛一个节点

> chef可以动态调整如何将一个节点转换成理想状态需要做的事情。如果运行中途被取消，下次它会自动从没完成的地方开始继续运行。该容错方法的关键是chef用来配置节点所使用的计划完全以数据驱动，取决于ohai运行后返回的结果。

### 剖析chef运行

1. 开始运行chef客户端

   chef-client进程在远程节点启动。进程可能由一个服务，cron任务或某用户手动启动。chef-client进程负责在目标节点上运行包含chef代码的配方单的菜谱。

2. 创建节点

   chef-client进程在内存中构建node（节点）对象。它运行ohai并收集所有关于这个节点的自动属性（比如主机名，FQDN，平台，用户等）

3. 同步

   运行清单被发送到节点。运行清单包含要在目标节点执行的配方单的清单。运行清单是一个有序的，完全展开且要在目标节点运行的配方单清单。运行清单所需的菜谱的URL也同样被下载到节点上。目标节点将这些所需的菜谱下载并缓存到一个本地文件储存中。

4. 加载

   菜谱和Ruby组件在此步骤被加载。菜谱级别的属性在此与第2步中ohai生成的自动属性相结合。菜谱的不同组件按以下顺序加载：

   - **库（Libraries）**加载每个菜谱中libraries/目录下的所有文件，这样语言扩展或更改将会在余下的chef运行步骤中可用。
   - **属性（Attributes）**加载每个菜谱中attributes/目录下的所有文件，并与ohai属性结合
   - **定义（Definitions）**加载每个菜谱中definitions/目录下的所有文件。这些文件定义在配方单中用到的类似资源的可重用代码。因此必须在加载配方单前加载。
   - **资源（Resource）**加载每个菜谱中resources/目录下的所有文件。资源必须在配方单之前加载因为配方单会使用资源代码。
   - **提供者（Providers）**加载每个菜谱中providers/目录下的所有文件。以便资源引用合适的提供者。
   - **配方单（Recipes）**加载并执行每个菜谱中recipes/目录下的所有文件。在这个阶段，配方单并未被执行来将节点转换为理想配置，配方单中的Ruby代码编译并转换成最后将在节点上执行的配方单。每个资源也在此时被加入到资源集合中。

5. 收敛

   收敛阶段则是每次chef运行中最重要的阶段。在这个时候chef配方单在目标节点执行并改变节点到理想状态，比如安装尚未安装的程序包，复制渲染号的模板或文件到目标位置等等

6. 报告

   如果chef客户端运行成功，节点会保存任何新的属性值，如果失败，客户端会抛出异常而节点对象也不会被更新，通知机制和异常处理器将运行来通知工作人员，比如发送email，发布消息到IRC或通知如PagerDuty之类的值班系统。

#### 运行清单

运行清单包含要在目标节点执行的配方单的清单。在之前的实验中都是把一个配方单作为参数直接传给chef-client，但如果是上百个配方单因如何管理？

运行清单的用途则是提供一个方便的方法来指定需要运行哪些配方单。

通过运行清单来指定在某个节点需要运行的配方单。

```shell
recipe['<菜谱名字>::<配方单名字>']
recipe['motd::default']
```

在生产环境中，运行清单作为节点的属性保存在chef服务器中。

![image-20210501161221599](https://i.loli.net/2021/05/01/PgdL24RADpinE7t.png)



### 菜谱结构

创建cookbook：

```shell
chef generate cookbook
knife cookbook create
```

**菜谱目录结构**：

![image-20210501161420663](https://i.loli.net/2021/05/01/25r9ldOowVYQG7I.png)

attributes:

可以在菜谱中提供自定义的属性来补充或覆盖ohai在目标节点上所生成的自动属性。属性经常用来定义应用程序分布路径，基于特定平台的值或在某个节点需要安装的软件的版本等

files：

files文件夹是此菜谱中集中存储将要分发到目标节点的文件的地方。文件可以为纯文本，图像，zip文件等。这些文件可以通过cookbook_file资源被分发到目标节点。files/目录下的文件结构控制是否将特定文件分发到特定节点。要分发到所有节点的文件都会放在files/default/子目录内

recipes:

recipes目录包含chef配方单。配方单文件包含chef代码。此目录中可以包含多个.rb配方单文件。默认recipe为default.rb。在每个节点上，chef只会运行该节点的运行清单中指定的配方单。运行清单存储在chef服务器中，.kitchen.yml文件中

templates：

templates目录存储chef的模板。templates目录和files目录类似，目的都是将文件分发到目标节点上，然后templates中的文件是erb模板文件，可包含rubu代码的纯文本文件，在复制到目标节点之前，文件中的ruby代码被执行并渲染成相应的文件内容。

当需要被分发到目标节点的文件中包含在不同情况下需要渲染成不同内容的情况下，则可以用模板来完成这任务。templates，目录和files目录使用一样的子目录命名方法来控制复制哪些文件到哪些特定的节点。



其他：

Berksfile,definitions,libraries,providers,resource





### 必须了解的四个资源

1. package

   使用正确的安装包管理器（yum,apt,pacman等）来安装一个程序包

2. service

   管理用package资源安装的所有后台进程/服务的生命周期

3. cookbook_file

   从菜谱的files目录复制文件到目标节点的指定位置

4. template

   类似cookbook_file的资源，允许你复制文件到目标节点，而由于文件为嵌入式Ruby模板，所以你可以用变量来控制复制到节点的文件内容。



### 创建菜谱

1. 前提工作
   - 名字
     - mysql
   - 用途
     - 在目标机器上安装和配置Mysql服务器以及Mysql客户端
   - 成功标准
     - 做最少的事情，让Mysql运行并提供一种创建Mysql用户，数据库和数据表
   - 应用/服务
     - 每个菜谱应该只管理一个应用或服务（基础架构的一个单位）。创建菜谱之前，应该了解这个将要被管理的应用或服务，如果没办法将范围缩小到单个应用或服务，就应该在前面的步骤中重新制订愿景来缩小范围。
   - 所需步骤
     - 明确手动做这些工作的步骤是什么，以及需要什么先决条件



![image-20210501164036987](https://i.loli.net/2021/05/01/cI7MfxwTquzARyH.png)



**package:**（action）

根据ohai收集的信息判断platform和platform_family结果调用不同的提供者（providers）

- :install(默认值)
- :upgrade
- :remove
- :puge



**service:**(action)

对应用服务进行管理，操控。可以使用数组将多个动作传递给service资源 

- :enable
- :start
- :restart
- :stop

```ruby
service "httpd" do
    action [ :enable,:start ]
end

chkconfig --list httpd |grep 3:on #查看自启动状态
```



**template:**

可以在目标节点上创建文件，额外的功能是可以在源文件里包含变量或其他ruby逻辑控制写出目标节点的文件的内容。

```ruby
template "/var/www/html/index.html" do
    source 'index.html.erb'
    mode '0644'
end
```

- source属性指定包含erb语句的模板的源文件，模板在templates子目录中，确保这写模板位于templates/default子目录内。

> default子目录是用来做什么的？
>
> 在files/default和templates/default中的default子目录究竟是干什么用的？chef允许你在菜谱中根据目标节点的平台指定不同的文件。chef需要你在files或templates目录下创建子目录来进行过滤。可以通过以下子目录命名格式过滤：
>
> - 节点的主机名（foo.bar.com）
> - 节点的平台-版本（redhat-6.5.1）
> - 节点的平台-部分版本（redhat-6.5和redhat-6）
> - 节点的平台（redhat）
> - default
>
> 在大多数时候，都只用default作为子目录名，表示其中的文件或模板将被复制到所有应用此菜谱的节点。

```shell
chef generate template index.html
touch template/default/index.html.erb
```

在erb文件中，当chef看到包含在<%= 和%>中间的语句时，chef会去执行其中的变量并将它的值渲染在文件内容中替代这个变量，比如以下的index.html.erb文件中的<%= node['hostname'] %>变量。chef读取node['hostname']变量的值，并以其替换<%= node['hostname'] %>的内容

```erb
This site was set up by <%= node['hostname'] %>
```

创建菜谱的过程：

![image-20210501173113026](https://i.loli.net/2021/05/01/lc9VJMUOZi2X6oG.png)

## 第八章: 属性

