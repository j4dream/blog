
先列举一些常用的缓存的头部字段，后面会慢慢解析。  
1. 通用字段，请求和响应都会用到的字段。

字段 | 说明
---|---
Cache-Control | 控制缓存行为
Pragma  | http 1.0 遗留下来的，值是 "no-cache" 时禁用缓存

2. 请求有关字段

字段 | 说明
---|---
If-Match | 比较 Etag 是否一致
If-None-Match | 比较 Etag 是否不一致
If-Modify-Since | 比较资源最后更新时间是否一致
If-Unmodify-Since | 比较资源最后更新时间是否不一致

3. 响应首部字段

字段 | 说明
---|---
ETag | 资源的匹配信息

4. 实体首部字段

字段 | 说明
---|---
Expires | http 1.0 遗留产物，实体主体过期时间
Last-Modified | 资源最后一次修改的时间

==Pargma== 值为 "no-cache" (貌似只有这个值) 客户端不缓存，每次都会发一次请求。

```
<meta http-equiv="Pargma" content="no-cache">
// 1. 仅 IE 识别， 其他浏览器仅能识别 "Cache-Control: no-store" 的 meta 标签
// 2. IE识别该标签，但不一定在请求字段加上该参数， 但会发出新的请求。 
```
==Expires== 控制启用缓存和定义缓存时间，它的值是格林尼治时间 (GMT), 用来告诉浏览器缓存的过期时间，没过这个时间点则不会发出新请求。


```
<meta http-equiv="Expires" content="Sat, 10 Dec 2016 13:21:14 GMT">
// 1. 仅 IE 识别， 若需要每次都要发新请求，可以把 content 的值改为 "-1" 或 "0"。
// 2. IE识别该标签，也不会在请求头发现该标记。
// 3. 如果是服务器返回的字段，则在任何浏览器中正确设置资源缓存时间。
```

如果用 Pargma 禁用缓存，又用Expires定义了一个缓存时间，刷新页面会发现 Pragma 字段优先级会高一点。  
注意系统时间跟服务器时间如果不一致，可能导致缓存没意义了。

## Cache-Control
Cache-Control 可以在请求或者响应报文中使用。若Pragma，Expires，Cache-Control 同时出现，则以 Cache-Control 为准。

## Last-Modified
如果客户端 If-Modified-Since 与服务端返回的 Last-Modified 不一致，则会返回新的资源。如果一致，直接返回 304 头部。
```
服务器: Last-Modified: Sat, 10 Dec 2016 15:21:21 GMT
客户端：If-Modified-Since: Sat, 10 Dec 2016 14:11:25 GMT
```

Last-Modified 有个问题，如果文件改动了，但实际内容没改动过，那么就会重新获取这个文件的内容（Last-Modified 不匹配）。

## Etag
Etag 是 http 1.1 新加的头部字段，他是服务器通过某种算法生成的值，这个值会返回给客户端，当再次请求的时候服务器发现 ETag 字段不一致，则会重新返回一个新资源。

## 200 OK from cache， 304 Not Modified
打开百度首页，从开发者工具中看到一些资源是 200 Ok from cache，而不是 304 Not Modified，这两者有什么区别呢？
其实 200 Ok from cache，是直接使用了浏览器中的缓存，而 304 Not Modified 是浏览器跟服务器确认了再使用的缓存。


## Chrome 的强制刷新
dev tools 勾上 disable cache，在请求资源的时候 Request Header 会加上这两个 Header

```
// 增加的 Header
Cache-Control:max-age=0, no-cache
Pragma:no-cache
```

并且会移除 If-Modified-Since 这个 Request Header。

```
// 移除的 Header
If-Modified-Since
```
