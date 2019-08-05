 ### 背景介绍
 
`axios` 作为一个在前端领域非常重要的 `HTTP` 请求库，不管是 `Vue`、`React` 还是 `Node`，甚至任何 `JS` 可以运行的地方，都有可能出现，可以说，`axios` 是前端绕不过去的一个槛，掌握 `axios` 的原理、对我们日常开发和技术学习有着非常重要的意义。
    
同时，`axios` 源码中，有着非常优秀的设计思想，是每一个 前端开发 都应该去学习和借鉴的，了解其中的原理并将其运用到我们代码中，是一件非常 `cool` 的事

### 系列计划
    
    1. 分析 axios 的每一个 feature，并在这过程中，使用 typescript 重写 axios 库
    2. 添加 微信小程序、支付宝小程序 adapter，并简化 axios 一些过于鸡肋的功能
    3. 完整的测试用例覆盖，确保 新的 axios 库能够稳定的运行在生产环境
    4. 将新的axios 库发布到 npm 上，供大家使用

###  axiox 介绍

> Promise based HTTP client for the browser and node.js 

    axios 是一个基于Promise 用于浏览器和 nodejs 的 HTTP 客户端，其主要也是对 XHR 基于 Promoise 
    的封装，所以它不兼容 ie9 及以下，在 ie9 中我们需要引入 ployfill

axios 主要的功能点有：

* 浏览器中创建 XMLHttpRequests
* Node中创建 http 请求
* 支持 Promise
* 支持拦截请求和响应
* 转换请求数据和响应数据
* 取消请求
* 自动转换 JSON 数据
* 客户端支持防御 XSRF
* ...

### 核心图示

为了避免 brothers 说我标题党，我在这里就将浓缩了 aioxs 源码的图片放出来，后面
大家可以看着这张图，去理解我的代码，更能容易理解....




