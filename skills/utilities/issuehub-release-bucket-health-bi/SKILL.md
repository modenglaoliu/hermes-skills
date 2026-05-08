---
name: issuehub-release-bucket-health-bi
description: 固定 HLB IssueHub 正式 BI V01 的生成口径、Owner与闭环分析结构、版本桶 626/725/828、Jira关闭/预计关闭双口径、静态趋势图和未关闭 Issue Classification 堆积图。
trigger: IssueHub 正式BI、HLB IssueHub BI、issuehub db、Owner与闭环、预计总体关闭、Ready To Test in UAT、Issue Classification 堆积、版本桶626/725/828、Jira关闭率
---

# HLB IssueHub 正式 BI V01 固定技能

## 必须遵守的总原则

用户已固定认可 IssueHub BI V01 的结构、样式和分析口径。以后生成 HLB IssueHub BI 时：

- 必须锚定官方 V01 结构：
  `/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_20260506_V01.html`
- 每日输出文件使用：
  - HTML：`/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_YYYYMMDD_V01.html`
  - JSON：`/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_YYYYMMDD_V01_data.json`
- 禁止生成或改名为 NextGen、作战日报、科技行长摘要、高级驾驶舱、Audited 等变体。
- 除非用户明确要求改版，否则不要重做布局、换风格、换结构；只更新数据、口径和必要局部细节。
- 风格使用银行/管理层风格：深蓝、银灰、金、深红；不要 Hermes 橙黑主题。
- 生成前后必须自检，不能把静态渲染失败的版本交给用户。

## 标准生成流程

当用户给出某个 IssueHub DB，例如：
`issuehub-2026-05-08T13-25-41-987Z.db`

执行：

```bash
cd /Users/zhangliang/hlb/04_Jira_jira/issuehub
ISSUEHUB_DB=/Users/zhangliang/hlb/04_Jira_jira/issuehub/daily_db/issuehub-YYYY-MM-DDTHH-MM-SS-xxxZ.db \
ISSUEHUB_BI_DATE=YYYYMMDD \
/usr/bin/python3 build_issuehub_bi_v01_today.py

ISSUEHUB_HTML=/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_YYYYMMDD_V01.html \
ISSUEHUB_JSON=/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_YYYYMMDD_V01_data.json \
ISSUEHUB_SNAPSHOT_LABEL='YYYY-MM-DD HH:MM' \
/usr/bin/python3 postprocess_owner_panel_v07_expected.py
```

已验证脚本：

- 主构建：`build_issuehub_bi_v01_today.py`
- 主口径实现：`build_issuehub_bi_v01_merged.py`
- Owner 静态后处理：`postprocess_owner_panel_v07_expected.py`

如果脚本还没有同步最新口径，先按本技能修复脚本，再生成。

## DB 与历史趋势规则

- 当前 DB 必须来自用户指定的最新切片，不要擅自改用旧 DB。
- daily_db 根目录按文件名排序保存每日最后切片；趋势图每个自然日只取最后一个快照。
- 趋势标签使用业务时间，例如 `2026-05-08 21:10`，不是文件下载 UTC 原文。
- Owner 与闭环顶部趋势图必须是静态 SVG，不依赖 JS/canvas。
- 每条趋势线必须至少 2 个点；如果只有单点，要画成短横线，避免静态渲染失败。

## Jira 关闭口径：固定

Jira 已关闭必须包括：

- `Closed`
- `Cancelled`
- `Canceled`
- `Resolved`

Jira 未关闭包括：

- `New`
- `In Progress`
- `Ready To Test in UAT`
- 其他非 Closed / Cancelled / Resolved 状态

实现建议：

```python
def norm_status(s):
    raw = (s or '').strip().strip('"')
    low = raw.lower().replace(' ', '')
    if low in ('closed', 'cancelled', 'canceled', 'resolved'):
        return 'Closed/Cancelled/Resolved'
    if low == 'readytotestinuat':
        return 'Ready To Test in UAT'
    if low == 'inprogress':
        return 'In Progress'
    if low == 'new':
        return 'New'
    return raw or 'Unknown'

def is_closed_status(s):
    return norm_status(s) == 'Closed/Cancelled/Resolved'
```

KPI 中：

- `closed_cancelled` 实际含义为 Jira Closed / Cancelled / Resolved
- `close_rate = closed_cancelled / total`
- `open = total - closed_cancelled`

## 预计总体关闭口径：固定

Owner 与闭环顶部“总体 / 版本桶关闭率趋势”中，必须在 Jira 关闭率之外增加一条新线：

- 展示名：`预计总体关闭`
- 建议颜色：绿色 `#1f6b45`
- 建议线型：虚线 `stroke-dasharray="6,4"`

预计总体关闭包括：

1. Jira 状态已关闭：
   - Closed
   - Cancelled / Canceled
   - Resolved
2. `Ready To Test in UAT`
   - 我方已部署到 HLB UAT，待客户验证，只是 Jira 未翻转。
3. `Issue Classification == 客户解释`
4. 评论、描述或外部评论中可判定为客户侧/解释类/暂不处理的问题，关键字包括：
   - 客户解释
   - 客户操作不当
   - 操作不当
   - 设计如此
   - 暂时不改
   - 不改
   - 已部署
   - 部署到HLB UAT
   - 已发版
   - 待客户验证
   - ready to test

实现建议：

```python
EXPECTED_CLOSE_COMMENT_RE = re.compile(
    r'客户解释|客户操作不当|操作不当|设计如此|暂时不改|不改|已部署|部署到HLB UAT|已发版|待客户验证|ready to test',
    re.I,
)

def is_expected_closed_status(s):
    return norm_status(s) in ('Closed/Cancelled/Resolved', 'Ready To Test in UAT')

def comment_expected_close_text(*parts):
    text = ' '.join(str(x or '') for x in parts)
    return bool(EXPECTED_CLOSE_COMMENT_RE.search(text))

expected_closed = bool(
    is_expected_closed_status(raw_status)
    or classification == '客户解释'
    or comment_expected_close_text(summary, description, external_description, external_comments, issuehub_comment_text)
)
```

KPI 必须输出：

- `expected_closed`
- `expected_close_rate = expected_closed / total`

趋势图上必须显示：

- Jira关闭率线：百分比 + `closed/total`
- 预计总体关闭线：百分比 + `expected_closed/total`
- 626/725/828 版本桶线：百分比 + `closed/total`

## 版本桶口径：固定

版本桶必须使用 `Golive Date` 字段，不再用 `Client ETA` 反推：

- `Golive Date = 2026-06-26` → 展示名统一为 `626`
- `Golive Date = 2026-07-25` → 展示名统一为 `725`
- `Golive Date = 2026-08-28` → 展示名统一为 `828`
- Golive Date 未填或不在三类日期内 → `未标记`

注意：

- `Client ETA` 仅作为每日承诺/SLA 和超期风险使用。
- 页面中禁止再出现 `6月底 / 7月底 / 8月底 / 520=7月25版本 / 626=8月底版本` 等旧标签。
- 页面展示只用 `626 / 725 / 828 / 未标记`。

## Issue Classification 堆积图口径：固定

Owner 展开详情里的 Issue Classification 堆积图只看未关闭 Issue。

固定标题：

- `Issue Classification 堆积（未关闭）`

计算逻辑：

```python
open_arr = [x for x in owner_or_group_issues if x['open']]
cnt = Counter(norm_class(x.get('classification')) for x in open_arr)
```

禁止用全部 Issue 做堆积图；否则会误导 Owner 当前未关闭风险结构。

固定分类顺序：

1. 程序缺陷
2. 未开发完成
3. 需求变更
4. 优化提升
5. 需求未对齐
6. 客户解释
7. 环境问题
8. 未分类

## Owner 与闭环 tab 固定结构

Owner 与闭环 tab 只保留以下结构：

1. `总体 / 版本桶关闭率趋势`
   - 静态 SVG
   - 5 条线：
     - Jira关闭
     - 预计总体关闭
     - 626
     - 725
     - 828
   - 上方显示：
     - 今日Jira关闭
     - 今日新增
     - 预计总体关闭
   - 点位显示百分比和数字，如 `54.7%` + `434/793`、`69.4%` + `550/793`

2. `版本桶未关闭事实`
   - 只展示事实指标：
     - 版本桶
     - 未关闭
     - S/H
     - S
     - H
     - 缺 Tester
   - 禁止健康分、Critical/Watch/At Risk/Healthy 等等级。

3. `Issue Classification 图例`
   - 说明堆积图只统计未关闭 Issue。

4. `模块组关闭率排名`
   - 按主模块组聚合。
   - 重复标签人员不重复计数。
   - 展示总数、Jira已关闭、Jira未关闭、关闭率、626/725/828关闭率。

5. `Owner 关闭率追踪`
   - 只保留“按模块分组清单”。
   - 使用原生 `<details>` 展开/收起，不依赖 JS。
   - 每个 Owner 展开后展示：
     - 当前关闭率
     - 总数
     - Jira已关闭
     - Jira未关闭
     - S/H open
     - ETA 超期
     - 客户解释类 open
     - 优化 open
     - Owner 关闭率趋势
     - Issue Classification 堆积（未关闭）
     - 未关闭负载
     - Issue 前后对比

禁止在 Owner tab 中出现：

- 单独“按关闭率排名追踪”全员排名区块
- Owner 单人趋势明细旧交互区
- Owner 未关闭负载旧区块
- S/H 缺 Tester 旧区块
- 未关闭 S/H Top 旧区块
- issue 明细清单
- 独立“版本健康度”tab
- 健康分 / score / Critical / Watch / At Risk / Healthy

## Owner 模块映射

固定主责归组：

- AIOCR：刘福龙、王宇佳、谢守坦、饶毅、危俊、周宽
- ITPM组：李科、杨志鹏、何聪、王朝阳
- CRC/CDPS组：游路、谢小威
- 风险引擎组：王勋彦、练紫颖、练紫莹、曹钰坤、王禹
- CED组：张文、骆京行、李强、易彤、谢邵恒、谢绍恒、陈一鎏
- calculator组：肖力、刘帅
- sales组：付勇、卢治东、张镕薪、柳勇、范恒旭、罗维、李斌、杨璞光、张广意
- UM组：王胜军、冯经宇
- 前端组：田娟、王玉珏、王玉玦、姚佳宇、郭研苹、王浩、王皓、柯常浩
- 技术平台组：伍健君、陈灿
- 放款组：张骏
- 测试组副标签：杨喜红、黄建、吴员英、韦日珍、曹钰坤、王禹

如果 owner 不在映射中：

- 不擅自归组；放入 `未归组`。
- 可在最终汇报中提醒用户哪些 Owner 待归组。

## 管理摘要模板：固定

BI 页面顶部 KPI 下方必须增加“管理摘要”卡片，放在原 Issue Classification 管理视图 note 之前。原分类管理视图可以保留为第二条 note，但管理摘要必须更靠前。

管理摘要用于给项目负责人/管理层一眼看懂当前闭环状态，固定分为四块：

1. `Jira 正式关闭`
   - 说明 Jira 口径：当前 Jira 已关闭 `closed_cancelled / total`，关闭率 `close_rate%`。
   - 同时给出 Jira 未关闭 `open` 和未关闭 S/H `critical_open`。

2. `预计可闭环`
   - 说明预计关闭口径：`已关闭 + Ready To Test in UAT + 客户解释/客户侧原因`。
   - 展示 `expected_closed / total` 和 `expected_close_rate%`。

3. `关闭率差距`
   - 展示 `expected_closed - closed_cancelled`。
   - 必须拆分说明主要来源：
     - Ready To Test in UAT 数量
     - Jira-open 且 Classification=客户解释数量
     - Jira-open 且由评论/描述判定为客户侧/暂不改/设计如此/已部署待验证数量
   - 注意三类来源建议互斥统计，避免重复解释。

4. `管理动作`
   - 固定动作方向：推动客户验证并翻转 Jira；优先处理 626 S/H。
   - 必须展示 626 版本桶未关闭数和 S/H 数。
   - 可追加 Owner 侧 S/H Top 3，用于当天盯人。

推荐文案模板：

```html
<div class="exec-summary-v01" id="execSummaryV01">
  <div class="exec-title">管理摘要</div>
  <div class="exec-grid">
    <div class="exec-item"><b>Jira 正式关闭</b><span>当前 Jira 已关闭 <strong>{closed} / {total}</strong>，关闭率 <strong>{close_rate}%</strong>；Jira 未关闭 <strong>{open}</strong> 条，其中 S/H <strong>{critical_open}</strong> 条。</span></div>
    <div class="exec-item"><b>预计可闭环</b><span>按“已关闭 + Ready To Test in UAT + 客户解释/客户侧原因”口径，预计可闭环 <strong>{expected_closed} / {total}</strong>，预计关闭率 <strong>{expected_close_rate}%</strong>。</span></div>
    <div class="exec-item"><b>关闭率差距</b><span>预计关闭比 Jira 正式关闭多 <strong>{gap}</strong> 条，主要来自 Ready To Test in UAT <strong>{ready}</strong> 条、客户解释类 <strong>{customer_explain_open}</strong> 条、评论/描述判定客户侧或暂不改 <strong>{comment_expected}</strong> 条。</span></div>
    <div class="exec-item action"><b>管理动作</b><span>优先推动客户验证并翻转 Jira；626 版本桶仍是主战场，未关闭 <strong>{release_626_open}</strong> 条、S/H <strong>{release_626_sh}</strong> 条；Owner 侧先盯 S/H 集中的 {owner_top3_text}。</span></div>
  </div>
</div>
```

推荐样式：

```css
.exec-summary-v01{background:#fff;border:1px solid #d7dee8;border-left:5px solid #9a7b2f;border-radius:10px;padding:14px 16px;margin:16px 0;box-shadow:0 1px 4px rgba(16,35,63,.05)}
.exec-title{font-weight:900;color:#0b1f3a;font-size:16px;margin-bottom:10px}
.exec-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:10px}
.exec-item{background:#f8fafc;border:1px solid #e5ebf2;border-radius:9px;padding:11px 12px;line-height:1.55}
.exec-item b{display:block;color:#0b1f3a;font-size:13px;margin-bottom:5px}
.exec-item span{font-size:12.5px;color:#344054}
.exec-item strong{color:#8f1d1d;font-weight:900}
.exec-item.action{background:#fff8e6;border-color:#efd99a}
@media(max-width:1200px){.exec-grid{grid-template-columns:1fr 1fr}}
@media(max-width:760px){.exec-grid{grid-template-columns:1fr}}
```

2026-05-08 21:10 已验证样例：

- Jira 正式关闭：434 / 793，54.7%；Jira 未关闭 359，S/H 251。
- 预计可闭环：550 / 793，69.4%。
- 差距：预计关闭比 Jira 正式关闭多 116；Ready To Test in UAT 67，客户解释类 14，评论/描述判定客户侧或暂不改 35。
- 管理动作：626 未关闭 195，S/H 169；Owner S/H Top 3：李强29条、骆京行19条、刘帅19条。

## 必须自检

每次输出给用户前必须执行自检。至少检查：

1. 数据源
   - JSON `source_db` 必须是用户指定 DB。

2. Jira 关闭口径
   - `Closed/Cancelled/Resolved` 的数量应计入已关闭。
   - `Ready To Test in UAT` 不计入 Jira 已关闭，但计入预计总体关闭。

3. 预计总体关闭
   - JSON 有 `expected_closed` 和 `expected_close_rate`。
   - Owner 趋势图中存在 `预计总体关闭`。
   - SVG 中存在 5 条顶部趋势线。
   - 最新点位显示 `expected_closed/total`。

4. Issue Classification 堆积图
   - 页面出现 `Issue Classification 堆积（未关闭）`。
   - 抽样 Owner 的堆积值必须等于该 Owner 未关闭 Issue 的分类统计。

5. 版本桶标签
   - 页面中不得出现旧标签：
     - `6月底版本`
     - `520=7月25版本`
     - `626=8月底版本`
     - `6月底`
     - `7月底`
     - `8月底`
   - 必须出现 `626 / 725 / 828`。

6. 结构禁项
   - 不得出现 `按关闭率排名追踪`。
   - 不得出现独立 `版本健康度` tab。

7. 静态渲染
   - 顶部趋势图每条 polyline 至少 2 个点，最好有历史 3 点。
   - 浏览器打开后 console JS error = 0。

建议自检代码片段：

```python
from pathlib import Path
import json, re
base = Path('/Users/zhangliang/hlb/04_Jira_jira/issuehub')
h = base/'IssueHub_BI_YYYYMMDD_V01.html'
j = base/'IssueHub_BI_YYYYMMDD_V01_data.json'
s = h.read_text()
d = json.loads(j.read_text())
owner = s[s.find('<section id="owner"'):s.find('<section id="eta"')]
assert d['source_db'].endswith('USER_SPECIFIED.db')
assert 'expected_closed' in d['kpi'] and 'expected_close_rate' in d['kpi']
assert '预计总体关闭' in owner and 'stroke-dasharray="6,4"' in owner
polys = re.findall(r'<polyline points="([^"]+)"', owner)
assert len(polys) >= 5 and all(len(x.split()) >= 2 for x in polys[:5])
assert 'Issue Classification 堆积（未关闭）' in owner
assert '按关闭率排名追踪' not in owner
for old in ['6月底版本','520=7月25版本','626=8月底版本','6月底','7月底','8月底']:
    assert old not in owner
```

然后用浏览器工具打开 HTML，切到 Owner 与闭环，检查 console：

- `js_errors == []`

## 已验证参考版本

2026-05-08 21:10 最新 DB 已验证：

- DB：`/Users/zhangliang/hlb/04_Jira_jira/issuehub/daily_db/issuehub-2026-05-08T13-25-41-987Z.db`
- HTML：`/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_20260508_V01.html`
- JSON：`/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_20260508_V01_data.json`
- 后处理脚本：`postprocess_owner_panel_v07_expected.py`

当时核心指标：

- 总数：793
- Jira 未关闭：359
- Jira Closed/Cancelled/Resolved：434
- Jira 关闭率：54.7%
- 预计总体关闭：550
- 预计总体关闭率：69.4%
- Ready To Test in UAT：67
- S/H 未关闭：251
- ETA 超期：88
- Golive Date 未填：95
- Classification 空值：27

版本桶未关闭：

- 626：195，S/H 169
- 725：31，S/H 17
- 828：25，S/H 8
- 未标记：108，S/H 57

已通过自检：

- 使用最新 DB：PASS
- Jira closed/cancelled/resolved 关闭口径：PASS
- 预计总体关闭 KPI：PASS
- 预计总体关闭趋势线存在：PASS
- 顶部趋势图 5 条线，每条 3 个点：PASS
- 最新点位显示 21:10：PASS
- 图上显示 434/793 与 550/793：PASS
- Issue Classification 堆积只看未关闭：PASS
- 版本桶标签为 626 / 725 / 828：PASS
- 无“按关闭率排名追踪”：PASS
- 浏览器控制台 JS 错误：0
