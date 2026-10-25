# MCP 接入说明

## MCP Server

- 地址：`https://mcp.mcd.cn`
- 协议：Streamable HTTP
- 鉴权：`Authorization: Bearer ${MCD_MCP_TOKEN}`
- 限流：每分钟 600 次请求

## 使用的官方工具

| 工具 | 用途 |
|---|---|
| `order-list` | 拉取历史订单，统计年度数据 |
| `query-my-account` | 查当前积分余额 |
| `query-my-coupons` | 查券包，找临期券 |
| `campaign-calendar` | 查今日活动 |
| `query-nearby-stores` | 查附近门店 |
| `query-meals` | 查菜单，推荐餐品 |
| `calculate-price` | 算价格 |
| `create-order` | 创建订单 |

## 调用流程

### 年度总结
```
用户："看看我的麦门年度总结"
  → order-list（全部订单）
  → query-my-account（积分）
  → query-my-coupons（券包）
  → 统计：次数/金额/最爱/月份/断食
  → 输出报告 + 询问开启提醒
```

### 关怀提醒（开启后）
```
用户打开对话
  → order-list（距今天数）
  → campaign-calendar（今日活动）
  → query-my-coupons（临期券）
  → 算宜吃指数 + 推荐餐品
  → 用户要下单 → query-meals → calculate-price → create-order
```

## 安全

- Token 仅通过环境变量 `${MCD_MCP_TOKEN}` 注入
- 仓库不包含真实 Token
- 所有金额来自 MCP 真实返回，不虚构
