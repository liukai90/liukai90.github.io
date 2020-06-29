## 测试

ng serve 即可在浏览器看到效果，但有时会通过http、webscoket、socketio等协议访问服务器，因此就会出现跨域问题。

解决跨域问题

配置一个代理文件proxy.config.json,配置文件名称无特定要求。

内容

```json
{
    "/": {
        //目标地址
        "target": "http://localhost:3000",
        "secure": "false"
    }
}
```

所在项目位置

![1593410363063](https://github.com/liukai90/liukai90.github.io/blob/master/cordova%2Bionic/img/1593410363063.png)

运行命令

```angular
ng serve  --proxy-config proxy.config.json
```



## 打包

测试打包用 ng build，正式需要打包ng build --prod 会启用aot优化以及其他文件的一些优化

详见

[构建说明](https://angular.cn/cli/build)

[angular.json配置说明]( https://angular.cn/guide/workspace-config )

关于ionic的图标问题，我的项目是基于angular+ionic，但是有个问题就是在项目里使用ionic的图标模块，所以导致了会把图标模块的所有图标都构建到包里，总共1200多小图标，但实际上我只用到了4个图标，我通过配置angular.json 文件解决了这个问题。

原配置

```json
{
                "glob": "**/*.svg",
                "input": "node_modules/ionicons/dist/ionicons/svg",
                "output": "./svg"
}
```

这种方式会将node_modules/ionicons/dist/ionicons/svg文件夹下的所有图标都打包进来。

改正后

```json
			{
                "glob": "**/bulb.svg",
                "input": "node_modules/ionicons/dist/ionicons/svg",
                "output": "./svg"
              },
              {
                "glob": "**/radio.svg",
                "input": "node_modules/ionicons/dist/ionicons/svg",
                "output": "./svg"
              },
              {
                "glob": "**/chevron-back.svg",
                "input": "node_modules/ionicons/dist/ionicons/svg",
                "output": "./svg"
              },
              {
                "glob": "**/chevron-forward.svg",
                "input": "node_modules/ionicons/dist/ionicons/svg",
                "output": "./svg"
              }
```

改正后只会构建几个命名的svg。

