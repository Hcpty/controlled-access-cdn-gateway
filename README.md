# Controlled-Access CDN
A note about Controlled-Access CDN.

### 控制访问的内容分发网络

请求和响应的过程：
- CDN从HTTP Request中读取Cookie，然后Cookie中读取session_id。
- CDN从HTTP Request中读取Verb。
- CDN从HTTP Request中读取resource_id。
- CDN从Database中查询session_id+Verb+resource_id -> local|remote|null的映射，这一步可以细分为三步：
  - CDN从Database中查询session_id -> user_groups的映射。
  - CDN从Database中查询resource_id -> resource_groups的映射。
  - CDN从Database中查询user_groups+Verb+resource_groups -> local|remote|null 的映射。
- 根据上一步的查询结果，CDN可以作出三种不同的反应：
  - 如果查询结果是local，则CDN使用resource_id到Datastore（存储资源的Database）中取出对应的资源响应请求。
  - 如果查询结果是remote，则CDN扮演反向代理将本次请求转发给Data Center处理。
  - 如果上一步的查询结果是null，则CDN可以拒绝请求。

这个Database由Data Center负责为CDN生成，CDN通过Database Replication机制在本地维持一个Database的只读副本，这个只读副本中包含如下数据类型：
- session_id -> user_groups
- resource_id -> resource_groups
- user_groups+Verb+resource_groups -> local|remote|null

这个Datastore也由Data Center负责为CDN生成，CDN通过Datastore Replication机制在本地维持一个Datastore的只读副本，这个只读副本中包含如下数据类型：
- resource_id -> resource_representation

### Credits
- Computer Networking: A Top-Down Approach, Eighth Edition
