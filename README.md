# Controlled-Access CDN
A note about Controlled-Access CDN.

### 带访问控制的内容分发网络

请求和响应的过程：
- CDN读取请求，这一步可以细分为三步：
  - CDN从HTTP Request中读取Cookie，然后从Cookie中读取session_id。
  - CDN从HTTP Request中读取 (Request) Method。
  - CDN从HTTP Request中读取 (URL) Path (with query)。
- CDN从Permission Database中查询session_id, Method, Path -> remote|local|null的映射，这一步可以细分为三步：
  - CDN从Permission Database中查询session_id -> user_groups的映射。
  - CDN从Permission Database中查询Path -> resource_groups的映射。
  - CDN从Permission Database中查询user_groups, Method, resource_groups -> remote|local|null 的映射。
- 如果权限查询的结果是remote或local，则说明有权限，否则说明没有权限，CDN可以作出三种不同的反应：
  - 如果查询结果是remote，则CDN扮演反向代理将请求转发给Data Center，并直接用Data Center的响应响应请求。
  - 如果查询结果是local，则CDN根据Path分别到Resource Database和Cache Database中查询last_modified，并比较两个last_modified的值：
    - 如果Resource Database中的last_modified晚于Cache Database中的last_modified，那么说明Cache Database中的缓存已经过期，则CDN先刷新缓存，再响应请求。
    - 如果Resource Database中的last_modified等于Cache Database中的last_modified，那么说明Cache Database中的缓存尚未过期，则CDN直接响应请求。
  - 如果上一步的查询结果是null，则CDN可以拒绝服务。

Permission Database由Data Center建设和运营，Permission Database中存储的数据结构：
- session_id -> user_groups
- Path -> resource_groups
- user_groups, Method, resource_groups -> remote|local|null

CDN对Permission Database的请求可能非常频繁，所以应该在靠近CDN的某个位置维持一个Permission Database的只读副本。

Resource Database由Data Center建设和运营，Resource Database中存储的数据结构：
- Path -> last_modified

CDN对Resource Database的请求可能非常频繁，所以应该在靠近CDN的某个位置维持一个Resource Database的只读副本。

Cache Database由CDN建设和运营，Cache Database中存储的数据结构：
- Path -> last_modified, resource_representation

CDN在Cache Database中刷新一个缓存时应该使用一个Mutex Lock。

### Credits
- Computer Networking: A Top-Down Approach, Eighth Edition
