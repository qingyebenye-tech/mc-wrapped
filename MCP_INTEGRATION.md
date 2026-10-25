# MCP 接入说明

## 使用的 MCP Server

- **接入地址**：`https://mcp.mcd.cn`
- **传输协议**：Streamable HTTP
- **鉴权方式**：`Authorization: Bearer ${MCD_MCP_TOKEN}`

## 使用的 Tool

| 工具 | 用途 |
|---|---|
| `query-order-history` | 拉取全部历史订单，统计年度数据 |
| `query-my-points` | 查当前积分余额 |
| `query-my-coupons` | 查券包，找快过期的券 |
| `activity-calendar` | 查今日活动（宜吃指数用） |
| `query-nearby-stores` | 查附近门店 |
| `query-meals` | 查菜单，推荐餐品 |
| `calculate-price` | 算价格 |
| `create-order` | 用户确认后下单 |

## 调用流程

### 年度总结
```
用户："看看我的麦门年度总结"
    │
    ▼
query-order-history → query-my-points → query-my-coupons
    │
    ▼
统计：总次数/总花费/最爱餐品/最频繁月份/最长连续/最长断食/距今天数
    │
    ▼
输出年度报告 + 询问是否开启关怀提醒
```

### 日常关怀提醒（开启后）
```
用户打开对话
    │
    ▼
自动检查：距今天数 + 今日活动 + 快过期券
    │
    ▼
断食警告 + 今日宜吃指数 + 推荐餐品
    │
    ▼
用户要下单 → query-meals → calculate-price → create-order
```

## 安全说明

- Token 仅通过环境变量 `${MCD_MCP_TOKEN}` 注入
- 仓库不包含真实 Token、密钥或个人信息
