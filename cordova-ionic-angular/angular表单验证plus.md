# 1.模板验证

```html
<!--
  Generated template for the LoginPage page.

  See http://ionicframework.com/docs/components/#navigation for more info on
  Ionic pages and navigation.
-->
<!--<ion-header>-->

  <!--<ion-navbar>-->
    <!--<ion-title>登录</ion-title>-->
  <!--</ion-navbar>-->

<!--</ion-header>-->
  <header [title]="title"></header>

  <ion-content scroll="false">
    <!--<form>-->
      <ion-item>
        <ion-input type="text" class="form-control"
                   name="username" #username="ngModel"
                   required maxlength="10" minlength="6"
                   placeholder="用户名"
                   [(ngModel)]="user.username"
                    ></ion-input>

      </ion-item>
      <p>ahdasidhasidashdudi</p>

      <ion-item>
        <ion-input type="password" class="form-control"
                   name="password" #password="ngModel"
                   required maxlength="16" minlength="6"
                   placeholder="密码" [(ngModel)]="user.password"></ion-input>
      </ion-item>

      <ion-item>
        <ion-label>记住密码</ion-label>
        <ion-toggle [(ngModel)]="pepperoni"></ion-toggle>
      </ion-item>

      <button ion-button block (click)="login()">登录</button>
      <ion-item>
        <button ion-button icon-start outline (click)="goRegistered()">
          去注册
        </button>

        <button ion-button icon-end outline>
          忘记密码
        </button>
      </ion-item>
      <h1 class="errorMessage">{{promptMessage}}</h1>
      <span *ngIf="username.invalid && (username.dirty || username.touched)"
            class="errorMessage">用户名必须为6到10位</span>
      <span *ngIf="password.invalid && (password.dirty || password.touched)" class="errorMessage">
        密码必须为6-16位
      </span>
    <!--</form>-->


  </ion-content>

```

# 2.响应式表单

```html
<!--
  Generated template for the LoginPage page.

  See http://ionicframework.com/docs/components/#navigation for more info on
  Ionic pages and navigation.
-->
<!--<ion-header>-->

  <!--<ion-navbar>-->
    <!--<ion-title>登录</ion-title>-->
  <!--</ion-navbar>-->

<!--</ion-header>-->
  <header [title]="title"></header>

  <ion-content scroll="false">
    <div>
      <ion-item [formGroup]="formGroup">
        <ion-input type="text"
                   class="form-control"
                   [formControl]="username"
                   placeholder="用户名"
                  required
                    ></ion-input>

      </ion-item>
      <p *ngIf="username.invalid && (username.dirty || username.touched)"
         class="error">用户名必须为6-10位</p>

      <ion-item [formGroup]="formGroup">
        <ion-input type="password"
                   class="form-control"
                   [formControl]="password"
                   placeholder="密码" required></ion-input>
      </ion-item>
      <p *ngIf="password.invalid && (password.dirty || password.touched)" class="error">
        密码必须为6-16位
      </p>

      <ion-item>
        <ion-label>记住密码</ion-label>
        <ion-toggle [(ngModel)]="pepperoni"></ion-toggle>
      </ion-item>

      <button ion-button block [disabled]="formGroup.invalid" (click)="login()" >登录</button>
      <ion-item>
        <button ion-button icon-start outline (click)="goRegistered()">
          去注册
        </button>

        <button ion-button icon-end outline>
          忘记密码
        </button>
      </ion-item>
      <h1 class="errorMessage">{{promptMessage}}</h1>
      <!--<span *ngIf="username.invalid && (username.dirty || username.touched)"-->
            <!--class="errorMessage">用户名必须为6到10位</span>-->
      <!--<span *ngIf="password.invalid && (password.dirty || password.touched)" class="errorMessage">-->
        <!---->
      <!--</span>-->
    </div>


  </ion-content>



```

ts

```ts
import { Component } from '@angular/core';
import { IonicPage, NavController, NavParams } from 'ionic-angular';
import { RegisteredPage} from "../registered/registered";
import { HttpClient,HttpHeaders } from "@angular/common/http";
import { TabsPage } from "../tabs/tabs";
import { FormControl, FormGroup, Validators,FormControlName } from "@angular/forms";
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

  formGroup:FormGroup

  title:string = '登录'
  promptMessage:string = ''

  private headers = new HttpHeaders({'Content-Type': 'application/json'});
  user={
    username:'',
    password:''
  }
  req={
    url:'http://localhost:8081/user/login',
    data:this.user,
  }





  constructor(public navCtrl: NavController, public navParams: NavParams,
              private http:HttpClient) {
    this.formGroup = new FormGroup(
      {
        'username':new FormControl(this.user.username,
          [
            Validators.required,
            Validators.minLength(6),
            Validators.maxLength(10),
          ]),
          'password':new FormControl(
            this.user.password,[
              Validators.required,
              Validators.minLength(6),
              Validators.maxLength(16)
            ]
          )
      }
    )

  }
  public get username(){
    return this.formGroup.get('username');
  }
  public get password(){
    return this.formGroup.get('password');
  }

  ionViewDidLoad() {
    console.log('ionViewDidLoad LoginPage');
  }
  login(){
    console.log(this.req);
    this.http.post(this.req.url,JSON.stringify(this.formGroup.value),{headers:this.headers}).
    subscribe((res:any)=>{
      console.log(res);
     if (res.code == 0){
       this.promptMessage = res.message;
       alert(res.message)
     } else {
       this.navCtrl.push(TabsPage);
     }

  });

  }
  goRegistered(){
    this.navCtrl.push(RegisteredPage);
  }

}

```

# 3.交叉验证