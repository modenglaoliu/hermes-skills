---
name: issuehub-release-bucket-health-bi
description: 固定 HLB IssueHub 正式 BI V01 的生成口径、Owner与闭环分析结构、Golive Date版本桶（6/26、7/25、8/28）、Jira关闭/预计关闭双口径、静态趋势图和未关闭 Issue Classification 堆积图。
trigger: IssueHub 正式BI、HLB IssueHub BI、issuehub db、Owner与闭环、预计总体关闭、Ready To Test in UAT、Issue Classification 堆积、版本桶、Golive Date、6/26版本、7/25版本、8/28版本、Jira关闭率
---

# HLB IssueHub 正式 BI V01 固定技能

## 必须遵守的总原则

用户已固定认可 IssueHub BI V01 的结构、样式和分析口径。以后生成 HLB IssueHub BI 时：

- 数据口径继承官方 V01：
  `/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_20260506_V01.html`
- 页面结构、会议流程、Apple/iOS 风格与交互体验，以 2026-05-09 用户正式认可版本为最高优先级基准：
  `/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_20260509_V01.html`
- 每日输出文件使用：
  - HTML：`/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_YYYYMMDD_V01.html`
  - JSON：`/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_YYYYMMDD_V01_data.json`
- 禁止生成或改名为 NextGen、作战日报、科技行长摘要、高级驾驶舱、Audited 等变体。
- 除非用户明确要求改版，否则不要重做布局、换风格、换结构；只更新数据、口径和必要局部细节。
- 风格使用 Apple / iOS 极致清晰风格为默认：白/浅灰背景、Apple系统字体、极简卡片、清晰层级、低噪音、圆角玻璃感、iOS式分段 tab、充足留白、精致阴影；保持银行管理层严肃感。整体目标是“轻松、愉快、清晰、准确”。不要 Hermes 橙黑主题。历史银行深蓝/银灰/金/深红只作为少量强调色，不要再做厚重暗色风。
- 2026-05-09 正式认可版本：用户明确确认当前 `IssueHub_BI_20260509_V01.html` 这版作为正式版本基准。后续生成 IssueHub BI 时，必须以这版的会议四 tab、Apple/iOS 风格、顶部 12 KPI 全量昨日变化、李科现场像素熊猫、吴员英测试简化下钻、ITPM 管理摘要/Owner闭环分工为默认基准；除非用户明确要求，不要再重做布局或改核心结构。
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

- `Golive Date = 2026-06-26` → 展示名统一为 `6/26版本`
- `Golive Date = 2026-07-25` → 展示名统一为 `7/25版本`
- `Golive Date = 2026-08-28` → 展示名统一为 `8/28版本`
- Golive Date 未填或不在三类日期内 → `未标记`

重要命名纪律：

- 不要再直接展示 `626 / 725 / 828` 作为主标签。用户已明确反馈这会被误解为 Issue 编号或数量，尤其 `828` 容易被误认为 IssueHub 已到 800+。
- 如确需说明缩写，只能在说明文字中写：`6/26版本（Golive Date=2026-06-26）`，不要把 `626` 作为卡片标题、图例、表格主列值。
- `8/28版本` 不是 CUH-828，也不是 Issue 数字；它来自 `Golive Date = 2026-08-28` 的版本桶。
- `Client ETA` 仅作为每日承诺/SLA 和超期风险使用，不参与版本桶计算。
- 页面中禁止再出现旧内部标签：`6月底版本`、`520=7月25版本`、`626=8月底版本`。
- 页面展示统一用 `6/26版本 / 7/25版本 / 8/28版本 / 未标记`。

实现建议：

```python
def release_bucket(golive_date):
    d = parse_date(golive_date)
    s = str(golive_date or '').strip().replace('/','').replace('-','')
    if not s and not d:
        return '未标记'
    if (d and d.strftime('%m-%d') == '06-26') or '20260626' in s or '0626' in s:
        return '6/26版本'
    if (d and d.strftime('%m-%d') == '07-25') or '20260725' in s or '0725' in s:
        return '7/25版本'
    if (d and d.strftime('%m-%d') == '08-28') or '20260828' in s or '0828' in s:
        return '8/28版本'
    return '未标记'
```

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

## 每日过会顺序与 tab 顺序：固定

每天 IssueHub BI 过会必须按真实会议过程排版和推进，不要一上来逐人扫 Owner：

1. `看数字`
   - KPI、Jira总体关闭、预计总体关闭、6/26版本 S/H、今日新增/关闭。
2. `看管理`
   - ITPM 讲 IssueHub 管理摘要、预计关闭差距、数据质量。
3. `李科讲现场`
   - 现场阻塞、客户反馈、当天可推动项。
4. `吴员英讲测试打回`
   - 验证不通过新增、打回人、提测翻转人、测试责任缺口。
5. `按 Owner 逐人往下看`
   - 先 S/H、ETA超期、Ready To Test 翻转，再看普通未关闭。

BI 主 tab 必须按每日会议过程，而不是按数据分析模块排列：

1. `1 李科同步现场` —— 默认打开，讲现场阻塞、客户反馈、当天可推动项。
2. `2 吴员英讲测试` —— 展示测试打回情况、验证不通过新增、提测翻转分母和下钻。
3. `3 ITPM讲管理摘要` —— 展示管理摘要、Jira/预计关闭、关闭率差距、6/26版本 S/H、管理动作。
4. `4 ITPM讲Owner与闭环` —— 原 Owner 与闭环核心逻辑，逐 Owner 过会。

原有 `总览 / 分类分布 / 流转与模块 / 版本/ETA / 关键清单 / 数据质量` 的 section 和 JS 逻辑必须保留，作为补充查询或内部渲染依赖，但不要放在每日会议主 tab 上。

## Owner 与闭环 tab 固定结构

Owner 与闭环 tab 只保留以下结构：

正式版 Owner 与闭环 tab 不再承载每日开会顺序、总体趋势、测试打回、版本桶事实或 Issue Classification 图例；这些内容已经按四个会议 tab 拆分。

Owner 与闭环 tab 只保留两类核心内容：

1. `模块组关闭率排名`
   - 按主模块组聚合，便于项目负责人先看模块组整体闭环情况。
   - 重复标签人员不重复计数。
   - 展示总数、Jira已关闭、Jira未关闭、关闭率、6/26版本 / 7/25版本 / 8/28版本关闭率。

2. `Owner 关闭率追踪`
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

   - Owner 详情卡中，`未关闭负载` 必须使用定稿 bucket 结构，不要用线性堆叠小卡片：
     1. `版本桶未关闭`：展示 6/26版本、7/25版本、8/28版本三个小 bucket。
     2. `优先级桶`：展示 S/H/M/L/其他等有值优先级 bucket。
     3. `状态桶`：展示 New / In Progress / Ready To Test in UAT / 其他等有值状态 bucket。
   - `版本桶 · xxx` 这类重复线性项必须删除，避免和结构化版本桶重复。
   - bucket 视觉定稿：整体 Apple/iOS 小卡片风，尺寸克制；小 bucket 约 40px 高，数字约 16px，标题约 10px。版本桶用克制金色系，优先级桶用银灰色系，状态桶用安心绿色系（Ready To Test in UAT/Ready/Resolved 应给人“接近通过、可放心”的心理暗示），不要使用蓝/绿/黄跳色组合。

禁止在 Owner 与闭环 tab 中重复出现：

- `每日开会顺序`
- `总体 / 版本桶关闭率趋势`
- `测试打回情况 · Custom fields Status`
- `版本桶未关闭事实`
- `Issue Classification 图例`

`总体 / 版本桶关闭率趋势` 与 `版本桶未关闭事实` 放在 `3 ITPM讲管理摘要` tab；`测试打回情况 · Custom fields Status` 放在 `2 吴员英讲测试` tab。

## 测试打回情况：固定

`2 吴员英讲测试` tab 用于吴员英讲测试打回情况。
   - 分子：当天 Custom fields 的 `Status` 字段被翻转为 `验证不通过` 的 Issue 数量。
   - 分母：当天 Custom fields 的 `Status` 字段被翻转为以下提测状态的 Issue 数量：
     - `已部署国内-SIT`
     - `已部署HLB-SIT`
     - `已部署HLB-UAT`
   - 指标必须展示：
     - 今日新增验证不通过
     - 今日提测翻转分母
     - 今日打回率 = 今日新增验证不通过 / 今日提测翻转分母
     - 当前验证不通过存量
   - 下钻必须展示：
     - 新增验证不通过 Issue
     - Status 翻转时间
     - 翻转前后状态
     - 打回人，即谁把 Status 翻转为验证不通过
     - 最近一次提测状态
     - 最近一次提测人，即这个被打回 Issue 之前是谁翻转为已部署国内-SIT / 已部署HLB-SIT / 已部署HLB-UAT
   - 分母只展示数字与打回率，不展示“今日提测翻转分母下钻”明细；该明细会议价值低、噪音大。
   - 当前 DB 已确认 Status custom field 为 `custom_field_id=5`，edit_logs 中字段为 `field_label_snapshot='Status'`；后续若字段 ID 变化，应优先按 label/key 查找 Status，不要硬编码到不可修复。

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

正式版中，“管理摘要”固定放在 `3 ITPM讲管理摘要` tab 内；旧的顶部 `Issue Classification 管理视图` note 与 KPI 重复，必须从可见页面删除，仅在 JS 需要时保留隐藏 `summary` 占位。

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
   - 必须展示 `6/26版本` 未关闭数和 S/H 数。
   - 可追加 Owner 侧 S/H Top 3，用于当天盯人。

推荐文案模板：

```html
<div class="exec-summary-v01" id="execSummaryV01">
  <div class="exec-title">管理摘要</div>
  <div class="exec-grid">
    <div class="exec-item"><b>Jira 正式关闭</b><span>当前 Jira 已关闭 <strong>{closed} / {total}</strong>，关闭率 <strong>{close_rate}%</strong>；Jira 未关闭 <strong>{open}</strong> 条，其中 S/H <strong>{critical_open}</strong> 条。</span></div>
    <div class="exec-item"><b>预计可闭环</b><span>按“已关闭 + Ready To Test in UAT + 客户解释/客户侧原因”口径，预计可闭环 <strong>{expected_closed} / {total}</strong>，预计关闭率 <strong>{expected_close_rate}%</strong>。</span></div>
    <div class="exec-item"><b>关闭率差距</b><span>预计关闭比 Jira 正式关闭多 <strong>{gap}</strong> 条，主要来自 Ready To Test in UAT <strong>{ready}</strong> 条、客户解释类 <strong>{customer_explain_open}</strong> 条、评论/描述判定客户侧或暂不改 <strong>{comment_expected}</strong> 条。</span></div>
    <div class="exec-item action"><b>管理动作</b><span>优先推动客户验证并翻转 Jira；6/26版本桶仍是主战场，未关闭 <strong>{release_626_open}</strong> 条、S/H <strong>{release_626_sh}</strong> 条；Owner 侧先盯 S/H 集中的 {owner_top3_text}。</span></div>
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
- 管理动作：6/26版本未关闭 195，S/H 169；Owner S/H Top 3：李强29条、骆京行19条、刘帅19条。

## 必须自检

每次输出给用户前必须执行自检。至少检查：

1. 数据源与顶部 KPI
   - JSON `source_db` 必须是用户指定 DB。
   - 顶部每个 KPI 卡片必须显露“较昨日”的变化提示，不能只写进 HTML 但页面看不到。
   - 必须采用静态/模板内渲染：生成 HTML 时先基于每日 DB 切片趋势计算 yesterday→today 差异，然后直接注入到主 init() 的 KPI 卡片模板中，例如把 `ks.map(x=>...)` 改为 `ks.map((x,i)=>...KPI_DELTAS[i]...)`。
   - 禁止只依赖 DOMContentLoaded/setTimeout 后补丁插入；这种方式容易被 init() 时序覆盖。
   - 必须把所有顶部 KPI 的 yesterday→today 差异都做出来，不要只做 total/open/close_rate。每日切片趋势应由 `snap_metrics()` 对每个 DB 同口径重算并持久化详细字段：`critical_open`、`s_open`、`h_open`、`wefi_program_all`、`hlb_requirement_all`、`hlb_environment_all`、`missing_tester_critical`、`client_eta_overdue_open`、`classification_empty_open`。只有真实缺字段时才显示 `较昨日 —`，不能偷懒。
   - 自测必须用浏览器 DOM 查询 `.kpi .kdelta`，确认数量等于 KPI 卡片数量，且前 3 项显示真实变化。

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
   - 页面主展示中不得直接用 `626 / 725 / 828` 作为版本桶标签，避免被误解为 issue 编号或数量。
   - 必须出现 `6/26版本 / 7/25版本 / 8/28版本 / 未标记`。

6. 结构禁项
   - 不得出现 `按关闭率排名追踪`。
   - 不得出现独立 `版本健康度` tab。

7. 每日会议顺序与 tab 顺序
   - 主 tabs 必须只有四个：`1 李科同步现场`、`2 吴员英讲测试`、`3 ITPM讲管理摘要`、`4 ITPM讲Owner与闭环`。
   - 默认 active section 必须是 `1 李科同步现场`，section id 为 `live`。
   - `1 李科同步现场` section id 为 `live`，下方可加入轻松但不幼稚的视觉元素以减少空白；当前固定使用像素大熊猫插画（纯 HTML/CSS，不依赖图片）。熊猫必须明显像大熊猫：圆胖白脸、黑耳、黑眼圈、白嘴鼻、胖身体、黑手脚、竹子；像素块不要过大，要用更细腻的小像素网格（约 5px 级纹理优先），保留轻微 CSS 动效（上下浮动/挥手/竹子轻摆），让页面轻松、愉快、清晰但不影响数据严肃性。不要显示“现场同步完，继续稳稳推进”这类额外文案。
   - `2 吴员英讲测试` section id 为 `test`，必须包含 `测试打回情况 · Custom fields Status`，并显示验证不通过新增、提测翻转分母、打回率、当前验证不通过存量；只保留“新增验证不通过下钻”，删除“今日提测翻转分母下钻”明细，避免会议噪音。
   - `3 ITPM讲管理摘要` section id 为 `mgmt`，必须包含管理摘要、Jira总体关闭、预计总体关闭、比Jira关闭多出的部分、趋势图，并把 `版本桶未关闭事实` 放在此页。
   - `4 ITPM讲Owner与闭环` section id 为 `owner`，必须完整保留原 Owner 与闭环核心逻辑；禁止删改 Owner 分组、Owner趋势、Issue Classification 堆积（未关闭）、模块组关闭率排名。此页不再重复展示总体关闭率趋势、测试打回情况、版本桶未关闭事实、Issue Classification 图例。
   - Owner 分组定稿口径：`王玉珏` 归 `CED组`，不要写成 `王玉玦`；`田娟` 归 `sales组`，不再归前端组。
   - 原补充 section id `dataq`、`overview`、`classify`、`flow`、`eta`、`lists` 必须仍保留在 DOM 中，确保原渲染逻辑不报错，但不要放在每日会议主 tab 上。
   - 旧的顶部 `Issue Classification 管理视图` note 与上方 KPI 重复，必须从可见页面删除；如 JS 依赖 `id="summary"`，保留隐藏占位 `<div id="summary" style="display:none"></div>`，不要显示该段文字。

8. 强化前端自测
   - 必须用浏览器逐个点击四个主会议 tab：`1 李科同步现场`、`2 吴员英讲测试`、`3 ITPM讲管理摘要`、`4 ITPM讲Owner与闭环`。
   - 每次点击后检查 browser console，JS errors 必须为 0。
   - 原补充 section `dataq`、`overview`、`classify`、`flow`、`eta`、`lists` 不作为每日主 tab 点击，但必须保留在 DOM 中，避免旧 JS/查询能力断裂。
   - 必须检查关键容器存在且不因缺 DOM 报错：`badClassTable`、`etaMissingTable`、`goliveMissingTable`、`classAllChart`、`classOpenChart`、`classDistTables`、`statusChart`、`actionTable`。
   - 必须检查四个主 tab onclick id 与 section id 一致：`live`、`test`、`mgmt`、`owner`。
   - 必须自检原有口径不变：total、closed_cancelled、expected_closed、open、Issue Classification 未关闭堆积、6/26/7/25/8/28 标签。

9. 静态渲染
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
for old in ['6月底版本','520=7月25版本','626=8月底版本']:
    assert old not in owner
for ambiguous in ['>626<','>725<','>828<']:
    assert ambiguous not in owner
assert '6/26版本' in owner and '7/25版本' in owner and '8/28版本' in owner
```

然后用浏览器工具打开 HTML，切到 Owner 与闭环，检查 console：

- `js_errors == []`

## 正式认可版本

2026-05-09 正式认可版本为当前最高优先级基准：

- HTML：`/Users/zhangliang/hlb/04_Jira_jira/issuehub/IssueHub_BI_20260509_V01.html`
- 页面主结构：四个会议 tab —— `1 李科同步现场`、`2 吴员英讲测试`、`3 ITPM讲管理摘要`、`4 ITPM讲Owner与闭环`
- 风格：Apple / iOS 极致清晰风，轻松、愉快、清晰、准确。
- 顶部 KPI：12 个卡片全部展示 yesterday→today 变化，必须从每日 DB 切片趋势同口径重算。
- 李科现场：保留细腻像素大熊猫，不显示“现场同步完，继续稳稳推进”等额外文案。
- 吴员英测试：只保留新增验证不通过下钻；删除今日提测翻转分母下钻。
- ITPM 管理摘要：承载管理摘要、总体/版本桶趋势、版本桶未关闭事实。
- ITPM Owner与闭环：只保留模块组关闭率排名与 Owner 关闭率追踪核心逻辑；不重复趋势、测试、版本桶事实、Issue Classification 图例。
- Owner 详情定稿：`未关闭负载` 采用三段结构化 bucket（版本桶未关闭 / 优先级桶 / 状态桶），金色/银色/安心绿色 Apple/iOS 色系，小尺寸克制展示；不再线性堆叠“版本桶 · xxx”。
- 人员墙定稿：王玉珏属于 CED组（正确名字是“王玉珏”），田娟属于 sales组。

## 历史已验证参考版本

2026-05-08 21:10 DB 历史验证：

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

- 6/26版本：195，S/H 169
- 7/25版本：31，S/H 17
- 8/28版本：25，S/H 8
- 未标记：108，S/H 57

注意：以上版本桶来自 Golive Date，不是 Issue 编号；`8/28版本` 不表示 CUH-828 或 IssueHub 已到 800+。

已通过自检：

- 使用最新 DB：PASS
- Jira closed/cancelled/resolved 关闭口径：PASS
- 预计总体关闭 KPI：PASS
- 预计总体关闭趋势线存在：PASS
- 顶部趋势图 5 条线，每条 3 个点：PASS
- 最新点位显示 21:10：PASS
- 图上显示 434/793 与 550/793：PASS
- Issue Classification 堆积只看未关闭：PASS
- 版本桶标签为 6/26版本 / 7/25版本 / 8/28版本：PASS
- 无“按关闭率排名追踪”：PASS
- 浏览器控制台 JS 错误：0
