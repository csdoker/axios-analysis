# 学习 axios 源码整体架构，打造属于自己的请求库

## 前言

>这是学习源码整体架构第五篇。整体架构这词语好像有点大，姑且就算是源码整体结构吧，主要就是学习是代码整体结构，不深究其他不是主线的具体函数的实现。本篇文章学习的是实际仓库的代码。

其余四篇分别是：
>1.[学习 jQuery 源码整体架构，打造属于自己的 js 类库](https://juejin.im/post/5d39d2cbf265da1bc23fbd42)<br>
>2.[学习 underscore 源码整体架构，打造属于自己的函数式编程类库](https://juejin.im/post/5d4bf94de51d453bb13b65dc)<br>
>3.[学习 lodash 源码整体架构，打造属于自己的函数式编程类库](https://juejin.im/post/5d767e1d6fb9a06b032025ea)<br>
>4.[学习 sentry 源码整体架构，打造属于自己的前端异常监控SDK](https://juejin.im/post/5dba5a39e51d452a2378348a)<br>

感兴趣的读者可以点击阅读。下一篇可能是学习 `axios` 源码。

TODO:
**导读**<br>

axios-analysis

umi-request

## chrome 和 vscode  调试 axios 源码方法

### chrome 调试浏览器环境 的 axios

前不久，在知乎回答了一个问题[一年内]()

调试方法

`axios`打包后有`sourcemap`文件。

```bash
git clone https://github.com/lxchuan12/axios-analysis.git
cd axios-analaysis/axios
npm install
npm run examples
# open [http://localhost:3000](http://localhost:3000)
#  source 控制面板  // webpack     .     lib 目录下，根据情况自行断点调试
```

把代码克隆下来后：
修改 `axios/examples/server.js` 以下两行代码：

```js
// ...
server = http.createServer(function (req, res) {
  var url = req.url;
// Process axios itself
  if (/axios\.min\.js$/.test(url)) {
    // 原来的代码 是 axios.min.js
    // pipeFileToResponse(res, '../dist/axios.min.js', 'text/javascript');
    pipeFileToResponse(res, '../dist/axios.js', 'text/javascript');
    return;
  }
  if (/axios\.min\.map$/.test(url)) {
    // 原来的代码 是 axios.min.map
     // pipeFileToResponse(res, '../dist/axios.min.map', 'text/javascript');
    pipeFileToResponse(res, '../dist/axios.min.map', 'text/javascript');
    return;
  }
  // ...
}
```

### vscode 调试 node 环境的 axios

在根目录下 `axios-analysis/`
创建`.vscode/launch`文件如下：

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "program": "${workspaceFolder}/axios/sandbox/client.js",
            "skipFiles": [
                "<node_internals>/**"
            ]
        },
    ]
}
```

按`F5`开始调试即可，按照自己的情况断点调试。

## axios 原理

## 源码

promise

Axios.prototype.request

```js
Axios.prototype.request = function request(config) {
  /*eslint no-param-reassign:0*/
  // Allow for axios('example/url'[, config]) a la fetch API
  if (typeof config === 'string') {
    config = arguments[1] || {};
    config.url = arguments[0];
  } else {
    config = config || {};
  }

  config = mergeConfig(this.defaults, config);

  // Set config.method
  if (config.method) {
    config.method = config.method.toLowerCase();
  } else if (this.defaults.method) {
    config.method = this.defaults.method.toLowerCase();
  } else {
    config.method = 'get';
  }

  // Hook up interceptors middleware
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

dispatchRequest

```js
var adapter = config.adapter || defaults.adapter;

return adapter(config)
```

## 总结

`Axios` 源码相对不多，打包后一千多行，非常值得学习，也比较容易看完。

如果读者发现有不妥或可改善之处，再或者哪里没写明白的地方，欢迎评论指出。另外觉得写得不错，对您有些许帮助，可以点赞、评论、转发分享，也是对笔者的一种支持，非常感谢呀。

## 推荐阅读

[@叫我小明呀：Axios 源码解析](https://juejin.im/post/5cb5d9bde51d456e62545abc)<br>
[@尼库尼库桑：深入浅出 axios 源码](https://zhuanlan.zhihu.com/p/37962469)<br>
[@小贼先生_ronffy：Axios源码深度剖析 - AJAX新王者](https://juejin.im/post/5b0ba2d56fb9a00a1357a334)<br>
[逐行解析Axios源码](https://juejin.im/post/5d501512518825159e3d7be6)<br>
[[译]axios 是如何封装 HTTP 请求的](https://juejin.im/post/5d906269f265da5ba7451b02)<br>

## 笔者精选文章

[学习 sentry 源码整体架构，打造属于自己的前端异常监控SDK](https://juejin.im/post/5dba5a39e51d452a2378348a)<br>
[学习 lodash 源码整体架构，打造属于自己的函数式编程类库](https://juejin.im/post/5d767e1d6fb9a06b032025ea)<br>
[学习 underscore 源码整体架构，打造属于自己的函数式编程类库](https://juejin.im/post/5d4bf94de51d453bb13b65dc)<br>
[学习 jQuery 源码整体架构，打造属于自己的 js 类库](https://juejin.im/post/5d39d2cbf265da1bc23fbd42)<br>
[面试官问：JS的继承](https://juejin.im/post/5c433e216fb9a049c15f841b)<br>
[面试官问：JS的this指向](https://juejin.im/post/5c0c87b35188252e8966c78a)<br>
[面试官问：能否模拟实现JS的call和apply方法](https://juejin.im/post/5bf6c79bf265da6142738b29)<br>
[面试官问：能否模拟实现JS的bind方法](https://juejin.im/post/5bec4183f265da616b1044d7)<br>
[面试官问：能否模拟实现JS的new操作符](https://juejin.im/post/5bde7c926fb9a049f66b8b52)<br>

## 关于

作者：常以**若川**为名混迹于江湖。前端路上 | PPT爱好者 | 所知甚少，唯善学。<br>
[个人博客-若川](https://lxchuan12.cn/posts/)，使用`vuepress`重构了，阅读体验可能更好些<br>
[掘金专栏](https://juejin.im/user/57974dc55bbb500063f522fd/posts)，欢迎关注~<br>
[`segmentfault`前端视野专栏](https://segmentfault.com/blog/lxchuan12)，欢迎关注~<br>
[知乎前端视野专栏](https://zhuanlan.zhihu.com/lxchuan12)，欢迎关注~<br>
[github blog](https://github.com/lxchuan12/blog)，相关源码和资源都放在这里，求个`star`^_^~

## 欢迎加微信交流 微信公众号

可能比较有趣的微信公众号，长按扫码关注。也可以加微信 `lxchuan12`，注明来源，拉您进【前端视野交流群】。

![若川视野](https://github.com/lxchuan12/blog/raw/master/docs/about/wechat-official-accounts-mini.jpg)
