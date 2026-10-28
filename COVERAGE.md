# 证据与缺口

- 上游源码：[miniflux/v2](https://github.com/miniflux/v2)，提交 `3945f3daa0a90040f761d79a5f62a678fe86ba4e`；README 标注 Apache 2.0。开发版本字符串 `2.3.x-dev` 见 `internal/version/version.go`。
- 阅读：README、`go.mod`、`internal/api/api.go`、API 认证中间件、分类处理器与部分 UI 路由。未阅读完全部请求处理、数据库模型、模板与迁移。
- 上游源码验证：在隔离检出中用 Go 1.27.3 运行 `go test ./internal/reader/parser ./internal/reader/rss ./internal/reader/json ./internal/reader/opml`，共 165 个测试通过。这是上游解析器单元测试，不是重建验收，也不证明整个产品运行正常。
- 未验证：实际安装、PostgreSQL 服务、浏览器 UI、用户账户、API 请求和外部集成；尚缺字段级 API 契约、完整源格式行为、数据库迁移与恢复、安全配置、设计和无障碍细节。尚无独立生成的软件或真实端到端验收。
- 依赖 `quality` 和 `design` 均为固定提交的通用合同；未声明普通源码仓库为 Recipkg 依赖。
