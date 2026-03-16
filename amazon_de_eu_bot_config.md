# 亚马逊德国/欧洲站商品分析机器人 — 小龙虾多功能配置方案

> **运行环境**：腾讯云 4核 8G 120GB SSD · 1500G流量包 · 10M带宽 · Linux  
> **模型套餐**：腾讯云 Coding Plan  
> **通道**：飞书（Feishu / Lark）  
> **目标站点**：Amazon.de（DE）及欧洲站（UK、FR、NL、SE、PL、ES、TR、BE、IE）

---

## 一、总体架构

```
飞书用户
   │
   ▼
飞书机器人（Webhook 接收 / 主动推送）
   │
   ▼
小龙虾调度中心（腾讯云 Linux 主机）
   ├── Agent 1：当前产品数据分析
   ├── Agent 2：同类竞品对比
   ├── Agent 3：Listing 优化建议
   ├── Agent 4：类目市场研究 & 选品评分
   └── Agent 5：定时报告 & 预警
```

所有 Agent 共用同一套数据采集层（Amazon DE/EU API + Keepa + Jungle Scout 等），
结果通过飞书卡片消息或富文本推送给用户。

---

## 二、各 Agent 功能定义

### Agent 1 — 当前产品数据分析（`product_analysis`）

**触发方式**：飞书指令 `/分析产品 <ASIN>` 或 `/pa <ASIN>`

**分析维度**

| 维度 | 说明 |
|---|---|
| BSR 排名趋势 | 近 30 / 90 天大类、小类排名曲线 |
| 价格历史 | 近 90 天最高 / 最低 / 均价；当前 Buy Box 价格 |
| 评分 & 评论 | 星级均值、评论数量趋势、最近 20 条评论情感分析 |
| 库存状态 | 当前库存深度（FBA / FBM）；缺货记录 |
| 流量指标 | 搜索词排名（集成 Helium10 / Jungle Scout 接口） |
| A+ 内容质量 | 图片数量、视频、EBC 模块完整度 |
| 广告可见性 | Sponsored 曝光频次（需授权数据） |

**输出**：飞书交互式卡片，含趋势折线图 + 文字摘要 + 优先行动建议。

---

### Agent 2 — 同类竞品对比（`competitor_comparison`）

**触发方式**：飞书指令 `/对比 <ASIN> [竞品ASIN1,竞品ASIN2,...]` 或 `/cc <ASIN>`（自动抓取 Top10 竞品）

**对比项目**

| 项目 | 说明 |
|---|---|
| 价格区间 | 各竞品价格分布，我方产品定价位置 |
| BSR 对比 | 同一类目下排名排列 |
| 关键词重叠度 | 共同关键词 vs. 差异关键词 |
| 主图/副图数量 | 图片质量评分（AI 视觉评估） |
| 评论数 & 星级 | 竞品用户口碑矩阵 |
| Listing 完整度 | 标题长度、Bullet 字数、A+ 有无 |
| FBA/FBM 比例 | 竞品物流模式分布 |
| 促销策略 | Coupon、Lightning Deal 频率 |

**输出**：飞书多维表格卡片 + 雷达图（我方 vs. 竞品均值）+ 竞争优/劣势总结。

---

### Agent 3 — Listing 优化建议（`listing_optimizer`）

**触发方式**：飞书指令 `/优化 <ASIN> [语言代码]` 或 `/opt <ASIN> DE`

**支持语言**：de-DE、en-GB、fr-FR、nl-NL、sv-SE、pl-PL、es-ES、tr-TR、fr-BE、nl-BE、en-IE

**优化模块**

| 模块 | 优化内容 |
|---|---|
| 标题优化 | 关键词密度、字符数合规（≤200）、卖点前置 |
| Bullet 优化 | Emoji 使用规范、合规提示（欧盟 GPSR / REACH）、字数均衡 |
| 描述 / A+ | 叙事结构、差异化卖点、转化引导语 |
| 搜索关键词 | 去重、高搜索量词补充、禁用词过滤 |
| 图片建议 | 主图背景合规、场景图种类、尺寸及压缩比 |
| 后端关键词 | 填充建议，避免重复标题已出现词 |

**当前产品参考**（ASIN B0FCD14NB7）：本仓库 `localization_content.md` 已包含 DE / EU 各语言 Listing，优化建议将基于此文件内容进行差量对比。

**输出**：飞书富文本消息，逐字段给出 "原文 → 优化建议 → 原因" 三栏对照表。

---

### Agent 4 — 类目市场研究 & 选品评分（`category_research`）

**触发方式**：飞书指令 `/选品 <类目关键词> [站点]` 或 `/cr Serviertablett DE`

**分析流程**

1. **类目概览**：类目规模（月销量估算）、竞争强度（Review 中位数、头部集中度）
2. **价格带分布**：各价格段产品数量、销量占比、建议切入区间
3. **需求验证**：关键词月搜索量、搜索趋势（季节性分析）、买家痛点词云
4. **竞争壁垒评估**：专利风险、品牌集中度、最低 Review 门槛
5. **选品评分模型**（0–100 分）

| 评分维度 | 权重 |
|---|---|
| 市场规模 | 20% |
| 竞争烈度（反向） | 25% |
| 利润空间（价格 - 成本估算） | 20% |
| 增长趋势 | 15% |
| 物流可行性（尺寸 / 重量） | 10% |
| 合规风险（欧盟 CE / GPSR） | 10% |

6. **Top 机会产品清单**：列出评分 ≥ 60 的产品，附 ASIN、当前排名、月销量估算、建议切入价格

**输出**：飞书交互卡片，含评分雷达图 + 机会产品表格 + 类目进入建议报告。

---

### Agent 5 — 定时报告 & 预警（`scheduled_reports`）

**触发方式**：定时任务（Cron）+ 飞书主动推送；可在飞书订阅 `/订阅日报 <ASIN>` 或 `/订阅周报 <ASIN>`

| 报告类型 | 频率 | 内容 |
|---|---|---|
| 日报 | 每天 08:00 CET | BSR 变化、价格变化、新增评论数 |
| 周报 | 每周一 09:00 CET | 7天销量趋势、关键词排名变化、库存预警 |
| 竞品预警 | 实时 | 竞品大幅降价（> 15%）、新竞品进入 Top 10、竞品库存清空 |
| 合规预警 | 每月 1 日 | 欧盟法规更新（GPSR、REACH、电池法）对产品的影响评估 |

---

## 三、腾讯云主机部署配置

### 3.1 系统依赖

```bash
# 更新系统（建议先在测试环境验证，再在生产环境执行）
sudo apt update && sudo apt upgrade

# 安装 Python 运行环境
sudo apt install -y python3.11 python3.11-venv python3-pip redis-server nginx git

# 安装 Node.js（飞书 SDK）
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt install -y nodejs

# 安装 Docker（可选，用于容器化部署）
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

### 3.2 项目目录结构

```
/home/ubuntu/amzbot/
├── agents/
│   ├── product_analysis.py        # Agent 1
│   ├── competitor_comparison.py   # Agent 2
│   ├── listing_optimizer.py       # Agent 3
│   ├── category_research.py       # Agent 4
│   └── scheduled_reports.py       # Agent 5
├── data/
│   ├── listings/                  # 本地 Listing 缓存（含 localization_content.md）
│   └── cache/                     # Redis 缓存持久化目录
├── config/
│   ├── settings.yaml              # 主配置文件（见 3.3 节）
│   └── feishu_credentials.yaml   # 飞书应用凭证（勿提交至 Git）
├── dispatcher.py                  # 飞书消息路由分发器
├── scheduler.py                   # 定时任务调度器（APScheduler）
└── requirements.txt
```

### 3.3 主配置文件 `config/settings.yaml`

```yaml
server:
  host: "0.0.0.0"
  port: 8080
  workers: 2                       # 4核机器建议 2–4 worker

feishu:
  app_id: "${FEISHU_APP_ID}"
  app_secret: "${FEISHU_APP_SECRET}"
  verification_token: "${FEISHU_VERIFICATION_TOKEN}"
  encrypt_key: "${FEISHU_ENCRYPT_KEY}"
  webhook_url: "https://<你的域名或IP>/feishu/webhook"

tencent_coding:
  api_key: "${TENCENT_CODING_API_KEY}"
  # OpenAI 兼容接口地址：https://api.hunyuan.cloud.tencent.com/v1
  # 模型标识符以腾讯云 Coding Plan 控制台显示为准，示例值：hunyuan-pro / hunyuan-standard
  model: "hunyuan-pro"
  max_tokens: 4096
  temperature: 0.3                 # 分析类任务建议低温度

amazon:
  marketplaces:
    - id: "A1PA6795UKMFR9"        # Amazon.de
      locale: "de-DE"
    - id: "A1F83G8C2ARO7P"        # Amazon.co.uk
      locale: "en-GB"
    - id: "A13V1IB3VIYZZH"        # Amazon.fr
      locale: "fr-FR"
    - id: "A1805IZSGTT6HS"        # Amazon.nl
      locale: "nl-NL"
    - id: "A2NODRKZP88ZB9"        # Amazon.se
      locale: "sv-SE"
    - id: "A1C3SOZRARQ6R3"        # Amazon.pl
      locale: "pl-PL"
    - id: "A1RKKUPIHCS9HS"        # Amazon.es
      locale: "es-ES"
  sp_api:
    client_id: "${SP_API_CLIENT_ID}"
    client_secret: "${SP_API_CLIENT_SECRET}"
    refresh_token: "${SP_API_REFRESH_TOKEN}"
    region: "eu-west-1"

data_sources:
  keepa_api_key: "${KEEPA_API_KEY}"
  jungle_scout_api_key: "${JUNGLE_SCOUT_API_KEY}"   # 可选
  helium10_api_key: "${HELIUM10_API_KEY}"             # 可选

redis:
  host: "127.0.0.1"
  port: 6379
  db: 0
  cache_ttl_seconds: 3600          # 数据缓存 1 小时

scheduler:
  # cron 表达式使用本地时区（Europe/Berlin，CET/CEST）
  # CET（UTC+1）：夏令时期间自动切换为 CEST（UTC+2）
  daily_report_cron: "0 8 * * *"   # CET 08:00（含夏令时自动调整）
  weekly_report_cron: "0 9 * * 1"  # CET 09:00 每周一（含夏令时自动调整）
  timezone: "Europe/Berlin"

logging:
  level: "INFO"
  file: "/var/log/amzbot/amzbot.log"
  max_bytes: 10485760              # 10 MB
  backup_count: 5
```

### 3.4 依赖清单 `requirements.txt`

> **提示**：以下版本号为最低兼容版本。生产部署时建议先执行 `pip install <package>` 获取实际安装版本，
> 再用 `pip freeze > requirements.lock` 生成精确锁定文件，以确保构建可复现。

```
# 飞书 / 消息通道
lark-oapi>=1.3.0

# Amazon SP-API
python-amazon-sp-api>=1.0.0

# Keepa
keepa>=1.7.0

# AI / LLM（兼容腾讯混元 OpenAI 兼容接口）
openai>=1.0.0

# 调度
APScheduler>=3.10.0

# 数据处理
pandas>=2.1.0
matplotlib>=3.8.0
Pillow>=10.0.0

# 缓存
redis>=5.0.0

# HTTP
httpx>=0.26.0
tenacity>=8.2.0                    # 自动重试

# 配置
PyYAML>=6.0.1
python-dotenv>=1.0.0
```

### 3.5 Nginx 反向代理配置

```nginx
# /etc/nginx/sites-available/amzbot
server {
    listen 80;
    server_name <你的域名或公网IP>;

    # 飞书 Webhook 入口
    location /feishu/webhook {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 30s;
    }

    # 健康检查
    location /health {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

### 3.6 Systemd 服务配置

```ini
# /etc/systemd/system/amzbot.service
[Unit]
Description=Amazon DE/EU Analysis Bot
After=network.target redis.service

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/amzbot
EnvironmentFile=/home/ubuntu/amzbot/.env
ExecStart=/home/ubuntu/amzbot/venv/bin/python dispatcher.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

启动命令：
```bash
sudo systemctl daemon-reload
sudo systemctl enable amzbot
sudo systemctl start amzbot
sudo systemctl status amzbot
```

---

## 四、飞书机器人配置步骤

详见 [`feishu_bot_setup.md`](./feishu_bot_setup.md)。

---

## 五、指令速查表

| 指令 | Agent | 说明 |
|---|---|---|
| `/pa <ASIN>` | Agent 1 | 分析指定产品数据 |
| `/分析产品 <ASIN>` | Agent 1 | 同上（中文全称） |
| `/cc <ASIN>` | Agent 2 | 自动抓取 Top10 竞品对比 |
| `/对比 <ASIN> <竞品1,...>` | Agent 2 | 手动指定竞品对比 |
| `/opt <ASIN> <语言>` | Agent 3 | Listing 优化建议 |
| `/优化 <ASIN>` | Agent 3 | 同上（中文全称） |
| `/cr <类目词> [站点]` | Agent 4 | 类目研究 & 选品评分 |
| `/选品 <类目词>` | Agent 4 | 同上（中文全称） |
| `/订阅日报 <ASIN>` | Agent 5 | 订阅每日产品报告 |
| `/订阅周报 <ASIN>` | Agent 5 | 订阅每周产品报告 |
| `/帮助` | — | 显示所有指令说明 |

---

## 六、资源占用估算

| 组件 | CPU | 内存 | 备注 |
|---|---|---|---|
| 飞书消息分发器 | 0.5 核 | 256 MB | 常驻进程 |
| Agent 1–4（按需） | 1–2 核 | 512 MB | 任务队列，并发 ≤ 2 |
| 定时报告调度器 | 0.1 核 | 128 MB | 常驻轻量进程 |
| Redis 缓存 | 0.1 核 | 256 MB | 数据缓存 |
| Nginx | 0.1 核 | 64 MB | 反向代理 |
| **合计** | **~4 核** | **~1.2 GB** | 4核8G 机器完全胜任 |

---

## 七、安全与合规注意事项

1. **凭证管理**：所有 API Key 及密钥通过 `.env` 文件注入，禁止硬编码，禁止提交至 Git。
2. **飞书加密**：启用飞书事件订阅加密（`encrypt_key`），防止 Webhook 被伪造。
3. **Amazon ToS**：数据采集严格使用官方 SP-API，禁止爬虫抓取 Amazon 页面，避免账户违规。
4. **欧盟 GDPR**：不收集、不存储买家个人信息；分析数据仅使用聚合统计口径。
5. **数据备份**：每日凌晨 02:00 自动备份 Redis 数据到腾讯云 COS 对象存储。
6. **访问控制**：飞书机器人仅对指定企业内部成员开放；服务器防火墙仅开放 80 / 443 / 22 端口。
