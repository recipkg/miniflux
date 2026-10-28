# API v1 路由清单和边界

固定来源：[Miniflux 提交 `3945f3daa0a90040f761d79a5f62a678fe86ba4e`](https://github.com/miniflux/v2/tree/3945f3daa0a90040f761d79a5f62a678fe86ba4e)，`internal/api/api.go` 的 51 条注册路由及相应处理器。清单按源码记录方法、路径及目的；它不意味着所有字段、权限、错误、副作用或客户端都已实测。`API-OBSERVED.md` 和 `API-ENTRIES.md` 单列隔离实例的样本。`{...}` 是路径参数，具体编码及非法值需逐项验证。

## 账户

| 方法和路径 | 作用 |
| --- | --- |
| `POST /v1/users` | 创建账户 |
| `GET /v1/users` | 列出账户 |
| `GET /v1/users/{identifier}` | 按数字 ID 或用户名查账户 |
| `PUT /v1/users/{userID}` | 修改账户 |
| `DELETE /v1/users/{userID}` | 删除账户 |
| `PUT /v1/users/{userID}/mark-all-as-read` | 将当前账户的文章全部标为已读 |
| `GET /v1/me` | 读取当前账户 |

## 分类

| 方法和路径 | 作用 |
| --- | --- |
| `POST /v1/categories` | 创建分类 |
| `GET /v1/categories` | 列出分类；`counts` 可请求订阅数 |
| `PUT /v1/categories/{categoryID}` | 修改分类 |
| `DELETE /v1/categories/{categoryID}` | 删除分类 |
| `PUT /v1/categories/{categoryID}/mark-all-as-read` | 标记分类下文章已读 |
| `GET /v1/categories/{categoryID}/feeds` | 列出分类订阅 |
| `PUT /v1/categories/{categoryID}/refresh` | 排队刷新分类订阅 |
| `GET /v1/categories/{categoryID}/entries` | 查询分类文章 |
| `GET /v1/categories/{categoryID}/entries/{entryID}` | 查询分类内单篇文章 |

## 订阅与 OPML

| 方法和路径 | 作用 |
| --- | --- |
| `POST /v1/discover` | 从 URL 发现订阅 |
| `POST /v1/feeds` | 添加订阅 |
| `GET /v1/feeds` | 列出订阅 |
| `GET /v1/feeds/counters` | 查询订阅读取计数 |
| `PUT /v1/feeds/refresh` | 排队刷新符合条件的订阅 |
| `PUT /v1/feeds/{feedID}/refresh` | 刷新指定订阅 |
| `GET /v1/feeds/{feedID}` | 读取订阅 |
| `PUT /v1/feeds/{feedID}` | 修改订阅 |
| `DELETE /v1/feeds/{feedID}` | 删除订阅 |
| `GET /v1/feeds/{feedID}/icon` | 读取订阅图标 |
| `PUT /v1/feeds/{feedID}/mark-all-as-read` | 标记订阅下文章已读 |
| `GET /v1/export` | 导出 OPML |
| `POST /v1/import` | 导入 OPML |
| `GET /v1/feeds/{feedID}/entries` | 查询订阅文章 |
| `POST /v1/feeds/{feedID}/entries/import` | 导入单篇文章 |
| `GET /v1/feeds/{feedID}/entries/{entryID}` | 查询订阅内单篇文章 |

## 文章、附件与集成

| 方法和路径 | 作用 |
| --- | --- |
| `GET /v1/entries/ids` | 分页查询文章 ID |
| `GET /v1/entries` | 查询文章及总数 |
| `PUT /v1/entries` | 批量修改文章已读或收藏状态 |
| `GET /v1/entries/{entryID}` | 读取单篇文章 |
| `PUT /v1/entries/{entryID}` | 修改文章标题或正文 |
| `PUT /v1/entries/{entryID}/bookmark` | 切换收藏状态 |
| `PUT /v1/entries/{entryID}/star` | 切换收藏状态（同一处理器） |
| `POST /v1/entries/{entryID}/save` | 发送到配置的第三方保存服务 |
| `GET /v1/entries/{entryID}/fetch-content` | 获取文章全文；可请求更新已有正文 |
| `PUT /v1/flush-history` | 异步清理阅读历史 |
| `DELETE /v1/flush-history` | 异步清理阅读历史（同一处理器） |
| `GET /v1/icons/{iconID}` | 读取图标 |
| `GET /v1/enclosures/{enclosureID}` | 读取附件 |
| `PUT /v1/enclosures/{enclosureID}` | 修改附件 |
| `GET /v1/integrations/status` | 查询保存服务是否可用 |
| `GET /v1/version` | 读取服务版本 |

## API key

| 方法和路径 | 作用 |
| --- | --- |
| `POST /v1/api-keys` | 创建当前账户的 key |
| `GET /v1/api-keys` | 列出当前账户的 key |
| `DELETE /v1/api-keys/{apiKeyID}` | 删除当前账户的 key |

## 源码观察的通用行为

- 外层 CORS 中间件处理 OPTIONS 并返回 204；其他方法依次经过 API key 和 Basic 认证，最后进入路由。带 key 时先验证 key；无 key 时尝试 Basic。Basic 在禁用本地认证的配置下不可用。具体反向代理、跨源和部署限制仍待验证。
- 指定用户删除采用后台任务，响应成功不等于此时删除已完成；分类和批量订阅刷新也会将工作交给后台。单个订阅刷新处理器直接调用刷新，并支持 `force` 查询参数；不能把其返回时机混同于批量刷新。调度约束和失败重试未全面验证。
- 新建订阅不指定分类时源码尝试使用用户第一个分类；文章列表支持多种过滤、排序和分页，细节与实测边界见 `API-ENTRIES.md`。`fetch-content` 只有在 `update_content` 为真时请求保存抓取的内容；是否成功依赖外部页面与配置。
- 用户及资源所属关系不是全局一条简单规则；管理员检查、按当前用户查找、同一个路径中错误优先顺序均由具体处理器决定。已运行的代表性边界见 `API-OBSERVED.md`；必须逐项补齐余下合同。
