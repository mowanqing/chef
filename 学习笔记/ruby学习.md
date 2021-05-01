## Ruby学习

### 数据类型：

Number:1

String:"jim"

Bool:true,false

Array:[1,2,3]

Hash: { :name => "jim", :age => 20 }

### 赋值：

Ruby中的变量，不需要做类型声明。直接就用：

```ruby
name=“jim”
# => jim
```

### 命名规则

常量：全都是大写字母.ANDROID_SYSTEM='android'

变量：如果不算@，@@，$的话，是小写字母开头.下划线拼接。例如:color,age,is_created

class,module: 首字母大写，骆驼表达法：Apple，Human

方法名： 小写字母开头，下划线拼接。可以以问号 ？或者等号= 结尾，例如：name，created？，color=

### Class的写法

作为面向对象语言，class毫无疑问是最重要的。Ruby中的任何变量都是object（不想java，int是基本数据类型，integer才是class）

具体的写法：

1. 名字首字母大写
2. class开头，end结尾
3. 文件名字与class名称一样，只是改为：下划线+小写

例如：

```ruby
Class Apple
	#这个方法就是在Apple.new 时自动调用的方法
	def initialize
        #instance variable,实例变量
        @color
    end

	# getter 方法
	def color
        return @color
    end

	# setter 方法
	def color=color
        @color=color
    end

	#private
	def i_am_private
    end
end

red_apple=Apple.new
red_apple.color='red'
puts "red_apple.color:#{red_apple.color}"
```

例2：

```ruby
ruby apple.rb
# => "red_apple.color:red"

class Apple
    #自动声明了@color，getter，setter
    attr_accessor 'color'
end
```

### 各种变量

类变量：class variable,例如：@@name，作用域：在这个类中，所有的多个instance会共享这个变量，用的很少。

实例变量：instance variable，例如：@color，作用域仅在instance之内，在rails中被大量使用（比如小明的name属性只是他自己的，小红的name属性是她自己的，不同的实例，name属性不同）

普通变量：local variable，例如：age=20，作用域仅在某个方法内，大量使用

全局变量：global variable,例如：$name="jim",作用域在全局。用的更少。



例如：

```ruby
class Apple
    @@from = 'china'
    
    def color= color
        @color = color
    end
    
    def color 
        return @color
    end
    
    def get_from
        @@from
    end
    
    def set_from= from
        @@from = from
    end
end
```

```ruby
red_one = Apple.new
red_one.color = 'red'
puts red_one.color

red_one.set_from('Japan')
puts red_one.get_from
```



### 方法：类方法与实例方法

类方法：

跟实例无关，可以由class直接调用的方法

实例方法：

由某个class的实例调用的方法

例如：

```ruby
class Apple
    #类方法
    def Apple.name
        'apple'
    end
    
    #实例方法
    def color
        'red'
    end
end

Apple.new.color

Apple.name
```

### 字符串

```ruby
single_line="我是一个字符串"
multiple_line=%Q{
    今天天气不错
    出去走走
    }
```

### Symbol

不变的字符串

```ruby
#内容永远不变，等同于一个常量
#特别适合做hash的key
#大量被用到

class Apple
    attr_accessor :color
end

#该 :color 就是symbol.不会变化的字符串，
#:name等同与"name".to_symbol
```

### 判断数据类型

```ruby
'abc'.class
:abc.class
'abc'.to_sym
```



### 字符串插值

```ruby
puts 'hi,#{name}!'
puts 'hi,jim'

#报错
a=1
puts "a is:" + a

puts "a is：#{a}"
```

### 数组

```ruby
包含同一数据类型的数组：
numbers = [1,2,3]
包含多种数据类型的数组：
[1,'two',:three,{ :name => 4 }]
```

### Hash: key/value

```ruby
{
  :name => 'jim',
  :age => 18
}

hash,也叫dictionary
```

同一个hash的三种写法

```ruby
#任何情况下都生效的语法： =>
jim = {
    :name => 'jim',
    :age => 20
    }

#Ruby1.9之后产生的语法：更加简洁
jim={
    name:'jim',
    age:20
    }

#也可以写成：
jim={}
jim[:name]='jim'
jim[:age]=20
```

hash的key:  symbol与string不同

但是，symbol与string，是不同的key，例如：

```ruby
a = {:name =>'jim','name'=>'hi'}
a[:name] #=> 'jim'
a['name'] #=> 'hi'
```

### 条件语句 if-else

if else end 是最常见的

```ruby
a=1
if a == 1
    puts "a is 1"
elsif a == 2
    puts "a is 2"
else 
    puts "a is not in [1,2]"
end
```

### case 分支语句

例如：

```ruby
a=1
case a
    when 1 then puts "a is 1"
    when 2 then puts "a is 2"
    when 3,4,5 then puts "a is in [3,4,5]"
    else puts "a is not in [1,2,3,4,5]"
end
```

### 三元表达式

```ruby
a=1
puts a == 1 ? 'one':'not one'
#=> one
```

### 循环：for,each,loop,while

for,each (前者是关键字，后者是普通方法)

```ruby
#for 与 each 几乎一样，例如：
    [1,2,3].each {
        |e|
        puts e
        }
#等同于下方
for e in [1,2,3]
    puts e
end
```

### eachd 与for的区别

for与each都可以做循环，一般使用each

区别在于：for是关键字，each是方法。for后面的变量，是全局变量，不仅仅存在与for...end这个作用域之内

### 循环：while与loop

loop与while是几乎一样的

```ruby
loop do
    #your code
    break if <condition>
end

while <condition>
   #code
end
```

例如：

```ruby
a=[2,1,0,-1,-2]
loop do 
    current_element =a.pop
    puts current_element
    break if current_element <0
end

count = 1

while count < 10
 puts count
 count = count + 1
end
```

### **Heredoc表示法**

```ruby
a = 'in here'

b = <<METHOD_DESCRIPTION
This is a  test_sting.
just fun with ruby #{a}
METHOD_DESCRIPTION

print b
```

### Bool

```shell
true    true
false   false
Object  true
0 		true
1 		true
-1		true
nil		false
""		true
[]		true
{}		true
```

### 内置方法

```shell

数组：
	1. 增加
		a. months  <<  "August"            往数组最后的位置加入新值
		b. months.push("September")  
		c. months.insert(2,"October")  往索引位置插入新值
	2. 删除
		a. months.pop 删除数组中最后一项，如果有参数，则删除相应个数的item
		b. months.delete_at(2) 删除指定索引位置的item
	3. 改
		a. months[索引] = 新的值  改变原数组的值
	4. 查
		a. months.Include?(检验的值)   检查参数的值是不是数组中的元素
	5. 内置方法
		a. months.sort  对数组进行排序
		b. months.flatten 将嵌套数组合并成一维数组
		c. months.each { |item| puts item } 数组的迭代器 将数组中的每个item运行一次block中定义的代码
		d. months.map { |item| item**2}  对数组每个元素调用块内的代码一次，返回包含新值的数组

哈希：
	1. 创建：
		a. person = { "key" => "value"}
		b. person = Hash.new
	2. 访问：
		a. person["key"]
	3. 增加：
		a. person["key"] = 'value'
	4. 删除：
		a. person.delete("key")
	5. 内置方法：
		a. Person.each do |key,value|
		  Puts "#{key} is #{value}"
		End
		b. person.has_key?("key")  检查hash中有没有特定的键
		c. person.select{ |key,value| key == "name" }  根据块中的条件，检索符合的键值对
		d. person.fetch("name") 返回指定键的值
集合
	1. 创建：
		require 'set'
		my_set = Set.new([5, 2, 9, 3, 1])
	2. 增加：
		a. my_set << 5 向数组尾部加入新值
		b.  my_set.add 1 
Range
	1. …  三点[1,10) 1到9
	2. ..  二点[1,10]  1到10
a  = Range.new(1,10)  和二点一样
```

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





