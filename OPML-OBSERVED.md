# OPML 导入与导出：源码和隔离实例样本

观察来源为 [Miniflux 固定提交](https://github.com/miniflux/v2/tree/3945f3daa0a90040f761d79a5f62a678fe86ba4e) 的 `internal/api/opml_handlers.go` 和 `internal/reader/opml/`。样本用该提交构建的 `2.3.x-dev`、Go 1.27.3、PostgreSQL 18.6，关闭调度服务，在临时数据库及两个新建账户运行；样本地址使用 `.invalid` 域名，无须访问外部网站。下列具体结果仅对应此版本与配置。

## 请求与导入

- 经认证的 `POST /v1/import` 请求体直接传 OPML XML；成功返回 201 和 JSON `{"message":"Feeds imported successfully"}`。不是将 XML 放进 JSON 或 multipart。常规 API 认证适用；其他认证设置尚未在此测试。
- 样本把带 `xmlUrl` 的 outline 作为订阅，将外层 `Technology`、内层 `Nested` 两级分类各自对应的订阅导入为不同分类；最内层分类名为 `Nested`，不会拼成 `Technology/Nested`。根层订阅进入新账户已有的 `All` 分类。源码显示无分类时使用该账户的第一个分类，不能假定所有语言或已有账户都叫 `All`。
- 两次导入相同内容都返回 201；在同一账户已有相同 feed URL 的订阅被跳过，样本导入后仍只有三个订阅。另一账户可导入同样的 URL，生成自己的订阅 ID。源码按当前用户检查重复 URL；尚未穷尽大小写、URL 规范化和并发边界。
- 导入样本只创建订阅记录，没有立即抓取文章；调度服务在此测试中关闭。后续刷新和运行调度时行为仍需单独核实。没有 `htmlUrl` 的样本导出时以 `xmlUrl` 补作站点 URL。

## 导出与错误

- `GET /v1/export` 返回 200、`text/xml; charset=utf-8` 和 OPML 2.0；导出样本按当前账户分类分组，订阅包含标题、`xmlUrl`、`htmlUrl` 等。把导出内容导入第二个新账户后，三个 URL、标题与分类对应关系保留；两账户的订阅 ID 不同。导出包含创建时间，测试比较 XML 结构及语义，不比较完整字节。
- 输入 `not xml` 在该样本返回 500 和 JSON `error_message`。XML 内先放有效 HTTPS 订阅，再放 `ftp://` 订阅时返回 500；先前有效订阅依然存在，而失败订阅未创建，失败订阅所在的新分类已经创建。也就是说不能把此版本的导入当作全有或全无的事务。其他错误及顺序不应由此概括。
- 解析器、处理器与序列化器还处理 Miniflux 自有命名空间中的抓取、改写和其他订阅设置；具体属性、默认值、转义和所有错误尚未逐项对照，不把它们标为已验证。

可运行的隔离测试见 [standard-library-verification](https://github.com/recipkg/standard-library-verification) 的 `tests/test_miniflux.py`。上游测试通过不代表根据本配方重建成功。
