## 1.编写redis自启动shell如下：

```shell
#!/bin/sh

dir="/usr/local"

redis="/redis-4.0.8"

redisdir={dir}{redis}

src="/src"

server={redisdir}{src}

conf="/redisConf"

redisConf={redisdir}{conf}

#script_path=$(pwd)

echo "start redis server"

cd $redisConf

recommand="redis-server $CP &"

PROGDIR="/usr/local/redis-4.0.8/src/redis-server"

for file in ls ${redisConf}

do

    if [ ! -d $file ]; then

        if [[ $file == *.conf ]]; then

            CP=$file

            if PROGDIR CP > /dev/null 2>&1 &

				then
					echo -e "$CP redis start OK"

            fi

        fi

    fi

done

sleep 2

```



## 2.校验启动服务并打印日志如下

```shell
#运行服务的基础路径

basepath="/usr/local"

#日志文件夹

log="/log"

#日志文件夹全路径

logdir=basepathlog

#获取当前系统日期

DATE=$(date +%Y-%m-%d)

#获取当前系统时间

TIME=$(date +%H:%M:%S)

xian="--------"

null=" "

next=xianDATEnulltime$xian

#日志文件类型

logType=".log"

#日志文件名称

logFileName=DATElogType

cd $logdir

#创建文件

touch $logFileName

echo next>>logFileName

echo "start redis-server">>$logFileName

#进入基础路径文件夹

cd $basepath

#运行redis服务

nohup ./start1.sh > /dev/null 2>&1 &

#下面验证redis服务是否全部启动成功

cd $logdir

#redis服务全名称

servername="/usr/local/redis-4.0.8/src/redis-server"

#定义所有redis服务的端口

ports=('6371' '6373' '6374' '6375' '6376' '6378' '6379' '6380' '6381' '6382' '6394' '6395' '6400' '6450')

#获取所有redis服务的信息

redis=ps -aux | grep $servername

flag=1

for port in ${ports[*]};

do

	if [[ redis =~ port ]]

		then 

			echo "redis:$port succeed start"

			echo "redis:port succeed start">>logFileName

		else 

			flag=0

			echo $flag

			echo "redis:$port fail start"

			echo "redis:port fail start">>logFileName

	fi

	

done

if test $flag -eq 0

	then

		cd $basepath

		nohup ./start1.sh > /dev/null 2>&1 &

		echo "restart"

fi

```

