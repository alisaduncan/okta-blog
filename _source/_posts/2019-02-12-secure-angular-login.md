---
disqus_thread_id: 7225682591
discourse_topic_id: 16996
discourse_comment_url: https://devforum.okta.com/t/16996
layout: blog_post
title: "Build Secure Login for Your Angular App"
author: holger-schmitz
by: contractor
communities: [javascript]
description: "Build an Angular application with authentication, a step-by-step tutorial."
tags: [angular, login, angular-cli, typescript, authentication]
tweets:
- "Learn how to build an Angular application with authentication in this step-by-step tutorial. #angular"
- "Angular is awesome! And you'll probably need to secure it. This tutorial shows you how to add a secure login to your app! #angular #authentication"
- "Have you played with the latest release of @angular? Now's your chance!"
image: blog/featured/okta-angular-bottle-headphones.jpg
type: conversion
changelog: 
- 2020-03-30: Updated to use Angular 11, the Okta Angular SDK 3.0.1, and the Okta Sign-In Widget 5.5.0. You can see the changes in the [example app on GitHub](https://github.com/oktadeveloper/okta-angular-calculator-example/pull/1). Changes to this article can be viewed in [oktadeveloper/okta-blog#610](https://github.com/oktadeveloper/okta-blog/pull/610).
---

Single page applications (SPAs) are becoming more and more popular. Their appeal is obvious. Fast loading times gives users the feeling of responsiveness even over slow networks. At some point, a developer of a SPA has to think about authentication and authorization. But what do these two terms actually mean? Authentication deals with ensuring that a user truly is who they claim to be. This usually involves a login page in which the user provides their credentials. Once logged in, authorization deals with restricting and granting access to specific resources. In the simplest case, access to pages is restricted to users who have authenticated themselves.

In this tutorial, I will show you how to implement secure login in a client application using Angular. Okta provides Angular-specific libraries that make it very easy to include authentication and access control.

**Table of Contents**{: .hide }
* Table of Contents
{:toc}

{% include integrator-org-warning.html %}

## Build an Angular SPA with Login

In this tutorial, I will focus purely on client-side security. I will not delve into the topic of server-side authentication or authorization. The application you will be implementing is a simple server-less online calculator. Access to the calculator will be restricted users which have logged in. Naturally, real-life applications will communicate with the server and authenticate themselves with the server to gain access to restricted resources.

I will assume that you have installed Node on your system and that you are somewhat familiar with the node packet manager `npm`. The tutorial will be using Angular 11. To install the Angular command line tool, open a terminal and enter the command:

```bash
npm install -g @angular/cli@11
```

This will install the global `ng` command. On some systems, Node installs global commands in a directory that is not writable by regular users. In this case, you have to run the command above using `sudo`. Next, create a new Angular application. Navigate to a directory of your choice and issue the following command.

```bash
ng new AngularCalculator
```

This starts a wizard that will walk you through creating a new application. The wizard will prompt you with three questions. When asked about stricter type checking, answer **Yes**. When asked whether to add Angular routing in your application, answer with **Yes**. Next, you are given the choice of the CSS technology. Choose plain **CSS** here since the application you will be developing requires only little styling. The next step is to install some libraries that will make it easier to create a pleasant responsive design. Change into the `AngularCalculator` directory that you just created and run the command:

```bash
ng add @angular/material@11
```

This will prompt you with a few options. In the first question you can choose the color theme. I chose **Deep Purple/Amber**. For the next two questions, answer **Yes** to both using typography styles and browser animations. On top of Material Design, I will be using Flex Layout components. Run the command:

```bash
npm install @angular/flex-layout@11.0.0-beta.33
```

Next, add some general styling to the application. Open `src/style.css` and replace the contents with the following.

```css
body {
  margin: 0;
  font-family: sans-serif;
}

h1, h2 {
  text-align: center;
}
```

The next step is to make the Material Design components available inside the Angular application. Replace the contents of the file `src/app/app.module.ts` with the following code.

```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpClientModule } from '@angular/common/http';

import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { FlexLayoutModule } from '@angular/flex-layout';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatMenuModule } from '@angular/material/menu';
import { MatIconModule } from '@angular/material/icon';
import { MatCardModule } from '@angular/material/card';
import { MatButtonModule } from '@angular/material/button';
import { MatTableModule } from '@angular/material/table';
import { MatDividerModule } from '@angular/material/divider';

@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    BrowserAnimationsModule,
    FlexLayoutModule,
    MatToolbarModule,
    MatMenuModule,
    MatIconModule,
    MatCardModule,
    MatButtonModule,
    MatTableModule,
    MatDividerModule,
    AppRoutingModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

The file `src/app/app.component.html` contains the template for the main application component. This component acts as a container for the complete application and all its components. I like to create a basic toolbar layout in this component. Open the file and replace the content with the following.

{% raw %}
```html
<mat-toolbar color="primary" class="expanded-toolbar">
  <span>
    <button mat-button routerLink="/">{{title}}</button>
    <button mat-button routerLink="/"><mat-icon>home</mat-icon></button>
  </span>
  <button mat-button routerLink="/calculator"><mat-icon>dialpad</mat-icon></button>
</mat-toolbar>
<router-outlet></router-outlet>
```
{% endraw %}

To add some styling, open `src/app/app.component.css` and add the following lines.

```css
.expanded-toolbar {
  justify-content: space-between;
  align-items: center;
}
```

The `<router-outlet>` tag in the HTML template acts as a placeholder for the components that are managed by the router. Create these components next. The application will be made up of two views. The home view shows a simple splash screen containing information about the application. The calculator component contains the actual calculator. To create the components for these views, open the terminal again in the application's main directory and run the `ng generate` command for each.

```bash
ng generate component home
ng generate component calculator
```

This will create two directories under `src/app`, one for each component. You will only add two simple headers to the splash screen. Open `src/app/home/home.component.html` and replace the content with the following.

```html
<h1>Angular Calculator</h1>
<h2>A simple calculator with login</h2>
```

The calculator component contains the main meat of the application. Start by creating the layout template for the calculator's buttons and display in `src/app/calculator/calculator.component.html`.

{% raw %}
```html
<h1 class="h1">Calculator</h1>
<div fxLayout="row" fxLayout.xs="column" fxLayoutAlign="center" class="products">
  <mat-card class="mat-elevation-z1 calculator">
    <p class="display">{{display}}</p>
    <div>
      <button mat-raised-button color="warn" (click)="acPressed()">AC</button>
      <button mat-raised-button color="warn" (click)="cePressed()">CE</button>
      <button mat-raised-button color="primary" (click)="percentPressed()">%</button>
      <button mat-raised-button color="primary" (click)="operatorPressed('/')">÷</button>
    </div>
    <div>
      <button mat-raised-button (click)="numberPressed('7')">7</button>
      <button mat-raised-button (click)="numberPressed('8')">8</button>
      <button mat-raised-button (click)="numberPressed('9')">9</button>
      <button mat-raised-button color="primary" (click)="operatorPressed('*')">x</button>
    </div>
    <div>
      <button mat-raised-button (click)="numberPressed('4')">4</button>
      <button mat-raised-button (click)="numberPressed('5')">5</button>
      <button mat-raised-button (click)="numberPressed('6')">6</button>
      <button mat-raised-button color="primary" (click)="operatorPressed('-')">-</button>
    </div>
    <div>
      <button mat-raised-button (click)="numberPressed('1')">1</button>
      <button mat-raised-button (click)="numberPressed('2')">2</button>
      <button mat-raised-button (click)="numberPressed('3')">3</button>
      <button mat-raised-button color="primary" class="tall" (click)="operatorPressed('+')">+</button>
    </div>
    <div>
      <button mat-raised-button (click)="numberPressed('0')">0</button>
      <button mat-raised-button (click)="numberPressed('.')">.</button>
      <button mat-raised-button color="primary" (click)="equalPressed()">=</button>
    </div>
  </mat-card>
</div>
```
{% endraw %}

You will notice the `(click)` property on the buttons. This property allows you to specify the member functions of the component's class that will be called when the button is clicked. Before you get around to implementing that class, add a little bit of styling in `src/app/calculator/calculator.component.css`.

```css
.display {
  background-color: #f8f8f8;
  line-height: 24px;
  padding: 5px 8px;
}

.calculator button {
  margin: 5px;
  width: 64px;
}

.calculator button.tall {
  float: right;
  height: 82px;
}
```

The component's class lives in the file `src/app/calculator/calculator.component.ts`. Open this file and replace its contents with the following code.

```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-calculator',
  templateUrl: './calculator.component.html',
  styleUrls: ['./calculator.component.css']
})
export class CalculatorComponent implements OnInit {
  private stack: (number | string)[] = [];
  display = '';

  constructor() { }

  ngOnInit(): void {
    this.display = '0';
    this.stack = ['='];
  }

  numberPressed(val: string): void {
    if (typeof this.stack[this.stack.length - 1] !== 'number') {
      this.display = val;
      this.stack.push(parseInt(this.display, 10));
    } else {
      this.display += val;
      this.stack[this.stack.length - 1] = parseInt(this.display, 10);
    }
  }

  operatorPressed(val: string): void {
    const precedenceMap: {[index: string]: any} = {'+': 0, '-': 0, '*': 1, '/': 1};
    this.ensureNumber();
    const precedence = precedenceMap[val];
    let reduce = true;
    while (reduce) {
      let i = this.stack.length - 1;
      let lastPrecedence = 100;

      while (i >= 0) {
        if (typeof this.stack[i] === 'string') {
          lastPrecedence = precedenceMap[this.stack[i]];
          break;
        }
        i--;
      }
      if (precedence <= lastPrecedence) {
        reduce = this.reduceLast();
      } else {
        reduce = false;
      }
    }

    this.stack.push(val);
  }

  equalPressed(): void {
    this.ensureNumber();
    while (this.reduceLast()) {}
    this.stack.pop();
  }

  percentPressed(): void {
    this.ensureNumber();
    while (this.reduceLast()) {}
    const result = this.stack.pop() as number / 100;
    this.display = result.toString(10);
  }

  acPressed(): void {
    this.stack = ['='];
    this.display = '0';
  }

  cePressed(): void {
    if (typeof this.stack[this.stack.length - 1] === 'number') { this.stack.pop(); }
    this.display = '0';
  }

  private ensureNumber(): void {
    if (typeof this.stack[this.stack.length - 1] === 'string') {
      this.stack.push(parseInt(this.display, 10));
    }
  }

  private reduceLast(): boolean {
    if (this.stack.length < 4) { return false; }
    const num2 = this.stack.pop() as number;
    const op = this.stack.pop() as string;
    const num1 = this.stack.pop() as number;
    let result = num1;
    switch (op) {
      case '+': result = num1 + num2;
                break;
      case '-': result = num1 - num2;
                break;
      case '*': result = num1 * num2;
                break;
      case '/': result = num1 / num2;
                break;
    }
    this.stack.push(result);
    this.display = result.toString(10);
    return true;
  }
}
```

This code contains a complete calculator. See how the callback function for the buttons in the HTML template has been implemented as member functions. The calculator knows the for basic operations `+`, `-`, `*`, and `/` and it is aware of operator precedence. I will not go into the details of how this is being achieved. I'll leave this for you to figure out as an exercise.

Before you can start testing the application, you need to register the new components with the router. Open up `src/app/app-routing.module.ts` and edit its contents to match the following.

```ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { HomeComponent } from './home/home.component';
import { CalculatorComponent } from './calculator/calculator.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'calculator', component: CalculatorComponent },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

The demo application is now complete and you can fire up the server. The Angular command line tool comes with a development server that is ideal for testing the application. It will automatically cause the browser to reload the application whenever you make any changes to the code. To start the development server, simply run the following command.

```bash
ng serve
```

Open up your browser, navigate to `http://localhost:4200`, and click on the calculator icon in the top right corner. You should see something like the image below.

{% img blog/angular-login/angular-calculator.png alt:"Calculator screenshot" width:"800" %}{: .center-image }

## Add Authentication to Your Angular App

In this section, I will show you how to add authentication to your application. Okta provides a simple solution to secure authentication with easy integration into Angular applications. The ready-made route guard lets you restrict access to selected routes simply by dropping it into the route specification. The application flow is as follows. Whenever a user requests a protected resource, the route guard will check if the user is logged in. 

If the user is not logged in the guard will redirect the user to a hosted login page on the Okta servers. Alternatively, the user may opt to click on a login link directly. In this case, the authentication service will redirect the user to the login page. Once the user is logged in, the login page will redirect the user back to a special route, usually called `/callback`, in the application. This route is managed by the `OktaCallbackComponent`. The callback component will decide where to redirect the user, depending on the original request and on the authentication status of the user.

{% include setup/cli.md type="spa" framework="Angular" loginRedirectUri="http://localhost:4200/callback" %}

To start implementing authentication in your application, you need to install the Okta Angular library. Open the terminal in the application directory and run the command:

```bash
npm install @okta/okta-angular@3.0.1
```

Open `src/app/app.module.ts` and add the following imports and Okta configuration. The `{yourOktaDomain}` and `{yourClientId}` placeholders should be replaced with the issuer and client ID that you obtained earlier.

```ts
import { OKTA_CONFIG, OktaAuthModule } from '@okta/okta-angular';
import { OktaAuthOptions } from '@okta/okta-auth-js';

const oktaConfig: OktaAuthOptions = {
  issuer: 'https://{yourOktaDomain}/oauth2/default',
  clientId: '{clientId}',
  redirectUri: window.location.origin + '/callback'
};
```

In the `imports` section of the `@NgModule` annotation, add `OktaAuthModule`. Provide `OKTA_CONFIG` with your Okta values too.

```ts
@NgModule({
  ...
  imports: [
    ...
    OktaAuthModule
  ],
  providers: [{ provide: OKTA_CONFIG, useValue: oktaConfig }]
})
```
 
Open `src/app/app.component.ts` and replace the contents of the file with the following.

```ts
import { Component, OnInit } from '@angular/core';
import { OktaAuthService } from '@okta/okta-angular';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  title = 'AngularCalculator';
  isAuthenticated = false;

  constructor(public oktaAuth: OktaAuthService) {
    // subscribe to authentication state changes
    this.oktaAuth.$authenticationState.subscribe(
      (isAuthenticated: boolean) => this.isAuthenticated = isAuthenticated
    );
  }

  async ngOnInit(): Promise<void> {
    // get authentication state for immediate use
    this.isAuthenticated = await this.oktaAuth.isAuthenticated();
  }

  async login(): Promise<void> {
    await this.oktaAuth.signInWithRedirect();
  }

  async logout(): Promise<void> {
    await this.oktaAuth.signOut();
  }
}
```

The `OktaAuthService` is injected into the main application component. The main component contains a flag `isAuthenticated` that keeps track of the authentication status of the user. By subscribing to the `$authenticationState` observable this flag is kept up-to-date whenever the status changes. The flag is initialized in the `ngOnInit` function. The `login` member function simply calls `OktaAuthService.signInWithRedirect()` which redirects the user to the hosted login page. Similarly, the `logout` member function calls `OktaAuthService.signOut()` which erases any user tokens and redirects the user to the main route.

Next, open `src/app/app.component.html` and add the following code into the `<mat-toolbar>` after the closing tag of the first `<span>` element.

```html
<span>
  <button mat-button *ngIf="!isAuthenticated" (click)="login()"> Login </button>
  <button mat-button *ngIf="isAuthenticated" (click)="logout()"> Logout </button>
</span>
```

By making use of the `isAuthenticated` flag, either a `Login` or a `Logout` button is shown. Each button calls the respective method of the application component to log the user in or out. In the final step, you need to modify the router settings. Open `src/app/app-routing.module.ts` and add the following import to the top of the file.

```ts
import { OktaCallbackComponent, OktaAuthGuard } from '@okta/okta-angular';
```

In the code above and in the Okta dashboard settings, you have specified that `/callback` route should handle the login callback. To register the `OktaCallbackComponent` with this route, add the following entry to the `routes` setting.

```ts
{ path: 'callback', component: OktaCallbackComponent }
```

The `OktaAuthGuard` can be used to restrict access to any protected routes. To protect the `calculator` route, modify its entry by adding a `canActivate` property in the following way.

```ts
{ path: 'calculator', component: CalculatorComponent, canActivate: [OktaAuthGuard] }
```

Now, if you try to access the calculator in the application, you will be redirected to the Okta login page. Only on successful login are you going to be redirected back to the calculator. The splash screen, on the other hand, is accessible without any authentication.

## Add Angular Authentication with the Login Widget

Redirecting the user to an external login page is OK for some use cases. In other cases, you don't want the user to leave your site. This is a use case for the login widget. It lets you embed the login form directly into your application. To make use of the widget you first have to install it. Open the terminal in the application directory and install the following packages.

```bash
npm install @okta/okta-signin-widget@5.5.0
```

Next, generate a component that hosts the login form. This component will not need any additional styling and the HTML template consists only of a single tag that can be inlined in the component definition.

```bash
ng generate component login --inline-style=true --inline-template=true
```

Open the newly created `src/app/login/login.component.ts` and paste the following contents into it. Make sure to replace the client ID and Okta domain placeholders with your values.

```ts
import { Component, OnInit } from '@angular/core';
import { Router, NavigationStart } from '@angular/router';

import { OktaAuthService } from '@okta/okta-angular';
import { Tokens } from '@okta/okta-auth-js';
import OktaSignIn from '@okta/okta-signin-widget';

const DEFAULT_ORIGINAL_URI = window.location.origin;

@Component({
  selector: 'app-login',
  template: `
    <div id="okta-signin-container"></div>`,
  styles: []
})
export class LoginComponent implements OnInit {
  widget = new OktaSignIn({
    baseUrl: 'https://{yourOktaDomain}',
    clientId: '{yourClientId}',
    redirectUri: DEFAULT_ORIGINAL_URI + '/callback'
  });

  constructor(private oktaAuth: OktaAuthService, router: Router) {
    // Show the widget when prompted, otherwise remove it from the DOM.
    router.events.forEach(event => {
      if (event instanceof NavigationStart) {
        switch (event.url) {
          case '/login':
          case '/calculator':
            break;
          default:
            this.widget.remove();
            break;
        }
      }
    });
  }

  ngOnInit(): void {
    this.widget.showSignInToGetTokens({
      el: '#okta-signin-container'
    }).then(async (tokens: Tokens | undefined) => {
      const originalUri = this.oktaAuth.getOriginalUri();
      if (originalUri === DEFAULT_ORIGINAL_URI) {
        this.oktaAuth.setOriginalUri('/');
      }

      // Remove the widget
      this.widget.remove();

      // In this flow the redirect to Okta occurs in a hidden iframe
      await this.oktaAuth.handleLoginRedirect(tokens);
    }).catch((err: any) => {
      // Typically due to misconfiguration
      throw err;
    });
  }
}
```

**NOTE**: You will get [an error](https://github.com/okta/okta-angular/issues/29) about `import OktaSignIn`. To fix it, create a `src/sign-in-widget.d.ts` with the following code.

```ts
declare module '@okta/okta-signin-widget';
```

In `src/index.html` add the following line inside the `<head>` tag to include the default widget styles.

```html
<link href="https://global.oktacdn.com/okta-signin-widget/5.5.0/css/okta-sign-in.min.css" type="text/css" rel="stylesheet"/>
```

Now add the login route to the route configuration. Open `src/app/app-routing.module.ts`. Add an import for the `LoginComponent` at the top of the file.

```ts
import { LoginComponent } from './login/login.component';
```

Next, add a function that tells the router what to do when the user is required to log in.

```ts
export function onAuthRequired(oktaAuth: OktaAuthService, injector: Injector): void {
  const router = injector.get(Router);
  router.navigate(['/login']);
}
```

You will need to import `OktaAuthService` and `Injector` for this to compile.

```ts
import { Injector, NgModule } from '@angular/core';
import { OktaAuthGuard, OktaAuthService, OktaCallbackComponent } from '@okta/okta-angular';
```

Make sure that the function is exported. In the `routes` specification, add the route for the login component.

```ts
{ path: 'login', component: LoginComponent }
```

Finally, modify the specification for the `calculator` route to include a reference to the `onAuthRequired()` function.

```ts
{
  path: 'calculator',
  component: CalculatorComponent,
  canActivate: [OktaAuthGuard],
  data: { onAuthRequired }
}
```

The next step is to make sure that the user is directed to the `login` when the login button in the top bar is pressed. Open `src/app/app.component.html` and change the line containing the login button to the following.

```html
<button mat-button *ngIf="!isAuthenticated" routerLink="/login"> Login </button>
```

You can also remove the `login()` function in `src/app/app.component.ts` as it is no longer needed.

That's it! Your application now hosts its own login form powered by Okta. Below is a screenshot of what the login widget might look like.

{% img blog/angular-login/angular-login.png alt:"Angular Calculator with Sign-In Widget" width:"800" %}{: .center-image }

## Learn More About Building Secure Login and Registration in Angular

In this tutorial, I showed you how to implement authentication and basic authorization in a single page application based on Angular. You have the choice between a hosted login page and a login widget embedded in your application. The hosted login is ideal when you know that there are multiple applications linked to a single user account. In this case, the hosted solution conveys the idea that the user is logging on to all applications in one central location. The login widget is the ideal solution when you want to provide a seamless experience in a single branded application.

Below are some links where you can find out more about single page applications, Angular, and authentication.

* [Check out how to add authentication to any web page](/blog/2018/06/08/add-authentication-to-any-web-page-in-10-minutes)
* [Learn more about what's new in Angular 7](/blog/2018/12/04/angular-7-oidc-oauth2-pkce)
* [See how to add a CRUD server to your Angular application](/blog/2018/10/30/basic-crud-angular-and-node)
* [Find out how to turn your Angular application into a Progressive Web Application](/blog/2019/01/30/first-angular-pwa)

The code for this tutorial can be found on GitHub at [oktadeveloper/okta-angular-calculator-example](https://github.com/oktadeveloper/okta-angular-calculator-example).

Did you like this tutorial? For more cool stuff, follow us on Twitter [@oktadev](https://twitter.com/OktaDev), [Facebook](https://www.facebook.com/oktadevelopers/), and [LinkedIn](https://www.linkedin.com/company/oktadev/)!

