---
name: "airport-overnight-guide"
description: "调研全国任意机场航站楼内的过夜场地，生成含价格、位置、购买渠道、场景图、优缺点的分析表 HTML。当用户输入机场名想查机场内过夜/中转/赶早班机住宿攻略时调用（如'查一下浦东机场哪里能过夜'、'首都机场过夜攻略'）。"
---

# 全国机场过夜场地调研

输入一个机场名 → 全网深挖该机场航站楼内全部过夜场地 → 生成设计感分析表 HTML（环境列放图片）。

## 硬性输出要求（用户已确认的偏好，不可违反）

1. **不设"周边酒店"档**：只收航站楼内、或经连廊步行直达的场地；需班车/打车往返的机场外酒店群一律不进表。
2. **环境列只放图片**，不写文字说明；图为 AI 生成场景示意图，页内须注明"非实拍"。
3. **必须有"购买渠道"列**：线上平台（携程/美团/飞猪/去哪儿/KKday/Klook/GetYourGuide）、权益卡（龙腾/PP卡/悦途/支付宝钻石/银联/航司高卡可否免费进）、淘宝第三方券、现场扫码等，写明哪个渠道便宜。
4. **优缺点极简**：每条一句话短语，每格 2–3 条，关键条目带来源角标。
5. 表结构七列：场地 | 位置 | 价格 | 购买渠道 | 环境 | 优点 | 缺点；五档分组行：🛋️免费档 / 🔥经济档 / 🛏️计时舱档 / ☕贵宾休息室档 / 🏨酒店档。
6. 页脚：编号来源列表（标题+URL）、免责声明；正文含"对号入座"决策卡片。
7. 全程使用用户提问的语言（默认中文）。

## 工作流程

### 第 1 步：解析机场

- 输入可能是简称（"虹桥"→上海虹桥国际机场 SHA；"宝安"→深圳宝安国际机场 SZX；"天府"→成都天府国际机场 TFU）。用官方全称 + IATA 代码做后续搜索。
- 小机场若楼内确无过夜设施：如实告知，不硬凑；可口头给 1 条最近替代建议，但不建表项。

### 第 2 步：五档系统性搜索（质量关键，必须逐档排查，不得漏档）

每档至少 1 轮 WebSearch；贵宾休息室和计时舱两档最容易漏，务必单独查。把 `{机场}` 替换为全称或常用简称。

**A. 免费档**
- `{机场} 过夜 攻略 免费 小红书`
- `{机场} 免费休息区 躺椅 沙发 位置 登机口`
- `{机场} 24小时餐厅 安检外 过夜`

**B. 经济档**
- `{机场} 折叠床 躺平区 价格`
- `{机场} 按摩椅 过夜 平躺`

**C. 计时舱档**
- `{机场} 睡眠舱 计时休息室 价格 预订`
- `{机场} 计时休息室 前台 携程 便宜`

**D. 贵宾休息室档**
- `{机场} 贵宾休息室 价格 购买 大众点评`
- `{机场} 休息室 龙腾 PP 支付宝 钻石 免费进`
- `{机场} CHUM OR 环亚 OR Plaza Premium OR 东航 OR 南航 休息室`
- `FLYERT {机场} 休息室 实测`

**E. 酒店档（仅楼内/连廊步行可达）**
- `{机场} 航站楼内 机场酒店 价格`
- `{机场} 机场酒店 按小时 计费`

**交叉验证与查漏**
- `{airport} sleep overnight guide`（英文攻略交叉验证免费点位）
- `{机场} 过夜 实测 微博 OR B站 OR 什么值得买`
- 常见过夜品牌直接按"品牌+机场"搜：如憩、刻睡、Aerotel、环亚 Plaza Premium、CHUM、时空胶囊、空港宾馆。

### 第 3 步：逐场地深挖

对搜到的每个场地，用 WebFetch 打开最有信息量的 1–2 个帖子（优先旅客实测帖），核实：

- **位置**：具体到登机口/楼层/门号；**必须标注安检内还是安检外**（这决定深夜落地者能不能用）。
- **价格**：优先旅客实测价，标注计价方式（按小时/包夜/按次/投币）。
- **营业时间**：休息室类多为早 5 点–晚 10 点半左右，不能整夜——必须写进缺点。
- **购买渠道**：线上平台报价 vs 前台价、权益卡准入、优惠券；能便宜多少写清楚。
- **安检口夜间是否关闭**：部分机场凌晨关闭安检，安检内场地对深夜落地者不可用——核实并写进提示。
- 优缺点从真实评价提炼，每条压缩成一句话短语。

### 第 4 步：生成场景图

用 GenerateImage（Seedream）为每个场地生成 `landscape_4_3` 场景图：

- 保存到输出目录 `assets/env-{场地slug}.jpg`。
- Prompt 模式：`Website illustration: {场景英文描述}, photorealistic interior photography`。
- 常用场景库（按场地类型套用，可按搜到的实际描述调整）：
  - 免费休息区 → rows of recliner lounge chairs and sofas by floor-to-ceiling windows, travelers resting with blankets
  - 24h 餐厅 → late-night cafe sofas with charging ports, warm lighting
  - 折叠床 → rows of simple folding cots, privacy screens, disposable sheets
  - 按摩椅 → row of black massage chairs, one traveler reclined sleeping
  - 计时舱 → compact private sleeping pods like train soft-sleeper compartments, reading light and power outlets
  - 贵宾休息室 → lounge booths with cushions, buffet counter with hot food, coffee machine, shower room sign
  - 楼内酒店 → compact hotel room, queen bed with white linens, private bathroom, runway view

### 第 5 步：产出 HTML

- 输出目录：`airport-overnight-guides/{机场slug}/index.html` + `assets/` 子目录。
- 复制本 skill 目录下的 `template.html`，替换 `{{占位符}}`；**样式不要改**（用户已按参考图确认过设计：深色表头 + 米色分组条）。
- 分组行档位注释写价格区间（如 `g-note`）。
- 关键数据带 `<sup><a href="#cite-N">[N]</a></sup>` 角标，页脚 `#cite-N` 编号列出全部来源（标题 + URL），编号一一对应。
- 免责声明注明：价格为汇总区间、环境图为 AI 示意图、出行前以机场官方渠道为准。

## 质量检查清单（交付前逐项过）

- [ ] 五档都排查过？贵宾休息室档查了龙腾/PP/支付宝权益？
- [ ] 每个场地有具体位置（登机口/楼层）+ 安检内外标注？
- [ ] 购买渠道列有干货（哪个平台便宜、什么卡免费进）？
- [ ] 优缺点每条一句话，无长句？
- [ ] 环境列全部是图片，无文字？
- [ ] 没有混入需班车往返的周边酒店？
- [ ] 休息室营业时间限制写进缺点了？
- [ ] 来源列表编号与正文角标一一对应？
- [ ] 图片 AI 生成声明在页内？
