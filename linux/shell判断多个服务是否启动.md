###需求

需求是需要写一个shell，开机自启动多个redis服务。这个很好实现，但是问题就在，执行开机自启动脚本后，不知redis服务是否全部启动成功，查询了shell中许多判断自启动是否执行成功的方法，经测试都失败。最终发现多种方法只能判断shell语句脚本是否执行成功，并不能来判断redis服务。

###初级解决

我开始思考，先去执行自启脚本，然后去判断进程是否存在这个进程，这样不就可以知道是否自启成功。通过下面的代码：

```shell
name="redis-server"

#通过服务名来判断服务器是否有这个进程

if test ( pgrep -f name | wc -l ) -eq 0 

then 

echo "bucunzai" 

else 

echo "cunzai" 

fi 

```

###加强版

虽然判断了redis服务是否执行成功，但是我的服务器有多个redis对象，多个redis服务，服务名称相同，只是端口不同，这样则只要启动成功一个或多个就会认为全部启动成功。通过查询有可以通过进程pid号来判断进程是否存在，但redis服务未启动之前我们谁也不知道它的进程pid是多少。最终我想到通过字符串来判断所有的redis服务是否全部启动成功代码如下

```shell
#服务名（尽量写详细）

servername="/usr/local/redis-4.0.8/src/redis-server"

#每个redis服务端口，每一个端口的配置文件启动一个redis服务

ports=('6371' '6373' '6374' '6375' '6376' '6378' '6379' '6380' '6381')

#获取所有的redis服务的进程信息

redis=ps -aux | grep $servername

echo $redis

#判断端口是否存在于redis服务信息中

for port in ${ports[*]};do

#这句话的意思是判断port这个字符串是否在redis这个字符串中

if [[ redis =~ port ]]

then 

echo $port

else 

echo "no"

fi

done

```