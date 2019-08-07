 ### 背景介绍
 
`axios` 作为一个在前端领域非常重要的 `HTTP` 请求库，不管是 `Vue`、`React` 还是 `Node`，甚至任何 `JS` 可以运行的地方，都有可能出现，可以说，`axios` 是前端绕不过去的一个槛，掌握 `axios` 的原理、对我们日常开发和技术学习有着非常重要的意义。
    
同时，`axios` 源码中，有着非常优秀的设计思想，是每一个 前端开发 都应该去学习和借鉴的，了解其中的原理并将其运用到我们代码中，是一件非常 `cool` 的事

### 系列计划
    
    1. 分析 axios 的每一个 feature，并在这过程中，使用 typescript 重写 axios 库（不包含 Node 端实现）
    2. 添加 微信小程序、支付宝小程序 adapter，并简化 axios 一些过于鸡肋的功能（例如 axios.all axios.spread）
    3. 完整的测试用例覆盖，确保 新的 axios 库能够稳定的运行在生产环境
    4. 将新的axios 库发布到 npm 上，供大家使用

###  axiox 介绍

> Promise based HTTP client for the browser and node.js 

    axios 是一个基于Promise 用于浏览器和 nodejs 的 HTTP 客户端，其主要也是对 XHR 基于 Promoise 
    的封装，所以它不兼容 ie9 及以下，在 ie9 中我们需要引入 ployfill

axios 主要的功能点有：

* 浏览器中创建 XMLHttpRequests
* Node中创建 http 请求（暂不实现）
* 支持 Promise
* 支持拦截请求和响应
* 转换请求数据和响应数据
* 取消请求
* 自动转换 JSON 数据
* 客户端支持防御 XSRF
* ...

### 核心图示

为了避免 brothers 说我标题党，我在这里就将个人对 axios 源码的理解的图片贴出来，后面
大家可以看着这张图，去看我的代码，更能容易理解 axios 的原理和设计思想。

<p style="color: #e4393c">因为一张图片太大，所以这里分成多张，需要完整图片的 brothers， 可以私信联系我</p>

![](https://user-gold-cdn.xitu.io/2019/8/6/16c66fd44edb34c4?w=2104&h=1432&f=png&s=191608)
![](https://user-gold-cdn.xitu.io/2019/8/6/16c66fa776379a20?w=2840&h=1444&f=png&s=213453)
![](https://user-gold-cdn.xitu.io/2019/8/6/16c66fb3a6424d89?w=2398&h=1540&f=jpeg&s=164822)
![](https://user-gold-cdn.xitu.io/2019/8/6/16c66fb1243304ec?w=2444&h=1058&f=jpeg&s=158654)

### 搭建环境

使用 typescript 来开发库，我们需要一个脚手架来帮我们做编译打包工作，我推荐大家使用 非常优秀的开源库 
 [typescript-library-starter](https://github.com/alexjoverm/typescript-library-starter) , 只需要简单的几步，我们就能搭建起一个非常完善的项目，不管是编译 ts 到 js，还是测试工具 [typescript-library-starter](https://github.com/alexjoverm/typescript-library-starter) 都已经为我们做好了，十分推荐使用 
 
 1. 开始使用
 
```bash

git clone https://github.com/alexjoverm/typescript-library-starter.git projectName

cd projectName

# Run npm install and write your library name when asked. That's all!
npm install
```

2. 项目名

在 npm install 时，脚手架会询问你项目名称，你只需要输入个人的项目名称，脚手架会自动更改项目内部的名称，帮我们把一切搞定
 
 
当我们安装好依赖后，我们会得到一些代码和配置文件，国际惯例，src 就是我们要编写代码的文件夹，那么，就让我们一起愉快的 coding 吧

### 开发前准备
在开发过程中，我们要不断的验证我们开发的结果，而不是一口气写完再回头去写测试用例，那么为了我们能够实时验证结果，我们就需要实时编译我们的代码，然后去测试，axios 源码是 js 所写，也不需要编译，但我们要将 ts 编译成 js，所以我们需要使用到配置中的 rollup 来做, 同时需要启动一个node进程，提供我们开发使用的接口,这里选择使用 express，并且用
express-static 将我们的 examples 文件夹添加静态资源访问

1. 编写测试接口服务
```javascript
// 项目根目录下新建 server.js
const express = require('express');
const app = express();

app.use(express.static(__dirname + '/examples'))

app.listen(3000, () => console.log('dev server listening on port 3000!'));
```

2. 将编译好的代码引入测试页面，我们需要在 rollup.config.js 中添加一段代码

```javascript
// rollup.config.js
output: [
    { file: pkg.main, name: camelCase(libraryName), format: 'umd', sourcemap: true },
    { file: pkg.module, format: 'es', sourcemap: true },
    // 这里是将编译后的代码生成一份到 examples 中，供我们测试使用
    { file: './examples/iny-request.js', name: camelCase(libraryName),  format: 'umd', sourcemap: true },
  ],
```
3. 在启动 rollup 监听编译时，启动测试服务

```javascript
// rollup.config.js
if (process.env.NODE_ENV === 'development') {
  require('./server')
}
```
4. 安装 cross-env，修改 package.json 中 start 命令，设置环境变量

```javascript
// 终端
npm install cross-env --save-dev

//package.json
"start": "cross-env NODE_ENV=development rollup -c rollup.config.ts -w",
```

到这里，我们的开发环境搭建完成，其实这里测试用例我们可以使用 webpack 或者 gulp 来做一些实时编译，直接引入我们开发中的代码进行测试，会对调试和soucemap支持的更好。但这不是这次的重点，所以这里就取巧是使用了 rollup 编译后的代码进行测试，后面我们可以对这里在进行优化

### 开始开发

#### 实现简单的 xhr 请求

作为整个系列开发的第一步，我们从最简单的 xhr 请求做起，现在我们开始开发一个可以发送请求的 xhr 函数

```typescript
function xhr (options: any): Promise<any> {
  return new Promise((resolve, reject) => {

    const {
      url,
      method,
      data
    } = options

    const request = new XMLHttpRequest()

    request.open(method, url, true)

    request.send(data)
  })
}
```

axios 是基于原生 Promise 开发的，所以我们这里直接使用原生 Promise，我们的 xhr 函数需要返回一个 Pormise对象，我们直接 new Promise 返回，至于 xhr的内容，相信大家都懂，不太熟悉的可以去 [MDN](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/open)去了解一下，然后，大家可以思考一下，一个最简单的请求，会有那些最基本的参数，比如 url、method、data 这些,所以我们就可以简单的写出我们的 xhr 函数了.

这里，我们可以大概的定义出我们发起请求需要的参数类型了，因为是使用 typescript 开发，所以我们后面会定义大量的类型，现在我们定义出我们的第一个类型吧


```typescript

interface AxiosRequestConfig {
    // url 必须有，发起请求的地址
    url: string,
    // 请求方法，可有可无, axios 默认 get
    method?: string,
    // 请求数据，可有可无，同时类型不太确定
    data?: any
}

```

然后在我们的项目中使用这个类型，同时使用我们刚刚所写的 xhr 函数

```typescript
// iny-request.js

import xhr from './xhr'
import { AxiosRequestConfig } from './types';

function axios (config: AxiosRequestConfig): Promise<any> {

  return xhr(config)
}

export default axios

// xhr.js
import { AxiosRequestConfig } from "./types";

export default function xhr (options: AxiosRequestConfig): Promise<any> {
  return new Promise((resolve, reject) => {

    const {
      url,
      method,
      data
    } = options

    const request = new XMLHttpRequest()

    request.open(method!, url, true)

    request.send(data)
  })
}
```
为了避免后续文章过于冗余，所以文字介绍会对应的少一些，代码会多一些，所以大家有什么疑问，可以在评论中留言

#### xhr函数测试

1. 编写测试需要用到的接口

```javascript
const express = require('express');
const bodyParser = require('body-parser')

const app = express();

app.use(express.static(__dirname + '/examples'))

app.use(bodyParser.json())

const router = express.Router()
app.use(router)

router.get('/base/test', function (req, res) {
  console.log(req.body, req.query)
  res.json({ code: 200, msg: 'success' })
})

```
2. 编写测试代码

```html
<script>
    inyRequest({
      url: '/base/test',
      method: 'GET',
      params: { a: 1 },
      data: { b: 1}
    }).then(res => {
      console.log(res)
    }).catch(err => {
      console.log(err)
    })
  </script>
```
3. 检查请求结果


![](https://user-gold-cdn.xitu.io/2019/8/7/16c67b1635b1521c?w=1526&h=1064&f=jpeg&s=216508)

这里请求成功，但是 params 和 data 没有添加到 url 和 body 中，那么接下来，我们来处理 params 

#### url、params、data 的处理

1、url 和 params 处理

我们新建 helpers 文件夹，专门来放置我们的工具函数，新建 buildURL.ts 文件，来专门处理 url

```typescript
  export function buildURL(url: string, params?: any, paramsSerializer?: (params: any) => string) {
  // 没有参数，直接返回 url
  if (!params) return url

  let resultURL: string = ''

  // axios 这里直接每一个请求直接传入一个 paramsSerializer 函数，用来格式化参数的
  // 所以这里我们先判断是否存在这个函数，有的话直接调用即可
  if (paramsSerializer) {

    resultURL = paramsSerializer(params)
  } else if (isURLSearchParams(params)) {
    // params 也有可能传入一个URLSearchParams 类型的数据，如果是，我们直接 toString 即可
    resultURL = params.toString()
  } else {
    // 这里就需要对普通对象、数组来进行处理了
    const part: string[] = []
  

    Object.keys(params).forEach(key => {
      let value = params[key]
  
      // 过滤掉 null 和 undefined
      if (value === null || typeof value === 'undefined') {
        return
      }
  
      // 判断是否是数组, 数组需要将参数格式化为 'a[]=1&a[]=3&a[]=4' 这种
      if (Array.isArray(value)) {
        key += '[]'
      } else {
  
        // 这里将不是 数组的 数组转换为数组，方便后面循环处理
        value = [value]
      }
  
      // 循环将 k v 处理成 ['a=1','b=2']，并放置于 part 容器中
      value.forEach((val: any) => {
  
        // 这里是 axios 会讲 k 和 v 通过 url 编码，并且将一些特殊字符编译回来
        part.push(`${encode(key)}=${val}`)
      })
    })
  
    // 将 ['a=1','b=2'] 处理成 'a=1&b=2' 格式
    resultURL = part.join('&')
  }


  if (resultURL) {
    // 针对带有 # 哈希的 url 进行处理
    const index = url.indexOf('#')
    if (index !== -1) {
      url = url.slice(0, index)
    }

    url += (url.indexOf('?') === -1 ? '?' : '&') + resultURL
  }
  
  return url
}
```

大家在上面看到了，我们 有个工具函数，是用来判断是否是 URLSearchParams 类型的数据，所以我们在 helpers 下面新建 utils.ts 来放置我们的一些工具函数，然后来写我们判断 URLSearchParams 类型数据的函数

```typescript
/*
  *  在 ts 中，我们如何来写一个判断类型的函数呢
  *  有两种方法，第一个种是返回一个 boolean 类型，告诉我们 true 或者 false
  *  第二个就是这种 value is 类型 的写法
  *  更为推荐第二种
*/


export function isURLSearchParams (value: any): value is URLSearchParams {

  // 我们这里使用 instanceof 来判断 URLSearchParams 的类型
  return typeof value !== 'undefined' && value instanceof URLSearchParams
}
```

这里，我们将 buildURL 函数已经写好来，我们把它用到我们的xhr函数中

```typescript
import { AxiosRequestConfig } from './types'
import { buildURL } from './helpers/buildUrl';

export default function xhr(options: AxiosRequestConfig): Promise<any> {
  return new Promise((resolve, reject) => {
    const { url, method, data, params } = options

    const request = new XMLHttpRequest()

    // 这里使用我们的buildURL
    // 大家可能注意到我们这里的 method 后面，添加了一个 ! ,那是因为在我们的
    // AxioxRequestConfig 中，method 不是必须的参数，但这里是必须有的，所以我们在这里断言
    // method 肯定不为空，在 ts 中可以在一个值后面，使用 !来断言一个值不为空

    request.open(method!, buildURL(url, params), true)

    request.send(data)
  })
}
```
#### 测试 buildURL 函数

我们已经将 buildURL 函数写好了，我们需要来测试一下


