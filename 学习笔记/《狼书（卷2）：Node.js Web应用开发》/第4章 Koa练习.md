# 第4章 Koa练习
## 4.5 API开发
### 4.5.1 API的简单写法
一个基于通用约定实现的模块：koa.res.api（使用之前需要安装，狼书自己写的）
#### 使用方法
koa.res.api是一个Koa中间件，需要将它挂载在ctx对象上
```javascript
var Koa = require('koa')
var app = new Koa()
var res_api = require('koa.res.api')

app.use(res_api())
```
#### 调用方式
##### 方式1：直接返回API接口
```javascript
// 代码
ctx.api(404, err, {
    code: 1,
    msg: 'delete failed'
})

// 响应头
HTTP/1.1 404 Not Found
Content-Type: application/json; charset=utf-8
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PATCH ,DELETE
Access-Control-Allow-Headers: X-Requested-With,content-type, Authorization
Content-Length: 77
Date: Sat, 02 May 2020 04:15:51 GMT
Connection: keep-alive

// 响应数据
{
  "data": err,
  "status": {
    "code": 1,
    "msg": "delete failed"
  }
}
```

##### 方式2：返回带有状态的json数据
挡状态码为200的时候可以不加状态，默认会带`{ code : 0, msg  : 'request success!' }`的状态
```javascript
// 代码
ctx.api(data, {
    code: 1,
    msg: 'delete failed'
})

// 响应头
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PATCH ,DELETE
Access-Control-Allow-Headers: X-Requested-With,content-type, Authorization
Content-Length: 139
Date: Sat, 02 May 2020 04:29:03 GMT
Connection: keep-alive

// 响应数据
{
  "data": data,
  "status": {
    "code": 1,
    "msg": "delete failed"
  }
}
```

##### 方式3：返回JSON API
```javascript
// 代码
ctx.api(data)

// 响应头
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PATCH ,DELETE
Access-Control-Allow-Headers: X-Requested-With,content-type, Authorization
Content-Length: 142
Date: Sat, 02 May 2020 04:32:14 GMT
Connection: keep-alive

// 响应数据
{
  "data": data,
  "status": {
    "code": 0,
    "msg": "request success!"
  }
}
```

##### 方式4：异常情况下返回错误结果
```javascript
// 代码
ctx.api_error(error)

// 响应头
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PATCH ,DELETE
Access-Control-Allow-Headers: X-Requested-With,content-type, Authorization
Content-Length: 112
Date: Sat, 02 May 2020 04:34:26 GMT
Connection: keep-alive

// 响应数据
{
  "data": err,
  "status": {
    "code": -1,
    "msg": "api error"
  }
}
```

### 4.5.2 响应处理
响应处理的两种方法
#### Lodash的`_.get`方法
根据对象路劲获取值，如果获取到的值是undefined。则会赋予解析结果默认值，有效减少异常处理
```javascript
const _ = require('lodash')
const object = {a: [{b: {c: 3}}]}
const c = _.get(object, "a[0].b.c", 1)
console.log(c) // 3
const d = _.get(object, "a[0].b.c", 1)
console.log(d) // 1
```
#### 使用Typescript
Typescript作为一门静态类型语言，提前了类型检查的时机。在TypeScript粒，接口的作用是为这些类型命名以及为代码定义契约。根据接口信息，可以对值所具有的类型进行类型检查。开启--strictNullChecks选项会启用新的严格空值检查模式

### 4.5.3 RESTful API
- REST是REpresentational State Transfer的缩写，可以翻译成“表现状态转换”，但是在绝大多数场合中只说REST
- 符合REST设计标准的API就是RESTful API
- REST架构设计遵循的各项标准和准则就是HTTP的表现，换句话说HTTP就是满足REST架构的协议模式
- REST的5个关键词资源、表述、状态转移、统一接口、超文本驱动（在《RESTful Web APIs中文版》中有介绍）
- URI（Uniform Resource Locator）的意思是统一资源定位符，用于描述Web资源所在的位置。RESTful API是以HTTP为依托，还原了URI的本质

### 4.5.4 API访问鉴权
#### 保障API接口的安全性上，需要遵循以下原则：
- 有调用者身份
- 请求具有唯一性
- 请求的参数不能被篡改
- 请求有效时间，即API对应的令牌（Token）的有效期要长一些

#### 常见的鉴权
目前流行的鉴权方式有两种：JSON Web Tokens（JWT）、OAuth

##### JWT鉴权
通过原始的JWT API进行签名和校验的示例如下：
```javascript
const jwt = require('jsonwebtoken')
const secret = '17koa.com'

var token = jwt.sign({
    data: {
        user_id: 10000,
        user_name: 'luwei',
        user_email: '1280380446@qq.com'
    }
}, sign, { expiresIn: '1h' })

// invalid token - synchronous
try {
    var decoded = jwt.verify(token, secret);
    console.log(decoded)
} catch (err) {
    // err
}
```
**服务端怎么获取客户端传过来的token呢？**
```javascript
    // 检查POST的信息、URL查询参数、头部信息
    const token = ctx.request.body.token || ctx.query.token || ctx.header['x-access-token']
```

##### OAuth鉴权
- OAuth是一个开放标准，允许第三方应用访问该用户在某一网站上存储的私密资源（如照片、视频等），而无需将用户名和密码提供给第三方应用
- OAuth目前主流的版本是2.0，OAuth 2.0是OAuth的下一版本，但是不兼容OAuth 1.0
- OAuth中包含两个方法：加密生成令牌、解密获取用户信息。加密的时候可以把过期时间加进去，如果对安全性要求特别高，可以把过期时间设置的短一些。如果想增加安全性，也可以叠加授权。
- OAuth 2.0的运行步骤如下
    1. 用户打开客户端以后，客户端要求用户给与授权
    2. 用户同意给与客户端授权
    3. 客户端使用上一步骤获得的授权，向认证服务器申请令牌
    4. 认证服务器对客户端进行认证以后，确认无误，同意发放令牌
    5. 客户端使用令牌，向资源服务器申请获取资源
    6. 资源服务器确认令牌无误，同意向客户端开发资源
- 显然自己实现OAuth流程需要写不少代码，而且每增加一个API Provider过程就得重复一遍。这时候应该借助适合的框架。可以通过oauth2-server模块来完成OAuth鉴权过程，几乎不需要修改各端SDK就可以使用

## 4.6 常用中间件
介绍4个有特色的的常见中间件，分别是会话、ETag、验证码和限制访问频率
- 会话是最常见的，用来保持客户端和服务端的状态
- ETag是Web缓存优化的常见中间件
- 验证码是采用OTP封装的中间件
- 限制访问频率是对抗暴力破解的有效手段

### 4.6.1 会话（session）
- HTTP本身是无状态协议，但是有时我们需要记录用户的状态，这时候就需要会话
- 会话是一种记录客户状态的机制，和Cookie类似，不同的是Cookie保存在客户端浏览器中，而会话保存在服务器上
- 当客户端浏览器访问服务器的时候，服务器会把客户端信息已某种形式记录下来，这就是会话；当客户端再次访问服务器时，只需从该会话中查找用户状态即可
- Koa中和会话相关的模块
    1. koa-session: 基于Cookie的简单会话实现
    2. koa-generic-session: Session Store的抽象层，目标是让会话能够存储在Redis或者MongoDB等自定义持久化存储中。它内置了Memory Store，即内存存储。如koa-redis是基于Redis存储，koa-generic-session-mongo是基于MongoDB存储的

#### 基于Redis的会话实现代码如下
以下代码有三个要点：
1. 依赖Redis，因此要先启动Redis服务器
2. 通过ctx.session进行会话处理
3. TTL是Session Store的超时时间，这个值一般是30分钟
```javascript
const Koa =require('koa')
const session = require('koa-generic=session')
const RedisStore = require('koa-redis')

const app = new Koa()
app.keys = ['keys', 'keyskeys']

// 加入全局中间件
app.use(session({
    store: new RedisStore(),
    ttl: 30 * 60 * 1000 // 半小时
}))

// 在路由中可以直接通过ctx.session对后面的中间件进行操作
app.use(ctx => {
    switch(ctx.path) {
        case '/get':
            ctx.session.user = { name: 'luwei' }
            ctx.body = ctx.session.user
            break
        case '/remove': 
            ctx.session = null
            ctx.body = 'removed'
            break
    }
})

app.listen(8080)
```
Session Store其实就是将会话存储在不同的持久化存储中以后抽象出来的通用层，其基本的操作是存、取和销毁

### 4.6.2 ETag
- ETag是前端缓存优化的重要部分
- ETag在服务器端生成以后，客户端将通过If-Match或者If-None-Match条件判断请求来验证资源是否被修改，其中比较常用的是If-None-Match
- 如果资源被修改返回正常值，如果资源没有被修改返回304状态码
- 一般的静态HTTP服务器都会根据文件内容来判断文件是否被修改，进而决定是否需要给客户端返回新的内容

#### Koa中ETag功能的实现
Koa中使用koa-conditional-get和koa-etag模块来提供Etag功能，示例如下
```javascript
    var conditional = require('koa-conditional-get')
    var etag = require('koa-etag')
    
    // etag模块通常和koa-conditional-get模块一起使用
    app.use(conditional())
    app.use(etag())
    
    // 说明
    // 1. 这里定义的etag就是Koa中用于生产ETag的中间件
    // 2. ETag缓存是通过conditional-get拦截才能生效的
    // 3. koa-condotional-get一定要放在koa-etag前面
```
ETag的核心实现就是koa-etag模块。首先要回去entity，一般是ctx.body的内容，然后etag模块会计算出ETag的值，并将这个值赋给ctx.response.etag
```javascript
    var calculate = require('etag')
    ...
    ctx.response.etag = calculate(entity, options)
```

### 4.6.3 验证码
- OTP的全程为One-Time Password，也叫作动态口令。是根据专门的算法每隔60s生成的一个与时间相关的、不可预测的随机数字组合（即口令），每个口令只能使用一次，一天可以生成43200个口令
- OTP分为HOTP和TOTP两种；HOTP是基于加法计数器和静态对称秘钥的算法；TOTP是基于时间的一次性密码算法，是支持将时间作为动态因素的，基于HMAC一次性密码算法的扩展算法
- 对于生成短信验证码这种需求，使用上面两种都可以；如果要求60s内不能重新生成，则使用TOTP

#### OTP的实现
OTP的实现步骤如下：
1. 在一定时间范围（一般是60s）内生成有效且复杂的字符串
2. 对字符串进行散列计算
3. 将结果转换为6位整数
4. 让服务器与客户端保持时间、算法、Key同步一致

```javascript
    var notp = require('notp')
    
    var opt = {
        window: 0
    }
    
    var app = {
        encode: function(key) {
            return notp.totp.gen(key, opt)
        },
        decode: function(key, token) {
            var login = notp.totp.verify(token, key, opt)
            if(!login) {
                console.log('Token incalid')
                return false
            }
            return true
        }
    }
    
    module.exports = app
```
以上代码包含encode方法和decode方法，即采用TOTP进行加密解密，核心参数是key。那么怎么保证key的唯一性呢？很简单，我们的业务都是与手机号或者用户名进行绑定的，通过手机号就可以保证唯一性

对以上代码稍作封装，即可得到koa-otp的中间件代码
```javascript
module.exports = function(key) {
    return {
        encode: function(cb) {
            return function(ctx, next) {
                var token = app.encode(key)
                ctx.otp_token = token
                if(cb) {
                    cb(ctx, next)
                } else {
                    return next()
                }
            }
        },
        decode: function(token, cb) {
            return function(ctx, next) {
                ctx.otp_valid = app.decode(key, token)
                if(cb) {
                    cb(ctx, next)
                } else {
                    return next()
                }
            }
        }
    }
}
```
获取otp_token的代码如下
```javascript
var otp = require(koa-opt)("jsdfgjdhsfjsdhvfsd")

app.use(otp.encode(function(ctx, next){
    ctx.body = {
        token: ctx.otp_token,
        valid: ctx.otp_valid
    }
}))
```
OTP唯一的问题是每天最多只能产生43200个密码，而对于有些应用来说是不够的，因此需要自己再设计一套机制，将密码和相关的服务器进行绑定，将服务器和访问次数进行绑定。当然也可用Redis和MongoDB的TTL特性来实现

### 4.6.4 限制访问频率
经常会有比如限制访问频率的需求，比如限制验证码只能60s之后才能发一次，以减少短信开支

#### 利用Redis来实现
- 最简单也最好的办法是利用Redis的expire命令来实现
- 也可以通过MongoDB的TTL索引特性来实现TTL集合，TTL可以通过一个后台进程读取索引中的值。然后清除过期的集合
- Redis的expire的原理是在Redis里设置Key的值，经过一定的时间之后，这个值就会被删除。这样做的好处是，Redis是内存数据库，读写非常快，另外Key过期就会被删除，非常节省空间

设置缓存的方法如下：
```javascript
function cache_expire(k, v) {
    console.log('------ cache_expire------')
    if(client) {
        client.set(k, v, redis.print)
        // 60s过期
        client.expire(k, 1 * 60)
    } else {
        console.log('redis client instance is not exist')
    }
}
```
每次来请求都要先从缓存中查询一下，如果相应的Key存在就不做任何处理，如果不存在就发送短信，并将此Key保存到缓存中
```javascript
// 首先检测缓存中是否有tel的Key
client.get(tel, function(err, replay){
    if(replay) {
        console.log('已存在，不做任何处理' + replay.toString())
    } else {
        console.log('不存在，发送短信')
        cache_expire(tel, a)
    }
})

// 要点
// 1. cache_expire 可以设置Redis的Key
// 2. client.get可以检测缓存里是否有tel的Key
```
如果需要保存发送历史记录，可以通过日志保存，也可以通过库表保存，需要自己实现

#### 使用ratelimiiter模块来实现
ratelimiiter模块可以通过限制用户的链接频率来防止暴力破解类的攻击
```javascript
var Limiter = require('ratelimiter')
var email = req.body.email
var db = require('isredis').createClient()

var limit = new Limiter({id: email, db: db})

limit.get(function(err, limit) {

})
```
我们可以将ratelimiter封装成一个中间件来使用，而且ratelimiter中本身也有koa-ratelimiter这个现成的中间件
```javascript
var ratelimit = require('koa-ratelimit')
var redis = require('redis')
var Koa = require('koa')
var app = new Koa()

var email = ratelimit({
    db: new Redis(),
    duration: 60000,
    erroeMessage: 'Something Yu Just Have to Slow Down.',
    id: (ctx) => ctx.email,
    headers: {
        remaining: 'Rate-Limit-Remaining',
        reset: 'Rate-Limit-Reset',
        total: 'Rate-Limit-Total'
    },
    max: 10
})

var ip = ratelimit({
    db: new Redis(),
    duration: 60000,
    erroeMessage: 'Something Yu Just Have to Slow Down.',
    id: (ctx) => ctx.ip,
    headers: {
        remaining: 'Rate-Limit-Remaining',
        reset: 'Rate-Limit-Reset',
        total: 'Rate-Limit-Total'
    },
    max: 100
})

app.post('/login', ip, email, function(ctx, next){...})
```
这里就是限制了在一段给定时间内用户可以尝试登陆的次数，通过IP地址和E-mail这两个维度进行限制可以降低用户密码被暴力破解的风险