# 文章查询与导入：源码和实测样本

来源是 [Miniflux 固定提交](https://github.com/miniflux/v2/tree/3945f3daa0a90040f761d79a5f62a678fe86ba4e) 的 `internal/api/entry_handlers.go`、`messages.go`、请求参数和校验器。实际样本使用该提交构建的 `2.3.x-dev`、PostgreSQL 18.6、新建账户、关闭调度和本机 RSS fixture；只有下文明确写为样本的行为已经黑盒验证。该文件还不是完整文章、搜索或存储合同。

## 查询

- `GET /v1/entries` 返回 JSON `{total, entries}`；`total` 是匹配总数，不是当前页长度。样本按 `order=id&direction=asc&limit=1&offset=1` 查询三篇文章时返回总数 3 与中间一篇。样本中 `status=read`、`starred=true`、`search=Alpha` 过滤结果正确；不据此推广为完整全文搜索语义。
- 源码的默认排序为 `published_at` 升序，默认 limit 100、offset 0；允许排序字段为 `id`、`status`、`changed_at`、`published_at`、`created_at`、`category_title`、`category_id`、`title`、`author`，方向为 `asc`、`desc`。状态限 `read`、`unread`；不合法状态、排序字段、方向和不存在的正 `feed_id` 在样本中返回 400。
- 列表 limit 最大为 1000，样本的 1001 返回 400；`limit=-1` 被请求参数解析器替换为默认值，样本返回 200，而不是 400。不要简单地把所有非法数字都归为相同错误。
- 源码还支持 `feed_id`、`category_id`、重复的 `status` 和 `tags`、`globally_visible`、`before_entry_id`、`after_entry_id`、`before`、`after`、`published_before`、`published_after`、`changed_before`、`changed_after` 过滤。时间、标签、可见性、边界比较及空值行为尚未逐项实测。
- `GET /v1/entries/ids` 返回 `{total, entry_ids}`，源码按 ID 降序，limit 默认及最大 10000，超出范围或不大于零采用该默认值。样本 `starred=true` 返回收藏文章 ID；不合法的 `starred` 或 `status` 返回 400。区别是文章列表的 `starred=bogus` 在样本中被忽略，返回未按收藏过滤的结果，而 ID 接口返回 400。

## 导入及状态

- `POST /v1/feeds/{feedID}/entries/import` 接收 JSON；源码支持 `url`、`title`、`content`、`author`、`comments_url`、`published_at`（Unix 秒）、`status`、`starred`、`tags`、`external_id`。`url` 必填；没有标题时源码使用 URL，没有状态时使用 `read`。创建时返回 201 和 `{id}`；重复身份时返回 200 和原 `{id}`。源码以 `external_id` 为身份输入，未提供时使用 URL；具体散列碰撞、跨订阅及墓碑规则仍需独立验证。
- 样本先导入 `external_id=first`、`status=unread`、`starred=true` 的文章，随后以相同外部 ID、不同标题和 `status=read` 再导入，返回同一 ID；已有标题保持原值、状态变为已读、收藏保持 true。重复导入省略 `starred` 不能解释为清除收藏。其他字段更新与并发情形未穷尽。
- 导入内容经过 HTML 清理，样本中的 `<script>` 不出现在后续 GET 结果；这只是单一恶意内容样本，不证明清理器已覆盖所有攻击或安全要求。
- `PUT /v1/entries` 源码要求非空 `entry_ids`，且至少提供 `status` 或 `starred`；按当前账户更新指定状态。`PUT /v1/entries/{entryID}/bookmark` 与 `/star` 使用同一切换处理器，不是幂等的“设置 true”。重复请求、失效 ID 和跨用户行为仍须进一步对照。

可复现样本见 [standard-library-verification](https://github.com/recipkg/standard-library-verification) 的 `tests/test_miniflux.py`。上游通过不代表本配方已完整或独立重建已通过。
