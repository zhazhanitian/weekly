#### 背景

也使用Axios很久了，但是并未详细了解过其内部的实现原理，以及其暴露出来的方法，这里坐一下学习记录



#### Axios是什么

一个基于 Promise 来管理 http 请求的简洁、易用且高效的代码封装库。通俗一点来讲，它是一个前端替代Ajax的一个东西，可以使用它发起http请求接口功能，它是基于Promise的，相比于Ajax的回调函数能够更好的管理异步操作，引用一下官网的介绍

引用一下官网的介绍

> Axios 是一个基于 promise 的 HTTP 库，可以用在浏览器和 node.js 中
> 从浏览器中创建 XMLHttpRequests
> 从 node.js 创建 http 请求
> 支持 Promise API
> 拦截请求和响应
> 转换请求数据和响应数据
> 取消请求
> 自动转换 JSON 数据
> 客户端支持防御 XSRF



#### Axios源码分析

##### npm安装包

<img src="https://qiniu-app.qtshe.com/1588503878876.jpg" style="zoom:45%;float:left;" />

【index.js】入口文件：

```javascript
module.exports = require('./lib/axios');
```

【/lib/axios.js】文件内容：

```javascript
/**
 * Create an instance of Axios
 *
 * @param {Object} defaultConfig The default config for the instance
 * @return {Axios} A new instance of Axios
 */
function createInstance(defaultConfig) {
  var context = new Axios(defaultConfig);
  var instance = bind(Axios.prototype.request, context);

  // Copy axios.prototype to instance
  utils.extend(instance, Axios.prototype, context);

  // Copy context to instance
  utils.extend(instance, context);

  return instance;
}

// Create the default instance to be exported
var axios = createInstance(defaults);

// Expose Axios class to allow class inheritance
axios.Axios = Axios;

// Factory for creating new instances
axios.create = function create(instanceConfig) {
  return createInstance(mergeConfig(axios.defaults, instanceConfig));
};

// Expose Cancel & CancelToken
axios.Cancel = require('./cancel/Cancel');
axios.CancelToken = require('./cancel/CancelToken');
axios.isCancel = require('./cancel/isCancel');

// Expose all/spread
axios.all = function all(promises) {
  return Promise.all(promises);
};
axios.spread = require('./helpers/spread');

module.exports = axios;

// Allow use of default import syntax in TypeScript
module.exports.default = axios;
```



##### defaultConfig参数

```javascript
var axios = createInstance(defaults);
```

【/lib/defaults.js】文件内容

在创建时传入一个默认参数，先看看这个参数defaults是什么

```javascript
// 通过nodejs中的process和浏览器的XMLHttpRequest来区别当前在前端还是nodejs中
function getDefaultAdapter() {
  var adapter;
  if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '[object process]') {
    adapter = require('./adapters/http');
  } else if (typeof XMLHttpRequest !== 'undefined') {
    adapter = require('./adapters/xhr');
  }
  return adapter;
}

var defaults = {
  adapter: getDefaultAdapter(),
  ... 
  timeout: 0,

  xsrfCookieName: 'XSRF-TOKEN',
  xsrfHeaderName: 'X-XSRF-TOKEN',

  maxContentLength: -1,
 
  validateStatus: function validateStatus(status) {
    return status >= 200 && status < 300;
  }
};

defaults.headers = {
  common: {
    'Accept': 'application/json, text/plain, */*'
  }
};
```

从上面便可以看出来，axios能够即在客户端使用又能在浏览器使用的奥秘，它是通过Nodejs和浏览器中各自的全局变量来区别当前在哪个环境下，然后底层各自实现，再暴露出一套统一的API出来给我们使用。同时它还默认了想 超时时间，Headers信息，alidateStatus等一些默认值进去，当我们在使用的时候不传递覆盖这些值时，即走默认的配置



##### 各环境中的实现

【/adapters/http.js】文件内容，看看Nodejs中的Axios的实现

```javascript
...
var http = require('http');
var https = require('https');
...
module.exports = function httpAdapter(config) {
  return new Promise(function dispatchHttpRequest(resolvePromise, rejectPromise) {
    ...
    var req = transport.request(options, function handleResponse(res) {
    ...
    // 处理response后返回
    })
    // req的error处理
    req.on('error', function handleRequestError(err) {
      ...
    });
    req.end(data);
  })
}
```

从上可以看出在Nodejs中，Axios的实现其实是基于nodejs的http或者http模块来发起请求的

【/adapters/xhr.js】文件内容，看看在浏览器中的Axios的实现

```javascript
...
module.exports = function xhrAdapter(config) {
  return new Promise(function dispatchXhrRequest(resolve, reject) {
    ...
    var request = new XMLHttpRequest();
    ...
    request.open(config.method.toUpperCase(), buildURL(config.url, config.params, config.paramsSerializer), true);
    ...
    request.onreadystatechange = function handleLoad() {
      ...
    };
    ...
    request.onabort = function handleAbort() {...}
    ...
    request.onerror = function handleError() {...}
    ...
    request.ontimeout = function handleTimeout() {...}
    ...
    request.send(requestData);
    ...
  })
}
```

一个完整的Ajax库封装流程，只不过axios暴露了一个Promise出去，所以axios在浏览器端和Ajax底层的原理是一样的，都是通过浏览器的XMLhttpRequest这个底层接口进行的一次封装



##### Axios构造函数

【/lib/core/Axios.js】上面已经看了在入口进去的axios文件中，createInstance函数传递的参数，接下来再看看createInstance内部的Axios构造函数做了什么

```javascript
...
var context = new Axios(defaultConfig);
...

function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}
```

Axios这个构造函数将刚才传入的defaultConfig参数挂到自己的this上，然后新增了一个interceptors拦截器对象，这个对象有request和response两个属性，接下来看一下这两个属性中的InterceptorManager 这个构造函数又是什么

【/lib/core/InterceptorManager.js】文件内容，拦截器里的构造函数

```javascript
InterceptorManager.prototype.use = function() {...}
InterceptorManager.prototype.eject =  function() {...}
InterceptorManager.prototype.forEach = function forEach(fn) {...}
```

拦截器暴露了三个方法use，eject，forEach三个方法，相信大家很多人在写自己的拦截器的时候都是用过use这个属性。后面两个比较少用，但是可以通过它的代码看出来，eject是删除use过的内容，forEach则是循环执行传入fn



#### Interceptors 拦截器

axios 官网中对 Interceptors 的使用方法如下： 用户可以通过 then 方法为请求添加回调，而拦截器中的回调将在 then 中的回调之前执行：

```javascript
// Add a request interceptor
axios.interceptors.request.use(function (config) {
 // Do something before request is sent
 return config;
}, function (error) {
 // Do something with request error
 return Promise.reject(error);
});
// Add a response interceptor
axios.interceptors.response.use(function (response) {
 // Do something with response data
 return response;
}, function (error) {
 // Do something with response error
 return Promise.reject(error);
});
```

之后需要能移除 Interceptors ：

```javascript
const myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

也可以为axios实例添加一个Interceptors：

```javascript
const instance = axios.create();
instance.interceptors.request.use(function () {/*...*/});
```

从其中的使用方法，我们可以知道 request 拦截器需要在请求之前执行，response 拦截器需要再请求之后执行。我们看一下axios的实现方式： 首先axios为拦截器定义了一个管理中心InterceptorManager：

```javascript
function InterceptorManager() {
 this.handlers = [];
}
InterceptorManager.prototype.use = function use(fulfilled, rejected) {
 this.handlers.push({
 fulfilled: fulfilled,
 rejected: rejected
 });
 return this.handlers.length - 1;
};
InterceptorManager.prototype.forEach = function forEach(fn) {
 utils.forEach(this.handlers, function forEachHandler(h) {
 if (h !== null) {
 fn(h);
 }
 });
};
```

其次，当实例化Axios时，分别创建一个request 和一个response拦截器：

```javascript
function Axios(instanceConfig) {
 this.defaults = instanceConfig;
 this.interceptors = {
 request: new InterceptorManager(),
 response: new InterceptorManager()
 };
}
```

然后我们需要通过use来分别添加拦截器时，便将我们定义的resolve和reject收入对应的request和response中。在此之前，我们需要对Promise有一个简单的了解：

> Promise then 方法返回的是一个新的 Promise 实例（注意，不是原来那个 Promise 实例）。因此可以采用链式写法，即 then 方法后面再调用另一个 then 方法

```javascript
getJSON("/posts.json").then(function(json) {
 return json.post;
}).then(function(post) {
 // ...
});
```

> 上面的代码使用then方法，依次指定了两个回调函数。第一个回调函数完成以后，会将返回结果作为参数，传入第二个回调函数。采用链式的then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个Promise对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用

```javascript
getJSON("/post/1.json").then(function(post) {
 return getJSON(post.commentURL);
}).then(function funcA(comments) {
 console.log("resolved: ", comments);
}, function funcB(err){
 console.log("rejected: ", err);
});
```

> 上面代码中，第一个then方法指定的回调函数，返回的是另一个Promise对象。这时，第二个then方法指定的回调函数，就会等待这个新的Promise对象状态发生变化。如果变为resolved，就调用funcA，如果状态变为rejected，就调用funcB

接下来，看看request方法是如何实现拦截器功能的:

```javascript
Axios.prototype.request = function request(config) {
 ...
 var chain = [dispatchRequest, undefined];
 var promise = Promise.resolve(config);
 this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
 chain.unshift(interceptor.fulfilled, interceptor.rejected);
 });
 this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
 chain.push(interceptor.fulfilled, interceptor.rejected);
 });
 while (chain.length) {
 promise = promise.then(chain.shift(), chain.shift());
 }
 return promise;
};
```

首先定义了一个数组调用链chain，然后通过unshift将 request interceptors推入数组头部，将response interceptors推入数组尾部。最后通过while这样一个循环执行promise.then来达到链式调用。 来看一张图：

<img src="https://qiniu-app.qtshe.com/qweqwe.jpeg"/>

通过巧妙的利用unshift、push、shift等数组队列、栈方法，实现了请求拦截、执行请求、响应拦截的流程设定，注意无论是请求拦截还是响应拦截，越先添加的拦截器总是越“贴近”执行请求本身

可以看到，axios在实现封装网络请求所需的各项扩展功能时，都是使用的最朴素JavaScript源生方法，并且总是通过简单的链式then方法将这些功能与核心的promise对象关联。此外各种优化性能的方法，也都是采用的很基本的原理



