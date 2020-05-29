# 3 - Generating components and Nx lib

##  1. Generate our first Nx lib

* Run the below command to see all the lib options

```bash
ng g lib --help
```

* Add a new lib called auth. We will not lazy load this lib as auth will always be used by our app

```text
ng g lib auth --routing --prefix=app --parent-module=apps/customer-portal/src/app/app.module.ts
```

* Inspect the index.ts file which is for exporting public api surface for the lib.
* Inspect angular.json libs array allow us to use --project flag with libs to get cli scaffolding and code generation.

{% hint style="info" %}
Currently issue with setting scss as default for lib. When ever you make a lib component please change style extention .scss
{% endhint %}

## 2. Add container and presentational components

Lets look at what this pattern is and what are the benefits in slides  
[https://docs.google.com/presentation/d/1xf8aPIvQjgjUVGH\_1sRkikvh5H73x2xvX7PnN4AjYt4/edit\#slide=id.g3bc936a676\_1\_4](https://docs.google.com/presentation/d/1xf8aPIvQjgjUVGH_1sRkikvh5H73x2xvX7PnN4AjYt4/edit#slide=id.g3bc936a676_1_4)

![Characteristics of Container and Presentational Components](.gitbook/assets/image%20%288%29.png)

* Add a new container component to the auth lib

```text
ng g c containers/login --project=auth
```

* Add a new presentational component to the auth lib

```text
ng g c components/login-form --project=auth
```

Note: You may decide not to make this a presentational component but it makes it easier to test and refactor.

{% hint style="info" %}
As this is a child route it is assumed that the consuming module will already prefix the route with "localhost:4200/auth" and when we say the path is "login" we mean relative to this so it will end up being "localhost:4200/auth/login".
{% endhint %}

## 3. Add a default route to the auth module

{% code title="libs/auth/src/auth.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './containers/login/login.component';
import { LoginFormComponent } from './components/login-form/login-form.component';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
@NgModule({
  imports: [CommonModule, RouterModule],
  declarations: [LoginComponent, LoginFormComponent]
})
export class AuthModule {}

```
{% endcode %}

## 4. Update the consuming Customer Portal App module

* Delete everything but the router-outlet on the apps app.component.html file.

{% code title="apps/customer-portal/src/app/app.components.html" %}
```typescript
<router-outlet></router-outlet>
```
{% endcode %}

* Add the Auth module to the main App Module

{% code title="apps/customer-portal/src/app/app.module.ts" %}
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';
import { NxModule } from '@nrwl/nx';
import { RouterModule } from '@angular/router';
import { authRoutes, AuthModule } from '@demo-app/auth';  //added

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    NxModule.forRoot(),
    RouterModule.forRoot([{path: 'auth', children: authRoutes}], { initialNavigation: 'enabled' }),
    AuthModule .     // added
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}

```
{% endcode %}

* Run the app `ng s` and navigate to [http://localhost:4200/auth/login](http://localhost:4200/auth/login)

![App running in the Browser](.gitbook/assets/image%20%2819%29.png)

## 4. Add presentational component to container component

* Add the presentational component to the container component.

{% code title="libs/auth/src/lib/containers/login/login.component.html" %}
```markup
<app-login-form (submit)="login($event)"></app-login-form>
```
{% endcode %}

* Add login function to container component class

{% code title="libs/auth/src/lib/containers/login/login.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {
  constructor() {}

  ngOnInit() {}

  login(authenticate:any) {
    console.log(authenticate);
  }
}
```
{% endcode %}

## 5. Add new folder for shared interfaces

* Make a folder called 'data-models' manually in the libs folder.

![data-models folder](.gitbook/assets/image%20%2811%29.png)

* Add a 'authenticate.d.ts' file to the folder and export the added data models from the index.ts file

{% code title="libs/data-models/src/authenticate.d.ts" %}
```typescript
export interface Authenticate {
    username: string;
    password: string;
}
```
{% endcode %}

* Export the interface

{% code title="libs/data-models/index.ts" %}
```typescript
export { Authenticate } from './authenticate';
```
{% endcode %}

* Add basic login form to presentational component

{% code title="libs/auth/src/lib/components/login-form/login-form.component.html" %}
```markup
<input #username placeholder="username" type="text">
<input #password placeholder="password" type="text">
<button (click)="login({username: username.value, password: password.value})">Login</button>
```
{% endcode %}

* Add an angular @Output to emit the event of a form submission

{% code title="libs/auth/src/lib/components/login-form/login-form.component.ts" %}
```typescript
import { Component, EventEmitter, Output } from '@angular/core';
import { Authenticate } from '@demo-app/data-models';
@Component({
  selector: 'app-login-form',
  templateUrl: './login-form.component.html',
  styleUrls: ['./login-form.component.scss']
})
export class LoginFormComponent {
  @Output() submit = new EventEmitter<Authenticate>();

  login(authenticate: Authenticate) {
    this.submit.emit(authenticate);
  }
}

```
{% endcode %}

## 6. Change the ChangeDetectionStrategy to OnPush

* Now that we are using the presentation and container component pattern and we know that we only need to check the child components for changes if a DOM event or a @Input or @Output passes new primitives or reference values. In this way we can tell Angular not check the whole component tree which can cause performance issues in larger applications.

{% code title="libs/auth/src/lib/containers/login/login.component.ts" %}
```typescript
import { Component, OnInit, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class LoginComponent implements OnInit {
  constructor() {}

  ngOnInit() {}

  login(authenticate: any) {
    console.log(authenticate);
  }
}

```
{% endcode %}

