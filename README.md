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

#### Understand how `baseURL` concats with `url`.

Don't think it as simple as `baseURL + url`. A bad example is,

```js
axios({
  baseURL: 'http://a.com/b?c=d',
  url: '&e=f'
})
```

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
const qs = require('qs'); // https://www.npmjs.com/package/qs

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

Here must be the most severely afflicted area.

### Response Schema

### Config Defaults

### Interceptors

### Handling Errors

### Cancellation

## Problem Solutions


[axios]: https://github.com/axios/axios
[request-method-aliases]: https://github.com/axios/axios#request-method-aliases
[oop]: https://en.wikipedia.org/wiki/Object-oriented_programming
