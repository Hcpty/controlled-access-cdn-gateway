# Controlled-Access CDN Gateway
A note about Controlled-Access CDN (Content Delivery Network) Gateway.

### 带访问控制的CDN网关

CDN网关处理请求的过程如下：
- CDN网关读取请求，这一步可以细分为三步：
  - CDN网关从HTTP Request中读取Cookie，然后从Cookie中读取Session ID。
  - CDN网关从HTTP Request中读取 (Request) Method。
  - CDN网关从HTTP Request中读取 (URL) Path (without query)。
- CDN网关从Permission Mark Database中查询Session ID, Method, Path -> Choice的映射，这一步也可以细分为三步：
  - CDN网关从Permission Mark Database中查询Session ID -> User Groups的映射。
  - CDN网关从Permission Mark Database中查询Path -> Resource Groups, Resource Mark的映射。
  - CDN网关从Permission Mark Database中查询User Groups, Method, Resource Groups -> Choice的映射。
- 如果Choice的值是remote或local，则说明可以提供服务，否则说明不可以提供服务，CDN网关可以作出三种不同的反应：
  - 如果Choice的值是remote，则CDN网关扮演反向代理将请求转发给Data Center，并直接用Data Center的响应响应请求，并且不对响应进行缓存。
  - 如果Choice的值是local，则CDN网关根据Path到Cache Database中查询Resource Mark，并比较Resource Mark的值：
    - 如果Permission Mark Database中的Resource Mark和Cache Database中的Resource Mark不同，那么说明Cache Database中的缓存已经过期，则CDN网关先刷新缓存，再响应请求。
    - 如果Permission Mark Database中的Resource Mark和Cache Database中的Resource Mark相同，那么说明Cache Database中的缓存尚未过期，则CDN网关直接使用缓存响应请求。
  - 如果Choice的值是null，则CDN网关可以提示无法提供服务。

Permission Mark Database：

```
Session ID -> User Groups
```

```
Path -> Resource Groups, Resource Mark
```

```
User Groups, Method, Resource Groups -> Choice
```

Cache Database：
```
Path -> Resource Mark, Resource Metadata, Resource Representation
```

注意事项：
- 未进行登录的用户可以属于nobody用户组。
- CDN网关对Permission Mark Database的请求可能非常频繁，所以要在CDN网关附近放置一些Permission Mark Database的只读副本。
- 为了刷新缓存，CDN网关可以通过一个Message Queue异步请求一个Cache Refresher进行刷新，并传入Path作为要刷新的目标，由Cache Refresher负责刷新。
- CDN网关根据Resource Mark进行Conditional GET，Resource Mark既可以是"If-Modified-Since: \<Last-Modified\>"的形式，也可以是"If-None-Match: \<ETag\>"的形式，还可以是两者同时存在的形式。

### Credits
- [Representational State Transfer Architectural Style - Fielding Dissertation](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
- [If-Modified-Since/Last-Modified - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since) and [If-None-Match/ETag - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match)
- [Understanding /etc/passwd File Format - nixCraft](https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format)
