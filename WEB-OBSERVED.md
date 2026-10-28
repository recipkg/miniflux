# Web 阅读主流程：浏览器观察样本

固定来源为 [Miniflux 提交](https://github.com/miniflux/v2/tree/3945f3daa0a90040f761d79a5f62a678fe86ba4e)，以该提交构建的 `2.3.x-dev`、PostgreSQL 18.6、临时账户、英文默认设置及仅本机的单条 RSS 夹具运行。浏览器为 Playwright 驱动的 Chromium；宽屏截图为 1280×720，窄屏为 375×812。以下是该配置下观察到的行为，不代表全部设置、视口或辅助技术均已验证。

## 登录和导航

- 未登录访问 `/` 显示 `Sign In - Miniflux` 页面：用户名、密码和 Login；错误密码后页面显示 `Invalid username or password.`，用户名保留。有效本地账户提交后进入 `/unread`。无需把测试账户或密码写入实际配方或软件。
- 宽屏主导航从左到右包括主页、Unread（有未读时显示数量）、Starred、History、Feeds（附添加入口）、Categories、Search、Settings、Logout；还有跳转到内容的链接。导航的可访问名称和快捷键提示出现在页面语义中，但本次没有逐个测试快捷键执行。
- 375×812 视口下主导航折叠为可展开的 Menu，展开后仍可找到 Feeds 等入口；单条订阅页面的操作链接纵向排列。两种视口只核验上述样本，未对所有页面或视觉无障碍作全面结论。

## 阅读与收藏

- 未读页显示标题和数量、按页/全部标记已读操作、分页、文章标题、分类、订阅名、时间、预计阅读时间及单条已读、收藏和外部链接操作。首尾页时对应分页入口不可用。
- 点击单条 Star 后该控件变为 Unstar；打开该未读文章进入 `/unread/entry/{id}`，显示文章内容、订阅与分类、原文链接及 Mark as unread 和 Unstar 等操作。此样本的默认设置下打开文章后 API 状态变为 `read`，收藏保持 `true`；这两个状态相互独立。不同自动标记已读设置尚未实测。
- 返回 `/unread` 后，样本列表为空并显示 `There are no unread entries.`；Starred 页仍可找到刚收藏的文章。状态应持久保存，不应只靠页面临时改字。

## 订阅管理

- Feeds 页展示订阅、未读和文章总数、分类、网站、上次检查时间，以及 Refresh、Edit、Remove。子导航含 Feeds、Add feed、Export、Import 和后台刷新全部。
- Add feed 页包含 URL、Category、Advanced Options 和 Find a feed。选择分类并提交本机直接指向 RSS 的有效 URL 后，本样本直接进入新订阅的文章页；没有在此观察 HTML 自动发现、多候选、错误和权限边界，不能据此推断它们的行为。

## 外观和交互边界

实测默认浅色主题以白底、深灰正文、蓝色链接、细分隔线和紧凑文字为主；宽屏内容居中，文章列表以文本和操作为核心，窄屏显示为单列。配方中的通用设计依赖应用于实际产品任务，不等同于对上游所有主题的还原。未测试深色主题、各设置页面、键盘完整流程、屏幕阅读器、长文及慢网；这些仍须提取和验收。

[standard-library-verification](https://github.com/recipkg/standard-library-verification) 提供可选的 Playwright CLI 浏览器样本，使用临时数据库、账户、浏览器会话和本机 fixture，对登录失败、阅读、收藏、添加订阅、持久化状态及窄屏菜单执行断言；视觉观察仍为手工检查。上游通过不证明独立重建通过。
