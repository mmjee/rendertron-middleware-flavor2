# rendertron-middleware-flavor2

[![CI](https://github.com/GoogleChrome/rendertron/workflows/CI/badge.svg)](https://github.com/GoogleChrome/rendertron/actions)
[![NPM version](http://img.shields.io/npm/v/rendertron-middleware.svg)](https://www.npmjs.com/package/rendertron-middleware)

An Express middleware for [Rendertron](https://github.com/GoogleChrome/rendertron).

Rendertron is a server which runs headless Chrome and renders web pages on the fly, which can be set up to serve pages to search engines, social networks and link rendering bots.

This middleware is different from the default rendertron middleware, it only serves the SPA version to browsers that can actually render the SPA. Thus, no maintaining UA lists of bots and such.

Think about it, when you host a "website", does it imply that the only people who can access the _data_ of your website are browsers, an incredibly complex, heavy piece of software that doesn't work everywhere. No, the nature of SPAs are a _implementation detail_, your users don't care if it's an SPA, MPA, or static. Everyone using HTTP shouldn't either.

User-Agent clearly indicates the capabilities of the software, so unless you are using an engine that is compatible with SPAs, you shouldn't see incompatible and generic HTML shoved down your throat.

## Usage

```sh
$ npm install --save express rendertron-middleware
```

```js
const express = require('express');
const rendertron = require('rendertron-middleware');

const app = express();

app.use(
  rendertron.makeMiddleware({
    proxyUrl: 'http://my-rendertron-instance/render',
  })
);

app.use(express.static('files'));
app.listen(8080);
```

## Configuration

The `makeMiddleware` function takes a configuration object with the following
properties:

| Property                | Default                                                                                                                                            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `proxyUrl`              | _Required_                                                                                                                                         | Base URL of your running Rendertron proxy service.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `excludeUrlPattern`     | A set of known static file extensions. [Full list.](https://github.com/samuelli/bot-render/blob/master/middleware/src/middleware.ts)               | RegExp for excluding requests by the path component of the URL.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `injectShadyDom`        | `false`                                                                                                                                            | Force the web components polyfills to be loaded. [Read more.](https://github.com/samuelli/bot-render#web-components)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `timeout`               | `11000`                                                                                                                                            | Millisecond timeout for the proxy request to Rendertron. If exceeded, the standard response is served (i.e. `next()` is called). This is **not** the timeout for the Rendertron server itself. See also the [Rendertron timeout.](https://github.com/googlechrome/rendertron#rendering-budget-timeout)                                                                                                                                                                                                                                                                                                              |
| `allowedForwardedHosts` | `[]`                                                                                                                                               | If a forwarded host header is found and matches one of the hosts in this array, then that host will be used for the request to the rendertron server instead of the actual host of the current request. This is usedful if this middleware is running on a different host which is proxied behind the actual site, and the rendertron server should request the main site. **Note:** For security, because the header info is untrusted, only those hosts which you explicitly allow will be forwarded, otherwise they will be ignored. Leaving this undefined or empty (the default) will disable host forwarding. |
| `forwardedHostHeader`   | `"X-Forwarded-Host"`                                                                                                                               | Header used to determine the forwarded host that should be used when building the URL to be rendered. Only used if `allowedForwardedHosts` is not empty.                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
