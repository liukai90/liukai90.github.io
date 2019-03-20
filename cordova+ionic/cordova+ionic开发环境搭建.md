#1. 环境依赖关系叙述#
移动端混合开发的一个明显优势就是，一套代码两套部署，开发一套项目代码，可分别打成Android的包和ios的包。无论是混合开发还是原生开发，都是会需要原生的平台。我们先以Android平台为例，首先肯定需要AndroidSDK,Android环境缘起于java，所以必须先安装JDk,这是平台的环境.混合开发顾名思义需要前端和原生两块环境内容。混合开发平台，我们选择的是cordova,那么它依赖于node.js环境，并且需要node.js的npm模块来帮它下载插件。创建项目还需调试运行，那么就会需要Android模拟器。由于原生的Android模拟器启动过于慢，而且由于网络限制，google的cpu渲染加速器环境也难以下载。我们这里使用的是一个国外的好用且免费的第三方模拟器Genymotion。东西是免费的，但是需要注册和登陆。
一共需要搭建的环境也就这么几个JDK，AndroidSDK,node.js,cordova, Genymotion。

### 开发环境

node.js

cordova6.0.0

ionic4

### 测试运行环境

JDK

AndroidSDK

Genymotion

#2.安装说明#
##2.1 JDK##
安装教程很多，记住安装时下载1.8版本
菜鸟教程：[http://www.runoob.com/java/java-environment-setup.html][1]
##2.2 AndroidSDK##

androidSDK由于国内限网，推荐一个国内的一个下载网站：[http://www.androiddevtools.cn/][2]
下载后根据提示安装，推荐下载的包就不要取消。系统一般会帮你默认勾选安卓最新版本Android9.0，但是Android9.0会出现模拟器启动不了的问题，坑很多，建议安装9.0以下的，我选的Android6.0，勾选下图的选项即可。

![clipboard.png](/img/bVbk5Na)


下载完配置Android环境变量
打开安卓的安卓目录如图，我画圈的两个目录，需要加入到path里面
D:\Program Files\android\platform-tools; D:\Program Files\android\tools;

![clipboard.png](/img/bVbk5Nz)

##2.3Genymotion 安装

- 官网下载

  官方只需注册即可免费使用，使用下面链接注册即可。

  注册：https://www.genymotion.com/

  下载：https://www.genymotion.com/download/

  下载后按照提示安装即可，打开软件时登录选择个人登录即可。

- 下载安卓镜像

 打开后如图，点击add。

 ![图片描述][3]



​	

 - 找自己需要的版本下载即可
   ![图片描述][4]

## 2.4安装nodejs

- 官网下载nodejs免安装版

  https://nodejs.org/en/download/

选择windows免安装版64位

![图片描述][5]

- 解压到要安装的目录

  ![图片描述][6]

- 添加环境变量

  计算机(右键)-->属性(左键)-->高级系统设置(左键)-->环境变量(左键)

 ![图片描述][7]

- 检查是否配置成功

  ```web-idl
  node -v
  npm -v
  ```

  正常显示出版本号则说明安装成功

- 更换npm镜像源

首先来说为什么要更换镜像源，由于npm的镜像源是国外的，我们使用npm工具安装软件环境时，由于网速问题会导致很多难以解决的问题，而且下载巨慢，故将镜像源更换为淘宝的镜像源。

执行下面命令更换软件源

```web-idl
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

查看是否更换成功

```web-idl
cnpm -v
```

不报错且出来一段信息则说明更换成功。

- 参考学习

  菜鸟教程：http://www.runoob.com/nodejs/nodejs-tutorial.html

  w3school:https://www.w3cschool.cn/nodejs/nodejs-install-setup.html

  npm基本使用：https://www.w3cschool.cn/nodejs/nodejs-npm.html



## 2.5安装cordova平台

- 官网

  https://cordova.apache.org/

- 使用npm安装平台

  ```web-idl
  cnpm install -g cordova@6.0.0
  ```

  上面安装指定版本的cordova，我们这里安装cordova6.0.0,建议不要安装7版本和8版本，后面创建项目时会出现很多问题。

  ```
  cordova -v
  ```

  检查是否安装成功，正确显示出版本号则说明安装成功。

- 项目相关命令

  ```shell
  #1.创建项目
  cordova create MyApp
  cd ./MyApp
  #2.添加平台
  cordova platform add browser #添加浏览器平台
  cordova platform add android #安卓平台
  cordova platform add ios #ios平台
  #注意添加相关平台需要拥有相关平台的环境
  #3.编译打包
  cordova build android #打成android平台的包 .apk
  cordova build ios #打成ios平台的包
  #4.运行到androidSDK自带的模拟器
  cordova emulate android
  #5.运行到第三方模拟器或真机
  cordova run android
  ```

- 参考学习

  w3school:https://www.w3cschool.cn/cordova/cordova_overview.html



## 2.6angularjs环境搭建

- 官网

  https://www.angular.cn/guide/quickstart

- 安装项目脚手架

  ```
  npm install -g @angular/cli 
  ```

  这里只是为了给ionic创建项目提供环境，但是要使用ionic开发就必须学会angularjs。



## 2.7ionic安装配置

- 官网

  https://ionicframework.com/docs/cli/

- 安装

  -g是全局的意思，latest是最新版的意思。

  ```
  cnpm install -g ionic@latest
  ```

- 项目相关命令

  ```shell
  #1创建项目
  ionic start myNewProject tabs #tabs是项目模板的一种，ionic平台自身会提供几种项目模板
  #进入到项目
  cd ./myNewProject
  #2.添加平台
  ionic cordova platform add android #平台同上一样可选
  #3.打包
  ionic cordova build android
  #4.运行
  ionic serve #运行在浏览器
  ionic cordova run android #运行到android,ios
  ```

- 可能会出现的问题

  创建项目时，会问你是否使用ionic4创建项目，选择n即可，也可创建尝试下，但运行性项目到android模拟器显示空白。

- 参考学习

  菜鸟教程：http://www.runoob.com/ionic/ionic-tutorial.html

  中文文档：http://www.ionic.wang/js_doc-index.html


![](C:\Users\liukai@acoinfo.co\Documents\mdNote\appSysImg\g.png)

- 参考学习

  w3school:https://www.w3cschool.cn/cuhkj/cuhkj-lf832655.html
#3.可能会遇到的问题
1.	node.js 使用免安装版能避免许多未知错误。
2.	cordova安装6.0.0不要安装7.0或8.0的，后面出现的错误会很多。
3.	cordova添加android平台，会多次失败，由于资源在国外会失败多次，插件下载完成就可以成功。
4.	安卓模拟器需要cpu加速器，不然会一直黑屏，所以选用了第三方模拟器。
5.	谷歌真机调试chrome://inspect。
6.	android9.0不能用使用。
