# Miniflux 配方（提取中）

```toml
format = "recipkg/1"

[package]
name = "miniflux"
version = "0.1.0"
kind = "software"
description = "Miniflux 阅读器行为与接口的重建配方；当前仍在提取。"
entry = "BUILD_PROMPT.md"

[dependencies.quality]
git = "https://github.com/recipkg/recipkg.git"
rev = "41e5b55c6612a9a5a10e499b5ace6a1a3fed0f2b"
subdir = "recipes/software-quality"

[dependencies.design]
git = "https://github.com/recipkg/design-foundations.git"
rev = "bee6e280571b96128ce1541dddd51d11ec4dbf29"
```

观察来源：[Miniflux](https://github.com/miniflux/v2)，提交 `3945f3daa0a90040f761d79a5f62a678fe86ba4e`；上游 README 标明 Apache 2.0。此提交的默认开发版本字符串为 `2.3.x-dev`，不把它当成已发布版本。配方仅部分提取，不能宣称完整重建；见 `COVERAGE.md`。
