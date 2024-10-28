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
  - CDN从HTTP Request中读取 (URL) Path (with query)。
- CDN从Permission Condition Database中查询session_id, Method, Path -> choice的映射，这一步可以细分为三步：
  - CDN从Permission Condition Database中查询session_id -> user_groups的映射。
  - CDN从Permission Condition Database中查询Path -> resource_groups的映射。
  - CDN从Permission Condition Database中查询user_groups, Method, resource_groups -> choice的映射。
- 如果choice的值是remote或local，则说明可以提供服务，否则说明不可以提供服务，CDN可以作出三种不同的反应：
  - 如果choice的值是remote，则CDN扮演反向代理将请求转发给Data Center，并直接用Data Center的响应响应请求。
  - 如果choice的值是local，则CDN根据Path分别到Permission Condition Database和Cache Database中查询last_modified，并比较两个last_modified的值：
    - 如果Permission Condition Database中的last_modified晚于Cache Database中的last_modified，那么说明Cache Database中的缓存已经过期，则CDN先刷新缓存，再响应请求。
    - 如果Permission Condition Database中的last_modified等于Cache Database中的last_modified，那么说明Cache Database中的缓存尚未过期，则CDN直接使用缓存响应请求。
  - 如果choice的值是null，则CDN可以提示无法提供服务。

未进行登录的用户可以属于nobody用户组。

注意CDN应该在内部对 query fields 先进行排序再进行存储和匹配，以防止重复缓存。

Permission Condition Database由Data Center建立和运行，Permission Condition Database中存储的数据结构：
- session_id -> user_groups
- Path -> resource_groups
- user_groups, Method, resource_groups -> choice
- Path -> last_modified

注意CDN对Permission Condition Database的请求可能非常频繁，所以最好在靠近CDN的位置上放置一些Permission Condition Database的只读副本，而且保证从副本到原本有一个较低的网络延迟。

注意last_modified使用GMT。

Cache Database由CDN建立和运行，Cache Database中存储的数据结构：
- Path -> last_modified, resource_metadata, resource_representation

注意CDN在Cache Database中刷新缓存时应该使用Network Mutex Lock，以减少因多个节点多个事件并发地向Data Center请求同一个资源而带来的开销。

注意要保证从CDN到Data Center有较好的网络带宽和较低的网络延迟。

对于CDN的缓存功能，在某些情况下，可以使用If-None-Match/ETag解决方案替代If-Modified-Since/Last-Modified解决方案。

### Credits
- Computer Systems: A Programmer's Perspective, Third Edition
- Computer Networking: A Top-Down Approach, Eighth Edition
- [Understanding /etc/passwd File Format - nixCraft](https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format)
- [Representational State Transfer (REST) Architectural Style - Fielding Dissertation](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
- [HTTP headers - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [If-Modified-Since/Last-Modified - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since)
