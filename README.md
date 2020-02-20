# You Don't Know Axios

> Promise based HTTP client for the browser and node.js

[axios][axios] is one of the most famous Javascript request libraries. According to Github's data, it is used by 2.1 millions repositories now (Feb 2020).

Purposes of this tutorial are,

- Clarifying misleading behaviors and usages
- Introducing design principles and internal implementations
- Helping users be able to solve problems by themselves
- Avoiding invalid or weak issues or pull requests opened
- Recording personal study notes
- Practicing English writing skills

I'd like to divide things into 3 questions,

- [Design Theories](#design-theories), why does axios look like this?
- [Usage Knowledges](#usage-knowledges), what didn't mention clearly in axios official document?
- [Problem Solutions](#problem-solutions), how to analyze a problem and open a good issue or pull request?

## Design Theories

<img src="assets/axios-design.png" width="80%" />

### Keep Simple

The most impressive design in axios is its flexible architecture, including basic configs, interceptors, transformers and adapters. The core is simple and stable, while users can achieve customized functionalities by providing their own implementations. Before requesting for new features, think twice whether it is important and common enough to be added, or it can be solved by current hooks.

### Promise Based

Make sure you are familiar with asynchronous programming when using axios, especially for Promise and async/await. Because axios connects internal things by Promise.

## Usage Knowledges

Let's follow the structure of official document. In each topic, I will give some examples to explain the misleading or unclear points. To keep this document not out-of-date, the detailed logic will not be introduced too much. It may change between different versions, please read specific axios source codes.

### axios API

#### Note the difference and relationship between `axios` and `Axios`.

Using terms in [object-oriented programming][oop], `Axios` is the *class* which provides core `request` method and other [method aliases][request-method-aliases], and `axios` is an instance of `Axios` created by `axios.create` with the default configs.

Before returning, `axios.create` will bind the instance to `Axios.prototype.request`, so `axios` is also a function same with `Axios.prototype.request`.

Something special is that, `axios` has lots of *static* members and methods beside of `axios.create`, i.e. `axios.Cancel/CancelToken/isCancel` and `axios.all/spread`.

```js
axios.Cancel // object
axios.create(config).Cancel // undefined
```

#### The position of parameter `config` is different among request methods.

For beginners, it may be a little confused. In fact, you can remember easily by asking yourself whether it has a `data` parameter.

```
axios.request([url, ]config) // first or second
axios.delete/get/head/options(url, config) // second
axios.post/put/patch(url, data, config) // third
```

### Request Config

<img src="assets/axios-configs.png" width="80%" />

The above diagram shows all request configs.

- The left dotted box contains 4 main configs(`method`, `url`, `headers`, `data`), which are corresponding to 4 parts in HTTP request format, and their related things.
- The right dotted box is `adapter` related. A separate line divides configs into browser only(`xhr.js`) and server only(`http.js`). Others configs should be applicable to both sides, but are not fully supported.
- The rest is configs that control the process, including cancellation, timeout and response transforming.

#### Should `method` be lower cases or upper?

According to HTTP specifications, the method field must be all upper cases. The default 2 adapters have done that internally. So axios users can use case-insensitive `method`.

I used to worry about headers merging. But in fact, axios will convert received `method` to lower cases and keep cases unchanged until sending out.

#### Understand how `baseURL` concats with `url`.

Don't think it as simple as `baseURL + url`. A bad example is,

```js
axios({
  baseURL: 'http://a.com/b?c=d',
  url: '&e=f'
})
```

#### Nightmares of `headers`: CORS, cookies and `auth`.

First of all, `headers` in axios are request headers, not response headers. Therefore, [CORS][mdn-cors] related problems can't be resolved by adding values in `headers`. Considering many users are confused with CORS, I'd like to give some tips about it.

- CORS problems are browser only, when your site requests a resource that has a different origin (domain, protocol, or port) from its own. Node.js scripts and [Postman][postman] don't have this kind of trouble.
- Sometimes, take it easy for those additional OPTIONS requests. They are [preflighted requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#Preflighted_requests), which are very normal things and not strange bugs caused by axios.
- If some headers couldn't be accessed in your codes, even though they were visible in the network panel, please make sure the server responses correct [Access-Control-Expose-Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#Access-Control-Expose-Headers) header.
- As [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#Credentialed_requests_and_wildcards) says, when responding to a credentialed request, the server must specify an origin in the value of the `Access-Control-Allow-Origin header`, instead of specifying the "*" wildcard. Or cookies will not be send, even though `withCredentials` has been set true in axios.

Some users complains cookies can't be set when the server has responded `Set-Cookie` header. You may check whether they are [HttpOnly or Secure](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Secure_and_HttpOnly_cookies), and scopes of cookies.

I prefer to set `Authorization` manually in `headers` to authorize, unless you know exactly what happens in axios. Here are some warnings for users,

- If no `auth` was set, http adapter will try to extract user and password from the url, while xhr adapter does nothing.
- And xhr adapter may not able to handle special characters well.

Merging of headers will be introduced in [Config Defaults](#config-defaults) section.

#### Distinguish `params` with `data`.

When you want to send request data, read the endpoint document carefully to make sure where it should be.

- If should be seen in the url, it is `params`, otherwise is `data`.
- If the method is `get`, it is `params` with 99% possibilities. Theoretically, `get` can also send with `data`, but is very rare.
- If the method is `post`, it is `data` with 80% possibilities. `post` usually works with `data` and less will have both.
- For other methods, apply the similar strategy.

```js
axios({
  url: 'http://a.com/b?c=d'
})

// is as well as

axios({
  url: 'http://a.com/b',
  params: {
    c: 'd'
  }
})
```

#### Serialize `params` correctly.

The default serialization can only handle simple `params`. If you find out the built url is not as expected, especially when your `params` contains arrays or nested objects as values, you may need to set `paramsSerializer`.


```js
var qs = require('qs'); // https://www.npmjs.com/package/qs

// url?a%5B0%5D=1&a%5B1%5D=2&b%5Bc%5D=42
axios(url, {
  params: {
    a: [1, 2],
    b: {
      c: 42
    }
  },
  paramsSerializer(params) {
    // or any other libraries you like
    return qs.stringify(params);
  }
})
```

#### Submit `data` successfully.

Here must be the most severely afflicted area. Lots of axios issues seek help due to it.

> In requests, (such as POST or PUT), the client tells the server what type of data is actually sent.

So, `data` must match with the header [Content Type][mdn-content-type]. Followings are its common values.

- `text/plain`
- `application/json`

In this case, `data` should be JSON format. If `data` is an object (not null), the default `transformRequest` will set Content-Type to it automatically.

```js
axios({
  data: {
    a: 42
  }
})

// equals to 

axios({
  data: JSON.stringify({a: 42})
})
```

- `application/x-www-form-urlencoded`

As the name indicated, `data` should be URL/URI encoded. If `data` is an instance of URLSearchParams, the default `transformRequest` will set Content-Type to it automatically.

Note that it treats numbers as strings, while `application/json` is type-sensitive.

```js
var data = new URLSearchParams();
data.append('a', 42)

axios({
  data: data
})

// equals to 

var qs = require('qs'); // https://www.npmjs.com/package/qs

axios({
  data: qs.stringify({a: '42'})
})
```

#### Why was't `timeout` fired at the right time?

axios supports `timeout` by underlayer APIs, XMLHttpRequest's [timeout event](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/timeout_event) and [request.setTimeout](https://nodejs.org/api/http.html#http_request_settimeout_timeout_callback) in Node.js. You may face browser compatibilities problems or Node.js environments problems.

If you set `timeout` to a small value, i.e. 1 or 2, make sure it doesn't conflict with Javascript's [event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop).

Now axios doesn't have a convenient way to validate a timeout error, except for finding special patterns in the error message. And `timeoutErrorMessage` is browser only yet.

Someone wishes other types of timeout. Looks like [got](https://github.com/sindresorhus/got#timeout) provides them very well.

#### Do you use the right `adapter`?

For enviroments like Electron or Jest, both XMLHttpRequest and process are existed in the global context. axios may not select the right `adapter` as you want.

```js
axios.defaults.adapter // [Function: httpAdapter] or [Function: xhrAdapter]
```

And you can set `adapter` explicitly.

```js
axios({
  adapter: require('axios/lib/adapters/http')
})
```

If you like more fashion [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), sorry that axios has not supported yet. You have write one by yourself or search in npm.

### Response Schema

### Config Defaults

### Interceptors

### Handling Errors

### Cancellation

## Problem Solutions


[axios]: https://github.com/axios/axios
[request-method-aliases]: https://github.com/axios/axios#request-method-aliases
[mdn-content-type]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type
[mdn-cors]: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
[oop]: https://en.wikipedia.org/wiki/Object-oriented_programming
[postman]: https://www.postman.com/
