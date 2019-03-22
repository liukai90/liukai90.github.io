# 1.描述

无论是使用angularjs做前端或是结合ionic混合开发移动端开发app都需要与后台进行交互，而angular给我提供了httpModule模块供我们使用。今天就展现一个http的封装和使用的一个具体流程。

# 2. HttpModule引入

找到app.module.ts文件

```typescript
import { NgModule, ErrorHandler } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { IonicApp, IonicModule, IonicErrorHandler } from 'ionic-angular';
import { MyApp } from './app.component';


import { LoginPage } from "../pages/login/login";
/**
引入HttpClientModule模块
*/
import { HttpClientModule } from "@angular/common/http";

import { RequestServiceProvider } from "../providers/request-service/request-service";
import { StatusBar } from '@ionic-native/status-bar';
import { SplashScreen } from '@ionic-native/splash-screen';

@NgModule({
  declarations: [
    MyApp,
    
    LoginPage,
   
  ],
  imports: [
    BrowserModule,
      /**
      导入模块
      */
    HttpClientModule,
   
    IonicModule.forRoot(MyApp,{
      tabsHideOnSubPages:'true',
      backButtonText:''
    })
  ],
  bootstrap: [IonicApp],
  entryComponents: [
    MyApp,
   
    LoginPage,
    
  ],
  providers: [
    StatusBar,
    SplashScreen,
    {provide: ErrorHandler, useClass: IonicErrorHandler},
    RequestServiceProvider,
   
  ]
})
export class AppModule {}

```

按照自己的项目导入HttpClientModule模块即可，我导入其他组件，不用考虑。

# 3.创建服务

```typescript
ionic g provider RequestService
```

执行完成后则会出现如下文件

![图片描述][1]



# 4.封装服务

```typescript
/**
导入http相关
*/
import { HttpClient,HttpHeaders } from '@angular/common/http';
import { Injectable } from '@angular/core';
import {Observable} from "rxjs";

/*
  Generated class for the RequestServiceProvider provider.

  See https://angular.io/guide/dependency-injection for more info on providers
  and Angular DI.
*/
@Injectable()
export class RequestServiceProvider {

    /**
    讲基础路径提取说出来，配置ip和端口时只需要在这修改
    */
  //basePath:string='http://10.4.0.205:8081'
  reserveBasePath:string='http://10.6.254.110:8081'
  basePath=this.reserveBasePath;
    /**
    封装固定的消息头相关
    */
  private headers = new HttpHeaders({'Content-Type': 'application/json'})
  // private headers = new HttpHeaders({'Access-Control-Allow-Origin':'*'});

/**
初始化http变量
*/
  constructor(public http: HttpClient) {
    console.log('Hello RequestServiceProvider Provider');
  }

    /**
    给外界提供了四个基础的方法只需要传入uri和data即可
    */
  get(req:any):Observable<any> {
    return this.http.get(this.basePath+req.uri,{headers:this.headers});
  }

  post(req:any):Observable<any>{
    return this.http.post(this.basePath+req.uri,req.data,{headers:this.headers});
  }
  put(req:any):Observable<any>{
    return this.http.put(this.basePath+req.uri,req.data,{headers:this.headers});
  }
  delete(req:any):Observable<any>{
    return this.http.delete(this.basePath+req.uri,{headers:this.headers});
  }

}

```

# 5.导入声明封装服务

找到app.module.ts文件和第一部类似

```typescript
import { NgModule, ErrorHandler } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { IonicApp, IonicModule, IonicErrorHandler } from 'ionic-angular';
import { MyApp } from './app.component';


import { LoginPage } from "../pages/login/login";
/**
引入HttpClientModule模块
*/
import { HttpClientModule } from "@angular/common/http";

/**
导入自定的服务
*/
import { RequestServiceProvider } from "../providers/request-service/request-service";
import { StatusBar } from '@ionic-native/status-bar';
import { SplashScreen } from '@ionic-native/splash-screen';

@NgModule({
  declarations: [
    MyApp,
    
    LoginPage,
   
  ],
  imports: [
    BrowserModule,
      /**
      导入模块
      */
    HttpClientModule,
   
    IonicModule.forRoot(MyApp,{
      tabsHideOnSubPages:'true',
      backButtonText:''
    })
  ],
  bootstrap: [IonicApp],
  entryComponents: [
    MyApp,
   
    LoginPage,
    
  ],
  providers: [
    StatusBar,
    SplashScreen,
    {provide: ErrorHandler, useClass: IonicErrorHandler},
    /**
    声明服务
    */
    RequestServiceProvider,
   
  ]
})
export class AppModule {}

```

# 6.使用服务

找到自己的页面所对应的ts文件如下面代码一样

```typescript
import { Component } from '@angular/core';
import { IonicPage, NavController, NavParams } from 'ionic-angular';
/**
导入声明
*/
import {RequestServiceProvider} from "../../providers/request-service/request-service";

/**
 * Generated class for the LoginPage page.
 *
 * See https://ionicframework.com/docs/components/#navigation for more info on
 * Ionic pages and navigation.
 */

@IonicPage()
@Component({
  selector: 'page-login',
  templateUrl: 'login.html',
})
export class LoginPage {
  title:string = '登录'
  promptMessage:string = ''

  user={
    username:'',
    password:''
  }
  req={
    login:{
      uri:'/user/login'
    }

  }



  constructor(public navCtrl: NavController, public navParams: NavParams,
               /**
               初始化服务对象
               */
              private requestService:RequestServiceProvider) {

  }
  
  ionViewDidLoad() {
    console.log('ionViewDidLoad LoginPage');
  }
 login(){
  
     /**
     调用post方法，subscribe()方法可以出发请求，调用一次发送一次，调用多次发多次
     */
    this.requestService.post({uri:this.req.login.uri,data:user}).subscribe((res:any)=>{
      console.log(res);
      if (res.code == 0){
        this.promptMessage = res.message;
      } else {     
	    this.promptMessage = res.message;
      }

    },
      error1 => {
      alert(JSON.stringify(error1))
      });

  }

 
}

```


[1]: /img/bVblqCr