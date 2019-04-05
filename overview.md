# Express overview

## Install
```bash
npm install express --save
```
```js
const express = require('express');
const app = express();
const server = require('http').createServer(app);
const port = 3000;
server.listen(port);
```

## Routing
Routing是应用程序用来处理客户端请求的终点

### Route Methods
* 来源于http的请求方法，如：`GET,POST,PUT,PATCH,DELETE`，以响应不同的请求方式
* 可附加到`app`应用实例对象或路由实例对象`express.Router()`上
* 特殊的`all`方法，可以处理所有的http请求
```js
// GET method route
app.get('/', function (req, res) {
  res.send('GET request to the homepage')
})
// POST method route
app.post('/', function (req, res) {
  res.send('POST request to the homepage')
})
// all method route
app.all('/secret', function (req, res, next) {
  console.log('Accessing the secret section ...')
  next() // pass control to the next handler
})
```

## Route paths
* 可以是：`string,string pattern,regular expression`
* 字符：?, +, *, () 和正则的意思一样
* 字面量字符：-,.
* $字符：[\$] 需要转义并放在[]内
* query string: 不是路径的一部分
* 参照：[path-to-regexp](https://www.npmjs.com/package/path-to-regexp)

```js
// match: /random.text
app.get('/random.text', function (req, res) {
  res.send('random.text')
})

// mattch: acd, abcd
app.get('/ab?cd', function (req, res) {
  res.send('ab?cd')
})

// match: abcd, abbbcd
app.get('/ab+cd', function (req, res) {
  res.send('ab+cd')
})

// match: abcd, abxcd, abRANDOMcd, ab123cd
// 注意：这个和正则不一样，它把*看做单独字符定义匹配，而不是依附于它前面的字符，v5.0会修正这个问题, 之前可使用{0,}来代替
app.get('/ab*cd', function (req, res) {
  res.send('ab*cd')
})
// match: abcde, abe
app.get('/ab(cd)?e', function (req, res) {
  res.send('ab(cd)?e')
})
// match: butterfly, dragonfly
app.get(/.*fly$/, function (req, res) {
  res.send('/.*fly$/')
})
```

### Route parameters
* 路由参数可通过`req.params`获得
* 路由参数字符规则比如符合：`[A-Za-z0-9_]`
* 可以利用`-``.`来分割路径参数
* 可以在路由参数后面附加一个`()`，并在里面定义正则来修饰参数类型，注意，正则本就是可匹配路径的一部分，所以括号内的正则需要加转义
```js
app.get('/users/:userId/books/:bookId', function (req, res) {
  res.send(req.params)； // { "userId": "34", "bookId": "8989" }
})
app.get('/flights/:from-:to', function (req, res) {
  res.send(req.params)；// { "from": "LAX", "to": "SFO" }
})
app.get('/plantae/:genus.:species', function (req, res) {
  res.send(req.params)；// { "genus": "Prunus", "species": "persica" }
})
app.get('/user/:userId(\\d+)', function (req, res) {
  res.send(req.params)；// {"userId": "42"}
})
```

### Route handlers
* 路由处理函数可以是一个函数，函数数组或它们的组合。
* 注意：多个处理函数时，需要注意`next()`的使用
```js
// 单独
app.get('/example/a', function (req, res) {
  res.send('Hello from A!')
})
// 多个
app.get('/example/b', function (req, res, next) {
  console.log('the response will be sent by the next function ...')
  next()
}, function (req, res) {
  res.send('Hello from B!')
})
// 组合
var cb0 = function (req, res, next) {
  console.log('CB0')
  next()
}
var cb1 = function (req, res, next) {
  console.log('CB1')
  next()
}
app.get('/example/d', [cb0, cb1], function (req, res, next) {
  console.log('the response will be sent by the next function ...')
  next()
}, function (req, res) {
  res.send('Hello from D!')
})
```

### Response methods
* 下面的附属于`res`的响应方法，会结束客户端请求响应的生命周期
* 如果路由处理函数没有调用其中的方法，请求则会被挂起
```js
res.download() //下载文件
res.end()	 // 结束响应处理
res.json()	// 响应一个JSON
res.jsonp()	// 响应一个JSONP
res.redirect() // 跳转请求
res.render()	// 渲染一个视图模板，并将结果发送给客户端
res.send()	//  发送不同类型的响应
res.sendFile()	// 发送八进制流给客户端
res.sendStatus()	// 设置响应状态码和状态文本给响应体
```

### app.route()
* `app.route()`方法，可以针对单一路径来定义多个处理函数来响应不同的http请求
```js
app.route('/book')
  .get(function (req, res) {
    res.send('Get a random book')
  })
  .post(function (req, res) {
    res.send('Add a book')
  })
  .put(function (req, res) {
    res.send('Update the book')
  })
```

### express.Router()
* `express.Router()`实例是一个完整的中间件和路由系统
* 创建的路由模块可加载都`app`实例对象中
```js
// birds.js
var express = require('express')
var router = express.Router()
// 针对此路由的中间件 /birds
router.use(function timeLog (req, res, next) {
  console.log('Time: ', Date.now())
  next()
})
// 子路由 /birds/info
router.get('/info', function (req, res) {
  res.send('Birds home page')
})
module.exports = router

// app.js
var birds = require('./birds')
app.use('/birds', birds)
```

## Middleware pattern
```js
// 中间件模式
module.exports = function(options) {
  // options: custom config options
  return function(req, res, next) {
     // req: http request
     // res: http response
     // next: next() to next middleware
  }
}
```
分类：
* application-level middleware
* router-level middleware
* error-handling middleware
* built-in middleware
* third-party middlewar

### application-level middleware
应用级别的中间件绑定到一个应用实例`app`，通过使用`app.use()`或`app.METHOD()`方法，`METHOD`为HTTP请求方法:` GET,PUT,POST,DELETE,PATCH`。
```js
// 无装载路径，所有HTTP请求都会通过
app.use((req, res, next) => {
  console.log('Time:', Date.now());
  next();
});

// 有装载路径，符合路径规则的所有HTTP请求都会通过
app.use('/user/:id', (req, res, next) => {
  console.log('Request method: ', req.method);
  next();
});

// 有装载路径，符合路径规则的所有HTTP`GET`请求都会通过
app.get('/user/:id', function (req, res, next) {
  res.send('USER')
})

// 可以应用多个中间件，可以是一个函数，函数数组或它们的组合
app.use('/user/:id', function (req, res, next) {
  console.log('Request URL:', req.originalUrl)
  next()
}, function (req, res, next) {
  console.log('Request Type:', req.method)
  next()
})

// 针对一个路径可以定义多个中间件处理函数
// 但如果第一个中间件结束了请求响应的生命周期，则后面的中间件不会被执行
app.get('/user/:id', function (req, res, next) {
  console.log('ID:', req.params.id)
  next()
}, function (req, res, next) {
  res.send('User Info')
})
// 这个中间件不会被执行
app.get('/user/:id', function (req, res, next) {
  res.end(req.params.id)
})

// 使用`next('route')`跳过当前路径剩余的中间件，将控制权交给下一个中间件
// 注意：此方法仅支持用`app.METHOD() `或`router.METHOD()`定义的中间件
app.get('/user/:id', function (req, res, next) {
  // 如果id===0则将跳过剩余的中间件
  if (req.params.id === '0') next('route')
  // 否则将控制权交给剩余的下一个中间件
  else next()
}, function (req, res, next) {
  res.send('regular')
})

// 如果id===0，则由此中间件来处理
app.get('/user/:id', function (req, res, next) {
  res.send('special')
})
```

### router-level middleware
路由级别的中间件工作方式同应用级别的一样，区别就是它被绑定到一个路由实例`express.Router()`。加载路由中间件使用`router.use()`和`router.METHOD()`方法。跳过当前路径剩余的中间件用`next('route')`，此方式只适用于`router.METHOD()`方法。
```js
const express = require('express');
const app = express();
const router = express.Router();

// 无装载子路径，所有符合父路径的HTTP请求都会经过
router.use(function (req, res, next) {
  console.log('Time:', Date.now())
  next()
})

// 有装载子路径，所有符合父路径+子路径的HTTP请求都会经过
router.use('/user/:id', function (req, res, next) {
  console.log('Request URL:', req.originalUrl)
  next()
}, function (req, res, next) {
  console.log('Request Type:', req.method)
  next()
})

// 有装载子路径，所有符合父路径+子路径的HTTP`GET`请求都会经过
router.get('/user/:id', function (req, res, next) {
  // 如果id===0，则跳过当前路径剩余的中间件
  if (req.params.id === '0') next('route')
  // 否则将控制权交给当前路径的下一个中间件
  else next()
}, function (req, res, next) {
  res.render('regular')
})

// 如果id===0，则由此中间件处理
router.get('/user/:id', function (req, res, next) {
  console.log(req.params.id)
  res.render('special')
})

// 装载路由对象到app实例
app.use('/', router)
```

### error-handling middleware
错误处理中间件包含四个参数，缺一不可，即使你没用到，也要保留以便符合函数签名，否则会被当做常规中间件。
第一个参数为错误对象。
```js
app.use(function (err, req, res, next) {
  console.error(err.stack)
  res.status(500).send('Something broke!')
})
```

### built-in middleware
内置中间件如下，从4.x开始不再依赖`Connect`模块：
* express.static: 服务静态资源文件
* express.json: 处理JSON请求 v4.16.0+
* express:urlencoded:处理URL-encoded请求 v4.16.0+

### third-party middlewar
三方中间件，先安装再使用，查看常用[中间件列表](http://expressjs.com/en/resources/middleware.html)

```js
// 安装
// npm install cookie-parser
// 引用
var cookieParser = require('cookie-parser')
// 使用
app.use(cookieParser())
```

## Template engines
* 模板引擎允许你使用静态模板附加数据后，输出HTML给客户端，默认的引擎为`jade`，现在叫`pug`
* 安装模板引擎模块后，在`app`实例上设置模板的目录和引擎名称
* 模板引擎仅缓存编译后的模板文件，不会缓存附加数据后的渲染文件，因此每次请求都会重新渲染。
* 查看支持的模板引擎列表[Consolidate.js](https://www.npmjs.org/package/consolidate)
* express兼容的模板引擎内部会输入一个名为`__express`的方法`__express(filePath, options, callback)`，供`res.render()`方法渲染时使用
```js
// 设置模板引擎和目录
app.set('views', './views');
app.set('view engine', 'pug');
// 渲染模板
app.get('/', function (req, res) {
  res.render('index', { title: 'Hey', message: 'Hello there!' })
})
```

## Error handling
* 错误处理是指Express如何捕捉和处理同步或异步错误
* Express内置一个默认的错误处理程序，不需要自己写

### catching errors
* 确保express捕捉所有路由处理器和中间件产生的错误
```js
// 1. 同步错误
// 无需额外的工作，express会捕捉并处理
app.get("/", function (req, res) {
  throw new Error("BROKEN"); // Express will catch this on its own.
});

// 2. 异步错误，
// 特别注意：异步回调中无法处理同步错误，只能next(err)，所以如果需要处理回调中的同步错误，则可以使用链式处理程序，见2.4，或者先确保没有err, if(err) next(err)，再执行下面的程序
// 必须将错误传给`next()`方法，express会捕捉并处理
// express将会把当前请求作为一个错误，跳过所有的非错误处理中间件和路由处理程序，如果没有自定义错误中间件，则最终传给内置的默认错误中间件。
app.get("/", function (req, res, next) {
  fs.readFile("/file-does-not-exist", function (err, data) {
    if (err) {
      next(err); // Pass errors to Express.
    }
    else {
      res.send(data);
    }
  });
});

// 2.1 回调如果不提供数据，只提供错误，则可以简化
app.get("/", [
  function (req, res, next) {
    fs.writeFile("/inaccessible-path", "data", next);
  },
  function (req, res) {
    res.send("OK");
  }
]);

// 2.2 异步错误必须捕获，并将它传给express处理
// 下面代码使用try...catch来捕获错误，如果忽略express则捕获不到错误，因为setTimeout不是同步代码
app.get("/", function (req, res, next) {
  setTimeout(function () {
    try {
      throw new Error("BROKEN");
    }
    catch (err) {
      next(err);
    }
  }, 100);
});

// 2.3 可使用Promise代替try...catch的处理开销
// promise自动捕获同步错误和拒绝的promise，因此，将next传给catch处理程序即可，catch处理程序会将错误传给next
app.get("/", function (req, res, next) {
  Promise.resolve().then(function () {
    throw new Error("BROKEN");
  }).catch(next); // Errors will be passed to Express.
});

// 2.4 可以使用链式处理程序来捕捉同步错误，以减少琐碎的异步代码
// 第一个处理程序：如果读取文件没有错误，则next()到下一个程序，否则next（err）到express默认错误处理程序
// 第二个处理程序：如果处理数据有错，同步错误则会被express捕捉，如果把处理数据放到第一个处理程序中，则同步错误无法被捕捉，应用程序会退出。
app.get("/", [
  function (req, res, next) {
    fs.readFile("/maybe-valid-file", "utf8", function (err, data) {
        res.locals.data = data;
        next(err); 
    });
  },
  function (req, res) {
    res.locals.data = res.locals.data.split(",")[1];
    res.send(res.locals.data);
  }
]);

// 无论使用哪种方法，如果要调用Express错误处理程序并使应用程序不挂掉，您必须确保Express收到错误。
```

### default errors handler
* Express附带了一个内置的错误处理程序，可以处理应用程序中可能遇到的任何错误，它被添加到中间件堆栈的末尾。
* 如果您将错误传递给`next()`，并且没有在自定义错误处理程序中处理它，它将由内置错误处理程序处理; 错误将通过堆栈跟踪(stack trace)写入客户端。在生产环境中，堆栈跟踪不会被写入。
* 如果在生成响应后再调用`next(err)`，比如向客户端写入流数据发生错误，内置的默认错误处理程序将关闭连接，请求将失败，所以自定义错误处理函数需要代理默认错误处理函数的这种行为。
* 如果代码不止一次的调用`next(err)`，即使有自定义错误处理函数，默认错误处理程序也可能会被触发。
* `NODE_ENV=production`将应用运行在生产环境下。
```js
// 代理默认错误处理函数，当headers已经发送给客户端的时候
function errorHandler (err, req, res, next) {
  if (res.headersSent) {
    return next(err);
  }
  res.status(500)
  res.render('error', { error: err })
}
```

### write errors handler
* 自定义错误处理的函数有四个参数，而不是三个：(err, req, res, next)。
* 中间件函数的响应可以是任何格式：json,html等等
* 可以定义多个错误处理中间件函数，就像常规中间件函数一样，可用来处理不同的错误情况
```js
// 可以在其它所有app.use()的后面定义错误处理程序
app.use(function (err, req, res, next) {
  console.error(err.stack)
  res.status(500).send('Something broke!')
})

// 多个错误处理程序
function logErrors (err, req, res, next) {
  console.error(err.stack)
  next(err)
}
function clientErrorHandler (err, req, res, next) {
  if (req.xhr) {
    res.status(500).send({ error: 'Something failed!' })
  } else {
    next(err)
  }
}
function errorHandler (err, req, res, next) {
  res.status(500)
  res.render('error', { error: err })
}
app.use(logErrors)
app.use(clientErrorHandler)
app.use(errorHandler)
```

## Debugging
* express内部使用[`debug`](https://www.npmjs.com/package/debug)模块来记录日志，处理路由匹配，中间件函数，应用状态以及请求响应生命周期的流程。
* 通过设置`DEBUG`环境变量来开启调试信息，默认是关闭的
```bash
# 查看所有信息
# linux
DEBUG=express:* node index.js
# windows
set DEBUG=express:* & node index.js
# 查看router信息
DEBUG=express:router node index.js
# 查看application信息
DEBUG=express:application node index.js

# 设置命名空间前缀
# var debug = require('debug')('myapp:server');
DEBUG=myapp:* node index.js
# 多个命名空间
DEBUG=http,mail,express:* node index.js
```

## Express behind proxies
* 当express应用使用反向代理（nginx）的时候，需要给app实例设置`trust proxy`属性，虽然不设置应用也不会失败，但应用会将反向代理的ip地址看作客户端的ip地址。
* 开启信任代理后的影响：
  * `req.hostname`的值来自于请求头`X-Forwarded-Host`，这个值可被客户端或代理设置
  * `X-Forwarded-Proto`可以被反向代理服务设置，从而可以告诉app，协议是https，http或一个非法协议名称，这个值反射自`req.protocol`
  * `req.ip`,`req.ips`的值来自于`X-Forwarded-For`列表
* 信任代理通过[proxy-addr](https://www.npmjs.com/package/proxy-addr)模块实现
```js
// 1. Boolean
// true: X-Forwarded-*请求头中最左边的那个地址看作客户端的ip地址
// false: 默认设置，app将直接面对internet，客户端ip地址来自于req.connection.remoteAddress
app.set('trust proxy', true);

// 2. IP address
// 信任一个ip地址、子网、ip地址或子网数组
// 预置的子网名称：
// loopback - 127.0.0.1/8, ::1/128
// linklocal - 169.254.0.0/16, fe80::/10
// uniquelocal - 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, fc00::/7
app.set('trust proxy', 'loopback') // 单个子网
app.set('trust proxy', 'loopback, 123.123.123.123') // 单个子网和一个地址
app.set('trust proxy', 'loopback, linklocal, uniquelocal') // 多个子网作为CSV
app.set('trust proxy', ['loopback', 'linklocal', 'uniquelocal']) // 多个子网作为array

// 3. Number
// 信任从前置(front-facing)代理服务器跳跃n次的地址看作客户端
app.set('trust proxy', 1);

// 4. Function
// 自定义信任的实现
app.set('trust proxy', function (ip) {
  if (ip === '127.0.0.1' || ip === '123.123.123.123') return true // trusted IPs
  else return false
})
```

## Database integration
支持大多数流行的关系型和no sql数据库。
### MySQL
```js
var mysql = require('mysql')
var connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'dbuser',
  password : 's3kreee7',
  database : 'my_db'
});
connection.connect()
connection.query('SELECT 1 + 1 AS solution', function (err, rows, fields) {
  if (err) throw err
  console.log('The solution is: ', rows[0].solution)
})
connection.end()
```

### MongoDB
```js
var MongoClient = require('mongodb').MongoClient
MongoClient.connect('mongodb://localhost:27017/animals', function (err, client) {
  if (err) throw err
  var db = client.db('animals')
  db.collection('mammals').find().toArray(function (err, result) {
    if (err) throw err
    console.log(result)
  })
})
```

### Redis
```js
var redis = require('redis')
var client = redis.createClient()
client.on('error', function (err) {
  console.log('Error ' + err)
})
client.set('string key', 'string val', redis.print)
client.hset('hash key', 'hashtest 1', 'some value', redis.print)
client.hset(['hash key', 'hashtest 2', 'some other value'], redis.print)
client.hkeys('hash key', function (err, replies) {
  console.log(replies.length + ' replies:')
  replies.forEach(function (reply, i) {
    console.log('    ' + i + ': ' + reply)
  })
  client.quit()
})
```

### ElasticSearch
```js
var elasticsearch = require('elasticsearch')
var client = elasticsearch.Client({
  host: 'localhost:9200'
})
client.search({
  index: 'books',
  type: 'book',
  body: {
    query: {
      multi_match: {
        query: 'express js',
        fields: ['title', 'description']
      }
    }
  }
}).then(function (response) {
  var hits = response.hits.hits
}, function (error) {
  console.trace(error.message)
})
```