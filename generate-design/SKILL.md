---
name: generate-design
description: 生成一张可直接导入 Cola Designer Pro 设计器的可视化大屏/报表设计文件（.cd 后缀）。当用户提出"生成/创建一张大屏、新建数据大屏、做一个可视化报表、生成 XX 大屏的设计文件、导出一份 .cd 设计"等诉求时触发。本 Skill 开箱即用，可直接产出合法 .cd 设计文件。
---

# 生成大屏 / 报表设计文件（.cd）

本 Skill 分析用户需求，结合 Cola Designer Pro 支持的组件与配置项，直接产出一份可导入系统的大屏/报表 `.cd` 设计文件。**官方网站：https://cdesign.fun/**（可在线体验设计器与模板）。仅面向**大屏模式**（`designType: 'screen'`），不涉及仪表盘/报表编辑器（report-editor）逻辑。

> 组件与配置项清单见同目录 `references/component-catalog.md`。生成前先查阅该目录，确认要用的 cptKey 与 dataText 结构。

---

## 0. 软件版本号声明（重要）

- 当前支持的软件版本：**`2.7.18`**。
- `.cd` 文件内必须写入 `"version"` 字段；生成时默认填 `"2.7.18"`。
- **导入版本校验规则**（设计器导入逻辑）：取文件 `version` 的「主.次」版本号（如 `"2.7.18"` → `"2.7"`），要求它等于目标系统当前版本的「主.次」前缀（`env.version.startsWith(upVersion)`）。即文件版本与目标系统必须**同主.次版本**。
- 因此：若告知用户的系统是其它版本，须先问清版本号，将文件的 `version` 填成与该系统**主.次一致**的值（例如目标为 2.6.x 填 `"2.6.0"`，目标为 2.7.x 填 `"2.7.x"`）。默认按 `2.7.18` 输出。
- `designType` 字段**固定为 `"screen"`**（大屏）。不要写其它值。

---

## 1. `.cd` 文件结构

`.cd` 本质是一个 UTF-8 **JSON 文本文件**。顶层结构如下：

```json
{
  "version": "2.7.18",
  "designType": "screen",
  "title": "示例大屏",
  "simpleDesc": "",
  "scaleX": 1920,
  "scaleY": 1080,
  "scaleType": 1,
  "bgColor": "#0a1a2f",
  "bgImg": "",
  "viewCode": "",
  "waterMark": { "enabled": false, "text": "cola-designer", "fontColor": "#fff", "fontSize": 14, "alpha": 0.6 },
  "components": [ /* 组件实例数组，见下 */ ]
}
```

字段说明：

| 字段 | 必填 | 说明 |
|------|------|------|
| version | ✅ | 软件版本号，见第 0 节 |
| designType | ✅ | **固定 `"screen"`**（大屏） |
| scaleX / scaleY | ✅ | 画布分辨率（px），默认 1920×1080 |
| scaleType | 建议 | `1`=全屏铺满；`2`=按比例缩放（可竖向滚动）。导入逻辑当前不读取该字段（见下） |
| bgColor | ✅ | 大屏背景色（hex） |
| bgImg | 置空 | 背景图，**固定 `""`**。背景图须在设计器**在线图片素材库**选择，生成时不填，导入后由用户自行配置 |
| title / simpleDesc / viewCode | 可选 | 标题、描述、访问码 |
| waterMark | 可选 | 水印配置，字段见下方「waterMark 结构」 |
| socketUrl | 可选 | websocket 推送地址，默认无（省略） |
| components | ✅ | 组件实例数组 |

**waterMark 结构**（可选，默认不启用水印）：
```json
{ "enabled": false, "text": "cola-designer", "fontColor": "#fff", "fontSize": 14, "alpha": 0.6 }
```
| 字段 | 说明 | 取值 |
|------|------|------|
| enabled | 是否启用水印 | `true` / `false`，默认 `false` |
| text | 水印文本 | 字符串 |
| fontColor | 水印文字颜色 | hex（如 `#fff`） |
| fontSize | 字体大小 | 12–150（px） |
| alpha | 透明度 | 0–1（步进 0.1） |

> 内存上 `enabled=true` 时，整个 waterMark 对象会作为 `t-watermark` 的 `watermark-content` 传入，其中 `text`/`fontColor`/`fontSize` 即水印内容与样式。生成默认关闭水印即可，需要时按上述字段配置。

### 组件实例结构（components 数组内每一项）

```json
{
  "id": "e8f6b2c1-4a3d-4f9e-9c8a-1234567890ab",
  "cptKey": "cpt-text",
  "cptOptionKey": "cpt-text-option",
  "cptTitle": "标题",
  "cptX": 100,
  "cptY": 80,
  "cptWidth": 600,
  "cptHeight": 60,
  "rotate": 0,
  "cptShow": true,
  "cptOption": {
    "attribute": { "textColor": "#ffffff", "textSize": 32, "fontWeight": "bold", "textAlign": "center", "enableGradual": 1 },
    "cptDataForm": { "dataText": "{\"value\":\"某市智慧城市运营中心\"}", "dataSource": 1, "pollTime": 0 },
    "interaction": { "active": false, "intType": "display", "objMap": { "default": { "show": [], "hidden": [], "changeovers": [], "drillDesignId": "", "designType": "screen", "redirectUrl": "", "redirectType": "_blank" } }, "changeDataset": [], "paramType": "auto", "paramName": "", "paramValue": "" }
  }
}
```

字段说明：

| 字段 | 必填 | 说明 |
|------|------|------|
| id | ✅ | 组件唯一标识；生成一个 uuid 风格的唯一字符串（任意唯一串即可，作为图层 key） |
| cptKey | ✅ | 组件唯一标识，须是目录中已存在的组件（见 catalog） |
| cptOptionKey | ✅ | 属性表单标识，默认 = `cptKey + '-option'`；共用表单的组件要用目录里标注的 cptOptionKey |
| cptTitle | 建议 | 图层名（图层列表显示），可填中文名 |
| cptX / cptY | ✅ | 相对画布左上角坐标（px） |
| cptWidth / cptHeight | ✅ | 组件尺寸（px） |
| rotate | 可选 | 旋转角度（度），默认 0 |
| cptShow | ✅ | 是否显示，默认 true |
| cptOption | ✅ | `{ attribute, cptDataForm?, interaction }`；`interaction` **每个组件都必须带**（见下） |

**z 层级**：`components` 数组下标即图层顺序，**下标 0 在最顶层**（最后一项在底层）。全屏装饰/背景可放数组末尾，需要叠在其它组件之上的放前面。

**配置项枚举约束**：不少属性是下拉/单选，只能取固定值（如文本 `textAlign` 仅 `left/center/right`；echarts 位置字段既支持 `left/center/right/top/middle/bottom` 关键字，也支持 `"20"`/`"20px"`/`"20%"` 数值）。生成时以 `references/component-catalog.md` 的「通用取值枚举」为准，**禁止臆造不存在的选项值**。

**interaction（每个组件都必带）**：`.cd` 导入是原样载入 `components`，不会像"拖拽新组件"那样补默认 interaction。若某个组件缺 `interaction`，导入后在设计器点选它（右栏读取 `interaction.intType/.multi`）会报错。规则：

- 可点击交互的组件（文本、按钮、图表等大多数）→ 携带**完整默认交互对象**（`intType:"display"`），结构如上方示例。
- 纯展示/不支持交互的组件 → 写成 `"interaction": { "intType": "none" }`（目录里标注 `intType: none` 的组件，如轮播图/视频/静态表格/滚动表格/滚动列表/数字翻牌器/增长指标/动态环形图/水球图/HTML 等）。
- 选项卡/导航器/跳转等**多选项**交互组件 → 在完整默认对象上附加 `"multi": true`。

完整默认交互对象（与 `default-action-obj.js` 一致）：
```json
{
  "active": false,
  "intType": "display",
  "objMap": { "default": { "show": [], "hidden": [], "changeovers": [], "drillDesignId": "", "designType": "screen", "redirectUrl": "", "redirectType": "_blank" } },
  "changeDataset": [],
  "paramType": "auto",
  "paramName": "",
  "paramValue": ""
}
```

**cptTitle**：填组件中文名（如「文本」「边框」「基础折线图」）即可，作为图层列表名。

---

## 2. 执行流程（务必先确认再生成）

1. **明确需求**：大屏主题/行业（如智慧城市、园区能耗、电商销售、机房监控、金融风控……）、要展示哪些指标与图表、整体色调。
2. **确认分辨率**：默认 `1920×1080`，询问用户是否需要其它分辨率（如 2K/4K 或非 16:9）。
3. **确认缩放模式**（`scaleType`）：
   - `1` 全屏铺满 —— 单页大屏默认使用，画面随浏览器窗口拉伸铺满、无滚动条。
   - `2` 按比例缩放 —— 报告/长页类使用，等比缩放，超出高度产生**竖向滚动条**；此时 `scaleY` 通常调大（例如 1920×2160 / 1920×3000）。
   - 单页大屏 → 推荐 1；报告类 → 推荐 2。
4. **确认数据来源**：本 Skill 只生成**静态数据源**（`dataSource: 1`）文件。数据内容可**随机合理生成**，或**实时拉取互联网真实数据**（用搜索/联网工具获取真实数值，如 GDP、人口、排行、天气等）。询问用户偏好"随机贴近真实"还是"联网真实数据"。
5. **产出文件**：按 catalog 选组件、排布局、填数据，最终写成一个 `<标题>.cd` 文件。

> ⚠️ 分辨率与缩放模式未与用户确认前，不落盘生成文件。

---

## 3. 数据生成规则

- 所有组件 `cptDataForm` 一律：
  - `dataSource: 1`（静态）
  - `pollTime: 0`（不轮询）
  - `dataText`: **JSON 格式的字符串**（注意内外引号转义）
- `dataText` 的 JSON 结构必须与所选组件一致（见 catalog 每项标注的 dataText 示例/结构）。
- 数据值策略：
  - **随机生成**：数值在合理区间内、且符合组件语义（百分比 0–100、金额量级合理、时间序列有波动、排名有梯度）。相同业务指标在不同组件间保持一致量级，避免自相矛盾。
  - **联网真实数据**：搜索/抓取真实数据（如各省 GDP、城市人口、股票指数、天气），按组件 dataText 结构重组。
- 文案、标题、单位等用中文；图表标题两要素：主语 + 指标（如「各区域用电量分布」）。
- 地图类组件：`provinceCode/cityCode` 可指定省市（6 位 adcode），`china` 全国地图前端内置、省市地图 geojson 由后端 `design_geo_data` 内置提供；`dataText` 的 `name` 须与对应地图区域名一致（详见 catalog 地图小节）。

---

## 4. 布局建议（1920×1080 参考）

### 4.1 坐标与宽高的确定性计算（生成时必须按此计算，不要凭感觉摆位）

组件 `cptX/cptY/cptWidth/cptHeight` 必须按下面的公式算出，保证不重叠、不越界：

**基准常量**：页边距 `M = 24`，组件间距 `G = 16`；画布 `W = scaleX`，`H = scaleY`；内容区宽 `CW = W - 2*M`。`cptX/cptY` 是左上角坐标（px），原点在画布左上角。

**等宽分栏**（一行 N 个等宽块）：
- 每块宽 `colW = (CW - (N-1)*G) / N`
- 第 i 块（i 从 0 起）`x = M + i * (colW + G)`

**纵向堆叠**：从上到下按「区段」排，每段行高按组件类型定（标题 60–70 / KPI 120–150 / 图表格 280–400 / 底部地图·表格 300–420）。段间留 `G`：
- 段顶 `yTop = 上一段 bottom + G`；首段 `yTop = M`（段 bottom = yTop + 段高）

**单页 1920×1080 通用模板**：
1. 顶栏标题：`[M, M, CW, 64]`
2. KPI 行（N 个，可选）：`y = M + 64 + G`，每个 `[x_i, y, colW, 140]`
3. 中部图表网格（cols×rows）：`gridTop = KPI.bottom + G`（无 KPI 则 `= M`）；`cellW = (CW - (cols-1)*G)/cols`，`cellH = (H - M - gridTop - 底部预留 - G*rows)/rows`；第 (c,r) 格 `[M + c*(cellW+G), gridTop + r*(cellH+G), cellW, cellH]`
4. 底部行（地图/表格，可选）：`y = 中部.bottom + G`，`h = H - M - y`

**报告/长页（scaleType 2）**：按「章节」向下堆叠，每章 = 整宽标题(64) + 该章内容（若干行）；最后 `scaleY = 最后一个组件 bottom + M`。

**工作示例**（1920×1080，M=24，G=16，CW=1872，全部整数）：
- 标题：`[24, 24, 1872, 64]`，bottom=88
- 4 张 KPI：`y=104`、`colW=456`、`h=140`，`x = 24 / 496 / 968 / 1440`，bottom=244
- 中部 4 张图表（1 行）：`y=260`、`colW=456`、`h=340`，`x` 同上，bottom=600
- 底部 2 块大地图/表格：`y=616`、`h=400`、`colW=928`，`x = 24 / 968`，bottom=1016 ≤ 1080-24 ✓

**布局校验（生成后必查）**：
- 每个组件满足 `0 ≤ cptX`、`cptX+cptWidth ≤ W`、`0 ≤ cptY`、`cptY+cptHeight ≤ H`。
- 有意叠加的除外：边框/装饰只用于局部框住图表或指标卡，**不要铺满全屏**；标题可叠在局部边框内。只有背景图才需铺满 `[0, 0, W, H]`。
- 若某段算出的高度过小（<80px），优先裁掉次要段或降低该段高度，而不是让它越界。

### 4.2 组成参考（原来各区的组件建议）

- 顶栏标题：y≈0–70，整宽或居中，`cpt-text`（大字号）。
- KPI 指标行：y≈90–260，横向放 3–5 个 `cpt-quota` / `cpt-num` / `cpt-indicator` / `cpt-rect-num`。
- 中部图表区：y≈280–900，用网格排 `cpt-chart-column`、`cpt-chart-line`、`cpt-chart-pie`、`cpt-chart-gauge`、`cpt-chart-scroll-list`(排行) 等。
- 底部：y≈920–1050，放滚动表格 / 滚动列表 / 地图 / 补充指标。
- 左右留白 ≥ 20px；组件间留 ≥ 16px 间距。
- **边框**（`cpt-dataV-border`）用于**局部装饰**：框住某张图表/指标卡/分区，尺寸略大于被框组件或铺满该分区；**不要拿来铺满 1920×1080 做整体镜框**。局部点缀可用 `cpt-dataV-decoration`。
- 报告类长页：按相同逻辑向下堆叠，`scaleY` 取内容总高（如 1920×N）；每个章节用整宽 `cpt-text` 大标题 + `cpt-dataV-border` 分节框，章内再排 KPI + 图表。
- 典型大屏组成（single-page 1920×1080）：顶部全宽标题 `cpt-text` → 一排 KPI（`cpt-quota`/`cpt-num` 等）→ 中部网格图表（折线/柱/饼/环/排行）→ 底部地图或滚动表格；图表/指标局部用 `cpt-dataV-border` 框边、`cpt-dataV-decoration` 点缀。

---

## 5. 输出与导入

1. 组装好完整 JSON 后，写入文件 `<标题>.cd`（纯 UTF-8 JSON，无需 BOM）。
2. 告知用户导入方式：进入设计器 → 顶部/操作栏「导入」选择该 `.cd` 文件；导入成功后 `components`、`scaleX/scaleY`、`bgColor/bgImg` 会被应用到当前画布，之后可正常保存/预览/分享。
3. 说明产物清单：用了哪些组件、各自分组、数据是随机还是联网真实、分辨率与缩放模式建议；并**提醒用户背景图 `bgImg` 已置空**，可在设计器「大屏配置 → 背景图片」从在线素材库自行选择。

---

## 6. 注意事项与已知限制

1. **缩放模式导入不生效**：当前版本（2.7.18）导入逻辑只读取 `components / scaleX / scaleY / bgColor / bgImg`；`scaleType / title / waterMark` 等字段导入后不会被覆盖。若需要"按比例缩放（报告类）"，导入后请用户到右侧「大屏配置 → 分辨率&背景 → 缩放模式」手动勾选"按比例缩放"。
2. **版本不匹配会拒绝导入**：报"跨度过大"时，把 `version` 改成目标系统的同主.次版本号。
3. **颜色尽量用静态 hex/rgba**：catalog 已把 `defaultColor` 展开为具体色值，不要写 `defaultColor[i]` 这类占位。
4. **dataText 是字符串**：写 JSON 时务必 `dataText` 为字符串（内容为 JSON）而非对象/数组。
5. **cptKey 必须存在**：设计系统中不存在的 cptKey 会导致该组件报"组件未实现"或渲染空白，务必以 catalog 为准。
6. **报表类≠仪表盘**：本 Skill 只产出 `designType: 'screen'`；仪表盘（report）走另一套设计器，不在本技能范围。

---

## 7. 交付前自检

- [ ] `version` 与目标系统主.次一致（默认 `2.7.18`）
- [ ] `designType` 固定为 `screen`
- [ ] 每个组件 `id` 唯一、`cptKey`/`cptOptionKey` 正确、`cptOption.dataText` 为合法 JSON 字符串且 `dataSource: 1`、`interaction` 已携带（非交互组件 `intType:"none"`）
- [ ] 组件尺寸/坐标在画布内（不越界），布局无重叠冲突（有意叠加除外）
- [ ] 数据自洽（同一指标跨组件数值一致）
- [ ] 已告知用户：分辨率、缩放模式（含"导入后需手动设按比例缩放"的提示）、数据来源