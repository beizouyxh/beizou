# 浏览器缓存

**浏览器缓存** 是浏览器将用户请求过的静态资源（html、css、js），存储到电脑本地磁盘中，当浏览器再次访问时，就可以直接从本地加载了，不需要再去服务端请求了。

但也不是说缓存没有缺点，如果处理不当，可能会导致服务端代码更新了，但是用户却还是老页面。所以前端们要针对项目中各个资源的实际情况，做出合理的缓存策略。

缓存的优点：

- 减少了冗余的数据传输，节省网费
- 减少服务器的负担，提升网站性能
- 加快了客户端加载网页的速度

## 强缓存

**如果资源没过期，就取缓存，如果过期了，则请求服务器。**

强缓存是利用http的返回头中的Expires或者Cache-Control两个字段来控制的，用来表示资源的缓存时间

`HTTP1.0`时，使用的是 **Expires**，`HTTP1.1`时，使用的是 **Cache-Control**

#### Expires

`Expires` 即过期时间，存在于服务端返回的响应头中，告诉浏览器在这个过期时间内可以直接从缓存里面获取数据，无需再次请求
它的值为一个绝对时间的GMT格式的时间字符串，比如：

```js
Expires: Mon, 22 Sept 2020 23:59:59 GMT
```

表示资源在 `2020年9月22日23:59:59` 过期失效，过期了就要向服务器端发请求
这种方式有一个明显的缺点，由于失效时间是一个绝对时间，`服务器的时间和浏览器的时间可能不一致`，所以服务器返回的这个过期时间可能就是不准确的，所以在 HTTP1.1 版本中被抛弃了

#### Cache-Control

在 HTTP1.1 中采取了 `Cache-Control`
它和 `Expires` **本质的不同是它没有采用具体的过期时间点这个方式**，而是采用过期时长来控制缓存，对应的字段是 `max-age`，它是一个相对时间，比如：

```js
Cache-Control:max-age=3600
```

代表资源的有效期是 3600 秒，响应返回后在 3600 秒，也就是一个小时之内可以直接使用缓存
常用的配合设置的值：

1. public：可以被所有的用户缓存，包括客户端和 CDN 等中间代理服务器(因为一个请求可能要经过不同的代理服务器才到达目标服务器)
2. private：只能被浏览器缓存，不允许 CDN 等中间代理服务器缓存
3. no-cache：不使用本地缓存，跳过当前的强缓存，直接进入 `协商缓存阶段`
4. no-store：不强缓存，也不协商缓存，基本不用，缓存越多才越好呢
5. s-maxage：和 `max-age` 比较像，但是区别在于 `s-maxage` 是针对代理服务器的缓存时间
   `当 Expires 和 Cache-Control 同时存在的时候，Cache-Control 会有效考虑`

#### 强缓存流程

![](https://user-gold-cdn.xitu.io/2019/1/18/168602e5bb5e5b7e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2019/1/18/168603039eeb6f12?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 协商缓存

触发条件：

1. Cache-Control 的值为 no-cache （不强缓存）
2. 或者 max-age 过期了 （强缓存，但总有过期的时候）

也就是说，不管怎样，都可能最后要进行协商缓存（no-store除外）

需要请求头中添加tag，服务器根据tag来判断是否使用缓存，所以被称为协商缓存。tag分为两种Last-Modified和ETag

- **Last-Modified**

最后修改时间。在第一次请求完毕后，服务器给浏览器返回的响应头里会带有Last-Modified，浏览器在下一次请求的时候会携带If-Modified-Since，表示服务器资源最后修改时间，最后进行相应的操作。否则返回304，但只能以秒为单位，所以不够精准(不在意这几秒的差距也OK)

- **ETag**

ETag是给当前的文件资源添加唯一的文件标识，只要内容有改动就值就会变。服务器会将其加在响应头中，浏览器会在下次请求的时候将其作为If-None-Match字段的内容发送给服务器。服务器根据值做不同的操作

- 两者对比：

ETag优先级比Last-Modified高，因为它可以精确的判断是否需要更新。虽然性能不如Last-Modified

## 缓存位置

强缓存命中或者协商缓存阶段服务器换回 304 的时候，直接从缓存中获取资源
浏览器中的缓存位置一共有四种，按优先级从高到低依次为：

1. Service Worker
2. Memory Cache
3. Disk Cache
4. Push Cache

#### Service Worker

`Service Worker` 借鉴了 Web Worker 的思路，即让 JS 运行在主线程之外，由于它脱离了浏览器的窗体，因此无法直接访问 DOM。虽然如此，但它仍然能帮助完成很多有用的功能，比如`离线缓存`、`消息推送`和`网络代理`等功能。其中的`离线缓存`就是 Service Worker Cache

#### Memory Cache 和 Disk Cache

`Memory Cache` 指的是内存缓存，从效率上讲它是最快的。但是从存活时间来讲又是最短的，当渲染进程结束后，内存缓存也就不存在了
`Disk Cache` 就是存储在磁盘中的缓存，从存取效率上讲是比内存缓存慢的，但是他的优势在于存储容量和存储时长

1. 比较大的JS、CSS文件会直接被丢进磁盘，反之丢进内存
2. 内存使用率比较高的时候，文件优先进入磁盘

#### Push Cache

即推送缓存，这是浏览器缓存的最后一道防线，是 HTTP/2 中的内容，虽然现在应用的并不广泛，但随着 HTTP/2 的推广，它的应用越来越广泛

## 强缓存和协商缓存的比较

![](https://user-gold-cdn.xitu.io/2019/1/21/1686c02dc305d66c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 总结

```
- 如果强缓存可用，直接使用  
- 如果强缓存不可能，进入协商缓存阶段，发送 HTTP 请求，服务器通过请求头中的 If-Modified-Since 或者 If-None-Match 字段检查判断是否资源是否更新  
  + 如果资源更新，返回资源和 200 状态码  
  + 如果资源未更新，返回 304，告诉浏览器直接从缓存获取资源  
```
