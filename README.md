# Controlled-Access CDN Gateway
A note about Controlled-Access CDN (Content Delivery Network) Gateway.

### 控制访问的CDN Gateway

CDN Gateway处理请求的过程：
- 先从HTTP Request中读取Cookie，接着从Cookie中读取Session ID，然后从HTTP Request中读取 (Request) Method和 (URL) Path (without query)。
- 然后从Permission Mark Database中查询`Session ID -> User Groups`的映射和`Path -> Resource Groups, Resource Mark`的映射，然后从Permission Mark Database中查询`User Groups, Method, Resource Groups -> Choice`的映射，然后查看查询结果：
  - 如果Choice的值是"dc"，说明将由DC (Data Center) Server提供服务，CDN Gateway将扮演反向代理让DC Server代为服务，并且不对该响应结果进行缓存。
  - 如果Choice的值是"cdn"，说明将由CDN Gateway提供服务，CDN Gateway先从Cache Database中查询`Path -> Resource Mark`的映射，然后比较两个Resource Mark的值，如果不同，说明缓存已过期，那么CDN Gateway先刷新缓存，再响应请求。如果相同，说明缓存未过期，那么CDN Gateway直接用缓存响应请求。
  - 如果未查询到Choice的映射，那么应该记录一条错误日志。

Permission Mark Database：

```
Path -> Resource Groups, Resource Mark
```

```
Session ID -> User Groups
```

```
User Groups, Method, Resource Groups -> Choice
```

Cache Database：
```
Path -> Resource Mark, Resource Metadata, Resource Representation
```

其他注意事项：
- 未注册用户默认属于nobody用户组，已注册用户默认属于somebody用户组。无效链接对应的资源默认属于nothing资源组，有效链接对应的资源默认属于something资源组。
- CDN Gateway对Permission Mark Database的请求可能非常频繁，所以要在CDN Gateway附近放置一些Permission Mark Database的只读副本。
- CDN Gateway根据Resource Mark进行Conditional GET，Resource Mark既可以是"If-Modified-Since: \<Last-Modified\>"的形式，也可以是"If-None-Match: \<ETag\>"的形式，还可以是两者同时存在的形式。

### Credits
- [Representational State Transfer Architectural Style - Roy Thomas Fielding](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
- [Understanding /etc/passwd File Format - nixCraft](https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format)
- [If-Modified-Since/Last-Modified - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since) and [If-None-Match/ETag - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match)
