# Axios Module
[![npm (scoped with tag)](https://img.shields.io/npm/v/@nuxtjs/axios/latest.svg?style=flat-square)](https://npmjs.com/package/@nuxtjs/axios)
[![npm](https://img.shields.io/npm/dt/@nuxtjs/axios.svg?style=flat-square)](https://npmjs.com/package/@nuxtjs/axios)
[![CircleCI](https://img.shields.io/circleci/project/github/nuxt-community/axios-module.svg?style=flat-square)](https://circleci.com/gh/nuxt-community/axios-module)
[![Codecov](https://img.shields.io/codecov/c/github/nuxt-community/axios-module.svg?style=flat-square)](https://codecov.io/gh/nuxt-community/axios-module)
[![Greenkeeper](https://img.shields.io/badge/greenkeepr-enabled-green.svg?style=flat-square)](https://greenkeeper.io/)
[![Dependencies](https://david-dm.org/nuxt-community/axios-module/status.svg?style=flat-square)](https://david-dm.org/nuxt-community/axios-module)
   
[![js-standard-style](https://cdn.rawgit.com/standard/standard/master/badge.svg)](http://standardjs.com)

> Secure and Easy [Axios](https://github.com/mzabriskie/axios) integration with Nuxt.js.
   
[📖 **Release Notes**](./CHANGELOG.md)

## Features

- [Automatically set base URL for client & server side](#baseurl)
- [Exposes `setToken` function to `$axios` so we can easily and globally set authentication tokens](#setheadername-value-scopescommon)
- [Throws *nuxt-friendly* errors and optionally redirect on specific error codes](#redirecterror)
- [Automatically enables `withCredentials` when requesting to base URL](#credentials)
- [Proxy request headers in SSR](#proxyheaders) (Useful for auth)
- [Fetch Style requests](#fetch-style-requests)
- [...and more!](https://github.com/mzabriskie/axios#features)

## Setup
- Add `@nuxtjs/axios` dependency using yarn or npm to your project
- Add `@nuxtjs/axios` to `modules` section of `nuxt.config.js`

```js
{
  modules: [
    // Simple usage
    '@nuxtjs/axios',

    // With options
    ['@nuxtjs/axios', { credentials: false }],
 ],

  // You can optionally use global options instead of inline form
  axios: {
    credentials: false
  }
}
```

## Usage

### Component `asyncData`

```js
async asyncData({ app }) {
  const ip = await app.$axios.$get('http://icanhazip.com')
  return { ip }
}
```

### Store `nuxtServerInit`
```js
async nuxtServerInit ({ commit }, { app }) {
  const ip = await app.$axios.$get('http://icanhazip.com')
  commit('SET_IP', ip)
}
```

### Store actions
(Needs Nuxt >= 1.0.0-RC8)

```js
// In store
{
  actions: {
    async getIP ({ commit }) {
      const ip = await this.$axios.$get('http://icanhazip.com')
      commit('SET_IP', ip)
    }
  }
}
```

## Options
You can pass options using module options or `axios` section in  `nuxt.config.js`

### `baseURL`
- Default: `http://[HOST]:[PORT]/api`

Base URL is required for requests in server-side & SSR and prepended to all requests with relative path.
You can also use environment variable `API_URL` which **overrides** `baseURL`.

### `browserBaseURL`
- Default: `/api`

Base URL which is used in client side prepended to all requests with relative path.
You can also use environment variable `API_URL_BROWSER` which **overrides** `browserBaseURL`.

- If `browserBaseURL` is not provided it defaults to `baseURL` value.
  - If hostname & port of `browserbaseURL` are equal to nuxt server, it defaults to relative part of `baseURL`.
    So if your nuxt application is being accessed under a different domain, requests go to same origin and prevents Cross-Origin problems.

### `credentials`
- Default: `true`

Adds an interceptor to automatically set `withCredentials` config of axios when requesting to `baseUrl`
which allows passing authentication headers to backend.

### `debug`
- Default: `false`

Adds interceptors to log all responses and requests

### `proxyHeaders`
- Default: `true`

In SSR context, sets client request header as axios default request headers.
This is useful for making requests which need cookie based auth on server side.
Also helps making consistent requests in both SSR and Client Side code.

> **NOTE:** If directing requests at a url protected by CloudFlare's CDN you should set this to false to prevent CloudFlare from mistakenly detecting a reverse proxy loop and returning a 403 error.

### `redirectError`
- Default: `{}`

This option is a map from specific error codes to page which they should be redirect.
For example if you want redirecting all `401` errors to `/login` use:
```js
axios: {
  redirectError: {
    401: '/login'
  }
}
```

### `requestInterceptor`
- Default: `null`

Function for manipulating axios requests. Useful for setting custom headers,
for example based on the store state. The second argument is the nuxt context.

```js
requestInterceptor: (config, { store }) => {
  if (store.state.token) {
    config.headers.common['Authorization'] = store.state.token
  }
  return config
}
```

### `responseInterceptor`
- Default: `null`

```js
responseInterceptor: (response, ctx) => {
  return response
}
```


Function for manipulating axios responses. 

### `init`
- Default: `null`

Function `init(axios, ctx)` to do additional things with axios. Example:

```js
axios: {
  init(axios, ctx) {
    axios.defaults.xsrfHeaderName = 'X-CSRF-TOKEN'
  }
}
```

### `errorHandler`
- Default: (Return promise rejection with error)

Function for custom global error handler. 
This example uses nuxt default error page.

```js
axios: {
  errorHandler (error) {
    this.error('Request Error: ' + error)
  }
},
```

## Helpers

### Fetch Style requests
Axios plugin also supports fetch style requests with `$` prefixed methods:
```js
// Normal usage with axios
let data = (await $axios.get('...')).data

// Fetch Style
let data = await $axios.$get('...')
```

### `setHeader(name, value, scopes='common')`
Axios instance has a helper to easily set any header.

Parameters:
- **name**: Name of the header
- **value**: Value of the header
- **scopes**: Send only on specific type of requests. Defaults
  - Type: *Array* or *String*
  - Defaults to `common` meaning all types of requests
  - Can be `get`, `post`, `delete`, ...

```js
// Adds header: `Authorization: 123` to all requests
this.$axios.setHeader('Authorization', '123')

// Overrides `Authorization` header with new value
this.$axios.setHeader('Authorization', '456')

// Adds header: `Content-Type: application/x-www-form-urlencoded` to only post requests
this.$axios.setHeader('Content-Type', 'application/x-www-form-urlencoded', ['post'])

// Removes default Content-Type header from `post` scope
this.$axios.setHeader('Content-Type', false, ['post'])
```

### `setToken(token, type, scopes='common')`
Axios instance has an additional helper to easily set global authentication header.

Parameters:
- **token**: Authorization token
- **type**: Authorization token prefix(Usually `Bearer`).
- **scopes**: Send only on specific type of requests. Defaults
  - Type: *Array* or *String*
  - Defaults to `common` meaning all types of requests
  - Can be `get`, `post`, `delete`, ...

```js
// Adds header: `Authorization: 123` to all requests
this.$axios.setToken('123')

// Overrides `Authorization` header with new value
this.$axios.setToken('456')

// Adds header: `Authorization: Bearer 123` to all requests
this.$axios.setToken('123', 'Bearer')

// Adds header: `Authorization: Bearer 123` to only post and delete requests
this.$axios.setToken('123', 'Bearer', ['post', 'delete'])

// Removes default Authorization header from `common` scope (all requests)
this.$axios.setToken(false)
```

## Dynamic API Backend
Please notice that, `API_URL` is saved into bundle on build, CANNOT be changed
on runtime! You may use [proxy](../proxy) module for dynamically route api requests to different backend on test/staging/production.

**Example: (`nuxt.config.js`)**

```js
{
  modules: [
    '@nuxtjs/axios',
    '@nuxtjs/proxy'
 ],
  proxy: [
    ['/api', { target: 'http://www.mocky.io', pathRewrite: { '^/api': '/v2' } }]
  ]
}
```

Start Nuxt
```
[AXIOS] Base URL: http://localhost:3000/api | Browser: /api
[HPM] Proxy created: /api  ->  http://www.mocky.io
[HPM] Proxy rewrite rule created: "^/api" ~> "/v2"
```

Now you can make requests to backend: (Works fine in both SSR and Browser)
```js
async asyncData({ app }) {
  // Magically makes request to http://www.mocky.io/v2/59388bb4120000dc00a672e2
  const nuxt = await app.$axios.$get('59388bb4120000dc00a672e2')

  return {
    nuxt // -> { nuxt: 'Works!' }
  }
}
```

Details
- `'@nuxtjs/axios'`
  - By default axios plugin sets base url to `http://[host]:[port]/api` which is `http://localhost:3000/api`

- `'/api': 'http://www.mocky.io/v2'`
  - This line creates a server middleware to pass requests from `/api` to `http://www.mocky.io/v2`
  - We used `pathRewrite` to remove `/api` from starting of requests and change it to `/v2`
  - For more information and advanced usage please refer to [proxy](../proxy) docs.

## License

[MIT License](./LICENSE)

Copyright (c) 2017 Nuxt Community
