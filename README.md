![Stage: Stable Release](https://img.shields.io/badge/stage-stable-green)
![Release](https://img.shields.io/github/v/release/auth0/auth0-angular)
[![CircleCI](https://img.shields.io/circleci/build/github/auth0/auth0-angular)](https://circleci.com/gh/auth0/auth0-angular)
[![Codecov](https://img.shields.io/codecov/c/github/auth0/auth0-angular)](https://codecov.io/gh/auth0/auth0-angular)
![Downloads](https://img.shields.io/npm/dw/@auth0/auth0-angular)

# Auth0 Angular SDK

A library for integrating [Auth0](https://auth0.com) into an Angular 9+ application.

## Table of Contents

- [Documentation](#documentation)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [Angular Universal](#angular-universal)
- [Development](#development)
- [Contributing](#contributing)
- [Support + Feedback](#support--feedback)
- [Vulnerability Reporting](#vulnerability-reporting)
- [What is Auth0](#what-is-auth0)
- [License](#license)

## Documentation

- [API Reference](https://auth0.github.io/auth0-angular/)
- [Quickstart Guide](https://auth0.com/docs/quickstart/spa/angular-next)

## Installation

Using npm:

```sh
npm install @auth0/auth0-angular
```

We also have `ng-add` support, so the library can also be installed using the Angular CLI:

```sh
ng add @auth0/auth0-angular
```

## Getting Started

- [Register the authentication module](#register-the-authentication-module)
- [Add login to your application](#add-login-to-your-application)
- [Add logout to your application](#add-logout-to-your-application)
- [Display the user profile](#display-the-user-profile)
- [Protect a route](#protect-a-route)
- [Call an API](#call-an-api)

### Register the authentication module

Install the SDK into your application by importing `AuthModule` and configuring with your Auth0 domain and client ID:

```js
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';

// Import the module from the SDK
import { AuthModule } from '@auth0/auth0-angular';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,

    // Import the module into the application, with configuration
    AuthModule.forRoot({
      domain: 'YOUR_AUTH0_DOMAIN',
      clientId: 'YOUR_AUTH0_CLIENT_ID',
    }),
  ],

  bootstrap: [AppComponent],
})
export class AppModule {}
```

### Add login to your application

Next, inject the `AuthService` service into a component where you intend to provide the functionality to log in, by adding the `AuthService` type to your constructor. Then, provide a `loginWithRedirect()` method and call `this.auth.loginWithRedirect()` to log the user into the application.

```js
import { Component } from '@angular/core';

// Import the AuthService type from the SDK
import { AuthService } from '@auth0/auth0-angular';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent {
  title = 'My App';

  // Inject the authentication service into your component through the constructor
  constructor(public auth: AuthService) {}

  loginWithRedirect(): void {
    // Call this to redirect the user to the login page
    this.auth.loginWithRedirect();
  }
}
```

By default the application will ask Auth0 will redirect back to the root URL of your application after authentication, but this can be configured by setting the [`redirectUri` option](https://auth0.github.io/auth0-angular/interfaces/authconfig.html#redirecturi).

On your template, provide a button that will allow the user to log in to the application. Use the `isAuthenticated$` observable to check that the user is not already authenticated:

```html
<button
  *ngIf="(auth.isAuthenticated$ | async) === false"
  (click)="loginWithRedirect()"
>
  Log in
</button>
```

### Add logout to your application

Add a `logout` method to your component and call the SDK's `logout` method:

```js
logout(): void {
  // Call this to log the user out of the application
  this.auth.logout({ returnTo: window.location.origin });
}
```

Then on your component's template, add a button that will log the user out of the application. Use the `isAuthenticated$` observable to check that the user has already been authenticated:

```html
<button *ngIf="auth.isAuthenticated$ | async" (click)="logout()">
  Log out
</button>
```

### Display the user profile

Access the `user$` observable on the `AuthService` instance to retrieve the user profile. This observable already heeds the `isAuthenticated$` observable, so you do not need to check if the user is authenticated before using it:

```html
<ul *ngIf="auth.user$ | async as user">
  <li>{{ user.name }}</li>
  <li>{{ user.email }}</li>
</ul>
```

### Handle errors

Errors in the login flow can be captured by subscribing to the `error$` observable:

```js
authService.error$.subscribe((error) => console.log(error));
```

### Protect a route

To ensure that a route can only be visited by authenticated users, add the built-in `AuthGuard` type to the `canActivate` property on the route you wish to protect.

If an unauthenticated user tries to access this route, they will first be redirected to Auth0 to log in before returning to the URL they tried to get to before login:

```js
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { HomeComponent } from './unprotected/unprotected.component';
import { ProtectedComponent } from './protected/protected.component';

// Import the authentication guard
import { AuthGuard } from '@auth0/auth0-angular';

const routes: Routes = [
  {
    path: 'protected',
    component: ProtectedComponent,

    // Protect a route by registering the auth guard in the `canActivate` hook
    canActivate: [AuthGuard],
  },
  {
    path: '',
    component: HomeComponent,
    pathMatch: 'full',
  },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

### Call an API

The SDK provides an `HttpInterceptor` that automatically attaches access tokens to outgoing requests when using the built-in `HttpClient`. However, you must provide configuration that tells the interceptor which requests to attach access tokens to.

#### Register AuthHttpInterceptor

First, register the interceptor with your application module, along with the `HttpClientModule`.

**Note:** We do not do this automatically for you as we want you to be explicit about including this interceptor. Also, you may want to chain this interceptor with others, making it hard for us to place it accurately.

```js
// Import the interceptor module and the Angular types you'll need
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { AuthHttpInterceptor } from '@auth0/auth0-angular';

// Register the interceptor with your app module in the `providers` array
@NgModule({
  declarations: [],
  imports: [
    BrowserModule,
    HttpClientModule,     // Register this so that you can make API calls using HttpClient
    AppRoutingModule,
    AuthModule.forRoot(...),
  ],
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthHttpInterceptor, multi: true },
  ],
  bootstrap: [AppComponent],
})
```

#### Configure AuthHttpInterceptor to attach access tokens

Next, tell the SDK which requests to attach access tokens to in the SDK configuration. These are matched on the URL by using a string, a regex, or more complex object that also allows you to specify the configuration for fetching tokens by setting the `tokenOptions` property.

If an HTTP call is made using `HttpClient` and there is no match in this configuration for that URL, then the interceptor will simply be bypassed and the call will be executed without a token attached in the `Authorization` header.

**Note:** We do this to help prevent tokens being unintentionally attached to requests to the wrong recipient, which is a serious security issue. Those recipients could then use that token to call the API as if it were your application.

Here are some examples:

```js
import { HttpMethod } from '@auth0/auth0-angular';

// Modify your existing SDK configuration to include the httpInterceptor config
AuthModule.forRoot({
  domain: 'YOUR_AUTH0_DOMAIN',
  clientId: 'YOUR_AUTH0_CLIENT_ID',
  redirectUri: window.location.origin,

  // The AuthHttpInterceptor configuration
  httpInterceptor: {
    allowedList: [
      // Attach access tokens to any calls to '/api' (exact match)
      '/api',

      // Attach access tokens to any calls that start with '/api/'
      '/api/*',

      // Match anything starting with /api/accounts, but also specify the audience and scope the attached
      // access token must have
      {
        uri: '/api/accounts/*',
        tokenOptions: {
          audience: 'http://my-api/',
          scope: 'read:accounts',
        },
      },

      // Matching on HTTP method
      {
        uri: '/api/orders',
        httpMethod: HttpMethod.Post,
        tokenOptions: {
          audience: 'http://my-api/',
          scope: 'write:orders',
        },
      },

      // Using an absolute URI
      {
        uri: 'https://your-domain.auth0.com/api/v2/users',
        tokenOptions: {
          audience: 'https://your-domain.com/api/v2/',
          scope: 'read:users',
        },
      },
    ],
  },
});
```

> Under the hood, `tokenOptions` is passed as-is to [the `getTokenSilently` method](https://auth0.github.io/auth0-spa-js/classes/auth0client.html#gettokensilently) on the underlying SDK, so all the same options apply here.

#### Use HttpClient to make an API call

Finally, make your API call using the `HttpClient`. Access tokens are then attached automatically in the `Authorization` header:

```js
export class MyComponent {
  constructor(private http: HttpClient) {}

  callApi(): void {
    this.http.get('/api').subscribe(result => console.log(result));
  }
}
```

## Angular Universal

This library makes use of the `window` object in a couple of places during initialization, as well as `sessionStorage` in the underlying Auth0 SPA SDK, and thus [will have problems](https://github.com/angular/universal/blob/master/docs/gotchas.md#window-is-not-defined) when being used in an Angular Universal project. The recommendation currently is to only import this library into a module that is to be used in the browser, and omit it from any module that is to participate in a server-side environment.

See [Guards, and creating separate modules](https://github.com/angular/universal/blob/master/docs/gotchas.md#strategy-2-guards) in the Angular Universal "Gotchas" document.

## Development

### Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory. Use the `--prod` flag for a production build.

### Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

### Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via [Protractor](http://www.protractortest.org/).

### Running the playground app

The workspace includes a playground application that can be used to test out features of the SDK. Run this using `ng serve playground` and browse to http://localhost:4200.

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI README](https://github.com/angular/angular-cli/blob/master/README.md).

## Contributing

We appreciate feedback and contribution to this repo! Before you get started, please see the following:

- [Auth0's general contribution guidelines](https://github.com/auth0/open-source-template/blob/master/GENERAL-CONTRIBUTING.md)
- [Auth0's code of conduct guidelines](https://github.com/auth0/open-source-template/blob/master/CODE-OF-CONDUCT.md)

## Support + Feedback

For support or to provide feedback, please [raise an issue on our issue tracker](https://github.com/auth0/auth0-angular/issues).

## Vulnerability Reporting

Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/responsible-disclosure-policy) details the procedure for disclosing security issues.

## What is Auth0?

Auth0 helps you to easily:

- Implement authentication with multiple identity providers, including social (e.g., Google, Facebook, Microsoft, LinkedIn, GitHub, Twitter, etc), or enterprise (e.g., Windows Azure AD, Google Apps, Active Directory, ADFS, SAML, etc.)
- Log in users with username/password databases, passwordless, or multi-factor authentication
- Link multiple user accounts together
- Generate signed JSON Web Tokens to authorize your API calls and flow the user identity securely
- Access demographics and analytics detailing how, when, and where users are logging in
- Enrich user profiles from other data sources using customizable JavaScript rules

[Why Auth0?](https://auth0.com/why-auth0)

## License

This project is licensed under the MIT license. See the [LICENSE](https://github.com/auth0/auth0-angular/blob/master/LICENSE) file for more info.
