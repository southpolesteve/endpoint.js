# endpoint.js

> Turns REST API endpoints into generic request options

[![@latest](https://img.shields.io/npm/v/@octokit/endpoint.svg)](https://www.npmjs.com/package/@octokit/endpoint)
[![Build Status](https://travis-ci.org/octokit/endpoint.js.svg?branch=master)](https://travis-ci.org/octokit/endpoint.js)
[![Coverage Status](https://coveralls.io/repos/github/octokit/endpoint.js/badge.svg)](https://coveralls.io/github/octokit/endpoint.js)
[![Greenkeeper](https://badges.greenkeeper.io/octokit/endpoint.js.svg)](https://greenkeeper.io/)

`@octokit/endpoint` combines [GitHub REST API](https://developer.github.com/v3/)
with your options and turns them into generic request options which you can
then pass into your request library of choice.

`@octokit/endopint` is meant to run in all JavaScript environments. Browser
builds can be downloaded from each [Release](https://github.com/octokit/endpoint.js/releases).
The bundle size is currently 9.15KB (minified and gzipped). [Help us reduce the
bundle size](Reduce bundle size for browsers) 🙏

## Usage

```js
const endpoint = require('@octokit/endpoint')

// Following GitHub docs formatting:
// https://developer.github.com/v3/repos/#list-organization-repositories
const options = endpoint('GET /orgs/:org/repos', {
  headers: {
    authorization: 'token 0000000000000000000000000000000000000001'
  },
  org: 'octokit',
  type: 'private'
})
```

Alternatively, pass in a method and a url

```js
const options = endpoint({
  // route options
  method: 'GET',
  url: '/orgs/:org/repos',
  headers: {
    authorization: 'token 0000000000000000000000000000000000000001'
  },
  // parameters
  org: 'octokit',
  type: 'private'
})
```

The method returns an object with 3 or 4 keys

<table>
  <thead>
    <tr>
      <th align=left>
        key
      </th>
      <th align=left>
        type
      </th>
      <th align=left>
        description
      </th>
    </tr>
  </thead>
  <tr>
    <th align=left><code>method</code></th>
    <td>String</td>
    <td>The http method. Always lowercase</td>
  </tr>
  <tr>
    <th align=left><code>url</code></th>
    <td>String</td>
    <td>The url with placeholders replaced with passed parameters</td>
  </tr>
  <tr>
    <th align=left><code>headers</code></th>
    <td>Object</td>
    <td>All header names are lowercased</td>
  </tr>
  <tr>
    <th align=left><code>body</code></th>
    <td>Any</td>
    <td>The request body if one is present. Only for <code>PATCH</code>, <code>POST</code>, <code>PUT</code>, <code>DELETE</code> requests</td>
  </tr>
</table>


The above examples shown above return

```js
{
  method: 'get',
  url: 'https://api.github.com/orgs/octokit/repos?type=private',
  headers: {
    accept: 'application/vnd.github.v3+json',
    authorization: 'token 0000000000000000000000000000000000000001',
    'user-agent': 'octokit/endpoint.js v1.2.3'
  }
}
```

### Using `@octokit/endpoint` with common request libraries

```js
// using with fetch (https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
fetch(options.url, ...options)
// using with request (https://github.com/request/request)
request(options)
// using with got (https://github.com/sindresorhus/got)
got[options.method](options.url, options)
// using with axios
axios(options)
```

## Options

<table>
  <thead>
    <tr>
      <th align=left>
        name
      </th>
      <th align=left>
        type
      </th>
      <th align=left>
        description
      </th>
    </tr>
  </thead>
  <tr>
    <th align=left>
      <code>baseUrl</code>
    </th>
    <td>
      String
    </td>
    <td>
      <strong>Required.</strong> Any supported <a href="https://developer.github.com/v3/#http-verbs">http verb</a>, case insensitive. <em>Defaults to <code>https://api.github.com</code></em>.
    </td>
  </tr>
    <th align=left>
      <code>headers</code>
    </th>
    <td>
      Object
    </td>
    <td>
      Custom headers. Passed headers are merged with defaults:<br>
      <em><code>headers['user-agent']</code> defaults to <code>octokit-endpoint.js/1.2.3</code> (where <code>1.2.3</code> is the released version)</em>.<br>
      <em><code>headers['accept']</code> defaults to <code>application/vnd.github.v3+json</code>.<br>
    </td>
  </tr>
  <tr>
    <th align=left>
      <code>method</code>
    </th>
    <td>
      String
    </td>
    <td>
      <strong>Required.</strong> Any supported <a href="https://developer.github.com/v3/#http-verbs">http verb</a>, case insensitive. <em>Defaults to <code>Get</code></em>.
    </td>
  </tr>
  <tr>
    <th align=left>
      <code>url</code>
    </th>
    <td>
      String
    </td>
    <td>
      <strong>Required.</strong> A path or full URL which may contain <code>:variable</code> or <code>{variable}</code> placeholders,
      e.g. <code>/orgs/:org/repos</code>. The <code>url</code> is parsed using <a href="https://github.com/bramstein/url-template">url-template</a>.
    </td>
  </tr>
</table>

All other options will passed depending on the `method` and `url` options.

1. If the option key is a placeholder in the `url`, it will be used as replacement. For example, if the passed options are `{url: '/orgs/:org/repos', org: 'foo'}` the returned `options.url` is `https://api.github.com/orgs/foo/repos`
2. If the `method` is `GET` or `HEAD`, the option is passed as query parameter
3. Otherwise the parameter is passed as request body.

## endpoint.defaults()

Override or set default options. Example:

```js
const request = require('request')
const myEndpoint = require('@octokit/endpoint').defaults({
  baseUrl: 'http://github-enterprise.acme-inc.com/api/v3',
  headers: {
    'user-agent': 'myApp/1.2.3',
    authorization: `token 0000000000000000000000000000000000000001`
  },
  org: 'my-project',
  per_page: 100
})

request(myEndpoint(`GET /orgs/:org/repos`))
```

You can call `.defaults()` again on the returned method, the defaults will cascade.

```js
const myProjectEndpoint = endpoint.defaults({
  baseUrl: 'http://github-enterprise.acme-inc.com/api/v3',
  headers: {
    'user-agent': 'myApp/1.2.3'
  },
  org: 'my-project'
})
const myProjectEndpointWithAuth = myProjectEndpoint.defaults({
  headers: {
    authorization: `token 0000000000000000000000000000000000000001`
  }
})
```

`myProjectEndpointWithAuth` now defaults the `baseUrl`, `headers['user-agent']`,
`org` and `headers['authorization']` on top of `headers['accept']` that is set
by the global default.

## Special cases

### The `data` parameter – set request body directly

Some endpoints such as [Render a Markdown document in raw mode](https://developer.github.com/v3/markdown/#render-a-markdown-document-in-raw-mode) don’t have parameters that are sent as request body keys, instead the request body needs to be set directly. In these cases, set the `data` parameter.

```js
const options = endpoint('POST /markdown/raw', {
  data: 'Hello world github/linguist#1 **cool**, and #1!',
  headers: {
    accept: 'text/html;charset=utf-8',
    'content-type': 'text/plain'
  }
})

// options is
// {
//   method: 'post',
//   url: 'https://api.github.com/markdown/raw',
//   headers: {
//     accept: 'text/html;charset=utf-8',
//     'content-type': 'text/plain',
//     'user-agent': userAgent
//   },
//   body: 'Hello world github/linguist#1 **cool**, and #1!'
// }
```

### Set parameters for both the URL/query and the request body

There are API endpoints that accept both query parameters as well as a body. In that case you need to add the query parameters as templates to `options.url`, as defined in the [RFC 6570 URI Template specification](https://tools.ietf.org/html/rfc6570).

Example

```js
endpoint('POST https://uploads.github.com/repos/octocat/Hello-World/releases/1/assets{?name,label}', {
  name: 'example.zip',
  label: 'short description',
  headers: {
    'content-type': 'text/plain',
    'content-length': 14,
    authorization: `token 0000000000000000000000000000000000000001`
  },
  data: 'Hello, world!'
})
```

## LICENSE

[MIT](LICENSE)
