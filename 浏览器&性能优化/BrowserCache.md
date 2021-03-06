## 浏览器缓存控制

浏览器缓存过程   
![浏览器缓存过程](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/01.png?x-oss-process=image/resize,w_500)  

* 浏览器每次发起请求，都会在浏览器缓存中查找该请求的结果和缓存标识
* 浏览器每次拿到返回的请求结果都会将该结果和缓存标识存入浏览器缓存中

### 1. 按缓存位置分类
> 优先级顺序：（由上到下寻找，找到即返回，找不到则继续，若最终未命中缓存，则发起网络请求）
> 1. Service Worker
> 2. Memory Cache
> 3. Dist Cache
> 4. Push Cache

1. Service Worker  
    `Service Worker`是运行在浏览器背后独立的线程，与浏览器其他内建缓存机制不同，它可以让我们自由控制缓存哪些文件、如何匹配缓存。如何读取缓存、并且缓存是持续性的。

    `Service Worker`实现缓存功能一般分为三个步骤：首先需要注册`Service Worker`，然后监听到install事件以后就可以缓存需要的文件，那么在下次用户访问的时候就可以通过拦截请求的方式查询是否存在缓存，存在缓存的话就直接读取缓存文件，否则就去请求数据。

    * 使用`Service Worker`的话，传输协议必须是HTTPS。在http页面上无法使用。
        ```js
        'serviceWorker' in navigator    // false
        ```
    * 这个缓存是永久性的，即使关闭Tab或浏览器。有两种情况会导致缓存被清除，手动调用API`cache.delete(rosource)`或者容量超出限制，被浏览器全部清空。
    * 经过`Service Worker`的`fetch()`方法获取的资源，即使没有命中`Service Worker`缓存，甚至实际走了网络请求，也会标注为`from ServiceWorker`。

2. Memory Cache  
    `Memory Cache`是内存缓存，与`Dist Cache(硬盘缓存)`相对，读取内存中的数据要比磁盘数据快。它保证了页面中如果存在两个相同的请求（例如两个`src`相同的`<img>`，两个`href`相同的`<link>`），都实际只会被最多请求一次，避免浪费。
    * 关闭Tab页面，内存中的缓存也会被释放。
    * preload，例如`<link rel="preload">`。这些显式指定的预加载资源，也会被放入`Memory Cache`中。
    * 内存缓存在缓存资源时，并不关心返回资源的HTTP缓存头Cache-Control是什么值
    * 在缓存匹配时，除了匹配URL，可能还会对`Content-Type`，CORS中的域名规则做校验。
3. Dist Cache  
    相比于内存缓存，`Dist Cache`胜在存储容量和实效性上，绝大部分的缓存都来自`Dist Cache`。

    浏览器会把哪些文件丢进内存中？哪些丢进硬盘中？
    * 对于大文件来说，大概率是不存储在内存中的，反之优先
    * 当前系统内存使用率高的话，优先存储进硬盘

4. Push Cache  
    // TODO: 资料较少，待查证
    `Push Cache(推送缓存)`是HTTP/2中的内容，当以上三种缓存都没命中时，它才会被使用。  
    它只在会话(Session)中存在，一旦会话结束就被释放，并且缓存时间也很短暂，在Chrome浏览器中只有5分钟左右。

### 2. 按失效策略分类
1. 强缓存  
    强缓存直接减少请求次数，是提升最大的缓存策略，在性能优化中应该被首先考虑。  
    若命中强缓存，在Chrome控制台的Network选项中可以看到该请求返回200的状态码，并且Size显示`from disk cache`或`from memory cache`。  
    控制强缓存的字段，`Expires`和`Cache-Control`（* 都是设置在Response Header上）。`Cache-Control`的优先级高于`Expires`，为了兼容HTTP/1.0和HTTP/1.1，实际项目中，两个字段一般都会设置。

    1. Expires  
        这是HTTP/1.0的字段，表示缓存到期时间，是一个绝对时间（当前时间+缓存时间），如：
        ```
        Expires: Thu, 10 Nov 2017 08:45:11 GMT
        ```
        缺点：
        1. 由于是绝对时间，用户修改本地时间就可能会导致浏览器判断缓存失效，重新请求该资源。
        2. 写法太复杂了。表示时间的字符串多个空格，少个字母，都会导致非法属性从而设置失效。
    2. Cache-Control  
        HTTP/1.1，表示一个相对时间。
        ```
        Cache-control: max-age=2592000
        ```
        指令：
        * `max-age`   `max-age=xxx` 表示缓存内容将在xxx秒后失效
        * `no-cache`  客户端缓存内容，是否使用缓存则需要经过协商缓存来验证决定。（并不是字面意思不使用缓存）
        * `no-store`  所有内容都不会被缓存，即不使用强制缓存，也不使用协商缓存
        * ... 还有很多，可以组合使用

2. 协商缓存（对比缓存）  
    当强制缓存失效(超过规定时间)时，就需要使用协商缓存，由服务器决定缓存内容是否失效。  
    流程：浏览器先请求缓存数据库，返回一个缓存标识。之后浏览器拿这个标识和服务器通讯。如果缓存未失效，则返回 HTTP 状态码 304 表示继续使用，于是客户端继续使用缓存；如果失效，则返回新的数据和缓存规则，浏览器响应数据后，再把规则写入到缓存数据库。  
    **对比缓存在请求数上和没有缓存是一致的**，但如果是 304 的话，返回的仅仅是一个状态码而已，并没有实际的文件内容，因此 **在响应体体积上的节省是它的优化点**。通过减少响应体体积，来缩短网络传输时间。所以和强缓存相比提升幅度较小，但总比没有缓存好。  
    协商缓存是可以和强缓存一起使用的，作为在强缓存失效后的一种后备方案。实际项目中他们也的确经常一同出现。
    1. `Last-Modified` & `If-Modified-Since`  
        1. 服务器通过 `Last-Modified` 字段告知客户端，资源最后一次被修改的时间，例如:
            ```
            Last-Modified: Mon, 10 Nov 2018 09:10:11 GMT
            ```

        2. 浏览器将这个值和内容一起记录在缓存数据库中。

        3. 下一次请求相同资源时，浏览器从自己的缓存中找出“不确定是否过期的”缓存。因此在请求头中将上次的 `Last-Modified` 的值写入到请求头的 `If-Modified-Since` 字段

        4. 服务器会将 `If-Modified-Since` 的值与 `Last-Modified` 字段进行对比。如果相等，则表示未修改，响应 304 和空的响应体；反之，则表示修改了，响应 200 状态码，并返回数据。

        缺点：
        * 如果资源更新的速度是秒以下单位，那么该缓存是不能被使用的，因为它的时间单位最低是秒。
        
    2. `Etag` & `If-None-Match`  
        `Etag` 存储的是文件的特殊标识(一般都是 hash 生成的)，服务器存储着文件的 `Etag` 字段，**只要资源有变化，Etag就会重新生成**。之后的流程和 `Last-Modified` 一致，只是 `Last-Modified` 字段和它所表示的更新时间改变成了 `Etag` 字段和它所表示的文件 hash，把 `If-Modified-Since` 变成了 `If-None-Match`。服务器同样进行比较，命中返回 304, 不命中返回新资源和 200。
        * 首先在精确度上，`Etag` 要优于 `Last-Modified`
        * `Etag` 的优先级高于 `Last-Modified`

### 3. 浏览器的行为
* 打开网页，地址栏输入地址： 查找 `disk cache` 中是否有匹配。如有则使用；如没有则发送网络请求。
* 普通刷新 (F5)：因为 TAB 并没有关闭，因此 `memory cache` 是可用的，会被优先使用(如果匹配的话)。其次才是 `disk cache`。
* 强制刷新 (Ctrl + F5)：浏览器不使用缓存，因此发送的请求头部均带有 `Cache-control: no-cache`(为了兼容，还带了 `Pragma: no-cache`)。服务器直接返回 200 和最新内容

### 4. 缓存的应用模式
1. 模式1：不常变化的资源  
    ```
    Cache-Control: max-age=31536000
    ```
    通常在处理这类资源资源时，给它们的 `Cache-Control` 配置一个很大的 `max-age=31536000` (一年)，这样浏览器之后请求相同的 URL 会命中强制缓存。而为了解决更新的问题，就需要在文件名(或者路径)中添加 `hash`， 版本号等动态字符，之后更改动态字符，达到更改引用 URL 的目的，从而让之前的强制缓存失效 (其实并未立即失效，只是不再使用了而已)。
    在线提供的类库 (如 `jquery-3.3.1.min.js`, `lodash.min.js` 等) 均采用这个模式。
    ```
    // 构建过程中常见的做法
    https://hm.baidu.com/hm.js?e23800c454aa573c0ccb16b52665ac26
    http://tb1.bdstatic.com/tb/_/tbean_safe_ajax_94e7ca2.js
    http://img1.gtimg.com/ninja/2/2016/04/ninja145972803357449.jpg
    ```

2. 模式2：经常变化的资源  
    ```
    Cache-Control: no-cache
    ```
    这里的资源不单单指静态资源，也可能是网页资源，例如博客文章。这类资源的特点是：URL 不能变化，但内容可以(且经常)变化。我们可以设置 `Cache-Control: no-cache` 来迫使浏览器每次请求都必须找服务器验证资源是否有效。  
    既然提到了验证，就必须 `ETag` 或者 `Last-Modified` 出场。这些字段都会由专门处理静态资源的常用类库(例如 `koa-static`)自动添加。  
    这样的做法虽然不能节省请求数量，但是能显著减少响应数据大小。

3. 模式3：非常危险的模式 1 和模式 2 的结合（反例）  
    ```
    Cache-Control: max-age=600, must-revalidate
    ```
    表面上看这很美好：资源可以缓存 10 分钟，10 分钟内读取缓存，10 分钟后和服务器进行一次验证，集两种模式之大成，但实际线上暗存风险。因为浏览器的缓存有自动清理机制，开发者并不能控制。  
    举个例子：当我们有 3 种资源： `index.html`, `index.js`, `index.css`。我们对这 3 者进行上述配置之后，假设在某次访问时，`index.js` 已经被缓存清理而不存在，但 `index.html`, `index.css` 仍然存在于缓存中。这时候浏览器会向服务器请求新的 `index.js`，然后配上老的 `index.html`, `index.css` 展现给用户。这其中的风险显而易见：不同版本的资源组合在一起，报错是极有可能的结局。

### 5. 引用
[一文读懂前端缓存](https://juejin.im/post/5c22ee806fb9a049fb43b2c5?utm_source=gold_browser_extension#heading-19)

[深入理解浏览器的缓存机制](https://www.jianshu.com/p/54cc04190252)