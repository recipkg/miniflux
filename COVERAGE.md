# 证据与缺口

- 上游源码：[miniflux/v2](https://github.com/miniflux/v2)，提交 `3945f3daa0a90040f761d79a5f62a678fe86ba4e`；README 标注 Apache 2.0。开发版本字符串 `2.3.x-dev` 见 `internal/version/version.go`。
- 阅读：README、`go.mod`、`internal/api/api.go`、API 认证中间件、分类处理器与部分 UI 路由。未阅读完全部请求处理、数据库模型、模板与迁移。
- 上游源码验证：在隔离检出中用 Go 1.27.3 运行 `go test ./internal/reader/parser ./internal/reader/rss ./internal/reader/json ./internal/reader/opml`，共 165 个测试通过。这是上游解析器单元测试，不是重建验收，也不证明整个产品运行正常。
- 本轮从固定提交以 Go 1.27.3 构建上游，使用 PostgreSQL 18.6 新建仅 Unix socket 的临时数据库、只绑定 `127.0.0.1` 的应用与 RSS fixture，并启动迁移及临时账户；真实运行 API 场景见 `API-OBSERVED.md` 和黑盒测试。测试结束已关闭本次服务并删除临时数据。未接触已有数据库。
- 固定提交的 OPML 处理器、解析器、序列化器和真实隔离账户样本进一步核对了嵌套分类、去重、导出再导入及失败后的部分写入；细节和边界见 `OPML-OBSERVED.md`。这些仍仅为局部证据，不是完整 OPML 或产品重建验证。
- 浏览器 UI：使用 Playwright Chromium 检查隔离上游实例的登录、阅读、收藏、直接添加 RSS、宽窄屏及菜单，观察细节和限制见 `WEB-OBSERVED.md`；其中部分流程有可重复的浏览器样本。未做全面 UI、主题或辅助技术验收。
- 未验证：完整 API 和用户管理、其他 Feed 格式及边界、数据库备份恢复、生产安全配置、外部集成、所有界面和无障碍。此测试是原软件的局部端到端行为，不是配方重建验收；尚无独立生成的软件。
- 依赖 `quality` 和 `design` 均为固定提交的通用合同；`design` 继续依赖固定提交的 `foundation`。未声明普通源码仓库为 Recipkg 依赖。
