title: HTTP缓存机制
date: 2020-05-18
categories:

- caomengqi

tags:

* 缓存

---

本文主要针对HTTP的缓存机制进行了介绍。

# 前言

web缓存一直是前端备受关注的一个话题，主要包括数据库缓存、服务器缓存（代理服务器缓存、CDN缓存）、浏览器缓存。其中浏览器缓存是把已经访问的过的页面（如 html js css 图片等）拷贝一份副本保存在浏览器中，当下一次访问该网站，缓存会根据缓存机制决定是直接使用副本响应访问请求，还是向源服务器再次发送请求。本文对浏览器缓存中的HTTP缓存进行了介绍。

## 缓存的作用
为什么使用缓存？原因是缓存能够提高 Web 项目的性能和用户体验，具体体现在以下几点：

* 加快浏览器加载网页的速度
* 减少冗余的数据传输，节省网络流量和带宽
* 减少服务器的负担，大大提高了网站的性能

当然缓存无法保存世界上每份文档的副本 ，对于到达缓存的请求存在两种情况：缓存命中和缓存未命中。

缓存命中(cache hit)：可以用已有的副本为某些到达缓存的请求提供服务。

缓存未命中(cache miss) ：到达缓存的请求可能会由于没有副本可用，而被转发给原始服务器。

# 缓存处理步骤

缓存处理步骤可以用一张图来表示：

![1589872657427](https://p0.meituan.net/spacex/689248b0db9ead4609e591284a38b67b.png)

分析上图，可以了解到缓存处理主要分为6个步骤：

1. **接收：**缓存从网络中读取抵达的请求报文

   缓存检测到一条网络连接上的活动，读取输入数据。高性能的缓存会同时从多条输入连接上读取数据，在整条报文抵达之前开始对事务进行处理。 

2. **解析：**缓存对报文进行解析，提取出 URL 和各种首部

   缓存将请求报文解析为片断，将首部的各个部分放入易于操作的数据结构中。这样，缓存软件就更容易处理首部字段并修改它们了 。解析程序还要负责首部各部分的标准化，将大小写或可替换数据格式之类不太重要的区别都看作等效的。而且，某些请求报文中包含有完整的绝对 URL，而其他一些请求中包含的则是相对 URL 和 Host 首部，所以解析程序通常都要将这些细节隐藏起来 。

3. **查询：**缓存查看是否有本地副本可用，如果没有，就获取一份副本(并将其保存在本地) 

   在第三步中，缓存获取了 URL，查找本地副本。本地副本可能存储在内存、本地磁盘，甚至附近的另一台计算机中。专业级的缓存会使用快速算法来确定本地缓存中是否有某个对象。如果本地没有这个文档，它可以根据情形和配置，到原始服务器或父代理中去取，或者返回一条错误信息。 

   已缓存对象中包含了服务器响应主体和原始服务器响应首部，这样就会在缓存命中时返回正确的服务器首部。已缓存对象中还包含了一些元数据(metadata)，用来记录对象在缓存中停留了多长时间，以及它被用过多少次等 。 

4. **新鲜度检测：**缓存查看已缓存副本是否足够新鲜，如果不是，就询问服务器是否有任何更新 

   HTTP 通过缓存将服务器文档的副本保留一段时间。在这段时间里，都认为文档是 “新鲜的”，缓存可以在不联系服务器的情况下，直接提供该文档。但一旦已缓存副本停留的时间太长，超过了文档的新鲜度限值(freshness limit)，就认为对象“过期”了，在提供该文档之前，缓存要再次与服务器进行确认，以查看文档是否发生了变化。

5. **创建响应：**缓存会用新的首部和已缓存的主体来构建一条响应报文

   为了使缓存的响应看起来就像来自原始服务器的一样，缓存将已缓存的服务器响应首部作为响应首部的起点。然后缓存对这些基础首部进行了修改和扩充。

   缓存负责对这些首部进行改造，以便与客户端的要求相匹配。比如，服务器返回的可能是一条 HTTP/1.0 响应，而客户端期待的是一条 HTTP/1.1 响应，在这种情况下，缓存必须对首部进行相应的转换。缓存还会向其中插入新鲜度信息(Cache-Control、 Expires 首部等)，而且通常会包含一个 Via 首部来说明请求是由一个代理缓存提供的。 

6. **发送：**缓存通过网络将响应发回给客户端 

   一旦响应首部准备好了，缓存就将响应回送给客户端。和所有代理服务器一样，代理缓存要管理与客户端之间的连接。高性能的缓存会尽力高效地发送数据，通常可以避免在本地缓存和网络 I/O 缓冲区之间进行文档内容的复制。  

# 缓存规则

为了方便理解，可以假设浏览器存在一个缓存数据库，用于存储缓存信息（实际上静态资源是被缓存到了内存和磁盘中），在浏览器第一次请求数据时，缓存数据库没有对应的缓存数据，则需要请求服务器，服务器将返回缓存规则和数据，浏览器将缓存规则和数据存储进缓存数据库。

![1589873289242](https://p0.meituan.net/spacex/f128a87935661c41f78fca0d311f87a9.png)

注意：在浏览器地址栏输入地址后，与页面相关的所有资源都会遵循缓存策略，而是否被缓存则依赖于缓存策略的设置。

HTTP 缓存有多种规则，根据是否需要向服务器发送请求主要分为两大类，强缓存和协商缓存。

## 强缓存

### 强缓存流程

强制缓存是第一次访问服务器获取数据后，在有效时间内不会再请求服务器，而是直接使用缓存数据，强制缓存的流程如下。

![1589873518965](https://p0.meituan.net/spacex/bb11a5a591085b6d09accf91d7cf5daf.png)

### 强缓存到期时间

通过特殊的 HTTP Cache-Control 首部(HTTP/1.0+)和 Expires 首部(HTTP/1.1 )，HTTP 让原始服务器向每个文档附加了一个“过期日期”。这些首部说明了在多长时间内可以将这些内容视为新鲜的。 

![1589873870042](https://p0.meituan.net/spacex/6611fde19d148f9031c38c776113ddcf.png)

在 HTTP 1.0 版本中，Expires 字段的绝对时间是从服务器获取的，由于请求需要时间，所以浏览器的请求时间与服务器接收到请求所获取的时间是存在误差的，这也导致了缓存命中的误差。在 HTTP 1.1 版本中，因为 Cache-Control 的值 max-age=xxx 中的 xxx 是以秒为单位的相对时间，所以在浏览器接收到资源后开始倒计时，规避了 HTTP 1.0 中缓存命中存在误差的缺点。

为了兼容低版本 HTTP 协议，正常开发中两种响应头会同时使用，HTTP 1.1 版本的实现优先级高于 HTTP 1.0。

### Cache-Control

* Cache-Control：no-store

禁止一切缓存。缓存通常会像非缓存代理服务器一样，向客户端转发一条 no-store 响应，然后删除对象。 

* Cache-Control：no-cache

强制客户端直接向服务器发送请求，也就是说每次请求都必须向服务器发送。服务器接收到请求，然后判断资源是否变更，是则返回新内容，否则返回304，未变更。这个很容易让人产生误解，使人误以为是响应不被缓存。实际上Cache-Control: no-cache是会被缓存的，只不过每次在向客户端（浏览器）提供响应数据时，缓存都要向服务器评估缓存响应的有效性。 

* Cache-Control：must-revalidate

在事先没有跟原始服务器进行再验证的情况下，不能提供这个对象的陈旧副本。缓存仍然可以随意提供新鲜的副本。如果在缓存进行 must-revalidate 新鲜度检查时，原始服务器不可用，缓存就必须返回一条 504 Gateway Timeout 错误。

* Cache-Control：max-age

首部表示的是从服务器将文档传来之时起，可以认为此文档处于新鲜状态的秒数。

* Cache-Control：s-maxage

和max-age是一样的，不过它只针对代理服务器缓存而言。

* Cache-Control：private

只能针对个人用户，而不能被代理服务器缓存。

* Cache-Control：public

可以被任何对象缓存，包括发送请求的客户端，代理服务器。

* Cache-Control：max-stale

缓存可以随意提供过期的文件。如果指定了参数 <s>，在这段时间内，文档就不能过期。这条指令放松了缓存的规则。

### Pragma

Pragma 只有一个属性值，就是 no-cache ，效果和 Cache-Control 中的 no-cache 一致，不使用强缓存，需要与服务器验证缓存是否新鲜，在 3 个头部属性中的优先级最高。

### 分析

使用express构建本地服务器进行强缓存验证，代码如下：

```js
const express = require('express');
const app = express();
var options = { 
  etag: false, // 禁用协商缓存
  lastModified: false, // 禁用协商缓存
  setHeaders: (res, path, stat) => {
    res.set('Cache-Control', 'max-age=10'); // 强缓存超时时间为10秒
  },
};
app.use(express.static((__dirname + '/public'), options));
app.listen(3001);
```

第一次请求，状态码200 OK，Response Header中增加Cache-Control，过期时间为10秒：

![1589887718530](https://p0.meituan.net/spacex/aa3728e363c3187709e4f65266e814ef.png)

十秒内第二次请求，状态码200 OK (from disk cache)，并在在Response Headers中，Date未更新：

![1589887736499](https://p0.meituan.net/spacex/a8a2611be26837e89249df0a693ac716.png)

## 协商缓存

### 协商缓存流程

协商缓存又叫对比缓存，设置协商缓存后，第一次访问服务器获取数据时，服务器会将数据和缓存标识一起返回给浏览器，客户端会将数据和标识存入缓存数据库中，下一次请求时，会先去缓存中取出缓存标识发送给服务器进行询问，当服务器数据更改时会更新标识，所以服务器拿到浏览器发来的标识进行对比，相同代表数据未更改，响应浏览器通知数据未更改，浏览器会去缓存中获取数据，如果标识不同，代表服务器更改过数据，所以会将新的数据和新的标识返回浏览器，浏览器会将新的数据和标识存入缓存中，协商缓存的流程如下。

![1589874616041](https://p0.meituan.net/spacex/200106127c37bd2eb59e04c4cd847b1f.png)

协商缓存和强制缓存不同的是，协商缓存每次请求都需要跟服务器通信，而且命中缓存服务器返回状态码不再是200，而是304。

### 协商缓存判断

强制缓存是通过过期时间来控制是否访问服务器，而协商缓存每次都要与服务器交互对比缓存标识，同样的，对于协商缓存的实现在 HTTP 1.0 版本和 HTTP 1.1 版本也有所不同。在HTTP1.0版本中，使用Last-Modified/If-Modified-Since来标识，在HTTP1.1版本中使用ETag/If-None-Match来标识。

#### Last-Modified/If-Modified-Since

Last-Modified/If-Modified-Since 的值代表的是文件的最后修改时间，如果在指定日期之后文档被修改过了，就执行请求。

- 如果自指定日期后，文档被修改了，If-Modified-Since 条件为真，通常 GET 就会成功执行。携带新首部的新文档会被返回给缓存，新首部除了其他信息之外，还包含了一个新的过期日期。
- 如果自指定日期后，文档没被修改过，If-Modified-Since条件为假，会向客户端返回一个304 Not Modified响应报文。
- If-Modified-Since 首部可以与 Last-Modified 服务器响应首部配合工作。第一次请求服务端会把资源的最后修改时间放到 Last-Modified 响应头中，第二次发起请求的时候，请求头会带上上一次响应头中的 Last-Modified 的时间，并放到 If-Modified-Since 请求头属性中，服务端根据文件最后一次修改时间和 If-Modified-Since 的值进行比较，如果相等，返回 304 ，并加载浏览器缓存。如果不相等， 则返回带有新主体的 200 响应。

#### ETag/If-None-Match

服务器通过 ETag 响应头来设置缓存标识，ETag一般为资源实体的哈希值，也服务器生成的一个标记，用来标识返回值是否有变化。且Etag的优先级高于Last-Modified。

* 浏览器接收到数据和唯一标识后存入缓存，下次请求时，通过 If-None-Match 请求头将唯一标识带给服务器，服务器取出唯一标识与之前的标识对比，不同，说明修改过，If-None-Match 首部就会执行所请求的方法返回新标识和数据；相同，则返回状态码 304 通知浏览器命中缓存。

注意：ETag 又有强弱校验之分，如果 hash 码是以 "W/" 开头的一串字符串，说明此时协商缓存的校验是弱校验的，只有服务器上的文件差异（根据 ETag 计算方式来决定）达到能够触发 hash 值后缀变化的时候，才会真正地请求资源，否则返回 304 并加载浏览器缓存。

### 分析

使用express构建本地服务器进行协商缓存验证，代码如下：

```js
const express = require('express');
const app = express();
var options = { 
  etag: true, // 开启协商缓存
  lastModified: true, // 开启协商缓存
  setHeaders: (res, path, stat) => {
    res.set({
      'Cache-Control': 'max-age=00', // 浏览器不走强缓存
      'Pragma': 'no-cache', // 浏览器不走强缓存
    });
  },
};
app.use(express.static((__dirname + '/public'), options));
app.listen(3000);
```

第一次请求，状态码为200 OK，响应头中带有资源的hash信息和最后一次修改时间：

![1589885754975](https://p0.meituan.net/spacex/4629ed3bec0e6af2c9619354076021c5.png)

第二次请求，服务端根据请求头中的 If-Modified-Since 和 If-None-Match 验证文件是否修改，未修改返回状态码304 Not Modified：

![1589885770173](https://p0.meituan.net/spacex/0b7f944872390c95114f5a49af558463.png)

## 强缓存vs协商缓存

- 两者的共同点是：如果命中，都是从客户端缓存中加载资源，而不是从服务器加载资源数据；
- 两者的区别是：强缓存不发请求到服务器，协商缓存会发请求到服务器。

# 总结

为了使缓存策略更加健壮、灵活，HTTP 1.0 版本 和 HTTP 1.1 版本的缓存策略会同时使用，甚至强制缓存和协商缓存也会同时使用，对于强制缓存，服务器通知浏览器一个缓存时间，在缓存时间内，下次请求，直接使用缓存，超出有效时间，执行协商缓存策略，对于协商缓存，将缓存信息中的 Etag 和 Last-Modified 通过请求头 If-None-Match 和 If-Modified-Since 发送给服务器，由服务器校验同时设置新的强制缓存，校验通过并返回 304 状态码时，浏览器直接使用缓存，如果协商缓存也未命中，则服务器重新设置协商缓存的标识。

相关参考链接：

* [HTTP缓存介绍](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching_FAQ)
* [图解HTTP缓存](https://juejin.im/post/5eb7f811f265da7bbc7cc5bd)
* [HTTP缓存机制](https://juejin.im/post/5a1d4e546fb9a0450f21af23)
* [前端缓存最佳实践](https://juejin.im/post/5c136bd16fb9a049d37efc47)