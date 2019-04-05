# API 4.x

## express() 
```js
const express = express();
```

### 方法
```js
/**
 * 内置中间件，用于处理JSON请求，基于boby-parser模块
 * 仅检查HTTP请求头的`Content-Type`属性与options的`type`选项是否匹配
 * 支持任何Unicode编码的请求体body，并支持自动gzip和deflate编码解压
 * 解析后的数据会附加到req.body对象中，或一个空对象{}（如果解析为空或失败的话）
 * 
 * @since v4.16.0+
 * @param {Object} [options]
 * @return {Middleware}
 */
express.json([options]);
const options = {
  inflate: true, // {Boolean} - 是否允许处理deflate压缩的body
  limit: '100kb', // {Number|String} - 请求体的最大bytes，字符串单位会使用`bytes`模块解析
  reviver: null, // {Function} - 作为JSON.parse()方法的第二个参数
  strict: true, // {Boolean} - 是否仅接受数组arrays和对象objects
  type: 'application/json', // {String|String[]|Function} - 解析的MIME类型，非函数时直接传给`type-is`模块解析。函数则调用fn(req)，如果函数返回truthy值则解析请求
  verify: undefined // {Function} - 自定义验证请求体方法verify(req, res, buf, encoding)，buf: 原始请求体缓冲，encoding: 请求编码类型。抛错可取消验证。
}

/**
 * 内置中间件，用于处理普通表单内容提交，基于body-parser
 * 仅检查HTTP请求头的`Content-Type`属性与options的`type`选项是否匹配
 * 仅支持UTF-8的Unicode编码请求体body，并支持自动gzip和deflate编码解压
 * 解析后的数据会附加到req.body对象中，或一个空对象{}（如果解析为空或失败的话）
 * 解析后的对象包括key-value值对，值可以是字符串或数组（extended:false），或任何类型（extended:true）
 * 
 * @since v4.16.0+
 * @param {Object} [options]
 * @return {Middleware}
 */
express.urlencoded([options]);
const options = {
  extended: true, // {Boolean} - 是否使用`qs`模块（提供类似JSON的数据格式体验）解析请求的URL-encoded编码数据，false则使用`querystring模块`。
  inflate: true, // {Boolean} - 是否允许处理deflate压缩的body
  limit: '100kb', // {Number|String} - 请求体的最大bytes，字符串单位会使用`bytes`模块解析
  parameterLimit: 1000, // {Number} - 允许的最多参数数量，超过则报错
  type: 'application/x-www-form-urlencoded', // {String|String[]|Function} - 解析的MIME类型，非函数时直接传给`type-is`模块解析。函数则调用fn(req)，如果函数返回truthy值则解析请求
  verify: undefined, // {Function} - 自定义验证请求体方法verify(req, res, buf, encoding)，buf: 原始请求体缓冲，encoding: 请求编码类型。抛错可取消验证。
}

/**
 * 内置中间件，用于服务静态资源文件，基于serve-static模块
 * 如果请求时未找到资源，则返回404响应
 * 注：最佳方案是使用反向代理服务器来代替，可以提高执行效率
 * 
 * @param {String} root - 静态资源根目录
 * @param {Object} [options]
 * @return {Middleware}
 */
express.static(root, [options]);
const options = {
  dotfiles: 'ignore', // {String} - 如何处理点开头的文件或目录,见`dotfiles`
  etag: true, // {Boolean} - 是否允许生成etag值，`express.static`总是发送weak ETags
  extensions: false, // {Mixed} - 定义搜索文件时的扩展名匹配['html','htm']
  fallthrough: true, // {Boolean} - 是否将客户端错误作为未处理的请求下行，否则转发客户端错误，见`fallthrough`
  immutable: false, // {Boolean} - 是否启用`Cache-Control`响应头中的`immutable`指令，启用则需要指定maxAge缓存值，指令会阻止客户端在缓存的生命周期内发出条件请求来检查文件是否已更改
  index: 'index.html', // {String|Boolean} - 定义目录索引文件，false为取消索引
  lastModified: true, // {Boolean} - 是否设置`Last-Modified`响应头的值
  maxAge: 0, // {Number|String} - 设置最大缓存时间毫秒数，字符串值则使用`ms`模块解析 
  redirect: true, // {Boolean} - 当路径是一个目录的时候，跳转到尾`/`路径，即在原有的路径后面增加`/`
  setHeaders: undefined // {Function} - 自定义HTTP响应头，必须同步执行，见`setHeaders`
}

// dotfiles，点文件
// 注意：默认情况下，不会忽略以点开头的目录中的文件
// 'allow' -> '对点文件不做特殊处理，即允许访问'
// 'deny' -> '拒绝对点文件的请求，响应403，再调用next()'
// 'ignore' -> '忽略点文件，响应404，再调用next()' 

// fallthrough，错误下行
// true: 客户端错误（错误请求或文件不存在）将调用`next()`以调用下一个中间件。适用于将多个物理目录映射到同一Web地址或路由以映射不存在的文件。
// false: 调用`next(err)`，即使是404错误。从而可以短路404请求，减少开销。

// setHeaders，需同步执行
// res: response object
// path: 正在发送的文件路径
// stat: 正在发送的文件状态stat对象
fn(res, path, stat) {
  res.set('x-timestamp', Date.now())
}

/**
 * 创建一个路由router对象
 * 可以给router对象添加中间件(use())或HTTP方法（GET,POST,PUT,PATCH,DELETE），类似app对象一样
 * 
 * @param {Object} [options]
 * @return {Router}
 */
express.Router([options]);
const options = {
  caseSensitive: false, // {Boolean} - 路径名称是否允许大小写敏感 
  mergeParams: false, // {Boolean} - v4.5.0+ 父路由的req.params是否可被子路由的同名参数覆盖
  strict: false // {Boolean} - 是否开启严格路由，即把'/foo'和'/foo/'看做相同路由
}
```

## Application
```js
const express = express();
const app = express();

app.get('/', function(req, res){
  res.send('hello world');
  // req.app === app
  // res.app === app
});
app.listen(3000);

// 注意：
// app和router对象都实现了中间件的接口，所以本身也可以被作为中间件来使用
// 如装载子应用`mount()`，定义路由对象express.Router()来使用中间件`use()`或处理路由等
```

### 属性
```js
/**
 * 定义应用级别的全局对象，持久化在应用的整个生命周期中
 * `res.locals`：仅存在请求响应的生命周期中
 */
app.locals;

app.locals.title = 'My App';
app.locals.email = 'me@myapp.com';

/**
 * 返回子应用的装载路径或路径数组
 * 子应用也是express的一个实例，可用来处理特定路径的请求
 * 类似`req.baseUrl`：返回匹配的URL路径，而不是匹配模式
 */
app.mountpath;

var app = express(); // the main app
var admin = express(); // the sub app
admin.get('/', function (req, res) {
  console.log(admin.mountpath); // [ '/adm*n', '/manager' ]
  res.send('Admin Homepage');
});
// app.use('/admin', admin); // mount sub app
app.use(['/adm*n', '/manager'], admin);
```

### 事件
```js
/**
 * 子应用装载事件
 * 注意子应用：有默认值的不会继承设置的值，没有默认值的继承设置的值
 * 
 * @param {String} 'mount' - 事件名称
 * @param {Function} callback(parent) - 回调，parent: 父应用
 */
app.on('mount', callback(parent));

var admin = express();
admin.on('mount', function (parent) {
  console.log('Admin Mounted');
  console.log(parent); // parent === app
});
admin.get('/', function (req, res) {
  res.send('Admin Homepage');
});
app.use('/admin', admin);
```

### 方法
* 应用程序设置：`app.set`,`app.get`，`app.disable`,`app.enable`,`app.path`
* 处理HTTP请求：`app.METHOD`,`app.param`
* 配置中间件：`app.route`,`app.use`
* 渲染视图：`app.render`
* 配置模板引擎： `app.engine`

```js
/**
 * 应用程序配置列表 application settings list
 * 
 * 注意在子应用中：
 *  - 有默认值则不会继承已设置的值
 *  - 没有默认值则继承已设置的值
 *  - 继承`trust proxy`设置，即使被重新设置也无效（为了向后的兼容性）
 *  - 在生产环境中，不会继承`view cache`设置
 */
// {Boolean} - 默认undefined, 是否开启大小写敏感，"/Foo"和"/foo"是否被看做一样
// - 子应用继承此设置
app.set('case sensitive routing', true); 
// {String} - 设置环境变量，默认process.env.NODE_ENV || 'development'
app.set('env', 'production');
// etag 设置
// {Boolean} - 默认true, 允许weak ETag, false关闭ETag
// {String} - 'strong': 允许strong ETag, 'weak': 允许weak ETag
// {Function} - 自定义ETag设定，如：
// app.set('etag', function (body, encoding) {
//   return generateHash(body, encoding); // consider the function is defined
// });
app.set('etag', 'weak');
// {String} - JSONP回调名称
app.set('jsonp callback name', 'callback');
// {Boolean} - 针对`res.json,res,res.jsonp,res.send`是否开启JSON转义响应
// 目的就是防止XSS攻击
// - 子应用将继承
app.set('json escape', undefined);
// {String} - JSON.stringify()所使用的replacer参数
// - 子应用将继承
app.set('json replacer', undefined);
// {Number} - JSON.stringify()所使用的space参数
// - 子应用将继承
app.set('json spaces', undefined);

// {Boolean|String|Function} - 解析查询字符串，默认'extended'
// Boolean: 是否开启解析
// String: 'simple'使用`querystring`模块解析, 'extended'使用`qs`模块解析
// Function: 自定义解析，参数为查询字符串，需要返回包含key-value的对象
app.set('query parser', 'extended');

// {Boolean} - 严格路由，"/Foo"和"/foo"是否被看做一样，默认 undefined
// - 子应用将继承
app.set('strict routing', false);

// {Number} - 删除点分隔的主机分隔数量来访问子域
app.set('subdomain offset', 2);

// {Boolean|String|String[]|Number|Function} - 配置信任代理，用于被反向代理服务时，默认false
// 见`trust proxy`选项列表，注意：`X-Forwarded-*`请求头容易被伪造
// - 子应用将继承，即使它也有一个设定值
app.set('trust proxy', false);

// {String|String[]} - 视图文件的目录，默认 process.cwd() + '/views'
app.set('views', './views');
// {String} - 模板引擎，默认undefined
// - 子应用将继承
app.set('view engine', 'pug');
// {Boolean} - 模板编译缓存，生产环境中为true，否则undefined
// - 子应用不会继承
app.set('view cache', true);

// {Boolean} - 是否开启'X-Powered-By:Expess'响应头
app.set('x-powered-by', true);

/**
 * 配置应用程序
 * 
 * @param {String} name - 配置项名称
 */
app.set(name, value);
app.enable(name); // === app.set(name, true);
app.disable(name); // === app.set(name, true);

app.set('title', 'My Site');
app.enable('trust proxy'); // true
app.disable('trust proxy'); // false

/**
 * 获取应用程序配置
 * 
 * @param {String} name - 配置项名称
 */
app.get(name);
app.enabled(name); // === app.get(name);
app.disabled(name); // === app.get(name);

app.get('title'); // "My Site"
app.enabled('trust proxy'); // true
app.disabled('trust proxy'); // false

/**
 * 获取应用的规范路径
 * 注：推荐使用`req.baseUrl`获取应用的规范路径
 * 
 * @return {String} app path
 */
app.path();

var app = express(),
    blog = express(),
    blogAdmin = express();
app.use('/blog', blog);
blog.use('/admin', blogAdmin);
console.log(app.path()); // ''
console.log(blog.path()); // '/blog'
console.log(blogAdmin.path()); // '/blog/admin'

/**
 * 处理HTTP请求的方法
 * 
 * Path对象，响应HTTP请求的路径对象
 * Type: 
 *  - String
 *  - String Pattern
 *  - RegExp
 *  - [String, String Pattern, RegExp, ...]
 * 
 * Middleware对象，响应HTTP请求的中间件回调
 * Type: 
 *  - 一个中间件
 *  - 逗号分隔的中间件列表
 *  - 中间件数组
 *  - 上面三个情况的组合
 * 
 * 支持的METHOD列表:
 * head,options,get,post,put,patch,delete
 * checkout,copy,lock,merge,mkactivity,mkcol,move,m-search,notify
 * purge,report,search,subscribe,trace,nlock,unsubscribe
 * 
 * @param {Path} path - 路径对象，默认：/
 * @param {Middleware} callback - 中间件
 */
app.METHOD(path, callback [, callback ...])

/**
 * 处理HTTP的所有请求（GET,POST,DELETE,PUT,PATCH,...）
 * 非标HTTP方法
 * 
 * @param {Path} path - 路径对象，默认：/
 * @param {Middleware} callback - 中间件
 */
app.all(path, callback [, callback ...])

// 全局逻辑
app.all('*', function (req, res, next) {
  console.log('global logic ...')
  next()
});
// 白名单，针对/api的所有请求需要验证授权
app.all('/api/*', requireAuthentication, loadUser);

/**
 * 处理HTTP的GET请求
 * 
 * @param {Path} path - 路径对象，默认：/
 * @param {Middleware} callback - 中间件
 */
app.get(path, callback [, callback ...])

app.get('/', function (req, res) {
  res.send('GET request to homepage');
});

/**
 * 处理HTTP的POST请求
 * 
 * @param {Path} path - 路径对象，默认：/
 * @param {Middleware} callback - 中间件
 */
app.post(path, callback [, callback ...])

app.post('/', function (req, res) {
  res.send('POST request to homepage');
});

/**
 * 处理HTTP的PUT请求
 * 
 * @param {Path} path - 路径对象，默认：/
 * @param {Middleware} callback - 中间件
 */
app.put(path, callback [, callback ...])

app.put('/', function (req, res) {
  res.send('PUT request to homepage');
});

/**
 * 处理HTTP的DELETE请求
 * 
 * @param {Path} path - 路径对象，默认：/
 * @param {Middleware} callback - 中间件
 */
app.delete(path, callback [, callback ...])

app.delete('/', function (req, res) {
  res.send('DELETE request to homepage');
});

/**
 * 处理请求路由参数
 * - 符合字符串参数，回调即触发执行一次
 * - 符合字符串参数数组，则按顺序对应每个参数都触发回调执行一次，回调内部的next()即调用下一个参数的回调，如果是最后一个参数，则传给下一个中间件
 * 
 * 注意：参数回调会在路由处理程序前执行，并且它们在请求-响应周期中仅被调用一次，即使参数在多个路由中被匹配
 * 
 * @param {String|String[]} [name] - 路由参数名称
 * @param {Function} callback(req,res,next,paramValue) - 回调
 */
app.param([name], callback);

// 单个字符串参数
// GET /user/42/3
// CALLED ONLY ONCE
// although this matches
// and this matches too
app.param('id', function (req, res, next, id) {
  console.log('CALLED ONLY ONCE');
  next();
});
app.get('/user/:id', function (req, res, next) {
  console.log('although this matches');
  next();
});
app.get('/user/:id', function (req, res) {
  console.log('and this matches too');
  res.end();
});

// 字符串参数数组
// GET /user/42/3
// CALLED ONLY ONCE with 42
// CALLED ONLY ONCE with 3
// although this matches
// and this matches too
app.param(['id', 'page'], function (req, res, next, value) {
  console.log('CALLED ONLY ONCE with', value);
  next();
});
app.get('/user/:id/:page', function (req, res, next) {
  console.log('although this matches');
  next();
});
app.get('/user/:id/:page', function (req, res) {
  console.log('and this matches too');
  res.end();
});

/**
 * 返回单个路由实例
 * 可附加任意HTTP请求处理，避免定义多个相同的路由名称
 * 注：restful架构可参考
 * 
 * @param {String} name
 * @return {Route}
 */
app.route();

var app = express();
app.route('/user')
  .all(function(req, res, next) {
    // runs for all HTTP verbs first
    // think of it as route specific middleware!
  })
  .get(function(req, res, next) {
    res.json(...);
  })
  .post(function(req, res, next) {
    // maybe add a new event...
  });

/**
 * 装载中间件或针对指定路径装载中间件
 * 
 * Path对象，响应HTTP请求的路径对象
 * Type: 
 *  - String
 *  - String Pattern
 *  - RegExp
 *  - [String, String Pattern, RegExp, ...]
 * 
 * Middleware对象，响应HTTP请求的中间件回调
 * Type: 
 *  - 一个中间件
 *  - 逗号分隔的中间件列表
 *  - 中间件数组
 *  - 上面三个情况的组合
 * 
 * @param {String} path - 默认: /
 * @param {Middleware} callback(req,res,next)
 */
app.use([path,] callback [, callback...])

// 中间件按装载的序列执行，因此顺序很重要
app.use(function (req, res, next) {
  console.log('Time: %d', Date.now());
  next();
});

// 错误处理中间件必须包含4个参数，即使没有用到也要包含，已便维护函数签名，不把它当做普通中间件处理
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});

// 静态文件服务和日志服务中间件
app.use(express.static(__dirname + '/public'));
app.use(logger());

/**
 * 返回渲染后的HTML
 * 类似：`res.render()`，但res.render可以发送响应到客户端
 * res.render内部使用app.render来渲染视图
 * 
 * @param {String} view - 视图模板名称
 * @param {Object} [locals] - 数据对象
 * @param {Function} callback - 回调，参数包含渲染后的HTML
 */
app.render(view, [locals], callback)

app.render('email', { name: 'Tobi' }, function(err, html){
  // ...
});

/**
 * 注册基于扩展名ext的模板引擎回调callback
 * 即针对ext扩展名的模板文件，使用哪种模板引擎处理
 * __express()为默认的专为expess提供的模板渲染方法
 * 渲染方法签名fn(path, options, callback)
 * 
 * @param {String} ext
 * @param {Function} callback(path, options, callback)
 */
app.engine(ext, callback)

app.engine('pug', require('pug').__express);
app.engine('html', require('ejs').renderFile);

// 可使用consolidate.js库来协同不同引擎的渲染方法
var engines = require('consolidate');
app.engine('haml', engines.haml);
app.engine('html', engines.hogan);

/**
 * 创建一个unix socket，并监听特定路径的链接
 * 等同于`http.Server.listen()`方法
 * 
 * @param {String} path
 * @param {Function} callback
 */
app.listen(path, [callback])

app.listen('/tmp/sock');

/**
 * 绑定并监听指定服务器和端口的链接
 * 等同于`http.Server.listen()`方法
 * 
 * @param {String} path
 * @param {String} host
 * @param {String} backlog
 * @param {Function} callback
 */
app.listen([port[, host[, backlog]]][, callback])

app.listen(3000, 'localhost');

// listen方法返回一个http.Server对象
app.listen = function() {
  var server = http.createServer(this);
  return server.listen.apply(server, arguments);
};

// app实际上是一个Function，可以传递给http.Server，作为回调去处理请求
var https = require('https');
var http = require('http');
var app = express();
http.createServer(app).listen(80);
https.createServer(options, app).listen(443);
```

## Request
通常约定http请求对象名为`req`，包含请求查询字符串、参数、body和HTTP headers等内容。它是`Node`请求对象的增强版本，支持所有[内建的域和方法](https://nodejs.org/api/http.html#http_class_http_incomingmessage)
```js
const express = express();
const app = express();
app.get('/', function(req, res){
  // req: http request object
  // ...
});
```
### 属性
Express 4中，`req`对象默认不包含`req.files`，即上传文件，访问上传文件，可以使用第三方的模块，如：`multer`,`formidable`,`busboy`,`multiparty`等。

```js
/**
 * 引用app实例
 * 可在中间件函数中通过req访问app实例的属性和方法
 * 
 * @return {App}
 */
req.app;

app.get('/', function(req, res){
  res.send('The views directory is ' + req.app.get('views'));
});

/**
 * router实例被装载的路径
 * 不同于`app.mountpath`返回的路径`pattern`，它返回的是匹配字符串
 * 类似`app.path()`返回的路径，当大多数情况下推荐使用此方法获取`baseUrl`
 * 
 * @return {String}  path
 */
 req.baseUrl;

var greet = express.Router();
greet.get('/hello', function (req, res) {
  // req.baseUrl; // /greet
  // req.app.mountpath() // ['/gre+t', '/hel{2}o']
  // req.app.path() // /greet
});
app.use(['/gre+t', '/hel{2}o'], greet); // load the router on '/greet'


/**
 * 获取包含key-value值对数据的请求体对象
 * 当使用`body-parser`或`multer`等中间件时，默认为`undefined`，通过中间件的解析来生成结果
 */
req.body;

var bodyParser = require('body-parser');
app.use(bodyParser.json()); // for parsing application/json
app.use(bodyParser.urlencoded({ extended: true })); // for parsing application/x-www-form-urlencoded
app.post('/user', function (req, res, next) {
  res.json(req.body); // bodyParser populate req.body
});

/**
 * 当使用[`cookie-parser`](https://www.npmjs.com/package/cookie-parser)中间件的使用，获取客户端cookies的对象。
 * 如果不包含cookies，则返回 {}，获取签名cookies，则使用`req.signedCookies`
 * 
 * @return {Object} {key:value} 
 */
req.cookies;

var cookieParser = require('cookie-parser');
app.use(cookieParser()); // for parsing cookies
// Cookie: name=tj
req.cookies.name // "tj"

/**
 * 检测请求是否是新的请求，基于`fresh`模块
 * 相对应的是`req.stale`，检测请求是否是旧的请求
 * 返回true的条件除了`cahce-control`的值不等于`no-cache`外，需满足下列任何条件：
 *  - 已指定`if-modified-since`, 并且`last-modified` <= `modified`
 *  - `if-none-match`的值为 *
 *  - `if-none-match`头在被解析成指令后，和`etag`响应头不匹配
 * 
 * @return {Boolean}
 */
req.fresh;

/**
 * 返回HTTP请求头的Host值，即主机名，不包含端口和协议
 * 当设置信任代理`trust proxy`后，将使用`X-Forwarded-Host`的值代替，此header可由客户端或代理设置
 * 
 * @return {String}
 */
req.hostname;

/**
 * 获取客户端的IP地址
 * 当设置信任代理`trust proxy`后，将使用`X-Forwarded-For`的最左侧值代替，此header可由客户端或代理设置
 * 
 * @return {String}
 */
req.ip;

/**
 * 获取客户端IP地址数组
 * 当设置信任代理`trust proxy`后，将使用`X-Forwarded-For`的IP地址数组代替，此header可由客户端或代理设置
 * if X-Forwarded-For is client, proxy1, proxy2
 * return: ["client", "proxy1", "proxy2"]
 * 
 * @return {String[]}
 */
req.ips;

/**
 * 返回客户端请求方法，如`GET`,`POST`,`DELETE`,...
 * 
 * @return {String}
 */
req.method;

/**
 * 获取原始请求Url
 * 类似`req.url`(来自Node的http模块，非Express属性)，但它包含原始请求Url，可重写`req.url`来内部路由，如使用`app.use()`来装载路由，将重写`req.url`来跳过装载路径
 * 中间件中：req.originalUrl = req.baseUrl + req.path
 * 
 * @return {String}
 */
req.originalUrl;

// GET 'http://www.example.com/admin/new?q=something'
app.use('/admin', function(req, res, next) {  
  console.log(req.originalUrl); // '/admin/new?q=something'
  console.log(req.baseUrl); // '/admin'
  console.log(req.path); // '/new'
  next();
});

/**
 * 获取包含路由参数的对象，默认为{}
 * 如果路由定义是用正则或通配符的话，返回数组 req.params[n]，n为捕捉组
 * 
 * @return {Object} params
 */
req.params;

// /user/:name
// GET /user/tj
req.params.name; // => "tj"

// /file/*
// GET /file/javascripts/jquery.js
req.params[0]
// => "javascripts/jquery.js"

/**
 * 获取客户的请求路径，不包括挂载点
 * 
 * @return {String}
 */
req.path;

// example.com/users?sort=desc
req.path // => "/users"

/**
 * 获取客户端请求的协议名称，`http` or `https`
 * 当设置信任代理`trust proxy`后，将使用`X-Forwarded-Proto`的值代替，此header可由客户端或代理设置
 * 
 * @return {String}
 */
req.protocol;

/**
 * 获取包含客户端查询字符串参数的对象，默认为 {}
 * 
 * @return {Object}
 */
req.query;

// GET /shoes?order=desc&shoe[color]=blue&shoe[type]=converse
req.query.order // => "desc"
req.query.shoe.color // => "blue"
req.query.shoe.type // => "converse"

/**
 * 返回匹配的路由对象
 * 
 * @return {Route}
 */
req.route;

app.get('/user/:id?', function userIdHandler(req, res) {
  req.route = { path: '/user/:id?',
                stack:[{ 
                    handle: [Function: userIdHandler],
                    name: 'userIdHandler',
                    params: undefined,
                    path: undefined,
                    keys: [],
                    regexp: /^\/?$/i,
                    method: 'get' 
                  }],
                methods: { get: true } }
  res.send('GET');
});

/**
 * 判断是否建立了TLS安全连接，即是否是`https`协议链接
 * 
 * @return {Boolean}
 */
req.secure;

// 相当于
req.protocol === 'https';

/**
 * 当使用`cookie-parser`模块时，它包含已签名和未签名的cookies对象，默认为 {}
 * 签名的cookie应保存在不同的对象中，以显示开发人员的意图；否则，可能会对req.cookie值进行恶意攻击。
 * 请注意，对cookie进行签名不会使其“隐藏”或加密；但只会防止篡改（因为用于签名的secret是私有的，即服务端持有secret字符串）。
 * @return {Object}
 */
req.signedCookies;

// Cookie: user=tobi.CP7AWaXDfAKIRfH49dQzKJx7sKzzSoPq7/AcBBRVwlI3
req.signedCookies.user; // => "tobi"

/**
 * 判断请求是否是旧的请求，与`req.fresh`相反
 * 
 * @return {Boolean}
 */
req.stale;

/**
 * 获取请求域名中的子域数组
 * app配置项：`subdomain offset`默认值是2，它决定了此属性的返回值
 * 
 * @return {String[]}
 */
req.subdomains;

// Host: "tobi.ferrets.example.com"
req.subdomains; // ["ferrets", "tobi"]

/**
 * 判断是否来自AJAX请求，如果`X-Requested-With` === 'XMLHttpRequest'
 * 
 * @return {Boolean}
 */
req.xhr;
```

### 方法
```js
/**
 * 检测客户端可接收的内容类型MIME，基于客户端请求头`Accept`
 * 参考[`accepts`](https://github.com/expressjs/accepts)
 * 
 * @param {String|String[]} types - 内容类型,类型扩展名或数组
 * @return {String|Boolean} 返回最佳匹配值或false
 */
req.accepts(types)

// Accept: text/*, application/json
req.accepts('text/html'); // => "text/html"
req.accepts(['json', 'text']); // => "json"
req.accepts('application/json'); // => "application/json"
req.accepts('image/png'); // => undefined

/**
 * 检测客户端可接收的字符集，基于客户端请求头`Accept-Charset`
 * 
 * @param {String} charset - 字符集字符串或数组
 * @return {String|Boolean} 返回匹配的第一个值或false
 */
req.acceptsCharsets(charset[,...])

/**
 * 检测客户端可接收的内容编码，基于客户端请求头`Accept-Encoding`
 * 
 * @param {String} Encoding - 编码字符串或数组
 * @return {String|Boolean} 返回匹配的第一个值或false
 */
req.acceptsEncodings(encoding [, ...])

/**
 * 检测客户端可接收的内容语言，基于客户端请求头`Accept-Language`
 * 
 * @param {String} Language - 语言类型字符串或数组
 * @return {String|Boolean} 返回匹配的第一个值或false
 */
req.acceptsLanguages(lang [, ...])

/**
 * 获取HTTP请求头的值，大小写不敏感
 * `Referrer`和`Referer`域可互换
 * 别名：req.header(field)
 * 
 * @param {String} field
 * @return {String} value
 */
req.get(field)

req.get('Content-Type'); // => "text/plain"
req.get('content-type'); // => "text/plain"
req.get('Something'); // => undefined

/**
 * 检测客户端可接收的内容类型MIME，基于客户端请求头`Content-Type`
 * 如果没有请求体body, 返回null, 其它情况返回false
 * 参考[`type-is`](https://github.com/expressjs/type-is)
 * 
 * @param {String} type - MIME, 类型扩展名
 * @return {String|Boolean} value
 */
req.is(type)

// With Content-Type: text/html; charset=utf-8
req.is('html');       // => 'html'
req.is('text/html');  // => 'text/html'
req.is('text/*');     // => 'text/*'
// When Content-Type is application/json
req.is('json');              // => 'json'
req.is('application/json');  // => 'application/json'
req.is('application/*');     // => 'application/*'
req.is('html'); // => false

/**
 * 使用`req.params,req,body,req.query`代替
 * 
 * @deprecated
 */
req.param(name[,defaultValue])

/**
 * 解析HTTP请求头Range
 * options -> {combine: Boolean}: 指定是否应组合重叠和相邻范围
 *  - false: 默认
 *  - true: 范围将被组合并返回
 * return: 
 *  - Object[]
 *  - -1: 不可满足的范围
 *  - -2: 错误的标题字符串信号
 *  
 * @param {Number} size - 获取资源的大小
 * @param {Object} [options] - {combine:Boolean}
 * @return {Object[]} range object Array
 */
req.range(size[,options])

// parse header from request
var range = req.range(1000);
if (range.type === 'bytes') {
  range.forEach(function (r) {
    // do something with r.start and r.end
  })
}
```

## Response
`res`对象为http响应对象，用户向客户端发送http响应。它是Node响应对象的增强版本，支持所有的[内建域和方法](https://nodejs.org/api/http.html#http_class_http_serverresponse)
```js
const express = express();
const app = express();
app.get('/', function(req, res){
  // res: http response object
   res.send('hello world!');
});
```

### 属性
```js
/**
 * 引用app实例
 * req.app === res.app;
 * 
 * @return {App}
 */
res.app;

/**
 * 检测HTTP响应头是否已发送
 * 
 * @return {Boolean}
 */
res.headersSent;

app.get('/', function (req, res) {
  console.log(res.headersSent); // false
  res.send('OK');
  console.log(res.headersSent); // true
});

/**
 * 作用于请求和响应生命周期内的对象，因此仅对视图渲染有用，否则此属性与`app.locals`相同
 * 对于暴露请求级别的信息（如请求路径，身份验证，用户设置等）非常有用
 * 
 * @return {Object}
 */
res.locals;

app.use(function(req, res, next){
  res.locals.user = req.user;
  res.locals.authenticated = ! req.user.anonymous;
  next();
});
```

### 方法
```js
/**
 * 设置客户端cookie
 * - 方法即设置`Set-Cookie`响应头，所有选项默认值参照[RFC 6265](http://tools.ietf.org/html/rfc6265)
 * 
 * @param {String} name - 名称
 * @param {String|JSON Object} value - 值
 * @param {Object} options - 配置项
 */
res.cookie(name, value[, options])

const options = {
  domain: 'localhost', // cookie作用的域名,默认：app的域名
  encode: encodeURIComponent, // 一个同步执行的编码方法，默认：encodeURIComponent，不支持异步方法
  expires: new Date(Date.now() + 900000), // 过期时间：GMT格式时间，没有值或0则创建一个session cookie
  httpOnly: true, // 是否仅http使用，即禁止客户端脚本访问
  maxAge: 900000, // 过期时间：现对于当前时间的毫秒数，与expires相比，取最先到期的值
  path: '/', // cookie作用路径，默认 /
  secure: true, // 仅https使用
  signed: true, // 是否是签名cookie
  sameSite: true // [`SameSite`](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00#section-4.1.1)
}

res.cookie('name', 'tobi', { domain: '.example.com', path: '/admin', secure: true， expires: new Date(Date.now() + 900000), httpOnly: true });

// 字符串编码结果：'some_cross_domain_cookie=http://mysubdomain.example.com; Domain=example.com; Path=/;'
res.cookie('some_cross_domain_cookie', 'http://example.com',{domain:'example.com', encode: String});

// 以可序列化为JSON的Object为值，请求时会被`bodyParser()`解析
res.cookie('cart', { items: [1,2,3] }, { maxAge: 900000 });

// 当使用`cookie-parser`中间件时，此方法也支持签名cookie
// `res.cookie()`将把`secret`传递给`cookieParser(secret)`来获取签名值
res.cookie('name', 'tobi', { signed: true });
// req.signedCookie.name;

/**
 * 清除客户端cookie
 * - web浏览器和其它一些兼容的客户端，仅仅清除那些options同设置时`res.cookie()`的options相同的cookie，`expires`和`maxAge`属性除外
 * 
 * @param {String} name - 要删除的cookie名称
 * @param {Object} options - 同`res.cookie()`的options
 */
res.clearCookie(name [, options])

res.cookie('name', 'tobi', { path: '/admin' });
res.clearCookie('name', { path: '/admin' });

/**
 * 添加特殊响应头属性
 * - 如果头属性不存在，则创建头属性并赋值
 * - 如果头属性已存在，则覆盖属性值
 * - `res.append`之后再调用`res.set()`则重置先前的header值
 * 
 * @since v4.11.0
 * @param {String} field 字段
 * @param {String|Array} [value] 值
 */
res.append(field[, value]);

res.append('Link', ['<http://localhost/>', '<http://localhost:3000/>']);
res.append('Set-Cookie', 'foo=bar; Path=/; HttpOnly');
res.append('Warning', '199 Miscellaneous warning');

/**
 * 设置`Content-Disposition`响应头为`attachment`
 * 如果有filename，则设置为`Content-Disposition: attachment; filename="filename"`，`Content-type: image/png`
 * - 告知客户端下载附件文件
 *  
 * @param {String} [filename] - 文件路径
 */
res.attachment([filename]);

res.attachment();
// Content-Disposition: attachment
res.attachment('path/to/logo.png');
// Content-Disposition: attachment; filename="logo.png"
// Content-Type: image/png

/**
 * 将指定路径的文件作为附件传输，即下载附件文件
 * - `Content-Disposition: attachment; filename="filename"`
 * - 使用`res.sendFile()`方法发送文件，options也将传给它
 * 
 * @param {String} path - 文件路径
 * @param {String} filename - 文件名称
 * @param {Object} options - v4.16.0+
 * @param {Function} fn - 发生错误或传输结束后的回调
 */
res.download(path [, filename] [, options] [, fn])

res.download('/report-12345.pdf', 'report.pdf');
res.download('/report-12345.pdf', 'report.pdf', function(err){
  if (err) {
    next(err);
  } else {
    res.send('completed');
  }
});

/**
 * 设置响应头的属性和值，可用对象来一次性设置多个属性和值
 * - res.header(field [, value])的别名
 * 
 * @param {String|Object} field - 
 * @param {String} value - 
 */
res.set(field [, value])

res.set('Content-Type', 'text/plain');
res.set({
  'Content-Type': 'text/plain',
  'Content-Length': '123',
  'ETag': '12345'
});

/**
 * 返回某个响应头的值
 * 
 * @param {String} field - 响应头
 */
res.get(field)

res.get('Content-Type'); // => "text/plain"

/**
 * 设置响应头的Link属性
 * 
 * @param {Object} links
 */
res.links(links)

// Link: <http://api.example.com/users?page=2>; rel="next", <http://api.example.com/users?page=5>; rel="last"
res.links({
  next: 'http://api.example.com/users?page=2',
  last: 'http://api.example.com/users?page=5'
});

/**
 * 设置响应头的Location属性
 * - 如果未对URL进行编码，Express会将指定的URL传递到浏览器中的Location，而不进行任何验证
 * - 浏览器负责从当前URL或Referer的URL以及在Location中指定的URL派生预期的URL,并相应地重定向用户。
 * 
 * @param {String} path - 地址
 */
res.location(path)

res.location('/foo/bar');
res.location('http://example.com');
res.location('back'); // 引用请求头Referer的值或/
res.sendStatus(302); // 实现重定向

/**
 * 响应跳转到指定的路径或地址
 * - 可设定状态码，否则默认返回 302 'Found'
 * - 路径如果是一个完整合格的url,则跳转到这个url
 * - 路径可以相对于host的绝对路径或相对于当前路径的相对路径
 * - back为返回来源页（referer请求头的值）
 * 
 * @param {String} status - 状态码, 302 
 * @param {String} path - 
 */
res.redirect([status,] path)

res.redirect('http://example.com');
res.redirect(301, 'http://example.com');

// relative path
res.redirect('../login');
// absolute path
res.redirect('/foo/bar');
// referer path , default /
res.redirect('back');

/**
 * 依据http请求头的Accept来进行内容协商`content-negotiation`
 * - 使用`req.accepts()`来选择最终处理请求的回调
 * - 如果请求头未设置Accepts，第一个回调被调用
 * - 如果没有发现匹配值，则响应406 'Not Acceptable'或调用default回调
 * - 当匹配的回调被执行时，`Content-Type`会被设置，也可以通过`res.set()`或 `res.type()`来自定义
 * 
 * @param {Object} object
 */
res.format(object)

res.format({
  'text/plain': function(){
    res.send('hey');
  },
  'text/html': function(){
    res.send('<p>hey</p>');
  },
  'application/json': function(){
    res.send({ message: 'hey' });
  },
  'default': function() {
    res.status(406).send('Not Acceptable');
  }
});
// 可使用类型扩展名来代替MIME类型
res.format({
  text: function(){
    res.send('hey');
  },
  html: function(){
    res.send('<p>hey</p>');
  },
  json: function(){
    res.send({ message: 'hey' });
  }
});

/**
 * 结束响应进程
 * - 此方法来自于http.ServerResponse的[`response.end()`](https://nodejs.org/api/http.html#http_response_end_data_encoding_callback)
 * - 用于不带任何数据的快速结束响应，如果需要带数据则可以使用`res.send()`和`res.json()`
 * 
 * @param {String} data 
 * @param {String} encoding 
 */
res.end([data] [, encoding])

res.end();
res.status(404).end();

/**
 * 发送HTTP状态码StatusCode，并将状态文本作为响应体
 * - 状态码参照[StatusCode](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
 * 
 * @param {Number} statusCode - 状态码
 */
res.sendStatus(statusCode)

res.sendStatus(200); // equivalent to res.status(200).send('OK')
res.sendStatus(403); // equivalent to res.status(403).send('Forbidden')
res.sendStatus(404); // equivalent to res.status(404).send('Not Found')
res.sendStatus(500); // equivalent to res.status(500).send('Internal Server Error')
// 非标码用字符串表示
res.sendStatus(2000); // equivalent to res.status(2000).send('2000')

/**
 * 设置statusCode，可链式调用，是Node的response.statusCode的别名
 * 
 * @param {code} code - 状态码
 */
res.status(code)

res.status(403).end();
res.status(400).send('Bad Request');
res.status(404).sendFile('/absolute/path/to/404.png');

/**
 * 发送HTTP响应
 * - body type：{String | Array | Object | Buffer Object}
 * - 根据body类型自动设置响应头的`Content-Type`属性，除非之前已定义
 * - 提供自动HEAD和HTTP缓存刷新支持
 * 
 * @param {Any} [body]
 */
res.send([body])

res.send('<p>some html</p>'); // Content-Type: text/html
res.send({ some: 'json' }); // Content-Type: application/json
res.send([1,2,3]); // Content-Type: application/json
res.send(new Buffer('whoop')); // Content-Type: application/octet-stream

/**
 * 依据给定的路径传输文件，基于[`send`](https://github.com/pillarjs/send)模块
 * - 基于文件的扩展名设置`Content-Type`响应头
 * - 除非options设置了root属性，否则path必须是绝对路径
 * - 当发生错误或完成时调用回调，必须手动处理错误或结束响应处理
 * 
 * @since v4.8.0+
 * @param {String} path - 文件绝对路径，除非options设置了root属性
 * @param {Object} [options] - 配置项
 * @param {Function} [fn] - 回调，当发生错误或完成时调用，fn(err)
 */
res.sendFile(path [, options] [, fn])

const options = {
  maxAge: 900000, // {Number|String} 毫秒数，`Cache-Control`的`max-age`属性值，或使用`ms`模块来解析毫秒数字符串,默认为 0
  root: __dirname + '/public/', // 文件的根目录，绝对路径 
  lastModified: '', // v4.9.0+，设置`Last-Modified`头，Date格式，设置false为禁止
  headers: {}, // 设置http headers
  dotfiles: 'ignore', // 'allow', 'deny', 'ignore',等同 express.static的设置
  acceptRanges: true, // v4.14+ 是否接受range请求
  cacheControl: true, //  v4.14+ 是否允许设置 `Cache-Control`响应头
  immutable: false //  v4.16+ 是否允许`Cache-Control`的`immutable`指令，如果允许则需要设置`maxAge`值来进行缓存，`immutable`指令将阻止客户端在缓存期间使用条件请求来检查文件是否更新
}

app.get('/file/:name', function (req, res, next) {
  var options = {
    root: __dirname + '/public/',
    dotfiles: 'deny',
    headers: {
        'x-timestamp': Date.now(),
        'x-sent': true
    }
  };
  var fileName = req.params.name;
  res.sendFile(fileName, options, function (err) {
    if (err) {
      next(err);
    } else {
      console.log('Sent:', fileName);
    }
  });
});

/**
 * 发送JSON响应
 * 
 * @param {JSON type} body
 */
res.json([body])

res.json(null);
res.json(undefined);
res.json({ user: 'tobi' });

/**
 * 发送JSONP响应，除了支持callback外，和`res.json()`一样
 * - app配置项中可以设置callback的名称
 * 
 * @param {Any} body
 */
res.jsonp([body])

res.jsonp(null); // => callback(null)
res.jsonp({ user: 'tobi' }); // => callback({ "user": "tobi" })

app.set('jsonp callback name', 'cb');
// ?cb=foo
res.status(500).jsonp({ error: 'message' }); // => foo({ "error": "message" })

/**
 * 渲染模板，并发送HTML给客户端
 * view: 模板名称，基于app配置中的`views`目录
 *  - 如果不含扩展名，则使用配置的模板引擎`view engine`编译
 *  - 如果包含扩展名，则使用`require()`加载扩展名的模板引擎，并调用引擎的`__express()`方法编译模板
 *  - 模板是执行操作系统级别的操作，安全原因，最好不要使用用户输入的模板
 * locals: 本地数据对象，供模板使用
 * callback: 渲染后的回调，包含error和渲染后的html，不会自动给客户端发送响应，需要手动处理
 * 
 * @param {String} view - 模板名称
 * @param {Object} locals -  本地数据对象
 * @return {Function} callback - 渲染后回调，包含html
 */
res.render(view [, locals] [, callback])

// 发送渲染后的HTML给客户端
res.render('index');
// 回调中包含渲染后的HTML
res.render('index', function(err, html) {
  res.send(html);
});
// 传递本地数据对象给模板
res.render('user', { name: 'Tobi' }, function(err, html) {
  if(err) next(err);
  res.send(html);
});

/**
 * 设置响应头的Content-Type属性
 * - 通过使用[`mime.lookup`](https://github.com/broofa/node-mime#mimelookuppath)模块来判定具体类型
 * - 如果type带有`/`字符，则完全使用type字符串作为Content-Type的属性值
 * 
 * @param {String} type
 */
res.type(type)

res.type('.html');              // => 'text/html'
res.type('html');               // => 'text/html'
res.type('json');               // => 'application/json'
res.type('application/json');   // => 'application/json'
res.type('png');                // => image/png:

/**
 * 添加Vary响应头
 * 
 * @param {String} field
 */
res.vary(field)

res.vary('User-Agent').render('docs');
```

## Router
* router对象是一个独立的中间件和路由实例，可以看做是一个`小程序`（mini-application）,仅用来执行中间件和路由功能，每个Express应用都有一个内置的router
* router实现了middleware的接口，因此可以被app实例挂载`app.use()`，也可以被其它router实例挂载`router.use()`。
* 通过express的静态方法创建router实例`express.Router()`
* router实例对象可以挂载中间件并处理路由
* 可以将router实例挂载到app中，以分割路由或作为一个独立的小应用
```js
const express = express();
const router = express.Router();
// 挂载中间件，处理此router挂载路径的所有请求
router.use(function(req, res, next) {
  //...
  next();
});
// 处理路由
router.get('/events', function(req, res, next) {
  // ...
});
// 挂载到app
app.use('/miniApp', router);
// 避免挂载到相同路径，否则路径匹配后都会被执行
app.use('/users', authRouter);
app.use('/users', openRouter);
```
### 方法
```js
/**
 * 挂载中间件，类似`app.use()`
 * - 没有定义挂载路径，则处理所有请求
 * - 中间件按挂载的序列执行，因此顺序很重要
 * 
 * @param {String} [path] - 可选挂载路径，默认 /
 * @param {Function} [Function] - 中间件
 */
router.use([path], [function, ...] function)

// 处理所有请求
router.use(function(req, res, next) {
  console.log('%s %s %s', req.method, req.url, req.path);
  next();
});
// 仅处理/bar的请求
router.use('/bar', function(req, res, next) {
  // ...
  next();
});

/**
 * 处理路由的特定HTTP请求，如`GET,POST,PUT,DELETE,...`
 * - 如果`router.get()`之前的路径没有调用`router.head()`方法，则`router.head()`方法会自动调用`router.get()`方法（除了get方法之外）
 * - path会被转为正则来匹配请求，但不考虑查询字符串
 * - 可定义多个回调，同中间件一样，用`next()`转递或绕过
 * 
 * @param {String} path - 路径
 * @param {Function} callback - 回调
 */
router.METHOD(path, [callback, ...] callback)

// GET /
// GET /?name=tobi
router.get('/', function(req, res){
  res.send('hello world');
});

// GET /commits/71dbb9c
// GET /commits/71dbb9c..4c084f9
router.get(/^\/commits\/(\w+)(?:\.\.(\w+))?$/, function(req, res){
  var from = req.params[0];
  var to = req.params[1] || 'HEAD';
  res.send('commit range ' + from + '..' + to);
});

/**
 * 处理路由的所有HTTP请求，类似`app.all`
 * 
 * @param {String} path - 路径
 * @param {Function} callback - 回调
 */
router.all(path, [callback, ...] callback)

// 白名单验证特定目录下的所有请求
router.all('/api/*', requireAuthentication);

/**
 * 返回单个路由实例，用来定义不同的HTTP处理方法，类似`app.route()`，避免定义多个同名路由
 * 
 * @param {String} path - 路径
 * @return {Route}
 */
router.route(path)

router.route('/users/:user_id')
  .all(function(req, res, next) {
    // 处理所有HTTP请求
    next();
  })
  .get(function(req, res, next) {
    res.json({result: {user: 'express'}});
  })
  .post(function(req, res, next) {
    res.json({msg: 'success'});
  });

/**
 * 处理路由参数，类似`app.param`
 * - 不同于`app.param`，不接受路由参数数组
 * - callback(req,res,next,nameValue,nameLabel)
 * - 回调仅仅当此路由的参数被匹配时执行，app或其它router的路由参数匹配时不会执行
 * - 在请求响应生命周期中，即使参数匹配多个路由定义，回调也仅仅会被执行一次
 * 
 * @param {String} name - 参数名称
 * @param {Function} callback - 回调
 */
router.param(name, callback)
router.param(callback) // v4.11.0开始此方法不推荐被使用
```