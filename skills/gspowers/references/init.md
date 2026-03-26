# gspowers 首次启动

state.json 不存在，执行首次启动流程。

## 1. 环境检查

用 Bash 逐一检查，缺失则提示安装并**停止**：

1. `test -d ~/.claude/skills/gstack && echo "gstack: OK" || echo "gstack: MISSING"`
   - MISSING → 告诉用户："请先安装 gstack：`npx skills add gstack -g -y`"
2. 确认你的可用 skills 列表中有 `superpowers:brainstorming`（superpowers 作为 Claude Code plugin 安装，不在 ~/.claude/skills/ 下）
   - 不可见 → 告诉用户："请先安装 superpowers plugin"
3. `test -d .git && echo "git: OK" || echo "git: MISSING"`
   - MISSING → 告诉用户："请先执行 `git init`"

全部通过后继续。

## 2. 收集项目信息

逐一询问以下问题（不要一次问完）：

**Q1**: "这是新项目还是已有项目的迭代？（新项目 / 已有项目）"

- 新项目 → `project_type = "new"`
- 已有项目 → `project_type = "existing"`
  - 告诉用户需要在项目根目录创建 `Update_Plan.md`，给出模板（见下方）
  - 自动设置 `mode = "quick"`，跳过 Q2

**Q2**（仅新项目）: "选择流程模式：完整模式（走全部步骤）还是快速模式（跳过 office-hours 和 plan-ceo-review）？"

- 完整模式 → `mode = "full"`
- 快速模式 → `mode = "quick"`

**Q3**: "这个项目有前端 UI 吗？（有 / 没有）"

- 有 → `has_ui = true`
- 没有 → `has_ui = false`

## 3. 计算起始状态

根据回答计算：

| project_type | mode | current_step | current_phase |
|-------------|------|-------------|---------------|
| new | full | office-hours | plan |
| new | quick | plan-eng-review | plan |
| existing | quick (自动) | plan-eng-review | plan |

计算 skipped_steps：
- mode = "quick" 或 project_type = "existing" → skipped_steps = ["office-hours", "plan-ceo-review"]
- mode = "full" → skipped_steps = []

## 4. 创建文件

执行以下操作：

1. 创建目录：`mkdir -p .gspowers/artifacts`
2. 确保 `.gspowers/` 在 `.gitignore` 中（state.json 包含本地路径和时间戳，不应提交）：
   用 Bash 检查并追加：`grep -q '^\.gspowers/' .gitignore 2>/dev/null || echo '.gspowers/' >> .gitignore`

2. 用 Write 工具写入 `.gspowers/state.json`：

```json
{
  "version": "1.0",
  "project_type": "{计算值}",
  "mode": "{计算值}",
  "has_ui": {计算值},
  "current_phase": "plan",
  "current_step": "{计算值}",
  "status": "in_progress",
  "failure_reason": null,
  "completed_steps": [],
  "skipped_steps": {计算值},
  "artifacts": {},
  "started_at": "{当前 ISO 8601 时间}",
  "last_updated": "{当前 ISO 8601 时间}"
}
```

3. 用 Write 工具写入 `.gspowers/handoff.md`（按 SKILL.md 中的 handoff 模板填写）

4. 显示流程地图（按 SKILL.md 中的模板，用计算出的状态标记填充）

5. 显示第一步的执行命令：
   - office-hours → 提示用户执行 `/office-hours`
   - plan-eng-review → 根据 project_type 提示：
     - new → `/plan-eng-review`（读取产品需求）
     - existing → `/plan-eng-review`（读取 Update_Plan.md）

## Update_Plan.md 模板

对 existing 项目，显示以下模板让用户在项目根目录创建：

```markdown
# 更新计划

## 问题现状
<!-- 当前遇到的具体问题，越具体越好 -->

## 期望结果
<!-- 修复/优化后应该达到的状态 -->

## 约束条件
<!-- 不能动的部分：已有依赖、兼容性要求、技术栈限制等 -->

## 影响范围
<!-- 本次更新涉及的模块/文件/功能 -->
```
