# vue3-google-oauth2
Handling Google sign-in and sign-out for Vue 3 applications.

**Note I:** This repository is a fork of the original [vue-google-oauth2](https://github.com/guruahn/vue-google-oauth2) 
and has been updated for Vue 3.

**Note II:** This repository is not yet backwards compatible with Vue 2 and does not yet support TypeScript.

**Note III:** The documentation below has not been updated from the original repository.

![npm bundle size](https://img.shields.io/bundlephobia/minzip/vue-google-oauth2.svg)
![GitHub](https://img.shields.io/github/license/guruahn/vue-google-oauth2.svg)
![vue-google-oauth2](https://img.shields.io/npm/dt/vue-google-oauth2.svg)

We support [TypeScript](https://www.typescriptlang.org/) and [Nuxt](https://ko.nuxtjs.org/). 😎 

[Front-end Demo](https://stupefied-darwin-da9533.netlify.com/)
## Installation
### Installation with npm
```
npm install vue-google-oauth2
```

### Installation with yarn
```
yarn add vue-google-oauth2
```

## Initialization
```javascript
//src/main.js
import GAuth from 'vue-google-oauth2'
const gauthOption = {
  clientId: 'CLIENT_ID.apps.googleusercontent.com',
  scope: 'profile email',
  prompt: 'select_account'
}
Vue.use(GAuth, gauthOption)

```
Please Don't use `plus.login` scope. [It will be deprecated.](https://developers.google.com/identity/sign-in/web/quick-migration-guide)

### Initialization for Nuxt
1. creates plug-in file for nuxt

	```javascript
	// plugins/vue-google-oauth2.js
	// file name can be changed to whatever you want
	import Vue from 'vue'
	import GAuth from 'vue-google-oauth2'

	const gauthOption = {
	  clientId: 'CLIENT_ID.apps.googleusercontent.com',
	  scope: 'profile email',
	  prompt: 'select_account'
	}
	Vue.use(GAuth, gauthOption)

	```

2. adds plugin to nuxt config file
	```javascript
	...
	plugins: [
	  ...
      './plugins/vue-google-oauth2'
	],

	...

	```

## Options
| Property     | Type     | Required        | Description     |
|--------------|----------|-----------------|-----------------|
| clientId     | String   | Required.       | The app's client ID, found and created in the Google Developers Console. |
| scope        | String   | Optional.       | Default value is `profile email`. [Full list of scopes](https://developers.google.com/identity/protocols/googlescopes). |
| prompt       | String   | Optional.       | This value using for [authCode.](https://developers.google.com/api-client-library/javascript/reference/referencedocs#gapiauth2offlineaccessoptions) The possible values are `select_account` or `consent`. Default value is `select_account`. To get refresh token from auth code, use `consent`.|
| fetch_basic_profile       | Boolean   | Optional.       | If set to true, `email profile openid` will [be automatically added as scope](https://developers.google.com/identity/sign-in/web/sign-in). Default value is `true`. |

## Methods
| Property     | Description        | Type     |
|--------------|--------------------|----------|
| GoogleAuth   | return of [gapi.auth2.getAuthInstance()](https://developers.google.com/identity/sign-in/web/reference#gapiauth2authresponse)   | Object |
| isAuthorized | Whether or not you have auth | Boolean  |
| isInit       | Whether or not api init | Boolean  |
| isLoaded     | Whether or not api init. will be deprecated. | Function  |
| signIn       | function for sign-in | Function  |
| getAuthCode  | function for getting authCode | Function  |
| signOut      | function for sign-out | Function  |


## Usages
### Getting authorization code
The `authCode` that is being returned is the `one-time code` that you can send to your backend server, so that the server can exchange for its own access_token and refresh_token.

The `access_token` and `refresh_token` can be saved in backend storage for reuse and refresh. In this way, you can avoid exposing your api key or secret key whenever you need to use various google APIs.

```javascript
const authCode = await this.$gAuth.getAuthCode()
const response = await this.$http.post('http://your-backend-server-api-to-use-authcode', { code: authCode, redirect_uri: 'postmessage' })
```

### Sign-in: Directly get back the `access_token` and `id_token`

```javascript
const googleUser = await this.$gAuth.signIn()
// googleUser.getId() : Get the user's unique ID string.
// googleUser.getBasicProfile() : Get the user's basic profile information.
// googleUser.getAuthResponse() : Get the response object from the user's auth session. access_token and so on
this.isSignIn = this.$gAuth.isAuthorized

```

refer to [google signIn reference : GoogleUser](https://developers.google.com/api-client-library/javascript/reference/referencedocs#googleusergetid)


### Sign-out
Handling Google sign-out
```javascript
const response = await this.$gAuth.signOut()
```

## Extra - Directly get `access_token` and `refresh_token` on Server-side
To get `access_token` and `refresh_token` in server side, the data for `redirect_uri` should be `postmessage`. `postmessage` is magic value for `redirect_uri` to get credentials without actual redirect uri.

### Curl
```
curl -d "client_id=YOUR_CLIENT_ID&\
  client_secret=YOUR_CLIENT_SECRET&\
  redirect_uri=postmessage&\
  grant_type=authorization_code&\
  code=YOUR_AUTH_CODE" https://accounts.google.com/o/oauth2/token
```

### Sample Code
- [Golang Sample Code](https://github.com/guruahn/vue-google-oauth2/blob/master/backend-samples/golang/main.go)
- [Python Sample Code](https://github.com/guruahn/vue-google-oauth2/blob/master/backend-samples/python/main.py)
- [Front Sample Code](https://github.com/guruahn/vue-google-oauth2-front-sample)

## Additional Help
- [sample login page HTML file](https://github.com/guruahn/vue-google-oauth2/blob/master/sample.html).
- [Google API Client Libraries : Methods and Classes](https://developers.google.com/api-client-library/javascript/reference/referencedocs)
- If you are curious of how the entire Google sign-in flow works, please refer to the diagram below
![Google Sign-in Flow](http://i.imgur.com/BQPXKyT.png)


## FAQ
### The failure of initialization happens
You can check the brower console to check errors which occur during initialization.
The most of errors are from inproper setting of google oauth2 credentials setting in Google Developer Console.
After changing the settings, you have to do hard refresh to clear your caches.
### Type Errors
Follow the documentation provided [here](https://vuejs.org/v2/guide/typescript.html#Augmenting-Types-for-Use-with-Plugins) to add `$gAuth` as a property for preventing lint errors.

