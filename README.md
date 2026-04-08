# anygen-solution-hub — Data 层数据适配器

> 行业数据源统一访问层 — 提供标准化的数据查询接口

## 定位

anygen-solution-hub 是 AnyGen 生态的 **Data 层**，专注统一各行业原始数据源的访问接口。

**配套项目：** [anygen-solution-spec](https://github.com/AnyGenIO/anygen-solution-spec) — Connector 层工具说明书（业务操作工具）

## Data 层 vs Connector 层

```
┌─────────────────────────────────────────────────────┐
│                   AI Agent                          │
└──────────────┬────────────────────┬─────────────────┘
               │                    │
      ┌────────▼──────────┐  ┌──────▼────────────┐
      │  Connector 层      │  │  Data 层 (本项目)  │
      │ (业务操作工具)      │  │ (原始数据源)       │
      ├────────────────────┤  ├───────────────────┤
      │ Salesforce CLI     │  │ Apollo.io         │
      │ HubSpot CLI        │  │ ZoomInfo          │
      │ Shopify CLI        │  │ Clearbit          │
      │ Google Ads API     │  │ Gong/Chorus       │
      │ Stripe CLI         │  │ Clari             │
      │ DocuSign SDK       │  │ Bright Data       │
      └────────────────────┘  └───────────────────┘
           操作导向               数据查询导向
      (创建订单、发邮件)        (查联系人、爬广告)
```

**分工原则：**
- **Data 层**（本项目）— 提供行业原始数据（联系人库、通话录音、竞品广告素材）
- **Connector 层**（anygen-solution-spec）— 提供业务操作工具（CRM、支付、电子签、广告投放）

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

## 已支持数据源（8 个）

| 行业 | 数据源 | 用途 | 状态 |
|------|--------|------|------|
| **销售运营** | Apollo.io | 2.75亿 B2B 联系人和公司数据库 | ✅ Spec 完成 |
| 销售运营 | ZoomInfo | 企业级联系人数据 + 意图信号 | ✅ Spec 完成 |
| 销售运营 | Clearbit | 公司信息富化（邮箱→公司画像） | ✅ Spec 完成 |
| 销售运营 | Lusha | 联系人邮箱和电话挖掘 | ✅ Spec 完成 |
| 销售运营 | Gong | 销售通话录音 + AI 分析 | ✅ Spec 完成 |
| 销售运营 | Chorus.ai | 对话智能（ZoomInfo 旗下） | ✅ Spec 完成 |
| 销售运营 | Clari | 收入预测和管道健康度数据 | ✅ Spec 完成 |
| **广告投放** | Bright Data | 竞品广告素材和价格爬取 | ✅ Spec 完成 |

**注：** 业务操作工具（Salesforce、HubSpot、Shopify 等）见 [anygen-solution-spec](https://github.com/AnyGenIO/anygen-solution-spec)

## 项目结构

```
anygen-solution-hub/
├── README.md
├── adapters/                  # 数据源适配器 Spec（YAML）
│   ├── sales/                # 销售运营数据源
│   │   ├── apollo.yaml       # Apollo.io 联系人数据库
│   │   ├── zoominfo.yaml     # ZoomInfo 企业数据
│   │   ├── clearbit.yaml     # Clearbit 信息富化
│   │   ├── lusha.yaml        # Lusha 联系人查找
│   │   ├── gong.yaml         # Gong 通话录音
│   │   ├── chorus.yaml       # Chorus 对话智能
│   │   └── clari.yaml        # Clari 收入预测
│   └── ads/                  # 广告投放数据源
│       └── bright-data.yaml  # Bright Data 竞品监控
└── (未来：Python SDK 实现统一 API)
```

**每个 YAML 包含：**
- `name`, `id`, `description` — 数据源基本信息
- `auth` — 认证方式、环境变量、获取步骤、定价
- `endpoints` — 统一数据查询接口定义
- `data_schema` — 返回数据结构
- `rate_limits` — 速率限制
- `docs_url` — 官方文档链接

## 使用方式

### 1. 查看数据源 Spec
```bash
# 查看 Apollo.io 数据源规范
cat adapters/sales/apollo.yaml

# 查看所有销售数据源
ls adapters/sales/
```

### 2. 根据 Spec 集成（示例：Apollo.io）
```python
import requests
import os

# 从 Spec 读取配置
API_KEY = os.getenv("APOLLO_API_KEY")
BASE_URL = "https://api.apollo.io"

# 搜索联系人（参考 apollo.yaml 的 endpoints.search_people）
response = requests.post(
    f"{BASE_URL}/v1/mixed_people/search",
    headers={"X-Api-Key": API_KEY},
    json={
        "person_titles": ["CTO", "VP Engineering"],
        "q_organization_domains": "stripe.com",
        "page": 1,
        "per_page": 10
    }
)

contacts = response.json()["people"]
# 返回数据结构见 apollo.yaml 的 data_schema.person
```

## 与 anygen-solution-spec 的关系

| 项目 | 层次 | 定位 | 示例 |
|------|------|------|------|
| **anygen-solution-hub** (本项目) | Data 层 | 原始数据源适配器 | Apollo.io、ZoomInfo、Gong |
| [anygen-solution-spec](https://github.com/AnyGenIO/anygen-solution-spec) | Connector 层 | 业务操作工具说明书 | Salesforce、HubSpot、Shopify |

**典型场景：**
- 销售线索拓展 → **Data 层**：Apollo 搜索联系人 → **Connector 层**：Salesforce 创建 Lead
- 广告投放优化 → **Data 层**：Bright Data 采集竞品素材 → **Connector 层**：Google Ads 创建广告

## 贡献

1. Fork 本仓库
2. 在 `anygen_hub/adapters/` 下添加新的行业适配器
3. 编写测试
4. 提交 PR

## License

MIT

---

**AnyGen** — AI 原生的行业工具连接器
