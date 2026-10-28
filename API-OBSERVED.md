# API 观察：分类、认证与 RSS 主路径

来源：Miniflux 提交 `3945f3daa0a90040f761d79a5f62a678fe86ba4e` 的 `internal/api/`、`internal/model/category.go`、`internal/model/entry.go` 和验证器；并以该提交构建的 `2.3.x-dev`、Go 1.27.3、PostgreSQL 18.6 在独立临时实例执行下列用例。默认认证和错误信息、配置与版本可能改变，不能将这些样本推广到全部部署或 API。

## 认证和账户隔离

- 在本实验中本地 Basic 认证启用：`GET /v1/categories` 无凭据、`GET /v1/me` 错误密码均返回 401，JSON 错误对象带 `error_message`；正确 Basic 返回 200 和当前用户 JSON。
- 已认证用户 `POST /v1/api-keys`，JSON `{"description":"fixture key"}`，返回 201 及 `id`、`token` 等字段；以 `X-Auth-Token` 请求 `/v1/me` 返回该用户。不存在的 token 返回 401；`DELETE /v1/api-keys/{id}` 返回 204 后该 token 返回 401。不要打印、暴露或在配方中存真实密钥。
- 管理员 `POST /v1/users`，JSON 包含 `username`、`password`、`is_admin:false`，本样本返回 201。另一账户 `GET /v1/categories` 只列出其分类；对属于首个账户的分类 ID 执行 PUT 返回 404。其他角色、接口和认证配置未完整实测，不能据此概括全部权限。

## 分类

- `POST /v1/categories` 接收 JSON `title`、`hide_globally`；有效请求返回 201，响应字段包括 `id`、`title`、`user_id`、`hide_globally`。空标题、同一用户的重复标题和无效 JSON 均在本样本返回 400。创建时同名检查作用于当前用户，字符边界和并发语义未全面验证。
- `GET /v1/categories` 返回当前用户分类数组。每个新账户在本次配置下有一个 `All` 分类，不将其标签/语言概括为所有配置。
- `PUT /v1/categories/{id}` 接受 `title`、`hide_globally` 的可选变更，成功返回 **201** 和更新后的对象；空字段、重复名称、无权限和不存在对象按具体场景测试。`DELETE` 成功返回 204，再次删除同一 ID 返回 404。删除带订阅分类的后果未在此样本测试。

## 订阅与文章

- 在本地测试中，以 `FETCHER_ALLOW_PRIVATE_NETWORKS=1` 显式允许访问隔离的本地 RSS fixture；这只是测试前提，不是生产默认或通用安全配置。`POST /v1/feeds` 的 `feed_url` 与 `category_id` 指向有效分类和单条文章的 RSS 2.0，返回 201 和 `{"feed_id":...}`；`GET /v1/feeds/{id}` 返回 200 和上游 feed title。
- `GET /v1/feeds/{id}/entries` 返回对象，`total` 为 1，`entries` 含标题、唯一 ID、`status:"unread"`、`starred:false`。`PUT /v1/entries` 提交 `{"entry_ids":[id],"status":"read","starred":true}` 返回 204，再读文章确认 `read` 和已收藏；无效 status 返回 400。
- `PUT /v1/feeds/{id}/refresh` 在该 fixture 返回 204；立即再查仍有同一文章 ID 且保留已读/收藏状态。仅说明该次相同 RSS 文档的行为，不推断全部去重、错误、更新或并发规则。`DELETE /v1/feeds/{id}` 返回 204，随后删除空分类也返回 204。

黑盒样本在 [standard-library-verification](https://github.com/recipkg/standard-library-verification) 的 `tests/test_miniflux.py`；提供实际二进制路径后运行。测试仅覆盖上述局部功能，配方没有因此成为完整软件规范。
