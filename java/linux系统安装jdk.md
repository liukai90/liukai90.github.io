 1.oracle官网下载jdk
---------------

网址：https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

选择则Linux系统1.8版本64位如图：
![图片描述][1]


2.把jdk压缩包解压到安装的目录
-----------------

例如我的：

    mv jdk-8u191-linux-x64.tar.gz /opt/soft/java/

解压：

    tar -zxvf jdk-8u191-linux-x64.tar.gz 

3.配置环境变量
--------

编辑配置文件：

    vi /etc/profile

向文件里追加配置：

    JAVA_HOME=/opt/soft/javajdk1.8.0_191 
    CLASSPATH=$JAVA_HOME/lib/
    PATH=$PATH:$JAVA_HOME/bin
    export PATH CLASSPATH JAVA_HOME

如图：
![图片描述][2]


4.启用配置
------

执行下面命令或重启：

    source /etc/profile

5.检测配置是否成功
----------

    java -version
    java
    javac

6.如果遇到权限不够问题
------------

    sudo su 

根据提示输入root用户密码即可。

7.安装原理
------

我们下载的压缩包是编译好的可执行文件，我们所要做的是把这些文件路径引入，就可知执行 java javac 这些命令。在Linux里引入环境变量是通过profile文件，我们配置JAVA_HOME意为java安装的路径，以便已后程序出现异常找到安装的位置。CLASSAPTH意为编译好的字节文件存放位置，我们可以找javac命令编译好的字节文件的位置。PATH引入JAVA可执行文件的路径，也就是java javac这些可执行文件。
