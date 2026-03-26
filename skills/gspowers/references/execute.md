# 执行期指令

当前阶段：`current_phase = "execute"`。根据 state.json 中的 `current_step` 执行对应指令。

---

## brainstorming

提示用户执行：

```
/brainstorm

请读取 .gspowers/artifacts/architecture.md 作为架构输入。
```

> brainstorming 完成后会自动衔接 writing-plans，无需返回调度器。
> writing-plans 完成后可直接选择子代理执行。
> 所以用户可能一次走完 brainstorming + writing-plans + subagent-dev。

当用户返回 `/gspowers` 时，逐步确认完成情况：

1. 询问："brainstorming 和 writing-plans 都完成了吗？还是只完成了 brainstorming？"

2. **只完成 brainstorming**（有 design-spec，无 implementation-plan）：
   - 收集 design-spec 文件路径，复制：`cp <路径> .gspowers/artifacts/design-spec.md`
   - 将 `"brainstorming"` 加入 `completed_steps`
   - 设 `current_step` 为 `"writing-plans"`
   - 更新 state.json + 重写 handoff.md
   - 继续到下方 writing-plans 指令

3. **brainstorming + writing-plans 都完成**（两个 artifact 都有）：
   - 收集两个文件路径，分别复制到：
     - `.gspowers/artifacts/design-spec.md`
     - `.gspowers/artifacts/implementation-plan.md`
   - 将 `"brainstorming"` 和 `"writing-plans"` 加入 `completed_steps`
   - 设 `current_step` 为 `"subagent-dev"`
   - 更新 state.json + 重写 handoff.md
   - 继续到下方 subagent-dev 指令

4. **全部完成**（包括 subagent-dev）：
   - 收集所有 artifact
   - 将 `"brainstorming"`、`"writing-plans"`、`"subagent-dev"` 加入 `completed_steps`
   - 设 `current_phase` 为 `"finish"`，`current_step` 为 `"review"`
   - 更新 state.json + 重写 handoff.md
   - 显示 /clear 提示（见下方 subagent-dev 完成后的提示）

---

## writing-plans

> 这个步骤仅在 brainstorming 完成后中断恢复时出现。

显示："上次 brainstorming 已完成但 writing-plans 未开始。"

提示用户执行：

```
/superpowers:writing-plans

请读取 .gspowers/artifacts/design-spec.md
```

用户完成后：
1. 收集 implementation-plan 文件路径
2. 复制：`cp <路径> .gspowers/artifacts/implementation-plan.md`
3. 将 `"writing-plans"` 加入 `completed_steps`
4. 设 `current_step` 为 `"subagent-dev"`
5. 更新 state.json + 重写 handoff.md

---

## subagent-dev

提示用户执行：

```
/superpowers:subagent-driven-development

请读取 .gspowers/artifacts/implementation-plan.md
```

用户完成后：
1. 将 `"subagent-dev"` 加入 `completed_steps`
2. 设 `current_phase` 为 `"finish"`
3. 设 `current_step` 为 `"review"`
4. 更新 state.json + 重写 handoff.md
5. 显示：

```
⚠️ 下一步进入收尾期（gstack），需要清理上下文。
请执行 /clear，然后输入 /gspowers
```
