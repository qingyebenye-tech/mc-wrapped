# WorkBuddy 使用与开发上下文

本项目使用**腾讯 WorkBuddy** 开发，参与麦当劳 1024 程序员节创意开发大赛 WorkBuddy 专项奖励。

---

## 在 WorkBuddy 中配置麦当劳 MCP

### 方式一：界面配置（推荐）

1. 打开 WorkBuddy → 左侧边栏【专家·技能·连接器】→ 切到【连接器】页签
2. 点击右上角【自定义连接器】→【配置MCP】
3. 在手动配置页面粘贴以下 JSON：

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer YOUR_MCP_TOKEN"
      }
    }
  }
}
```

4. ⚠️ **把 `YOUR_MCP_TOKEN` 替换为你在 [open.mcd.cn/mcp](https://open.mcd.cn/mcp) 申请到的真实 Token**
5. 点击【保存】
6. 回到【自定义连接器】，将 `mcd-mcp` **启用**

### 方式二：配置文件

写入 `~/.workbuddy/mcp.json`：

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer YOUR_MCP_TOKEN"
      }
    }
  }
}
```

> 若该文件已有其他服务器，**只往 `mcpServers` 里合并 `mcd-mcp` 这一个 key**，不要整文件覆盖。

---

## 安装 Skill

1. 在 WorkBuddy 中新建自定义 Skill
2. 将 [`SKILL.md`](SKILL.md) 全文粘贴为 Skill 提示词
3. 建议触发的关键词：
   - 麦门年度总结
   - 年度账单
   - 今日宜吃
   - 断食提醒
   - 麦门指数

---

## 验证

在对话框输入：

```
看看我的麦门年度总结
```

**验证成功的标志：** AI 实际调用了 `order-list`、`query-my-account`、`query-my-coupons` 等麦当劳 MCP 工具，并基于返回的真实数据输出了年度账单（含次数、金额、最爱餐品、断食天数、麦门指数分档）。

若提示工具不可用，排查顺序：

| 检查项 | 说明 |
|---|---|
| MCP 是否启用 | 连接器列表里 `mcd-mcp` 需处于「已启用」 |
| Token 是否正确 | 复制时是否带空格 / 换行 |
| 网络是否可达 | 需能访问 `https://mcp.mcd.cn` |
| 工具名是否匹配 | 确认为 `order-list` 而非 `query-order-history` |

---

## 开发过程对话上下文

本项目由 WorkBuddy 协助完成，主要工作流：

| 阶段 | WorkBuddy 承担的工作 |
|---|---|
| 规则与能力盘点 | 连接 `mcd-mcp` 后只读调用各接口，摸清 33 个工具的真实名称、参数与返回结构，避免使用臆造的工具名 |
| Skill 设计与撰写 | 把「年度总结 + 每日关怀 + 宜吃指数」拆成明确工作流，写入 `SKILL.md` |
| 文档与物料生成 | 生成 `README.md`、`MCP_INTEGRATION.md`，以及 `images/banner.svg` 主视觉（矢量绘制，无字体/emoji 依赖） |
| 参赛材料核对 | 逐字比对 `CONTEST_DECLARATION.md` 与官方原文，确保一字不差 |
| 提交与合规 | 生成脱敏配置模板 `mcp-config.example.json`，确认仓库无真实 Token |

> ⚠️ 重要经验：初期曾在 `SKILL.md` 中使用臆造的工具名（如 `query-order-history`、`activity-calendar`），实际接入后才发现官方工具名为 `order-list`、`campaign-calendar`，已全部更正。**接入 MCP 后第一步应当只读探测真实工具清单。**

---

## 参赛合规自查

- [x] 仓库公开可访问
- [x] 包含 `README.md`（介绍、安装、使用示例、目标用户）
- [x] 包含 `MCP_INTEGRATION.md`（Server / Tool / 调用流程 / 业务价值）
- [x] 包含 `CONTEST_DECLARATION.md`（官方原文，一字未改）
- [x] 包含 `mcp-config.example.json`（仅环境变量占位符）
- [x] 包含 `workbuddy.md`（本文件）
- [x] 包含项目主体内容（`SKILL.md`）
- [x] 真实调用麦当劳 MCP 能力
- [x] 仓库内无真实 Token、密钥、个人信息
