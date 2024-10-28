# Controlled-Access CDN
A note about Controlled-Access CDN.

### 控制访问的内容分发网络

请求和响应的过程：
- CDN从HTTP Request中读取Cookie，然后从Cookie中读取session_id。
- CDN从HTTP Request中读取 (Request) Method。
- CDN从HTTP Request中读取 (URL) Path (with query)。
- CDN从Permission Database中查询session_id, Method, Path -> remote|local|null的映射，这一步可以细分为三步：
  - CDN从Permission Database中查询session_id -> user_groups的映射。
  - CDN从Permission Database中查询Path -> resource_groups的映射。
  - CDN从Permission Database中查询user_groups, Method, resource_groups -> remote|local|null 的映射。
- 根据上一步的查询结果，CDN可以作出三种不同的反应：
  - 如果查询结果是remote，则CDN扮演反向代理的角色将本次请求转发给Data Center处理。
  - 如果查询结果是local，则CDN根据Path分别到Cache Database和Resource Database中查询last_modified，并比较两个last_modified的值：
    - 如果Cache Database中的last_modified早于Resource Database中的last_modified，那么说明Cache Database中的缓存已经过期，则CDN先刷新缓存，然后响应请求。
    - 如果Cache Database中的last_modified等于Resource Database中的last_modified，那么说明Cache Database中的缓存尚未过期，则CDN直接响应请求。
  - 如果上一步的查询结果是null，则CDN可以拒绝服务本次请求。

Permission Database：
- session_id -> user_groups
- Path -> resource_groups, last_modified
- user_groups, Method, resource_groups -> remote|local|null

Resource Database：
- Path -> last_modified

Cache Database：
- Path -> last_modified, resource_representation

### Credits
- Computer Networking: A Top-Down Approach, Eighth Edition
