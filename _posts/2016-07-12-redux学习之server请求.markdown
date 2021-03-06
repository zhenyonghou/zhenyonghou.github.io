---
layout: post
date:   2016-07-12 23:00
categories: blog
---

### 1. 介绍

在redux的Example代码中有个网络请求的serverAPI，本文是讲解一下serverAPI。

serverAPI的工作：

1. 自身作为一个Middleware会被加入Middleware链，会判断action的SERVER_API标识，如果是属于SERVER_API的action，就处理，如果不是，放行。

2. serverAPI使用fetch做异步请求，请求前发request action，获得服务器返回的结果后发起成功或失败的action。


Middleware是啥？

Middleware是用来处理发起action之后的动作的，当调用dispatch发起一个action的时候，会遍历Middleware链。Middleware能在reducer之前截获到自己关心的action并做一些处理。关于Middleware可以看这个教程：

http://cn.redux.js.org/docs/advanced/Middleware.html



### 2. 构造个serverAPI能处理的网络请求

先来看看如何向服务器端发送请求的。下面fetchUserinfo请求为例。fetchUserinfo()是个action，请求接口的时候会dispatch(fetchUserinfo())。

SERVER_API是个Symbol，后面会看到。

types, url, param, method, normalizeFunc是必须要有的，serverAPI会处理这些。

{% highlight javascript %}

import { SERVER_API } from '../middleware/serverApi'

export function fetchUserinfo() {
  return {
    [SERVER_API]: {
      types: [ActionType.USERINFO_REQUEST, ActionType.USERINFO_SUCCESS, ActionType.USERINFO_FAILURE],
      url: config.apiDomain + '/account/userinfo',
      param: JSON.stringify({
        token: config.testToken
      }),
      method: 'POST',
      normalizeFunc: json => json
    }
  }
}

...

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    loadUserinfo: () => {
      dispatch(fetchUserinfo())
    }
  }
}

{% endhighlight %}

注意[SERVER_API]的写法，后面的实现中这么用的：action[SERVER_API] 获取action数据。


### 3. serverAPI中间件

{% highlight javascript %}

function callServerApi(url, param, method, normalizeFunc) {
  return fetch(url, {method: `${method}`, body: param})
    .then(response =>
      response.json().then(json => ({ json, response }))
    ).then(({ json, response }) => {
      if (!response.ok || json.ret !== 1) {
        return Promise.reject(json);
      }
      return normalizeFunc(json);
    })
}

{% endhighlight %}

callServerApi函数是发网络请求的，用到了fetch方法，最后将结果用normalizeFunc做扁平化处理。

关于fetch使用参考这里：

https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch


{% highlight javascript %}

export const SERVER_API = Symbol('Server API')

export default store => next => action => {     // middleware都这样写，里边再使用next方法
  const serverAPI = action[SERVER_API]
  if (typeof serverAPI === 'undefined') {       // 由于是middleware，所以所有的diapatch的action都要经过此函数，在这里判断serverAPI过滤下。
    return next(action)
  }
  let {types, url, param, method, normalizeFunc} = serverAPI;     // 从serverAPI取出各个字段，如果不知道各字段代表啥含义可以参看上面的用法
  if (typeof url !== 'string') {
    throw new Error('Specify a string url.')
  }
  if (typeof param !== 'string') {
    throw new Error('Specify a string param.')
  }
  if (!Array.isArray(types) || types.length !== 3) {
    throw new Error('Expected an array of three action types.')
  }
  if (!types.every(type => typeof type === 'string')) {
    throw new Error('Expected action types to be strings.')
  }
  function actionWith(data) {
    const finalAction = Object.assign({}, action, data)
    delete finalAction[SERVER_API]                                 // 注意，这里移除了[SERVER_API]数据，原因是这些数据已经没用了
    return finalAction
  }
  const [ requestType, successType, failureType ] = types          // types是一组action的type，包含这3个type。
  next(actionWith({ type: requestType }))                          // 在请求发出前先diapatch个requestType

  return callServerApi(url, param, method, normalizeFunc).then(    // 调用请求成功、失败的处理，注意这里用了then
    response => next(actionWith({
      response,
      type: successType
    })),
    response => next(actionWith({
      type: failureType,
      response,
      msg: response.msg || response.message || 'Something bad happened',
    }))
  )
}

{% endhighlight %}

疑问：
1. 调用callServerApi函数的后面为什么还能跟then? fetch返回的对象后面还可以跟then?


（完）


------

作者 [侯振永][1]     
写于2016 年 7月 12日

[1]: https://zhenyonghou.github.io/