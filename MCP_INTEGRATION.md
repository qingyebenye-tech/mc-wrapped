# MCP 接入说明

## MCP Server

| 项 | 值 |
|---|---|
| 地址 | `https://mcp.mcd.cn` |
| 协议 | Streamable HTTP |
| 鉴权 | `Authorization: Bearer ${MCD_MCP_TOKEN}` |
| 限流 | 每分钟 600 次请求 |
| 官方文档 | [麦当劳 MCP Server GitHub 使用指南](https://github.com/M-China/mcd-mcp-server) |
| 开放平台 | [open.mcd.cn/mcp](https://open.mcd.cn/mcp) |

---

## 使用的官方工具

本项目实际调用以下 **8 个**麦当劳 MCP 工具：

| 工具 | 用途 | 用于哪个模块 |
|---|---|---|
| `order-list` | 拉取用户历史订单列表 | 年度总结 / 断食检查 / 宜吃指数 |
| `query-my-account` | 查当前积分余额 | 年度总结 / 麦门指数 |
| `query-my-coupons` | 查我的券包及有效期 | 年度总结 / 临期券推荐 |
| `campaign-calendar` | 查麦麦日历当日活动 | 今日宜吃指数 |
| `query-nearby-stores` | 查附近门店 | 下单引导（确定门店） |
| `query-meals` | 查菜单与在售餐品 | 下单引导（确认餐品在售） |
| `calculate-price` | 试算价格 / 优惠后金额 | 下单引导（算价） |
| `create-order` | 创建订单 | 下单引导（用户确认后） |

**读写分离原则：**

- 只读工具（`order-list`、`query-my-account`、`query-my-coupons`、`campaign-calendar`、`query-nearby-stores`、`query-meals`、`calculate-price`）可自由调用
- 写工具（`create-order`）**必须**在展示确认单并获得用户明确同意后才调用

---

## 调用流程

### 年度总结

```
用户："看看我的麦门年度总结"
  → order-list（全部订单）
  → query-my-account（积分）
  → query-my-coupons（券包）
  → 统计：次数 / 金额 / 最爱 / 月份 / 星期 / 连续 / 断食
  → 计算麦门指数（活跃30% + 消费25% + 积分25% + 券包20%）
  → 输出报告 + 询问是否开启关怀提醒
```

### 每日关怀（开启后每次对话）

```
用户打开对话
  → order-list（取最后订单 → 断食天数）
  → campaign-calendar（今日活动）
  → query-my-coupons（临期券）
  → 加权计算宜吃指数（断食30% + 活动25% + 临期券25% + 星期20%）
  → 输出简报 + 推荐餐品
  → 用户回「点」→ query-meals → calculate-price → 展示确认单
  → 用户确认 → create-order
```

---

## 业务价值

| 用户痛点 | 本项目如何用 MCP 解决 |
|---|---|
| 记不清自己一年吃了多少次、花了多少钱 | `order-list` 拉全量订单，本地聚合统计，生成年度账单 |
| 优惠券临过期才发现，白白浪费 | `query-my-coupons` 查有效期，`query-my-coupons` + `campaign-calendar` 主动提示 |
| 不知道今天值不值得吃 | `campaign-calendar` + 断食天数 + 临期券，加权成 0-100 宜吃指数 |
| 纠结吃什么 | `query-meals` + `calculate-price` 直接给出可下单的组合与实付价 |
| 下单步骤繁琐 | `query-nearby-stores` → `query-meals` → `calculate-price` → `create-order` 全链路打通 |

**核心洞察：** 麦当劳 MCP 提供的是「能力接口」，而本 Skill 提供的是「使用这些能力的行为说明书」。年度总结之所以能成立，是因为 `order-list` 返回的是**真实订单流水**——用户不需要自己记账，数据是现成的、可信的、可立即变现成一份报告的。

---

## 安全设计

- Token 仅通过环境变量 `${MCD_MCP_TOKEN}` 注入，仓库中不含任何真实 Token
- 配置文件只保留环境变量占位符，符合参赛信息安全声明要求
- 所有金额、次数、餐品信息均来自 MCP 真实返回，不虚构
- 不存储、不缓存用户订单数据
- 写操作（`create-order`）前强制用户确认

---

## 错误处理与降级

| 情况 | 处理方式 |
|---|---|
| 单个工具调用失败 | 跳过该维度，用已有数据继续，末尾注明「部分数据获取失败」 |
| `order-list` 完全失败 | 告知无法拉取订单，引导用户检查 MCP 连接与 Token |
| 菜品位售罄 | `query-meals` 返回不可售时推荐同类在售餐品 |
| 优惠不可用 | `calculate-price` 返回原价时说明「此单不适用当前优惠」 |
| 限流触发 | 退避重试一次，仍失败则降级提示稍后再试 |

---

## 复现步骤

```bash
# 1. 申请 Token：https://open.mcd.cn/mcp

# 2. 配置 MCP（WorkBuddy / Cursor / Cherry Studio / Claude Code 通用）
cp mcp-config.example.json mcp.json
# 编辑 mcp.json，把 ${MCD_MCP_TOKEN} 替换为真实 Token
# 或改用环境变量：export MCD_MCP_TOKEN=your_token

# 3. 加载 Skill
#    将 SKILL.md 全文作为系统提示词 / Skill 文件加载

# 4. 验证
#    对话输入："看看我的麦门年度总结"
```

预期：AI 调用 `order-list`、`query-my-account`、`query-my-coupons` 并返回年度账单。
