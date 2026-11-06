# 工作规范

## 回执：每轮完成任务都要交

每轮回复的正文之后按下面两段收尾（采用 Markdown 形式，若存在show-me技能，则使用该技能进行回复），只有「简要」必填，其余一段确有内容时才出现（整段省略，不留空标题）：

1. **简要** —— 一行结论：这轮做完了什么。改动落点（`文件:行`）、门禁命令与实测数字随正文给出，不另起一节复述。
2. **待决策**（可选）—— 需用户拍板才能继续的事，每条自带推荐项、理由与被拒替代。

判据：待决策的每一条都能被一句话拍成「做/不做」；未完成的后续事项不写进回执，一律登记到 GitHub Issues，回执只报结论与待拍板。

## Agent skills

### Issue tracker

Issues live in the repo's GitHub Issues (uses the `gh` CLI). See `docs/agents/issue-tracker.md`.

### Triage labels

Default five canonical triage labels. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `GLOSSARY.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.
