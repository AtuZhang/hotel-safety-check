---
name: hotel-safety-check
license: MIT
github: https://github.com/AtuZhang/hotel-safety-check
description: >
  Analyzes hotel reviews for female safety risks (harassment, hidden cameras, lock issues, etc.).
  Invoke when user asks to check/evaluate hotel safety for women, solo female travelers,
  or mentions keywords like 女性安全/酒店安全/一个人住酒店安全.
metadata:
  author: "AtuZhang"
  version: "1.1.0"
---

# Hotel Safety Check

Analyzes hotel reviews from platforms like Ctrip to assess female safety risks.

## Trigger Scenarios

- User asks "这个酒店安全吗" / "女生一个人住安全吗"
- User provides a hotel URL and asks about safety
- User mentions keywords: 女性安全、独自入住、偷拍、骚扰、酒店安全
- User wants to compare multiple hotels on safety dimensions

## Prerequisites

- WebSearch / WebFetch tools available (built-in)
- Optionally: web-access skill for CDP browser mode (for sites with anti-scraping)

## Analysis Workflow

### Step 1: Gather Hotel Information

From the provided URL or hotel name, extract:
- Hotel name, address, brand, chain
- Star rating / brand tier (economy, mid-range, luxury)
- Total review count and overall rating

### Step 2: Collect Reviews

**Strategy: Multi-source, prioritize first-hand data**

| Source | Method | Priority | Notes |
|--------|--------|----------|-------|
| Ctrip hotel page | WebFetch or CDP browser | Primary | Rating, review count, visible reviews |
| Ctrip review API | curl with proper headers | Primary | `https://hotels.ctrip.com/DomesticComment/API/GetComment` |
| Xiaohongshu | CDP browser (recommended) or WebSearch | **Critical** | Real user experiences, often contains safety details that platforms filter |
| WebSearch | Search hotel name + keywords | Supplementary | Brand history, news, incidents |

#### 2.1 Xiaohongshu Search (Critical Source)

Xiaohongshu is one of the most important sources for female safety analysis. Users frequently share real hotel experiences including safety concerns that are filtered or downplayed on booking platforms.

**Method A: CDP Browser (recommended, full content access)**

Xiaohongshu heavily relies on JavaScript rendering and has strict anti-scraping. Use CDP browser mode via web-access skill:

```bash
# 1. Ensure CDP Proxy is running
node "${CLAUDE_SKILL_DIR}/../web-access/scripts/check-deps.mjs"

# 2. Open Xiaohongshu search page
curl -s --noproxy '*' -X POST --data-raw 'https://www.xiaohongshu.com/search_result?keyword=酒店名称+避雷' http://localhost:3456/new

# 3. Wait for content to load, then extract posts
curl -s --noproxy '*' -X POST "http://localhost:3456/eval?target=TAB_ID" \
  -d '(() => { const posts = []; document.querySelectorAll("section.note-item, [class*=note-item], [class*=feed-item]").forEach(el => { const title = el.querySelector("[class*=title]")?.innerText; const desc = el.querySelector("[class*=desc]")?.innerText; if (title) posts.push({title, desc: desc?.substring(0, 200) || ""}); }); return posts.slice(0, 20); })()'

# 4. Scroll to load more
curl -s --noproxy '*' "http://localhost:3456/scroll?target=TAB_ID&direction=bottom"

# 5. Click into individual posts for full content
curl -s --noproxy '*' -X POST "http://localhost:3456/click?target=TAB_ID" -d 'section.note-item:first-child a'

# 6. Close tab when done
curl -s --noproxy '*' "http://localhost:3456/close?target=TAB_ID"
```

**Method B: WebSearch (fallback, limited but still valuable)**

If CDP browser is unavailable, use targeted search queries:

```
# Search for hotel name + safety keywords on Xiaohongshu
WebSearch: "site:xiaohongshu.com 酒店名称 避雷"
WebSearch: "site:xiaohongshu.com 酒店名称 安全"
WebSearch: "site:xiaohongshu.com 酒店名称 偷拍"
WebSearch: "site:xiaohongshu.com 酒店名称 一个人住"
WebSearch: "site:xiaohongshu.com 酒店名称 女生"
```

> **Tip**: Xiaohongshu users often use informal language. Also search for: 避雷 (avoid/horror story)、踩坑 (pitfall)、劝退 (dissuade)、千万别 (never ever)、翻车 (disaster).

**Recommended search queries for Xiaohongshu:**

| Query Intent | Search Template |
|-------------|----------------|
| General safety | `{酒店名称} 避雷` or `{酒店名称} 踩坑` |
| Female-specific | `{酒店名称} 女生 一个人住` or `{酒店名称} 独自` |
| Hidden cameras | `{酒店名称} 偷拍` or `{酒店名称} 针孔` |
| Harassment | `{酒店名称} 骚扰` or `{酒店名称} 前台` |
| Room quality/safety | `{酒店名称} 隔音` or `{酒店名称} 门锁` |
| Brand-level | `{品牌名} 酒店 避雷 安全` |

#### 2.2 CDP Browser Mode (General)

For any site with anti-scraping protection:

```bash
# 1. Check proxy
node "${CLAUDE_SKILL_DIR}/../web-access/scripts/check-deps.mjs"

# 2. Open hotel page
curl -s --noproxy '*' -X POST --data-raw 'HOTEL_URL' http://localhost:3456/new

# 3. Extract visible reviews from DOM
curl -s --noproxy '*' -X POST "http://localhost:3456/eval?target=TAB_ID" \
  -d 'document.body.innerText'

# 4. Close tab when done
curl -s --noproxy '*' "http://localhost:3456/close?target=TAB_ID"
```

> **Important**: Always use `--noproxy '*'` with curl if the system has `http_proxy` set, otherwise CDP Proxy requests will fail silently.

### Step 3: Safety Keyword Scan

Scan ALL collected review text for these keyword categories:

**Direct Safety (highest priority):**
- 骚扰、跟踪、偷拍、摄像头、针孔、微型摄像头
- 门锁、门缝、猫眼、反锁、门禁
- 陌生人、闯入、半夜敲门、刷卡进入
- 不安全、害怕、恐惧、可怕

**Indirect Safety (medium priority):**
- 隔音、吵、噪音（隔音差 → 隐私暴露风险）
- 前台态度、服务态度（差服务 → 遇事无人帮助）
- 监控、安保、保安、巡逻
- 走廊、楼道、照明、暗

**Female-specific:**
- 女性、女生、一个人、独自、单身
- 独自住、一个人住、女生住

**Positive Safety Signals:**
- 门禁系统、需要刷卡、安全
- 隔音好、安静、前台好
- 保安、监控、安保人员

### Step 4: Analyze and Score

Rate each dimension on a 3-level scale:

| Dimension | Good | Caution | Risk |
|-----------|------|---------|------|
| Security facilities | 门禁+监控+保安 | 有监控但无门禁 | 无安保措施 |
| Lock & door safety | 门锁好评、可反锁 | 无相关提及 | 差评提及门锁问题 |
| Privacy (soundproofing) | 多人好评隔音好 | 无相关提及 | 差评提及隔音差 |
| Location safety | 地铁口旁、灯火通明 | 一般地段 | 偏僻、照明差 |
| Staff reliability | 24h前台、好评多 | 有一般投诉 | 差评提到态度恶劣 |
| Review sentiment | 无安全相关差评 | 少量模糊差评 | 有明确安全差评 |

### Step 5: Output Report

**Report structure (in Chinese):**

```markdown
## [酒店名称] 女性安全评估

### 结论：[安全/基本安全/需谨慎/不推荐]

### 评分概览
- 平台评分：X.X/5
- 总评论数：X,XXX
- 差评数：XX (X.X%)

### 安全关键词扫描结果
| 关键词类别 | 命中数 | 风险等级 |
|-----------|--------|---------|

### 各维度评估
[表格]

### 发现的安全相关问题
[具体差评内容和来源]

### 正面安全信号
[安保设施、隔音好评等]

### 建议
[针对女性住客的具体建议]
```

## Important Notes

- **Correlation ≠ Causation**: A lack of safety complaints does not guarantee safety. Always present findings as "no negative signals found" rather than "proven safe."
- **Brand history check**: Some chains have had incidents at specific locations. If the user provides a chain hotel, search for "[brand] 酒店 安全事件" to check for historical incidents.
- **Recency matters**: Prioritize reviews from the last 12 months. Older reviews may not reflect current conditions.
- **Platform bias**: Ctrip may filter or highlight certain reviews. Cross-reference with Xiaohongshu or Dianping when possible.
- **If CDP browser is unavailable**: Fall back to WebFetch + WebSearch. The analysis can still be comprehensive with static page content and search results.

## Dependencies

- **Required**: WebSearch, WebFetch (TRAE built-in tools)
- **Optional**: web-access skill (for CDP browser mode on anti-scraping sites)
