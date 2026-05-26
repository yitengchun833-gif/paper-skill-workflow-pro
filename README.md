[README_论文工作流使用说明.md](https://github.com/user-attachments/files/28281684/README_.md)
# 论文工作流使用说明

> 适用对象：Codex / Claude Code / Claude Skills / Cursor / Windsurf / 其他支持 `SKILL.md` 或项目规则的 AI Agent  
> 当前定位：论文写作、文献检索、引用判断、AIGC 表达优化、Word 论文套格式、科研图表、论文转 PPT、多 Agent 审查的一体化工作流  
> 重要原则：不覆盖原始文件、不伪造数据、不泄露 API Key、不把完整未公开论文交给联网搜索工具。

---

## 1. 这个工作流是什么

这是一套面向论文任务的 Agent Skills 整合包，核心目标是把论文任务拆成多个专用模块，让 AI 按任务类型自动分流，而不是一次性全仓库扫描、全文件乱改。

它主要解决以下问题：

1. 论文初稿写作、章节扩写、摘要与结论优化。
2. 文献检索、DOI / PMID / arXiv 信息整理。
3. 判断文献是否能支撑论文正文中的某个表述。
4. 根据 AIGC / PaperPass 等检测报告优化表达，降低模板化痕迹。
5. 按学校格式要求处理 Word 论文格式。
6. 生成论文汇报、答辩 PPT、图表审查建议。
7. 通过多 Agent 分工做最终审查。
8. 使用 AnySearch 实现实时联网检索和网页内容提取。
9. 使用轻量模型处理简单任务，复杂任务再升档，减少 Codex 额度浪费。

---

## 2. 总体工作流架构

推荐理解成下面这个结构：

```text
用户任务
  ↓
AGENTS.md / multi-agent-workflow 判断任务类型
  ↓
分配到对应 Skill
  ↓
执行任务
  ↓
输出结果 / 新文件 / 审查报告
```

常见路由如下：

| 任务类型 | 优先使用的 Skill |
|---|---|
| 实时联网检索、网页提取、官方资料核验 | `anysearch` |
| 学术文献检索、DOI / PMID / arXiv 初筛 | `anysearch` + `nature-academic-search` |
| 判断文献是否支撑正文表述 | `nature-citation` |
| 阅读、翻译、总结论文 PDF | `nature-reader` |
| 写摘要、引言、结果、讨论、结论 | `nature-writing` |
| 学术润色、中英翻译 | `nature-polishing` |
| 审稿意见回复、修改说明 | `nature-response` |
| Word 论文格式识别与修改 | `thesis-format-module-editor` |
| AIGC 表达优化、降低模板感 | `aigc-academic-expression-optimizer` |
| 段落级表达优化 | `aigc-academic-expression-optimization` |
| 数据可用性、FAIR、数据说明 | `nature-data` |
| 科研图件检查、图注优化 | `nature-figure` |
| 论文转 PPT、答辩汇报 | `nature-paper2ppt` |
| 多角色综合审查 | `multi-agent-workflow` |

---

## 3. 文件夹结构建议

共享版建议采用以下结构：

```text
paper-skill-workflow-share/
├── README_使用说明.md
├── FILE_LIST.txt
├── paper-skill-workflow/
│   ├── README.md
│   ├── AGENTS.md
│   ├── workflow_manifest.json
│   ├── install_workflow_skills.ps1
│   └── ...
└── codex-skills/
    ├── anysearch/
    ├── thesis-format-module-editor/
    ├── aigc-academic-expression-optimizer/
    ├── aigc-academic-expression-optimization/
    ├── multi-agent-workflow/
    ├── nature-academic-search/
    ├── nature-citation/
    ├── nature-data/
    ├── nature-figure/
    ├── nature-paper2ppt/
    ├── nature-polishing/
    ├── nature-reader/
    ├── nature-response/
    └── nature-writing/
```

---

## 4. 安装到 Codex

### 4.1 推荐安装位置

Windows 上推荐安装到：

```text
%USERPROFILE%\.codex\skills
```

你的本机通常对应：

```text
C:\Users\llc\.codex\skills
```

### 4.2 手动安装方法

把 `codex-skills` 下面的每个 Skill 文件夹复制到：

```text
C:\Users\llc\.codex\skills
```

复制后结构应类似：

```text
C:\Users\llc\.codex\skills\anysearch
C:\Users\llc\.codex\skills\nature-academic-search
C:\Users\llc\.codex\skills\nature-citation
C:\Users\llc\.codex\skills\thesis-format-module-editor
```

### 4.3 PowerShell 批量安装示例

在共享包根目录运行：

```powershell
$Source = ".\codex-skills"
$Target = "$env:USERPROFILE\.codex\skills"

if (!(Test-Path $Target)) {
    New-Item -ItemType Directory -Force -Path $Target | Out-Null
}

Get-ChildItem $Source -Directory | ForEach-Object {
    $SkillName = $_.Name
    $Dest = Join-Path $Target $SkillName

    if (Test-Path $Dest) {
        $BackupRoot = Join-Path $Target ".trash"
        if (!(Test-Path $BackupRoot)) {
            New-Item -ItemType Directory -Force -Path $BackupRoot | Out-Null
        }

        $BackupPath = Join-Path $BackupRoot ("{0}_{1}" -f $SkillName, (Get-Date -Format "yyyyMMdd_HHmmss"))
        Move-Item $Dest $BackupPath -Force
        Write-Host "已备份旧 Skill：$BackupPath"
    }

    Copy-Item $_.FullName $Dest -Recurse -Force
    Write-Host "已安装 Skill：$SkillName"
}

Write-Host "安装完成，请重启 Codex。"
```

---

## 5. AnySearch 配置说明

### 5.1 AnySearch 是什么

`anysearch` 是实时搜索 Skill，适合用于：

1. 普通网页搜索。
2. 学术文献发现。
3. DOI / PMID / arXiv 等信息初筛。
4. 官方资料核验。
5. 批量关键词搜索。
6. 网页内容提取。

在论文工作流中，它不是替代论文 Skill，而是作为“联网搜索底层工具”。

推荐链路：

```text
anysearch → nature-academic-search → nature-citation
```

含义：

```text
联网查资料 → 筛选学术来源 → 判断是否支撑论文正文
```

### 5.2 是否必须配置 API Key

AnySearch 可以匿名使用，但匿名模式额度和速率较低。若希望长期稳定使用，建议配置 API Key。

### 5.3 获取 API Key

打开：

```text
https://anysearch.com/console/api-keys
```

登录后创建 API Key。

安全要求：

1. 不要把 API Key 发到聊天里。
2. 不要截图展示 API Key。
3. 不要把 API Key 写入 README。
4. 不要把 API Key 提交到 GitHub。
5. 不要把 `.env` 分享给别人。

### 5.4 配置 `.env`

进入 AnySearch Skill 目录：

```powershell
cd "$env:USERPROFILE\.codex\skills\anysearch"
```

创建 `.env`：

```powershell
$ApiKey = Read-Host -Prompt "请粘贴 ANYSEARCH_API_KEY"
"ANYSEARCH_API_KEY=$ApiKey" | Set-Content -Path ".env" -Encoding UTF8
```

注意：只有出现 `请粘贴 ANYSEARCH_API_KEY:` 提示时，才粘贴 Key。不要把 Key 直接粘到 PowerShell 命令行，否则 PowerShell 会把它当成命令执行。

### 5.5 验证 `.env`

```powershell
$EnvPath = "$env:USERPROFILE\.codex\skills\anysearch\.env"

if (Test-Path $EnvPath) {
    Write-Host ".env 存在"
    if ((Get-Content $EnvPath) -match "^ANYSEARCH_API_KEY=") {
        Write-Host "ANYSEARCH_API_KEY 已存在"
    } else {
        Write-Host "ANYSEARCH_API_KEY 未写入"
    }
} else {
    Write-Host ".env 不存在"
}
```

此命令只检查是否存在，不会显示 Key 内容。

---

## 6. runtime.conf 配置说明

### 6.1 runtime.conf 的作用

`runtime.conf` 用来告诉 Codex：以后调用 AnySearch 时优先用哪个运行时。

推荐优先级：

```text
Python > Node.js > PowerShell
```

### 6.2 Python 版本示例

如果 Python 可用，`runtime.conf` 推荐内容：

```text
Runtime: Python
Command: python C:\Users\llc\.codex\skills\anysearch\scripts\anysearch_cli.py
```

### 6.3 共享包里的写法

如果要发给别人，不建议保留本机绝对路径。共享包中应提供：

```text
runtime.conf.example
```

内容：

```text
Runtime: Python
Command: python scripts/anysearch_cli.py
```

让对方根据自己的路径自行配置。

### 6.4 检查核心文件是否存在

```powershell
Test-Path "$env:USERPROFILE\.codex\skills\anysearch\.env"
Test-Path "$env:USERPROFILE\.codex\skills\anysearch\runtime.conf"
Test-Path "$env:USERPROFILE\.codex\skills\anysearch\SKILL.md"
```

如果全部返回：

```text
True
True
True
```

说明核心安装正常。

---

## 7. AGENTS.md 路由规则

为了让 Codex 稳定使用 AnySearch，而不是有时用、有时不用，建议在项目根目录的 `AGENTS.md` 中加入以下规则：

```markdown
## AnySearch routing rule

When the task requires current web search, real-time source lookup, webpage extraction, batch search, DOI lookup, literature discovery, official source verification, or online fact checking, use `anysearch` first.

Use `anysearch` for:
- latest information
- official source lookup
- academic literature discovery
- DOI / PMID / arXiv lookup
- webpage extraction
- batch keyword search
- citation source discovery
- verifying whether a source exists online

Do not use `anysearch` for:
- rewriting thesis text
- editing Word formatting
- private document analysis
- full thesis draft reading
- AIGC report rewriting
- passwords
- API keys
- private identity data
- unpublished full thesis drafts
- school account information
- detection report raw content

For thesis workflows:
- real-time literature search → anysearch + nature-academic-search
- citation support → nature-citation
- paper reading → nature-reader
- thesis writing → nature-writing
- Word formatting → thesis-format-module-editor
- AIGC expression optimization → aigc-academic-expression-optimizer
- workflow coordination → multi-agent-workflow
```

---

## 8. Codex 省额度使用方法

### 8.1 推荐模式

| 场景 | 推荐 profile | 模型策略 |
|---|---|---|
| 文件检查、目录确认、简单复制 | `light` | 轻量、低推理 |
| README、Skill 描述、普通修改 | `normal` | 中等推理 |
| Word 格式失败诊断、复杂流程分析 | `deep` | 高推理 |
| 最终提交前审查 | `final` | 最高推理 |

### 8.2 使用示例

简单任务：

```powershell
codex --profile light "只检查文件名和目录结构，不要全项目扫描"
```

普通任务：

```powershell
codex --profile normal "优化 README 和 Skill 描述，只改当前文件"
```

复杂任务：

```powershell
codex --profile deep "分析论文 Word 格式流程为什么失败，先给计划，不要直接修改"
```

最终审查：

```powershell
codex --profile final "最终检查论文格式处理流程，重点检查是否误改正文"
```

---

## 9. 常用任务提示词

### 9.1 测试 AnySearch 是否生效

```text
请使用 anysearch 检索 AlGaN/GaN HEMT self-heating diamond substrate，最多返回 3 条结果，并说明是否成功调用 anysearch。不要改任何文件。
```

### 9.2 测试论文文献链路

```text
请先用 anysearch 检索 AlGaN/GaN HEMT diamond substrate self-heating 的论文资料，再交给 nature-academic-search 筛选学术来源，最后让 nature-citation 判断哪些文献可以支撑“金刚石衬底有利于缓解 GaN HEMT 自热效应”这个表述。不要修改文件。
```

### 9.3 检索可用于论文的文献

```text
请使用 anysearch + nature-academic-search + nature-citation，围绕“金刚石衬底缓解 AlGaN/GaN HEMT 自热效应”检索 5 篇可用于本科毕业论文的文献，要求输出：题名、作者、年份、DOI、可支撑的论文表述、适合放在第几章。不要修改文件。
```

### 9.4 写论文段落

```text
请使用 nature-writing，根据我提供的图表和实验说明，撰写第 3.2 节输出特性与转移特性分析。要求：只基于我已有图表，不编造原始数据；所有图像读数写成“由图估算”或“约为”；不要写我没有的晶格温度图或焦耳热密度图。
```

### 9.5 判断文献是否支撑句子

```text
请使用 nature-citation 判断以下文献是否能支撑这句话：“金刚石衬底有利于缓解 AlGaN/GaN HEMT 自热效应”。请输出支撑强度、适合引用的位置、是否需要改写句子。
```

### 9.6 Word 格式处理

```text
请使用 thesis-format-module-editor，根据学校格式要求检查我的论文 Word。仅限检查和修改目录、大标题、小标题、页眉页脚、页码。禁止修改正文语义。修改前先备份，输出新文件。
```

### 9.7 AIGC 表达优化

```text
请使用 aigc-academic-expression-optimizer，根据检测报告定位高风险段落，只做学术表达自然化、证据绑定和模板化表达修正。不要改变论文结论，不要编造数据，不承诺具体检测分数。
```

### 9.8 多 Agent 最终审查

```text
请使用 multi-agent-workflow，从格式、数据一致性、文献支撑、AIGC 表达、答辩风险五个角度审查我的论文。先给审查计划，不要直接修改文件。
```

---

## 10. AlGaN/GaN HEMT 论文专用规则

如果用于 AlGaN/GaN HEMT、自热效应、蓝宝石与金刚石衬底对比论文，请遵守以下规则：

1. 论文重点偏向热效应、自热效应、散热路径、热源分布。
2. 蓝宝石组与金刚石组的核心区别是底部衬底材料不同。
3. 其他主要结构参数应默认保持一致，用于隔离衬底热效应。
4. 如果只有曲线图，没有原始导出数据，数值必须写成“由图读数估算”。
5. 如果没有晶格温度图，不要写晶格温度分布。
6. 如果没有焦耳热密度图，不要写焦耳热密度分布。
7. 如果只有温度云图或热分布图，就围绕已有图进行解释。
8. 不要把“仿真结果”写成“实验结果”，除非用户明确说明是真实实验。
9. 引用文献只能支撑一般物理机制，不能替代用户自己的仿真数据。
10. 对“金刚石衬底改善散热”这类结论应写得有边界，例如：

```text
由于金刚石具有较高热导率，将其作为衬底或散热结构通常有利于降低 GaN HEMT 沟道温升，从而缓解器件自热效应；但实际改善程度仍受 GaN/diamond 界面热阻、外延转移质量及器件结构等因素影响。
```

---

## 11. 安全规则

### 11.1 严禁分享的内容

不要分享或上传：

```text
.env
ANYSEARCH_API_KEY
OpenAI API Key
GitHub Token
学校账号
检测报告原文
身份证号
完整未公开论文原稿
个人隐私数据
```

### 11.2 联网检索注意事项

AnySearch 适合查公开资料，不适合查私密内容。不要把以下内容交给联网搜索工具：

```text
完整论文原稿
未公开实验数据
检测报告原文
个人身份信息
账号密码
API Key
```

### 11.3 GitHub 提交前检查

提交或打包前检查：

```powershell
Select-String -Path ".\*" -Pattern "ANYSEARCH_API_KEY","as_sk_","password","secret","github_token" -Recurse
```

如果发现敏感内容，不要提交，不要打包，先删除或替换为示例值。

---

## 12. 共享给别人前的脱敏规则

共享包必须排除：

```text
.git
.env
*.env
.env.*
node_modules
__pycache__
.pytest_cache
.mypy_cache
.ruff_cache
.DS_Store
Thumbs.db
.trash
*.tmp
*.bak
runtime.conf 中含本机绝对路径的版本
任何包含 ANYSEARCH_API_KEY 的文件
任何包含 as_sk_ 的文件
```

推荐只保留：

```text
.env.example
runtime.conf.example
README_使用说明.md
SKILL.md
scripts/
references/
workflow_manifest.json
install_workflow_skills.ps1
```

---

## 13. 共享包使用流程

别人拿到共享包后，应按以下步骤使用：

### 第一步：复制 Skill

把：

```text
codex-skills/*
```

复制到：

```text
%USERPROFILE%\.codex\skills
```

### 第二步：配置 AnySearch API Key

进入：

```text
%USERPROFILE%\.codex\skills\anysearch
```

复制：

```text
.env.example
```

为：

```text
.env
```

填写：

```text
ANYSEARCH_API_KEY=自己的key
```

### 第三步：配置 runtime.conf

根据自己的系统选择运行时：

```text
Python / Node.js / PowerShell
```

建议优先 Python。

### 第四步：重启 Codex

关闭 Codex 后重新打开，让它重新加载 Skill。

### 第五步：测试

发给 Codex：

```text
请使用 anysearch 检索 AlGaN/GaN HEMT self-heating diamond substrate，最多返回 3 条结果，并说明是否成功调用 anysearch。不要改任何文件。
```

---

## 14. 常见问题

### 14.1 PowerShell 把 API Key 当成命令怎么办

错误示例：

```powershell
PS C:\Users\llc> as_sk_xxxxx
```

这是错的。PowerShell 会把 Key 当命令执行。

正确方法：

```powershell
$ApiKey = Read-Host -Prompt "请粘贴 ANYSEARCH_API_KEY"
"ANYSEARCH_API_KEY=$ApiKey" | Set-Content -Path ".env" -Encoding UTF8
```

只有出现提示后再粘贴 Key。

### 14.2 AnySearch 不生效怎么办

检查：

```powershell
Test-Path "$env:USERPROFILE\.codex\skills\anysearch\.env"
Test-Path "$env:USERPROFILE\.codex\skills\anysearch\runtime.conf"
Test-Path "$env:USERPROFILE\.codex\skills\anysearch\SKILL.md"
```

如果不是三个 `True`，说明安装不完整。

### 14.3 Codex 没有主动调用 AnySearch 怎么办

在提示词中明确写：

```text
请使用 anysearch 检索……
```

或者：

```text
请先用 anysearch，再用 nature-academic-search 和 nature-citation……
```

### 14.4 API Key 泄露怎么办

立刻去 AnySearch 后台删除旧 Key，重新生成新 Key，然后更新本地 `.env`。

### 14.5 Word 格式任务为什么还要人工检查

Word 的页码、页眉页脚、目录跳转、图表位置、公式编号等内容存在视觉差异。AI 可以自动处理大部分格式，但最终提交前仍必须人工打开 Word 检查。

---

## 15. 推荐最终论文流程

```text
1. 整理论文初稿
2. 使用 anysearch 检索补充文献
3. 使用 nature-academic-search 筛选学术来源
4. 使用 nature-citation 判断引用支撑关系
5. 使用 nature-writing / nature-polishing 优化正文表达
6. 使用 aigc-academic-expression-optimizer 降低模板感
7. 使用 thesis-format-module-editor 套学校格式
8. 使用 multi-agent-workflow 做最终审查
9. 人工打开 Word 做视觉检查
10. 导出 PDF
```

---

## 16. 推荐最终审查清单

最终提交前检查：

- [ ] 题目、姓名、学院、专业、学号是否正确。
- [ ] 摘要和关键词是否完整。
- [ ] 英文摘要是否与中文摘要一致。
- [ ] 目录是否自动生成，是否能 Ctrl + 左键跳转。
- [ ] 一级、二级、三级标题格式是否统一。
- [ ] 页眉页脚是否符合学校要求。
- [ ] 页码是否正确。
- [ ] 图号、表号是否连续。
- [ ] 公式是否居中，编号是否右对齐。
- [ ] 参考文献格式是否统一。
- [ ] 正文中引用与参考文献列表是否对应。
- [ ] 没有把仿真写成真实实验。
- [ ] 没有编造不存在的数据图。
- [ ] 没有把蓝宝石和金刚石结论写反。
- [ ] 没有出现 API Key、账号、隐私内容。
- [ ] 已另存为最终 Word 和 PDF。
- [ ] 已人工打开 PDF 检查排版。

---

## 17. 推荐给 Codex 的总控提示词

```text
你现在是我的论文工作流 Agent。请严格按照以下规则执行：

1. 不覆盖原始文件，所有正式修改必须输出新版本。
2. 不编造数据、图表、参考文献、DOI 或结论。
3. 需要联网检索时，优先使用 anysearch。
4. 文献检索后，使用 nature-academic-search 筛选学术来源。
5. 判断文献是否支撑正文表述时，使用 nature-citation。
6. 写论文正文时，使用 nature-writing。
7. 学术润色时，使用 nature-polishing。
8. Word 格式处理时，使用 thesis-format-module-editor。
9. AIGC 表达优化时，使用 aigc-academic-expression-optimizer。
10. 多角度审查时，使用 multi-agent-workflow。
11. 如果任务涉及 Word、PDF、长日志，先说明为什么需要读取。
12. 如果用户限定“只改标题、目录、格式”，不得修改正文。
13. 不要打印、提交或分享 API Key、.env、学校账号、检测报告原文。
14. 复杂任务先给计划，再执行。
15. 最终输出要说明改了哪些文件、备份在哪里、是否需要人工检查。

当前论文主题：AlGaN/GaN HEMT 中蓝宝石与金刚石衬底对自热效应的影响。
重点方向：热效应、自热效应、散热路径、热源分布。
核心限制：只根据已有图表和仿真数据写，不写没有数据支撑的内容。
```

---

## 18. 版本维护建议

建议每次大改后创建一个版本：

```text
paper-skill-workflow-share_YYYYMMDD_HHMM.zip
```

建议保留版本记录：

```text
CHANGELOG.md
```

示例：

```markdown
# CHANGELOG

## 2026-05-27
- 新增 anysearch 联网检索集成。
- 新增 AGENTS.md 路由规则。
- nature-academic-search 加入 External search backend。
- multi-agent-workflow 加入 anysearch 路由。
- 新增 README_使用说明.md。
- 移除共享包中的 .env 和 API Key。
```

---

## 19. 最终定位

这套工作流适合：

```text
本科毕业论文
课程论文
科研论文初稿
论文格式处理
文献检索
引用判断
AIGC 表达优化
答辩 PPT
多角色审查
```

它不适合：

```text
伪造实验数据
伪造引用
规避学术诚信
替代导师最终审核
替代人工最终排版检查
处理敏感个人信息
```

最终定位：

> 这是一个论文专用 Codex 工作台，可以显著提升论文修改、查文献、套格式和审查效率，但最终提交前仍必须人工确认数据、图表、格式、引用和学校要求。

---

## 20. 参考信息

- AnySearch API Key 可选但建议配置；无 Key 可匿名使用，但额度和速率较低。
- AnySearch 运行时优先级建议为 Python > Node.js > Shell / PowerShell。
- AnySearch 的 routine 调用建议读取 `runtime.conf`，避免每次都重复运行 `doc`。
- 搜索查询、提取 URL 和 API Key 会发送到 AnySearch 服务端，不要用于敏感内容。
