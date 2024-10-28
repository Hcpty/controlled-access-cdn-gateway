# Controlled-Access CDN
A note about Controlled-Access CDN.

### 带访问控制的内容分发网络

CDN的主要用途包括：
- 对来自Client的HTTP Request进行访问控制、反向代理和缓存服务。
- 对来自Data Center的HTTP Response进行缓存。

CDN执行任务的过程：
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

Permission Database由Data Center建立和运行，Permission Database中存储的数据结构：
- session_id -> user_groups
- Path -> resource_groups
- user_groups, Method, resource_groups -> remote|local|null

CDN对Permission Database的请求可能非常频繁，所以最好在靠近CDN的某个位置维持一些Permission Database的只读副本。

当权限查询结果是remote时，请求被转发到Data Center，Data Center应该再次对请求进行权限查询，因为write是更加敏感的操作。

Resource Database由Data Center建立和运行，Resource Database中存储的数据结构：
- Path -> metadata (包括last_modified等)

CDN对Resource Database的请求可能非常频繁，所以最好在靠近CDN的某个位置维持一些Resource Database的只读副本。

Cache Database由CDN建立和运行，Cache Database中存储的数据结构：
- Path -> metadata (包括last_modified等), resource_representation

CDN在Cache Database中刷新一个缓存时应该使用一个Mutex Lock。

注意last_modified使用GMT。

### Credits
- Computer Networking: A Top-Down Approach, Eighth Edition
