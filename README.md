# Controlled-Access CDN
A note about Controlled-Access CDN.

### 控制访问的内容分发网络

请求和响应的过程：
- CDN从HTTP Request中读取Cookie，然后Cookie中读取session_id。
- CDN从HTTP Request中读取action。
- CDN从HTTP Request中读取resource_id。
- CDN从Database中查询session_id+can_action+resource_id -> true|null的映射，这一步可以细分为三步：
  - 从Database中查询session_id -> user_group的映射。
  - 从Database中查询resource_id -> resource_group的映射。
  - 从Database中查询user_group+can_action+resource_group -> true|null 的映射。

CDN通过Database Replication在本地维持一个Database的只读副本。

### Credits
- Computer Networking: A Top-Down Approach, Eighth Edition
