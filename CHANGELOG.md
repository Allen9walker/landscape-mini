# Changelog / 变更日志

All notable changes to this project will be documented in this file.
本文件记录项目的所有重要变更。

## [Unreleased]

### Changed / 变更

- Replace the legacy variant model with an explicit build identity model based on `base_system`, `include_docker`, and `output_formats`, update local build UX, rework tests to consume metadata, and fold `pve-ova` into the normal exporter pipeline / 用 `base_system`、`include_docker`、`output_formats` 替换旧的 variant 模型，更新本地构建体验，让测试改为消费 metadata，并将 `pve-ova` 纳入常规导出流水线
- Rewrite CI, Custom Build, retest, release promotion, and repository docs around tuple-based build identities instead of named variants / 将 CI、Custom Build、复测、release promotion 与仓库文档统一重写为基于 tuple 的构建身份模型，不再围绕命名 variant 运转

## [0.2.6] - 2026-04-14

### Fixed / 修复

- Publish only release image archives and stop uploading duplicate metadata files as GitHub release assets / 发布时仅上传镜像压缩包，不再将重复的 metadata 文件作为 GitHub release 资产上传
