---
title: express-session 文档翻译
date: 2017-03-09 07:23:21
tags:
- Express
- Node.js
- 前端
---

> [文档原文](https://www.npmjs.com/package/express-session)
> express-session版本：1.15.1

<!-- more -->

## 安装

该模块是通过 npm 仓库进行管理的 Node.js 模块，可使用 `npm install` 命令进行安装：

```bash
$ npm install express-session
```

---

## 接口

```javascript
var session = require('express-session');
```

### session(options)

根据传入的 options 创建 session 的实例。

**注意** 保存到 cookie 里面的数据并非 session 本身，只是 session ID，session 的内容保存在服务端。

**注意** 1.5.0 版本之后， `cookie-parser` 中间件不再是本模块的依赖环境，模块直接在 req/res 中读写 cookies。新版本中如果 `cookie-parser` 中的 `secret` 配置项和本模块中的 `secret` 配置项不一致的话会引发异常。

**警告** 本模块使用的默认服务器端存储介质是 MemoryStore （内存），这并不是为生产环境设计的。在很多情况下都会自动释放内存，也不能存储大量的数据，这意味着经常会出现异常。

想了解可以使用哪些介质来存储 Session 的话，可以查阅 与 [session 兼容的存储介质](https://www.npmjs.com/package/express-session#compatible-session-stores)

### Options

express-session 接受以下这些配置项。

#### cookie

设置保存 session ID 的 cookie 对象的属性，默认值是：

```javascript
{ path: '/', httpOnly: true, secure: false, maxAge: null }
```

以下是该配置项的一些可配置项。

##### cookie.domain-默认域名

指定 Domain Set-Cookie 属性的默认属性，不配置默认域名的话，大部分客户端会只在当前路径应用 cookie。

##### cookie.expires-期限

指定 Expires Set-Cookie 配置项的值为 Date 对象，在默认情况下该值为空，大部分浏览器会将这类 cookie 视为 非持续性cookie 并且在一些默认行为例如关闭浏览器的行为下删除该 cookie。

**注意** 如果 expires 和 maxAge 属性都有设置了值，最后被定义的那个配置项为生效项。

**注意** expires 这个配置项不应该被直接定义，大部分情况下使用 maxAge 配置项来替代。

##### cookie.httpOnly

指定 HttpOnly Set-Cookie 配置项的值为 布尔 值。当其为真，HttpOnly 值被设置，反之不被设置。默认该值为真。

**注意** 注意当该值设置为 true 的时候，兼容模式的客户端将不被允许使用客户端的 javascript 来通过 document.cookie 访问 cookie。

##### cookie.maxAge

指定该值的值为数字（单位毫秒）以计算 Expires Set-Cookie 属性。该值生效的原理是通过服务器的当前系统时间来计算出 Expires 的值。默认情况下该值没有内容。

**注意** 如果 expires 和 maxAge 属性都有设置了值，最后被定义的那个配置项为生效项。

##### cookie.path

指定 Path Set-Cookie 的值，该值默认为 /，域名的根目录。

##### cookie.sameSite

指定 SameSite Set-cookie 的值为 boolean 或者 string。

- true 将 SameSite 属性值设置为 Strict
- false 将不启用 SameSite 属性
- lax 将 SameSite 属性值设置为 Lax
- strict 和 true 效果一样。

关于 SameSite-cookies 安全机制的更多信息可以查看下面两个文章：

> [再见，CSRF：讲解set-cookie中的SameSite属性](http://bobao.360.cn/learning/detail/2844.html)
> https://tools.ietf.org/html/draft-west-first-party-cookies-07#section-4.1.1

**注意** 这个属性还没有得到很普遍的支持，将来也有可能会改变，这意味着大部分浏览器在适用该属性之前会忽略这个配置项。

##### cookie.secure

指定 Secure Set-Cookie 配置项的属性为 boolean 值，当为真时，启用 Secure 属性，反之不启用该属性。默认情况不启用 Secure 属性。

**注意** 当把 Secure 属性设置为真时，如果兼容客户端没有使用 https 协议进行连接的时候，将不能把 cookie 发回到服务器。

需要一提的是 Secure: true 是推荐属性，不过这需要和一个支持 https 的站点配合使用。https 属性对于 Secure cookie 来说是必须的。当启用该属性的时候，如果你使用 http 来进入你的网站的时候，cookie 将不会被保存。如果你的 node.js 是挂载在使用 secure: true 属性的代理下面的时候，你需要设置 express 的 trust proxy 属性：

```javascript
var app = express()
app.set('trust proxy', 1) // trust first proxy 
app.use(session({
  secret: 'keyboard cat',
  resave: false,
  saveUninitialized: true,
  cookie: { secure: true }
}))
```

想要只在产品上线的时候使用 secure cookie，在生产环境下不是用的话，可以使用基于 NODE_ENV 全局配置变量的代码逻辑：

```javascript
var app = express()
var sess = {
  secret: 'keyboard cat',
  cookie: {}
}
 
if (app.get('env') === 'production') {
  app.set('trust proxy', 1) // trust first proxy 
  sess.cookie.secure = true // serve secure cookies 
}
 
app.use(session(sess))
```

cookie.secure 配置项也可以设置为 auto，这样会自动适配当前链接的安全性，不过要小心当该网站同时使用 http 和 https 的时候，当 cookie 在 https 下被设置的时候，它在 http 的连接方式的时候会不可见。这在我们使用 express 的 trust proxy 设置的时候简化关于生产环境和上线环境的配置很有用。

#### genid

产生一个新 session ID 的函数，返回一个可以用作 session ID 的字符串。当你想在生成 session ID 的时候将某些值附加到 req 里，把 req作为第一个参数传入到该函数。

默认使用 uid-safe 库里面的值来生成 session ID。

**注意** 你应该为你的 session 生成一些具有唯一性的 ID，以避免冲突。

```javascript
app.use(session({
  genid: function(req) {
    return genuuid() // use UUIDs for session IDs 
  },
  secret: 'keyboard cat'
}))
```

#### name

在响应头中返回的保存 session ID 的 cookie 名。默认为 connect.sid。

**注意** 如果你有多个网络应用同时运行在同一个域名下，你可能需要甄别彼此保存 session 的 cookie 值，这时最好的方法是为不同的网络应用使用不同的 name 来保存 session ID。

#### proxy

当使用 Secure cookie 的时候信任 reverse proxy。

默认值是 undefined。

- true 使用 X-Forwarded-Proto 头部
- false 所有头部将被忽略，只有使用 TLS/SSL 连接的时候才允许连接。
- undefined 使用 express 的 trust proxy 设置

#### resave

强制将接收到的 session 保存到存储介质中，即使在本次请求过程中没有进行过更改。根据你使用的存储介质的不同，这可能会是必选配置项，不过有一种竞争条件是当客户端传来了两个并发请求都修改到 cookie 的时候，先修改到的 cookie 会被后修改到的 cookie 给覆盖，即使这个覆盖可能并不会改变原来的 cookie （这种情况的表现也跟你使用的存储介质是什么有关）。

该配置项默认值为 true，但是该选项的默认配置已经被弃用，默认值在将来版本可能会变更，所以最好按照你的需要去手动设置该值，通常情况下，你可能需要将其设置为 false。

我要怎么样才能知道这个选项对于我所使用的存储介质来说是不是必要的呢？最好的办法是去检查你使用的存储介质有没有实现 touch 方法。如果有，你可以放心的将之设为 resave: false。如果它没有实现 touch 方法并且会为你的 session 设置到期日期的话，你最好将该值设为 resave: true。

**注意** 关于[竞争条件(race condition)](http://blog.csdn.net/silentpebble/article/details/6900162)

#### rolling

强制在每一个响应头中设置存储 session ID 的 cookie。失效日期的设置会根据 maxAge 属性重算。

该属性的默认值为 false。

**注意** 当本属性设置为 true 而 saveUninitialized 设置为 false 的时候，未初始化的 session 的 cookie 将不会在响应的时候被保存。

#### saveUninitialized

强制保存将那些未“初始化”的 session。session 在它被新建但是还没有设置属性的时候是被视为“未初始化”。设置为 false 对于实现登录 session 的时候很有用，并可以减少存储空间的使用比例，同时也是遵循了在设置 cookie 之前需要得到允许的原则，同时也有利于处理客户端在没有 session 的情况下发出多个并发请求的情况。

该选项的默认值为 true，但使用默认选项已经被弃用，默认值在将来版本可能会变更，所以最好按照你的需要去手动设置该值。

**注意** 如果你使用的 session 是和 PassportJS 进行关联的话，Passport 会在用户进行验证之后为 session 添加一个空的 Passport 对象。这会被视为为 session 进行了初始化，从而引发该 session 的保存。这在 PassportJS 0.3.0 中已经被修复。

#### secret

必填项

这是用来为保存 session ID 的 cookie 加密的秘钥，该选项可以是一个秘钥字符串，也可以是一个保存复数秘钥的数组。如果传入的是秘钥数组的话，只有第一个元素会被用来加密保存 session ID 的cookie，不过所有的元素都会在校验请求中的 cookie 的时候被使用。

#### store

session 保存的实例，默认是保存在内存中。

#### unset

控制如何处理关于 req.session 的取消设置请求（通过 delete 方法，或者将之设置为 null之类的）

该选项的默认值为 keep。

- ‘destroy’ session 在请求结束的时候会被销毁(删除)
- ‘keep’ 存储空间中的 session 会被保留，不过在请求过程中对其作出的更改将被忽略且不被保存。

### req.session

存储或者访问 session，通过请求的参数进行调用 req.session，通常使用 json 作为数据格式，下面的例子是保存用户的视图计数器。

```javascript
app.use(session({ secret: 'keyboard cat', cookie: { maxAge: 60000 }}))
 
// Access the session as req.session 
app.get('/', function(req, res, next) {
  var sess = req.session
  if (sess.views) {
    sess.views++
    res.setHeader('Content-Type', 'text/html')
    res.write('<p>views: ' + sess.views + '</p>')
    res.write('<p>expires in: ' + (sess.cookie.maxAge / 1000) + 's</p>')
    res.end()
  } else {
    sess.views = 1
    res.end('welcome to the session demo. refresh!')
  }
})
```

#### Session.regenerate(callback)

调用本方法来初始化 session，当本方法执行完毕时，新的 SID 和 session 实例会初始化到 req.session 然后执行回调函数。

```javascript
1
2
3
req.session.regenerate(function(err) {
  // will have a new session here 
})
```

#### Session.destroy(callback)

删除 session 并且解除 req.session 属性。当执行完成时，执行回调函数。

```javascript
req.session.destroy(function(err) {
  // cannot access session here 
})
```

#### Session.reload(callback)

从存储介质中重新读取 session 数据并且挂载在 req.session 属性中，执行完成时执行回调函数。

```javascript
req.session.reload(function(err) {
  // session updated 
  // 应该是异步方法，所以在回调中才能读到新的有效的session
})
```

#### Session.save(callback)

将 session 保存回存储介质中，将内存中的 session 内容替换到存储介质中的对应内容（虽然存储介质可能会做点其他事情，查看你所使用的存储介质的文档看看有哪些额外的行为）

如果 session 被修改过的话，这个方法会在 http 的响应逻辑的结尾被自动调用。（尽管这个行为会被中间体的构造函数中的多个选项影响到）所以这个方法通常不需要手动调用。

有几个情况手动调用本函数是有用的，比如 websockets 中的长持续请求。

```javascript
req.session.save(function(err) {
  // session saved 
})
```

#### Session.touch()

更新 session 的 .maxAge 属性，通常这个函数是没有必要手动调用的，session 中间件会为你处理这个事情。

#### req.session.id

每个 session 都有一个唯一的 ID 与它关联，本属性包含了 session ID 并且无法被改写。

#### req.session.cookie

每个 session 都有一个唯一的 cookie 对象，我们可以为不同的用户改变他们的 session cookie。例如我们可以设置 req.session.cookie.expires 为 false 以使 cookie 只在用户代理的持续时间内保留。

#### Cookie.maxAge

req.session.cookie.maxAge 会以毫秒为单位返回 cookie 的剩余时间，以使我们可以根据情况调整它的 .expires 属性，以下两条语句的作用是相同的。

```javascript
var hour = 3600000
req.session.cookie.expires = new Date(Date.now() + hour)
req.session.cookie.maxAge = hour
```

假设 maxAge 属性被设置为 60000(一分钟)，在 30 秒之后，除非本次请求结束了，该属性的值会更新为 30000，也就是在那个时候自动调用了 req.session.touch() 的原因。

```javascript
req.session.cookie.maxAge // => 30000
```

#### req.SessionID

访问 req.SessionID 属性能够访问当前所加载的 session 的 ID，这是一个只读属性，只有在这个 session 被创建或是被加载了之后才能够获取到。

---

## Session Store Implementation - session 存储介质的实现方案

Session 存储方案必须是以 EventEmitter 为骨架，并且实现了一些具体的方法，下面是要实现的一些方法列表，分别有标注为 必须实现、推荐实现 和 可选实现。

- 必须实现：这种方法是模块会一直都调用到的方法。
- 推荐实现：这种方法是如果存储介质有实现的话就会调用到的方法。
- 可选实现：这种方法模块不一定会调用，但是能够帮助用户实现呈现统一的存储介质。

例如 connect-redis 的实现：

### store.all(callback)

可选实现

该可选实现能够获取存储中的所有 session 并以数组形式返回。回调函数的形式应该为：callback(error, sessions)

### store.destroy(sid, callback)

必须实现

该必须实现是通过传入的 sid 来删除存储介质中的 session 的。当 session 被删除了之后，回调的形式应该为 callback(error)。

### store.clear(callback)

可选实现

该可选实现是用于将存储介质中的所有 session 都删除。回调函数的形式应该为：callback(error)。

### store.length(callback)

可选实现

该可选实现是用于获取存储介质中 session 的数量，回调函数的形式应该为：callback(error, len)。

### store.get(sid, callback)

必须实现

该必须实现是根据传入的 sid 获取指定的 session，回调函数的形式应该为：callback(err, session)。

如果找不到指定的 session 的话，返回的 session 参数值为 null 或者 undefined (并且无发生错误信息 error)，一种特殊情况是当发生 error.code === 'ENOENT' 时，返回将是 callback(null, bull)。

### store.set(sid, session, callback)

必须实现

该必须实现是根据传入的 sid 和 session 来更新存储介质中对应的 session 。回调函数的形式应该为：callback(error)。

### store.touch(sid, session, callback)

推荐实现

该推荐实现是根据传入的 sid 和 session 来更新指定 session 的状态，回调函数的形式应该为：callback(error)。

当存储介质打算自动删除那些空闲的 session 的时候，该方法可以用来标记指定的 session，告诉存储介质该 session 是激活状态，将其定时器复位。

---

## 兼容的 session 存储介质

以下的[列表](https://www.npmjs.com/package/express-session#compatible-session-stores)是跟本模块兼容的实现了作为 session 存储介质。
