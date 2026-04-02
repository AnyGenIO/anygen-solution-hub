# anygen-solution-hub — 统一数据 API 网关

> 连接行业工具与 AI Agent 的统一数据层

## 定位

anygen-solution-hub 是 AnyGen 生态的数据 API 网关，为 AI Agent 提供统一的行业工具调用接口。

**配套项目：** [anygen-solution-spec](https://github.com/AnyGenIO/anygen-solution-spec) — AI 行业工具说明书（工具是什么、怎么用）

```
┌──────────────────────────────────────────────────────┐
│                    AI Agent                           │
│              (Hermes / GPT / Claude)                  │
└──────────────┬───────────────────────────────────────┘
               │ 统一 API 调用
┌──────────────▼───────────────────────────────────────┐
│          anygen-solution-hub (本项目)                  │
│                                                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │ 跨境电商 │  │ 销售运营 │  │ 广告投放 │  │ 投研金融 │ │
│  │ adapter  │  │ adapter  │  │ adapter  │  │ adapter  │ │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘ │
└───────┼────────────┼────────────┼────────────┼───────┘
        │            │            │            │
   ┌────▼───┐  ┌────▼───┐  ┌────▼───┐  ┌────▼───┐
   │Shopify │  │Salesforce│ │Google  │  │OpenBB  │
   │Amazon  │  │HubSpot  │  │Ads API │  │AkShare │
   │TikTok  │  │Apollo   │  │Meta API│  │yfinance│
   └────────┘  └────────┘  └────────┘  └────────┘
```

## 核心功能

### 1. 统一认证管理
```yaml
# 一次配置，所有工具共享
credentials:
  shopify:
    access_token: ${SHOPIFY_ACCESS_TOKEN}
    store_url: ${SHOPIFY_STORE_URL}
  salesforce:
    access_token: ${SF_ACCESS_TOKEN}
    instance_url: ${SF_INSTANCE_URL}
```

### 2. 统一 API 格式
```python
from anygen_hub import SolutionHub

hub = SolutionHub()

# 跨境电商 — 查询 Shopify 订单
orders = hub.ecommerce.get_orders(platform="shopify", status="unfulfilled")

# 销售运营 — 搜索 CRM 线索
leads = hub.sales.search_leads(query="AI startup", source="apollo")

# 投研 — 获取股票数据
stock = hub.finance.get_stock("AAPL", period="1y")

# 广告 — 查看广告效果
report = hub.ads.get_campaign_report(platform="google", days=7)
```

### 3. 跨工具编排
```python
# 典型工作流：SDR 线索拓展
pipeline = hub.workflow("sdr-prospecting")
pipeline.run(
    steps=[
        ("apollo", "search_contacts", {"title": "CTO", "industry": "SaaS"}),
        ("outreach", "create_sequence", {"template": "cold-outreach-v2"}),
        ("salesforce", "create_leads", {"source": "outbound"}),
    ]
)
```

## 已支持行业

| 行业 | Adapter | 工具 | 状态 |
|------|---------|------|------|
| 跨境电商 | `hub.ecommerce` | Shopify, Amazon SP-API, TikTok Shop | 🚧 开发中 |
| 销售运营 | `hub.sales` | Salesforce, HubSpot, Apollo, Outreach | 🚧 开发中 |
| 广告投放 | `hub.ads` | Google Ads, Meta Ads, TikTok Ads | 🚧 开发中 |
| 投研金融 | `hub.finance` | OpenBB, AkShare, yfinance, Tushare | 🚧 开发中 |
| 客服运营 | `hub.support` | Zendesk, Intercom, Freshdesk | 🚧 开发中 |
| 法务助理 | `hub.legal` | 法律检索, 合同审查, DocuSign | 🚧 开发中 |
| 税务财务 | `hub.finance_tax` | 发票管理, 记账, VAT 合规 | 🚧 开发中 |

## 项目结构

```
anygen-solution-hub/
├── README.md
├── setup.py
├── anygen_hub/
│   ├── __init__.py
│   ├── hub.py                 # SolutionHub 主入口
│   ├── auth/                  # 统一认证管理
│   │   └── credential_store.py
│   ├── adapters/              # 行业适配器
│   │   ├── ecommerce/        # 跨境电商
│   │   ├── sales/            # 销售运营
│   │   ├── ads/              # 广告投放
│   │   ├── finance/          # 投研金融
│   │   ├── support/          # 客服运营
│   │   ├── legal/            # 法务助理
│   │   └── tax/              # 税务财务
│   └── workflow/              # 跨工具编排引擎
│       └── engine.py
├── config/
│   └── credentials.yaml.example
└── tests/
```

## 快速开始

```bash
# 安装
pip install anygen-hub

# 配置认证
cp config/credentials.yaml.example config/credentials.yaml
# 编辑填入你的 API key

# 使用
python -c "from anygen_hub import SolutionHub; hub = SolutionHub(); print(hub.status())"
```

## 与 anygen-solution-spec 的关系

| 项目 | 角色 | 类比 |
|------|------|------|
| [anygen-solution-spec](https://github.com/AnyGenIO/anygen-solution-spec) | 说明书 — 告诉 AI 有什么工具、怎么用 | API 文档 |
| **anygen-solution-hub** (本项目) | 网关 — 统一调用接口，实际连接工具 | API Gateway |

## 贡献

1. Fork 本仓库
2. 在 `anygen_hub/adapters/` 下添加新的行业适配器
3. 编写测试
4. 提交 PR

## License

MIT

---

**AnyGen** — AI 原生的行业工具连接器
