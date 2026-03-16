# 飞书机器人配置指南 — 亚马逊分析主龙虾 / 小龙虾

本文档详细说明如何在飞书开放平台创建机器人应用，并将其与运行在腾讯云上的主龙虾 / 小龙虾分析服务对接。

---

## 一、在飞书开放平台创建应用

1. 登录 [飞书开放平台](https://open.feishu.cn/app)，点击 **创建企业自建应用**。
2. 填写应用名称（如 `亚马逊分析助手`）和描述，上传应用图标。
3. 进入 **凭证与基础信息** 页，记录以下信息：
   - `App ID`
   - `App Secret`
4. 在 **事件订阅** 页：
   - 开启 **加密策略**，记录 `Encrypt Key`
   - 填入 **验证 Token**（`Verification Token`）
   - 填写请求地址：`https://<你的域名>/feishu/webhook`
5. 订阅以下 **事件**：
   - `im.message.receive_v1`（接收消息）
   - `im.chat.member.bot.added_v1`（机器人被拉入群）
6. 在 **权限管理** 页开启以下权限：
   - `im:message`（读取消息）
   - `im:message:send_as_bot`（发送消息）
   - `im:chat`（获取群信息）
7. 将上述凭证填入腾讯云服务器的 `.env` 文件（见下方模板）。

---

## 二、服务器端环境变量配置

在 `/home/ubuntu/amzbot/.env` 文件中填写以下内容（**禁止提交至 Git**）：

```dotenv
# 飞书应用凭证
FEISHU_APP_ID=cli_xxxxxxxxxxxxxxxx
FEISHU_APP_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
FEISHU_VERIFICATION_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
FEISHU_ENCRYPT_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# 主龙虾 / 小龙虾模型配置
TENCENT_HY_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TENCENT_HY_API_BASE=https://<tencent-hy-compatible-endpoint>/v1
HUNYUAN_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
MINIMAX_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
MINIMAX_API_BASE=https://<minimax-compatible-endpoint>/v1
KIMI_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
KIMI_API_BASE=https://<kimi-compatible-endpoint>/v1
GLM_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
GLM_API_BASE=https://<glm-compatible-endpoint>/v1

MASTER_LOBSTER_MODEL=hunyuan
PRODUCT_ANALYSIS_MODEL=tencent-hy-2.0
COMPETITOR_COMPARISON_MODEL=minimax
LISTING_OPTIMIZER_MODEL=kimi-k2.5
CATEGORY_RESEARCH_MODEL=glm-5
SCHEDULED_REPORTS_MODEL=hunyuan

# Amazon SP-API
SP_API_EU_CLIENT_ID=amzn1.application-oa2-client.eu.xxxxxxxx
SP_API_EU_CLIENT_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
SP_API_EU_REFRESH_TOKEN=Atzr|xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
SP_API_NA_CLIENT_ID=amzn1.application-oa2-client.na.xxxxxxxx
SP_API_NA_CLIENT_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
SP_API_NA_REFRESH_TOKEN=Atzr|xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Keepa（价格历史数据）
KEEPA_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# 可选数据源
HELIUM10_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
JUNGLE_SCOUT_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

> ⚠️ `.env` 文件已在 `.gitignore` 中排除，请勿手动提交。

---

## 三、飞书消息卡片模板说明

所有分析结果均使用飞书**交互式消息卡片**（Card JSON），主要模板如下：

> 说明：对外展示仍由主龙虾统一发送飞书卡片；小龙虾仅向主龙虾返回结构化结果。

### 3.1 产品分析卡片（小龙虾 1，经主龙虾统一发送）

```json
{
  "config": { "wide_screen_mode": true },
  "header": {
    "title": { "tag": "plain_text", "content": "📊 产品分析报告 · {{ASIN}}" },
    "template": "blue"
  },
  "elements": [
    {
      "tag": "div",
      "fields": [
        { "is_short": true, "text": { "tag": "lark_md", "content": "**当前 BSR**\n{{BSR_RANK}}" } },
        { "is_short": true, "text": { "tag": "lark_md", "content": "**Buy Box 价格**\n{{CURRENCY}}{{PRICE}}" } },
        { "is_short": true, "text": { "tag": "lark_md", "content": "**评论数**\n{{REVIEW_COUNT}}" } },
        { "is_short": true, "text": { "tag": "lark_md", "content": "**平均星级**\n{{STAR_RATING}} ⭐" } }
      ]
    },
    { "tag": "img", "img_key": "{{TREND_CHART_KEY}}", "alt": { "tag": "plain_text", "content": "BSR趋势图" } },
    {
      "tag": "action",
      "actions": [
        { "tag": "button", "text": { "tag": "plain_text", "content": "查看竞品对比" }, "type": "primary", "value": { "action": "cc", "asin": "{{ASIN}}" } },
        { "tag": "button", "text": { "tag": "plain_text", "content": "获取优化建议" }, "type": "default", "value": { "action": "opt", "asin": "{{ASIN}}" } }
      ]
    }
  ]
}
```

### 3.2 选品评分卡片（小龙虾 4，经主龙虾统一发送）

```json
{
  "config": { "wide_screen_mode": true },
  "header": {
    "title": { "tag": "plain_text", "content": "🔍 选品评分 · {{CATEGORY}}" },
    "template": "green"
  },
  "elements": [
    {
      "tag": "div",
      "text": { "tag": "lark_md", "content": "**综合评分：{{SCORE}}/100**\n\n{{SCORE_BREAKDOWN}}" }
    },
    {
      "tag": "div",
      "text": { "tag": "lark_md", "content": "**Top 机会产品**\n{{OPPORTUNITY_TABLE}}" }
    }
  ]
}
```

---

## 四、飞书群机器人推荐使用方式

| 使用场景 | 推荐设置 |
|---|---|
| 个人日常分析 | 与机器人单聊，直接发送指令 |
| 团队协作 | 将机器人加入群聊，支持 `@机器人 /pa B0FCD14NB7` 格式 |
| 自动定时报告 | 机器人主动向指定群或个人推送 |
| 移动端使用 | 飞书手机 App 完整支持卡片交互 |

推荐将飞书机器人只绑定到主龙虾入口服务：所有用户消息先到主龙虾，再由主龙虾按任务路由到对应小龙虾，避免小龙虾直接暴露在飞书事件入口。

---

## 五、Webhook 安全验证流程

飞书事件回调包含 `X-Lark-Signature` 请求头，服务端须验证签名：

```python
import hashlib
import hmac
import time

def verify_feishu_signature(timestamp: str, nonce: str, body: str,
                             encrypt_key: str, signature: str,
                             max_age_seconds: int = 300) -> bool:
    """验证飞书 Webhook 签名，防止伪造及重放攻击。"""
    # 时间戳有效性检查（防止重放攻击，默认允许 5 分钟偏差）
    try:
        ts = int(timestamp)
    except ValueError:
        return False
    if abs(time.time() - ts) > max_age_seconds:
        return False
    content = timestamp + nonce + encrypt_key + body
    computed = hashlib.sha256(content.encode("utf-8")).hexdigest()
    return hmac.compare_digest(computed, signature)
```

---

## 六、常见问题排查

| 问题 | 可能原因 | 解决方法 |
|---|---|---|
| 机器人无响应 | Webhook 地址未公网可达 | 检查 Nginx 配置及腾讯云安全组 80/443 端口 |
| 签名验证失败 | `encrypt_key` 填写错误 | 核对 `.env` 中 `FEISHU_ENCRYPT_KEY` |
| 消息发送失败 | 权限未开启 | 开放平台 → 权限管理 → 开启 `im:message:send_as_bot` |
| API 调用超限 | 腾讯 Coding Plan 请求频率限制 | 启用 Redis 缓存，减少重复调用；高频场景升级套餐 |
| 数据延迟 | Keepa/SP-API 配额不足 | 检查 API 配额消耗；非实时数据可降低刷新频率 |
| 服务崩溃重启 | 程序异常退出 | `systemctl status amzbot` 查看日志；已配置自动重启 |
