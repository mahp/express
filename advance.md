# Advance topics

## Template engines
```js
var fs = require('fs');
 // define the template engine
app.engine('ntl', function (filePath, options, callback) {
  fs.readFile(filePath, function (err, content) {
    if (err) return callback(err);
    // simple handle
    var rendered = content.toString().replace('#title#', '<title>' + options.title + '</title>')
    .replace('#message#', '<h1>' + options.message + '</h1>');
    return callback(null, rendered);
  });
})
// index.ntl
// <h1>#title#</h1>
// <p>#message#</p>

// set app view engine
app.set('views', './views');
app.set('view engine', 'ntl');
// use index.ntl template render result
app.get('/', function (req, res) {
  res.render('index', { title: 'Hey', message: 'Hello there!' })
});
```

## Process managers
流行的PM列表：
* [Forever](http://www.expressjs.com.cn/advanced/pm.html#forever)
* [PM2](http://www.expressjs.com.cn/advanced/pm.html#pm2)
* [StrongLoop Process Manager](http://www.expressjs.com.cn/advanced/pm.html#strongloop-process-manager)
* [SystemD](http://www.expressjs.com.cn/advanced/pm.html#systemd)

## Security updates
* Node.js漏洞直接影响Express。因此，请密切关注[Node.js漏洞](http://blog.nodejs.org/vulnerability/)，并确保使用最新的稳定版Node.js.
* 如在Express中发现了安全漏洞，请参阅[安全策略和过程](http://www.expressjs.com.cn/en/resources/contributing.html#security-policies-and-procedures)

## Production Best Practices: Security
* 通常`development`和`production`环境需要有不同的配置，`development`环境需要详细的错误跟踪以及调试，无需担心可扩展性、可靠性及性能，而这些对于`production`环境来说则尤其重要。
* `production`环境中的安全最佳实践：
  * 不要使用已弃用或易受攻击的Express版本：比如2.x,3.x
  * 使用TLS：建议在反向代理服务上配置SSL安全连接，比如：Nginx
  * 使用`helmet`模块：9项HTTP头安全设置，[查看详情](https://www.npmjs.com/package/helmet)
  * 安全使用cookies：
    * express-session: 仅仅将sessionID保存在cookie中，其它会话数据默认保存在内存中，生产环境建议保存在类似[`redis`](https://github.com/expressjs/session#compatible-session-stores)这样的三方服务中
    * cookie-session: 序列号所有会话数据到cookie中，客户端可看见cookie数据，大小不允许超过4096个字节，仅在会话数据相对较小且易于作为原始值编码时使用。
    * 不要使用默认的session cookie name：类似`X-Powered-By`，攻击者可使用它对服务器进行指纹识别，并相应地攻击目标。可使用`express-session`重新生成cookie name
    * 设置cookie安全选项：如secure,httpOnly,domain,path,expires等等
  * 确保您的依赖项是安全的：使用`npm audit`来分析依赖关系树或使用[`snyk`](https://snyk.io/)来检查或修补应用的依赖项的任何已知漏洞。
  * 避免其他已知漏洞：熟悉[Node Sercurity Project](https://nodesecurity.io/advisories)和web相关的[漏洞](https://www.owasp.org/index.php/Top_10-2017_Top_10)，及时采取措施预防。
  * 其他考虑因素：[Node.js的安全检查列表](https://blog.risingstack.com/node-js-security-checklist/)

## Production best practices: performance and reliability

### 开发阶段 dev
* 开启gzip压缩：减少响应体的大小，加快应用的速度，可使用`compression`模块，生产环境可使用反向代理，并开启gzip模块
* 不要使用同步方法：应该总是使用异步方法，同步方法仅仅在应用启动阶段使用，[`--trace-sync-io`](https://nodejs.org/api/cli.html#cli_trace_sync_io)检查应用是否有同步api
* 正确记录日志：调试可使用`console`或`debug`模块，记录应用的运行信息可使用`winston`或`bunyan`模块
* 正确处理异常：未捕获的异常将会导致应用崩溃
  * Node使用`错误优先回调`约定来从异步函数中返回错误，即回调的一个参数必须是错误对象
  * 不要监听`uncaughtException`事件，在未捕获的异常后继续运行应用是一种危险的做法，自动重启是从错误中恢复的最可靠方法
  * 使用`try...catch`可捕捉同步代码的异常，对于异步（特别是生成环境中），不会捕捉很多异常
  * 使用promise可处理使用`then()`的异步代码块中的任何异常（显式和隐式），只需添加`.catch(next)`到方法链的末尾即可
  * 同步错误和异步错误都会传给错误中间件，确保所有异步代码都必须返回promises，如果不返回，则可用help转换`Bluebird.promisifyAll()`

### 运维阶段 ops
* 设置环境变量`NODE_ENV=production`
  * 开启视图模板缓存
  * 缓存从css扩展生成的css文件
  * 生成较少的错误信息
```bash
# for Upstart
# /etc/init/env.conf
env NODE_ENV=production

# for systemd
# /etc/systemd/system/myservice.service
Environment=NODE_ENV=production
```

* 确保应用可自动重启
  * 首先确保应用已经过完整测试并处理了所有异常
  * 使用进程管理器来重启App，如PM2,Forever
    * PM是一个应用`container`，方便部署，提供高可用，并可在运行时管理应用
    * PM可深入了解运行时性能和资源消耗
    * PM可动态修改设置来提升性能
    * 集群控制（PM2,StrongLoop PM）
    * about StrongLoop PM:
      * 本地构建打包应用，安全部署在生产环境中
      * 应用因任何原因崩溃则自动重启
      * 远程管理集群
      * 查看CPU分析和堆快照来优化性能并诊断内存泄漏
      * 查看应用性能指标
      * 通过Nginx负载均衡的整合控制，轻松扩展到多个主机
  * 当OS重启的时候，可使用OS的init system（如systemd,Upstart）来重新启动PM（推荐使用这种方法），也可以使用init system启动应用而不使用PM（但无法使用PM的优势）
    * Systemd: Linux系统服务管理器，大多数Linux主要发行包都使用它作为操作系统的默认初始化器，配置文件称为`unit file`，服务文件名为`filename.service`，[更多参考](http://www.freedesktop.org/software/systemd/man/systemd.unit.html)

    * Upstart: 很多Linux发行包把它作为一个系统管理工具，在操作系统启动的时候用来启动任务和服务，关闭的时候停止它们，配置文件成为`job`，服务文件名为`filename.conf`,[更多参考](http://upstart.ubuntu.com/cookbook)

* 在集群中运行应用
  * 利用操作系统多核的特性，每个CPU内核上运行一个实例，在实例之间分配负载和任务，即运行多个进程来集群运行应用，以提升应用的性能
  * 但集群应用无法共享内存空间，每个实例中对象都是独立的，即无法维护应用的状态，因此需要将共享数据保存到第三方服务中，如`Redis`这样的内存数据库中，这样可以水平扩展应用，无论是使用多个进程或在多个物理服务器上进行集群
  * 在集群应用中，除了性能优势之外，故障隔离也是运行应用程序进程集群的另一个原因，worker进程的崩溃不会影响剩余的进程，务必记录事件并使用`cluster.fork()`生成新进程
  * 使用集群：
    * [Node的集群模块](https://nodejs.org/dist/latest-v4.x/docs/api/cluster.html)
    * [StrongLoop PM](https://docs.strongloop.com/display/SLC/Clustering)
    * [PM2](https://pm2.keymetrics.io/docs/usage/cluster-mode/)

* 缓存请求结果
  * 另一种提高生产性能的策略是缓存请求的结果，以便您的应用程序不会重复操作相同的请求
  * 缓存服务器[Varnish](https://www.varnish-cache.org/)或[Nginx](https://serversforhackers.com/nginx-caching/)可以大大提高应用程序的速度和性能

* 使用负载均衡
  * 无论应用程序如何优化，单个实例只能处理有限的负载和流量
  * 扩展应用程序的一种方法是运行它的多个实例并通过负载均衡分配流量
  * 负载均衡器通常是反向代理，用于协调进出多个应用程序实例和服务器的流量
  * 如果需要将特定SessionID的请求关联到发起的进程，则可以开启粘性会话`sticky session`来实现，如Nginx上面的`ip_hash`
  * 一般使用[`Nginx`](https://www.nginx.com/)和[`HAProxy`](http://www.haproxy.org/)

* 使用反向代理服务
  * 反向代理可以处理错误页面，压缩，缓存，提供静态文件和负载平衡等
  * 通常将反向代理置于应用的前端以分担应用的任务，从而让应用执行专门的任务
  * 一般使用[`Nginx`](https://www.nginx.com/)和[`HAProxy`](http://www.haproxy.org/)

## Health Checks and Graceful Shutdown
* 优雅关闭：当需要版本升级时，PM首先发送`SIGTERM `信号通知应用，它将要被杀死。一旦应用收到信号，它将停止接收新的请求，结束进行中的请求，释放占用的资源，包括数据库连接和文件锁等。
* 健康检查：负载均衡LB使用健康检查来判断一个应用实例是否健康，并可以接受请求。[K8S](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)有2种健康检查：
  - liveness: 判断何时重启容器
  - readiness: 判断容器何时准备开始接受流量，当pod未准备好的情况下，它将从服务的负载均衡中被删除
* 第三方解决方案：[Terminus](https://github.com/godaddy/terminus)
  - 支持优雅关闭
  - 支持 K8S liveness/readiness 健康检查
