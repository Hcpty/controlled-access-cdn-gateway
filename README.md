# Controlled-Access CDN
A note about Controlled-Access CDN.

### 带访问控制的内容分发网络

CDN的主要用途包括：
- 对来自Client的HTTP Request进行访问控制、反向代理和缓存服务。
- 对来自Data Center的HTTP Response进行缓存。

CDN处理任务的过程：
- CDN读取请求，这一步可以细分为三步：
  - CDN从HTTP Request中读取Cookie，然后从Cookie中读取session_id。
  - CDN从HTTP Request中读取 (Request) Method。
  - CDN从HTTP Request中读取 (URL) Path (without query)。
- CDN从Permission Mark Database中查询session_id, Method, Path -> choice的映射，这一步可以细分为三步：
  - CDN从Permission Mark Database中查询session_id -> user_groups的映射。
  - CDN从Permission Mark Database中查询Path -> resource_groups, resource_mark的映射。
  - CDN从Permission Mark Database中查询user_groups, Method, resource_groups -> choice的映射。
- 如果choice的值是remote或local，则说明可以提供服务，否则说明不可以提供服务，CDN可以作出三种不同的反应：
  - 如果choice的值是remote，则CDN扮演反向代理将请求转发给Data Center，并直接用Data Center的响应响应请求。
  - 如果choice的值是local，则CDN根据Path到Cache Database中查询resource_mark，并比较resource_mark的值：
    - 如果Permission Mark Database中的resource_mark和Cache Database中的resource_mark不同，那么说明Cache Database中的缓存已经过期，则CDN先刷新缓存，再响应请求。
    - 如果Permission Mark Database中的resource_mark和Cache Database中的resource_mark相同，那么说明Cache Database中的缓存尚未过期，则CDN直接使用缓存响应请求。
  - 如果choice的值是null，则CDN可以提示无法提供服务。

未进行登录的用户可以属于nobody用户组。

Permission Mark Database由Data Center建立和运行，Permission Mark Database中存储的数据结构：
- session_id -> user_groups
- Path -> resource_groups, resource_mark
- user_groups, Method, resource_groups -> choice

注意CDN对Permission Mark Database的请求可能非常频繁，所以最好在CDN上放置一些Permission Mark Database的只读副本，而且保证从副本到原本有一个较低的网络延迟。

Cache Database由CDN建立和运行，Cache Database中存储的数据结构：
- Path -> resource_mark, resource_metadata, resource_representation

为了刷新缓存，CDN可以通过一个Message Queue异步请求一个Cache Refresher进行刷新，并传入Path作为要刷新的目标，由Cache Refresher负责刷新。

注意要保证从CDN到Data Center有较大的网络带宽和较低的网络延迟。

注意CDN根据resource_mark进行Conditional GET，resource_mark可以是"If-Modified-Since: \<Last-Modified\>"的形式，也可以是"If-None-Match: \<ETag\>"的形式，如果两者都存在，则仅使用后者。

### Credits
- Computer Systems: A Programmer's Perspective, Third Edition
- Computer Networking: A Top-Down Approach, Eighth Edition
- [Understanding /etc/passwd File Format - nixCraft](https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format)
- [Representational State Transfer (REST) Architectural Style - Fielding Dissertation](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
- [If-Modified-Since/Last-Modified - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since)
- [If-None-Match/ETag - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match)
- [Available, Big and Fast - Hcpty](https://github.com/hcpty/available-big-and-fast)
