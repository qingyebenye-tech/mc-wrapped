# WorkBuddy 适配说明

## 在 WorkBuddy 中配置

1. 打开 WorkBuddy → 左侧边栏【专家·技能·连接器】→【连接器】
2. 点右上角【自定义连接器】→【配置MCP】
3. 粘贴 `mcp-config.example.json` 内容，替换 `${MCD_MCP_TOKEN}` 为你的真实 Token
4. 保存后启用 `mcd-mcp`

## 安装 Skill

把 `SKILL.md` 的内容作为自定义 Skill 的提示词：
- WorkBuddy 中新建 Skill，将 `SKILL.md` 全文粘贴
- 触发词："麦门年度总结"、"今日宜吃"、"断食提醒"

## 验证

配置完成后在对话框输入：
> "看看我的麦门年度总结"

如果 AI 调用了 `order-list`、`query-my-account` 等麦当劳 MCP 工具并返回数据，说明配置成功。
