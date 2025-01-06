# sso-js-sdk

This is SSO's SDK for js will allow you to easily connect your application to the SSO Imigrasi authentication system without having to implement it from scratch.

SSO SDK is very simple to use. We will show you the steps below.

## Init SDK

Initialization requires 5 parameters, which are all string type:

| Name (in order)  | Must | Description                                                                                    |
|------------------|------|------------------------------------------------------------------------------------------------|
| serverUrl        | Yes  | your sso server URL                                                                        |
| clientId         | Yes  | the Client ID of your sso application                                                      |
| appName          | Yes  | the name of your sso application                                                           |
| organizationName | Yes  | the name of the sso organization connected with your sso application                   |
| redirectPath     | No   | the path of the redirect URL for your sso application, will be `/callback` if not provided |
| signinPath       | No   | the path of the signin URL for your sso application, will be `/api/signin` if not provided |

```typescript
import {SDK, SdkConfig} from 'sso-js-sdk'

const sdkConfig: SdkConfig = {
    serverUrl: "http://add0275a8800a486e93515678dd6b876-1265442895.ap-southeast-3.elb.amazonaws.com",
    clientId: "014ae4bd048734ca2dea",
    appName: "app-casnode",
    organizationName: "sso",
    redirectPath: "/callback",
    signinPath: "/api/signin",
}
const sdk = new SDK(sdkConfig)
// call sdk to handle
```

## API reference interface

#### Get sign up url

```typescript
getSignupUrl(enablePassword)
```

Return the sso url that navigates to the registration screen

#### Get sign in url

```typescript
getSigninUrl()
```

Return the sso url that navigates to the login screen

#### Get user profile page url

```typescript
getUserProfileUrl(userName, account)
```

Return the url to navigate to a specific user's sso personal page

#### Get my profile page url

```typescript
getMyProfileUrl(account)
```

#### Sign in

```typescript
signin(serverUrl, signinPath)
```

Handle the callback url from sso, call the back-end api to complete the login process

#### Determine whether silent sign-in is being used

```typescript
isSilentSigninRequested()
```

We usually use this method to determine if silent login is being used. By default, if the silentSignin parameter is included in the URL and equals one, this method will return true. Of course, you can also use any method you prefer.

#### silentSignin


````typescript
silentSignin(onSuccess, onFailure)
````

First, let's explain the two parameters of this method, which are the callback methods for successful and failed login. Next, I will describe the execution process of this method. We will create a hidden "iframe" element to redirect to the login page for authentication, thereby achieving the effect of silent sign-in.

#### popupSignin


````typescript
popupSignin(serverUrl, signinPath)
````
Popup a window to handle the callback url from sso, call the back-end api to complete the login process and store the token in localstorage, then reload the main window.

### OAuth2 PKCE flow sdk (for SPA without backend)

#### Start the authorization process

Typically, you just need to go to the authorization url to start the process. This example is something that might work in an SPA.

```typescript
signin_redirect();
```

You may add additional query parameters to the authorize url by using an optional second parameter:

```typescript
const additionalParams = {test_param: 'testing'};
signin_redirect(additionalParams);
```

#### Trade the code for a token

When you get back here, you need to exchange the code for a token.

```typescript
sdk.exchangeForAccessToken().then((resp) => {
    const token = resp.access_token;
    // Do stuff with the access token.
});
```

As with the authorizeUrl method, an optional second parameter may be passed to the exchangeForAccessToken method to send additional parameters to the request:

```typescript
const additionalParams = {test_param: 'testing'};

sdk.exchangeForAccessToken(additionalParams).then((resp) => {
    const token = resp.access_token;
    // Do stuff with the access token.
});
```

#### Parse the access token

Once you have an access token, you can parse it into JWT header and payload.

```typescript
const result = sdk.parseAccessToken(accessToken);
console.log("JWT algorithm: " + result.header.alg);
console.log("User organization: " + result.payload.owner);
console.log("User name: " + result.payload.name);
```

#### Get user info

Once you have an access token, you can use it to get user info.

```typescript
getUserInfo(accessToken).then((resp) => {
    const userInfo = resp;
    // Do stuff with the user info.
});
```

#### Refresh access token

You could use a refresh token, to get a new token from the oauth server when token expired.

```typescript
sdk.refreshAccessToken(refreshToken).then((resp) => {
    const token = resp.access_token;
    // Do stuff with new access token
});
```

#### A note on Storage
By default, this package will use sessionStorage to persist the pkce_state. On (mostly) mobile devices there's a higher chance users are returning in a different browser tab. E.g. they kick off in a WebView & get redirected to a new tab. The sessionStorage will be empty there.

In this case it you can opt in to use localStorage instead of sessionStorage:

```typescript
import {SDK, SdkConfig} from 'sso-js-sdk'

const sdkConfig = {
  // ...
  storage: localStorage, // any Storage object, sessionStorage (default) or localStorage
}

const sdk = new SDK(sdkConfig)
```
