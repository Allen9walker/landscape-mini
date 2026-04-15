# Contributing

## 开发流程

默认流程：

1. 从 `main` 拉新分支
2. 开发并做必要验证
3. 如属于用户可感知变更，更新 `CHANGELOG.md` 的 `Unreleased`
4. 提交 commit
5. push 分支
6. 开 PR 合并到 `main`
7. 等 CI 通过后合并

## 发版流程

`release.yml` 现在只会：

- 查找该 tag 所指向 commit 在 `main` 上对应的成功 `ci.yml` 运行记录
- 下载那次 CI 已验证的 4 个 tuple artifacts
- 校验 metadata / git SHA / tuple 完整性
- 压缩 `.img` 并创建 GitHub Release

它**不会**在 tag 触发时重新构建镜像。

当前 tuple 覆盖要求：

- `debian + include_docker=false`
- `debian + include_docker=true`
- `alpine + include_docker=false`
- `alpine + include_docker=true`
