# 论文工作流 Agent Skills 整合包

这个文件夹是论文工作流 Agent Skills 整合包，包含论文写作、文献检索、引用判断、AIGC 表达优化、Word 格式处理、PPT/图表、多 Agent 审查和 AnySearch 联网检索集成。

## 如何安装到 Codex

将 `codex-skills` 下的各个 skill 文件夹复制到：

`%USERPROFILE%\.codex\skills`

## 如何配置 AnySearch

1. 到 https://anysearch.com/console/api-keys 创建 API Key
2. 在 anysearch 文件夹中复制 `.env.example` 为 `.env`
3. 填入：

```text
ANYSEARCH_API_KEY=你的key
```

不要把 `.env` 发给别人，不要上传 GitHub。

## 如何使用

示例：

请使用 anysearch 检索 AlGaN/GaN HEMT self-heating diamond substrate，最多返回 3 条结果。

示例：

请使用 anysearch + nature-academic-search + nature-citation 检索并判断文献是否能支撑我的论文表述。

## 安全提醒

- 不要分享 `.env`
- 不要分享 API Key
- 不要分享完整未公开论文原稿
- 不要把检测报告原文、学校账号、个人信息交给联网搜索工具

## 适用范围

- 本科毕业论文
- 课程论文
- 科研论文初稿
- 文献检索
- Word 套格式
- AIGC 表达优化
- 论文转 PPT
- 多角色审查