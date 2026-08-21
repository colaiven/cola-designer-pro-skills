# 组件目录与配置项（供 .cd 生成参考）

本文件是 Cola Designer Pro 大屏模式下可用组件的**参考目录**，用于生成 `.cd` 文件时产出合法的 `cptOption`。官方产品地址：https://cdesign.fun/

## 0. 默认色板与字段约定

组件 attrs 中出现的 `defaultColor` 展开为下列固定色值（生成文件时直接写 hex，不要写 `defaultColor[i]`）：

| 下标 | 色值 |
|------|------|
| 0 | `#0061C2` |
| 1 | `#409EFF` |
| 2 | `#31adfb` |
| 3 | `#8FC7FF` |
| 4 | `#DEEEFF` |
| 5 | `#bbd3fb` |

整组色板（用于 `color` / `lineColors` 等数组字段）：
```json
["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"]
```

字体中文默认 `微软雅黑`；数字字体常用 `lccd` / `DIN` / `CAI978`（液晶/数码体）。凡 `attribute` 里的颜色字段，正文都能直接改 hex/rgba。

### 通用取值枚举（写死取值，禁止臆造不存在的选项值）

**跨组件通用**
- `textAlign`（文本水平对齐）：`"left"`(靠左) / `"center"`(居中) / `"right"`(靠右)
- `fontWeight`（字重）：`"lighter"` / `"normal"` / `"bold"`
- `fontStyle`（字形）：`"normal"` / `"italic"`
- `textDecoration`（装饰线）：`"none"` / `"underline"` / `"overline"` / `"line-through"`
- `borderStyle`（边框样式）：`"none"` / `"dotted"` / `"dashed"` / `"solid"` / `"double"` / `"groove"` / `"ridge"` / `"inset"` / `"outset"`
- `fit`（图片填充，image/carousel 用）：`"fill"` / `"contain"` / `"cover"` / `"none"` / `"scale-down"`
- `direction`（柱状图）：`"x"`(竖向) / `"y"`(横向)；轮播图 `direction`：`"horizontal"` / `"vertical"`
- `orient`（图例/漏斗）：`"horizontal"`(横向) / `"vertical"`(纵向)

**echarts 位置取值（`title.left/top`、`legend.left/top`、`grid.left/top` 等）**

位置字段既支持**关键字**，也支持**数值**（像素/百分比），一律写成字符串：
- 水平方向（left/x）：`"left"` / `"center"` / `"right"`，或 `"20"`、`"20px"`、`"20%"`（`20%` 相对画布宽）
- 垂直方向（top/y）：`"top"` / `"middle"`(亦可用 `"center"`) / `"bottom"`，或 `"20"`、`"20px"`、`"20%"`（相对画布高）
- 饼图/环图圆心 `series.center`：`[x, y]` 数组，常用 `[50, 50]`（即 50%）
- 例：`"left":"center"`、`"top":"top"`、`"left":"10"`（10px）、`"left":"8%"` 均合法

**组件自有枚举**
- 饼图 `series.roseType`：`"false"`(不展示) / `"radius"`(圆心角玫瑰) / `"area"`(扇区)
- 饼图 `series.label.position`：`"outside"`(外侧) / `"inside"`(内部) / `"center"`(中心)
- 雷达图 `radar.shape`：`"polygon"`(多边形) / `"circle"`(圆形)
- 水位图 `shape`：`"rect"` / `"roundRect"` / `"round"`
- 漏斗图 `series.orient`：`"vertical"` / `"horizontal"`；`series.sort`：`"descending"` / `"ascending"` / `"none"`
- 排行榜 `sort`：`"desc"`(降序) / `"asc"`(升序) / `"cus"`(自定义)；`scrollType`：`"one"`(单列滚动) / `"all"`(整列滚动)
- 轮播图 `trigger`：`"hover"` / `"click"`；`type`：`"default"` / `"card"`；`animation`：`"slide"` / `"fade"`；`navigation.type`：`"dots"` / `"bars"` / `"dots-bar"`；`navigation.showSlideBtn`：`"always"` / `"never"` / `"hover"`
- 柱状图 `series.barType`：`"bar"`(普通柱) / `"pictorialBar"`(象形柱)；仅当 `"pictorialBar"` 时 `series.symbol` 生效，取值：`"circle"` / `"rect"` / `"roundRect"` / `"triangle"` / `"diamond"` / `"arrow"` / `"pin"`，或以 `"path://"` 开头的 SVG path 字符串（默认水滴形 path）
- 当前时间 `format`（**仅日期部分，下拉选项**）：`"yyyy-MM-dd"` / `"yyyy年MM月dd日"` / `"yyyy/MM/dd"` / `"yyyy.MM.dd"`；时间部分恒为 `hh:mm:ss`，由 `showTime` 决定是否拼接到日期后
- 当前时间 `hourType`：`1`(12小时制) / `2`(24小时制)；`dataType`：`0`(当前时间) / `1`(指定时间，取 `dataText.value` 时间戳)；`showDate` / `showTime` / `showWeek` 为布尔开关
- 装饰 `decorationType`（**下拉 12 选 1**）：`dv-decoration-1` ~ `dv-decoration-12`；仅 `7/9/11/12` 显示文字（`text`/`textColor`），`2/4/8` 支持竖放（reverse），`9/12` 带动画（dur）。都是横向为主的矢量装饰，宽度应明显大于高度（详见「装饰」条目）

### 组件实例 interaction 字段（每个组件都要带）

`.cd` 导入会**原样**载入 components，不像"拖拽新组件"那样自动补默认 interaction。为此每个组件实例的 `cptOption.interaction` 都不能缺，否则导入后点选组件会报错。写入规则：

- **可交互组件**（本目录下文 JSON 里**未**标注 `interaction` 的组件，如文本/按钮/图表/指标等）→ 携带完整默认交互对象：
  ```json
  { "active": false, "intType": "display",
    "objMap": { "default": { "show": [], "hidden": [], "changeovers": [], "drillDesignId": "", "designType": "screen", "redirectUrl": "", "redirectType": "_blank" } },
    "changeDataset": [], "paramType": "auto", "paramName": "", "paramValue": "" }
  ```
- **非交互组件**（本目录下文 JSON 里标注了 `"interaction": { "intType": "none" }` 的组件）→ **同样用完整默认对象，仅把 `intType` 改为 `"none"`**。目录示例里的 `"intType": "none"` 是**简写**，落盘时必须展开为完整默认对象——不能只写 `{ "intType": "none" }`，否则缺 `objMap`，导入后点选该组件会报错。
- **多选项交互组件**（选项卡 `cpt-tab`、导航器 `cpt-navigator`、跳转 `cpt-jumper` 等，目录里简写为 `interaction:{multi:true}`）→ 用**完整默认对象 + `"multi": true`** 写入，不要只写 `{multi:true}`。

## 1. 组件目录总表

分组与默认尺寸（生成时可按布局自行覆盖尺寸）：

| cptKey | 中文名 | 分组 | cptOptionKey | 默认尺寸 W×H |
|--------|--------|------|--------------|-------------|
| cpt-dataV-border | 边框 | 基础组件 | cpt-dataV-border-option | 400×300 |
| cpt-icon | 图标 | 基础组件 | cpt-icon-option | 120×120 |
| cpt-button | 按钮 | 基础组件 | cpt-button-option | 150×40 |
| cpt-text | 文本 | 基础组件 | cpt-text-option | 150×40 |
| cpt-scroll-text | 滚动文字 | 基础组件 | cpt-scroll-text-option | 250×50 |
| cpt-image | 图片 | 基础组件 | cpt-image-option | 400×300 |
| cpt-carousel | 轮播图 | 基础组件 | cpt-carousel-option | 400×300 |
| cpt-dataV-decoration | 装饰 | 基础组件 | cpt-dataV-decoration-option | 400×300 |
| cpt-iframe | iframe | 基础组件 | cpt-iframe-option | 400×300 |
| cpt-table | 静态表格 | 基础组件 | cpt-table-option | 600×300 |
| cpt-scroll-table | 滚动表格 | 基础组件 | cpt-scroll-table-option | 400×300 |
| cpt-scroll-list | 滚动列表 | 基础组件 | cpt-scroll-list-option | 400×300 |
| cpt-datetime | 当前时间 | 基础组件 | cpt-datetime-option | 250×50 |
| cpt-video-player | 视频 | 基础组件 | cpt-video-player-option | 400×300 |
| cpt-audio | 音频 | 基础组件 | cpt-audio-option | 150×40 |
| cpt-weather | 天气 | 基础组件 | cpt-weather-option | 210×140 |
| cpt-tab | 选项卡 | 控件 | cpt-tab-option | 500×40 |
| cpt-input | 输入框 | 控件 | cpt-input-option | 300×50 |
| cpt-select | 下拉框 | 控件 | cpt-select-option | 300×50 |
| cpt-date-picker | 日期选择 | 控件 | cpt-date-picker-option | 300×50 |
| cpt-jumper | 跳转 | 控件 | cpt-jumper-option | 300×50 |
| cpt-navigator | 导航器 | 控件 | cpt-navigator-option | 400×20 |
| cpt-tree-select | 树形选择 | 控件 | cpt-tree-select-option | 400×50 |
| cpt-num | 数值文本 | 指标卡 | cpt-num-option | 200×80 |
| cpt-quota | 指标卡 | 指标卡 | cpt-quota-option | 350×150 |
| cpt-icon-quota | 统计指标 | 指标卡 | cpt-icon-quota-option | 140×170 |
| cpt-rect-num | 数字翻牌器 | 指标卡 | cpt-rect-num-option | 350×150 |
| cpt-time-run | 计时器 | 指标卡 | cpt-time-run-option | 500×80 |
| cpt-indicator | 增长指标 | 指标卡 | cpt-indicator-option | 500×80 |
| cpt-quota-progress | 进度指标 | 指标卡 | cpt-quota-progress-option | 250×120 |
| cpt-link-card | 链接卡片 | 指标卡 | cpt-link-card-option | 400×160 |
| cpt-vue-data-ui (VueUiSparkline) | 趋势指标 | 指标卡 | cpt-dataui-sparkline-option | 300×140 |
| cpt-vue-data-ui (VueUiGizmo) | 电池 | 指标卡 | cpt-dataui-gizmo-option | 300×140 |
| cpt-vue-data-ui (VueUiKpi) | KPI | 指标卡 | cpt-dataui-kpi-option | 300×140 |
| cpt-rate | 评分 | 指标卡 | cpt-rate-option | 300×100 |
| cpt-chart-column | 基础柱状图 | 柱状图 | cpt-chart-column-option | 400×300 |
| cpt-chart-column（单列/横向/旋风等派生） | 柱状图变体 | 柱状图 | cpt-chart-column-option | 见注 |
| cpt-chart-column-stack | 堆积柱状图 | 柱状图 | cpt-chart-column-option | 400×300 |
| cpt-chart-column-region | 区间柱状图 | 柱状图 | cpt-chart-column-option | 400×300 |
| cpt-chart-column-stereo | 立体柱状图 | 柱状图 | cpt-chart-column-stereo-option | 400×300 |
| cpt-chart-column-3d | 3D柱状图 | 柱状图 | cpt-chart-column-3d-option | 500×350 |
| cpt-chart-scroll-list | 排行榜 | 柱状图 | cpt-chart-scroll-list-option | 400×300 |
| cpt-chart-pie | 基础饼图 | 饼/环图 | cpt-chart-pie-option | 400×300 |
| cpt-ring-pie | 同心环形图 | 饼/环图 | cpt-ring-pie-option | 400×300 |
| cpt-dataV-activeRing | 动态环形图 | 饼/环图 | cpt-dataV-activeRing-option | 400×300 |
| cpt-chart-polycyclic | 聚环图 | 饼/环图 | cpt-chart-polycyclic-option | 400×300 |
| cpt-chart-pie-3d | 3D饼图 | 饼/环图 | cpt-chart-pie-3d-option | 400×300 |
| cpt-dataV-waterLevel | 方形水位图 | 水位图 | cpt-dataV-waterLevel-option | 120×100 |
| cpt-water-prop | 水球图 | 水位图 | cpt-water-prop-option | 200×200 |
| cpt-chart-line | 基础折线图 | 折线/折柱图 | cpt-chart-line-option | 400×300 |
| cpt-chart-line-3d | 3D折线图 | 折线/折柱图 | cpt-chart-line-3d-option | 400×300 |
| cpt-chart-line-dual | 双Y轴折线柱图 | 折线/折柱图 | cpt-chart-line-dual-option | 600×350 |
| cpt-chart-line-multiple | 多Y轴折线柱图 | 折线/折柱图 | cpt-chart-line-multiple-option | 700×400 |
| cpt-chart-candlestick | 基础K线图 | k线/趋势 | cpt-chart-candlestick-option | 400×300 |
| cpt-chart-gauge | 仪表盘 | 仪表盘/进度 | cpt-chart-gauge-option | 300×200 |
| cpt-dataV-percentPond | 进度池 | 仪表盘/进度 | cpt-dataV-percentPond-option | 300×200 |
| cpt-chart-progress-ring | 环形进度(组) | 仪表盘/进度 | cpt-chart-progress-ring-option | 660×300 |
| cpt-chart-ring | 环形刻度图 | 仪表盘/进度 | cpt-chart-ring-option | 300×300 |
| cpt-chart-scatter | 散点图/气泡图 | 散点图 | cpt-chart-scatter-option | 500×400 |
| cpt-chart-radar | 雷达图 | 雷达图 | cpt-chart-radar-option | 400×300 |
| cpt-g6-dom | 关系图 | 关系图 | cpt-g6-dom-option | 400×300 |
| cpt-chart-funnel | 漏斗图 | 漏斗图 | cpt-chart-funnel-option | 400×300 |
| cpt-chart-map-gc | 渐变地图 | 地图 | cpt-chart-map-gc-option | 900×600 |
| cpt-chart-map-migrate | 飞线地图 | 地图 | cpt-chart-map-migrate-option | 900×600 |
| cpt-chart-map-3d | 3D地图 | 地图 | cpt-chart-map-3d-option | 900×600 |
| cpt-chart-map-3d-line | 3D飞线地图 | 地图 | cpt-chart-map-3d-line-option | 900×600 |
| cpt-threejs-dom | 模型 | 3D | cpt-threejs-dom-option | 900×600 |
| cpt-chart-wordcloud | 词云图 | 其它 | cpt-chart-wordcloud-option | 800×800 |
| cpt-img-quota | 告警图 | 其它 | cpt-img-quota-option | 140×170 |
| cpt-icon-ring | 线路状态 | 其它 | cpt-icon-ring-option | 200×200 |
| cpt-img-state | 状态图 | 其它 | cpt-img-state-option | 150×100 |
| cpt-html-viewer | HTML | 其它 | cpt-html-viewer-option | 300×250 |
| cpt-time-line | 时间轴 | 其它 | cpt-time-line-option | 680×200 |

> 柱状图/饼图/折线图存在若干**派生变体**，共用同一 `cptOptionKey` 与渲染组件，仅 `attribute` 个别字段不同（见第 8 节）。

---

## 2. 基础组件

### cpt-icon 图标（TDesign 内置图标）
无 dataText（纯配置）。渲染为 `t-icon`，`color` 图标颜色，图标字号跟随组件宽度。
```json
{ "attribute": { "name": "api", "color": "#0034b5" } }
```
> `name` 是 tdesign-vue-next 内置图标名（即 `t-icon` 的 `name` 值）。常用：`api`、`alarm`、`user`、`setting`、`money`、`edit`、`search`、`add`、`delete`、`chevron-right`、`close`、`help-circle`、`star`、`star-filled`、`dashboard`、`cloud`、`wifi`、`video`、`sound`。完整图标名见 TDesign 图标库 / 项目 `src/assets/icon/icon-manifest`；写错不报错，仅图标不显示。按钮的 `icon`、指标卡 `icon.name`、时间轴 `icon` 等其它字段同样是该 tdesign 图标名。

### cpt-text 文本
dataText 结构：`{"value":"文本内容"}`
```json
{
  "cptDataForm": { "dataText": "{\"value\":\"某市智慧城市运营中心\"}", "dataSource": 1, "pollTime": 0 },
  "attribute": {
    "textColor": "#ffffff", "textSize": 32,
    "enableGradual": 1, "gradualColor": ["#54f857", "#f31794"],
    "fontWeight": "bold", "textLineHeight": 0, "letterSpacing": 0,
    "textFamily": "微软雅黑", "textAlign": "center",
    "fontStyle": "normal", "textDecoration": "none", "bgColor": "rgba(255,255,255,0)",
    "borderRadius": 0, "borderStyle": "none", "borderWidth": 1, "borderColor": "#ccc",
    "alarmConfig": { "operator": ">", "threshold": "", "textColor": "#f00", "bgColor": "" }
  }
}
```
> `enableGradual`: 1=纯色文本（用 textColor），2=左右渐变，3=上下渐变（用 gradualColor）。渐变文本时 textColor/bgColor 设为 transparent。

### cpt-scroll-text 滚动文字
dataText：`{"value":"滚动公告内容"}`
```json
{ "cptDataForm": { "dataText": "{\"value\":\"重要通知：系统将于今晚 22:00 升级维护\"}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "textColor": "#ffffff", "fontSize": 18, "step": 2, "scrollType": 1 } }
```

### cpt-button 按钮
dataText：`{"value":"按钮文字"}`
```json
{ "cptDataForm": { "dataText": "{\"value\":\"进入系统\"}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "borderColor": "#0061C2", "bgColor": "#0061C2", "radius": 0,
    "theme": "primary", "textColor": "#ffffff", "icon": "api", "fontSize": 14,
    "variant": "base", "shape": "rectangle", "fullscreen": false } }
```

### cpt-image 图片
无 dataText（纯配置）。`attribute.url` 指向 `/file` 资源库路径，或留空。
```json
{ "attribute": { "url": "", "fit": "fill", "preview": false, "autoRotate": false, "rotateDuration": 2, "bgColor": "rgba(0,0,0,0)" } }
```
> 建议生成时 `url` 留空（需用户上传素材），`fit` 取 contain/cover。

### cpt-carousel 轮播图
dataText：`[{"name":"图片名","value":"图片url"}]`
```json
{ "cptDataForm": { "dataText": "[{\"name\":\"图1\",\"value\":\"\"}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "trigger": "hover", "fit": "contain", "imgUrls": [], "type": "default",
    "animation": "slide", "direction": "horizontal", "duration": 300, "interval": 2000,
    "navigation": { "placement": "inside", "showSlideBtn": "always", "size": "small", "type": "dots" },
    "titleSize": 14 },
  "interaction": { "intType": "none" } }
```

### cpt-dataV-border 边框（`dv-border-box-1` 等 DataV 边框盒）
无 dataText。用于**局部装饰**：框住某张图表/指标卡/分区，尺寸取「被框组件尺寸 + 边框留白」或铺满该分区，`borderTitle` 可填分区标题。**不要拿它铺满整屏当镜框**。
```json
{ "attribute": { "borderType": "dv-border-box-1", "borderColor1": "#0061C2", "borderColor2": "#409EFF",
    "backgroundColor": "rgba(0,0,0,0)", "borderTitle": "标题1", "titleWidth": 250,
    "dur": 3, "reverse": false } }
```

### cpt-dataV-decoration 装饰
无 dataText。`attribute.color1/color2` 为装饰主色（双色）。
```json
{ "attribute": { "decorationType": "dv-decoration-1", "color1": "#0061C2", "color2": "#409EFF",
    "text": "装饰文案", "textColor": "#dddddd" } }
```

> `decorationType` 是**下拉 12 选 1**（`dv-decoration-1` ~ `dv-decoration-12`），对应 DataV 矢量装饰，随组件宽高缩放。按适用尺寸分组：

| 分类 | 类型 | 适用尺寸建议 |
|------|------|-------------|
| 横向装饰条/线 | `1` / `3` / `5` / `6` / `10` | 横向长条，宽 200–600、高 20–80 |
| 可竖放装饰线 | `2` / `4` / `8` | 默认横向；做竖向分隔线时宽小高大（如 5×150） |
| 带文字标题装饰 | `7` / `9` / `11` / `12` | 标题条，宽 300–500、高 50–70，文字用 `text` 配置 |
| 带动画 | `9` / `12` | 标题类 + 动画，动画时长 `dur` |

- 仅 `7/9/11/12` 显示文字（`text` / 颜色3 `textColor`），其余类型忽略 `text`。
- 通用：这些是横向为主的矢量图形，**宽度要明显大于高度**（竖放用法除外），不要做成接近正方形或铺满大画布，否则图案变形。

### cpt-datetime 当前时间
dataText：`{"value":<时间戳ms>}`（1=当前时间场景忽略 value）
```json
{ "cptDataForm": { "dataText": "{\"value\":1672506061000}", "dataSource": 1, "pollTime": 0,
    "apiUrl": "/text", "sql": "" },
  "attribute": { "textColor": "#ffffff", "fontSize": 18, "fontWeight": "normal", "fontFamily": "微软雅黑",
    "dataType": 0, "format": "yyyy-MM-dd", "showWeek": false, "showDate": true, "showTime": true,
    "textAlign": "center", "hourType": 2 } }
```
> 枚举：`format` 只能取 `"yyyy-MM-dd"` / `"yyyy年MM月dd日"` / `"yyyy/MM/dd"` / `"yyyy.MM.dd"`（仅日期部分，时间恒为 `hh:mm:ss` 由 `showTime` 拼接）；`hourType` `1`=12小时 / `2`=24小时；`dataType` `0`=当前时间 / `1`=指定时间（取 `dataText.value` 时间戳）；`showDate`/`showTime`/`showWeek` 为布尔开关（true/false）。

### cpt-table 静态表格
dataText：`[{"colKey":"值",...}]` 数组；列定义在 `attribute.columns`。
```json
{ "cptDataForm": { "dataText": "[{\"name\":\"张三\",\"age\":18,\"sex\":\"男\",\"addr\":\"北京市海淀区\"}]", "dataSource": 1, "pollTime": 0 },
  "interaction": { "intType": "none" },
  "attribute": { "bordered": true, "textAlign": "left", "lineHeight": 20,
    "theadBg": "#0061C2", "theadColor": "#ffffff", "theadSize": 13,
    "tbodyColor": "#dddddd", "tbodySize": 13, "oddRowBg": "#00000000", "evenRowBg": "#0061C2", "pageSize": 5,
    "columns": [
      { "colKey": "name", "title": "姓名", "width": 0, "id": "c1" },
      { "colKey": "age", "title": "年龄", "width": 0, "id": "c2" },
      { "colKey": "sex", "title": "性别", "width": 0, "id": "c3" },
      { "colKey": "addr", "title": "地址", "width": 200, "id": "c4" } ] } }
```
> 生成时 `columns[i].colKey` 须与 dataText 每行对象的 key 一致；`id` 用唯一短串（如 c1/c2…）。

### cpt-scroll-table 滚动表格
dataText：对象数组；列定义在 columns，`type` 支持 `text` / `img`。
```json
{ "cptDataForm": { "dataText": "[{\"dev\":\"节点1\",\"ip\":\"10.0.0.1\",\"status\":\"正常\"}]", "dataSource": 1, "pollTime": 0 },
  "interaction": { "intType": "none" },
  "attribute": { "theadBg": ["#0061C2", "#409EFF"], "theadHeight": 40, "theadColor": "#ffffff", "theadSize": 14,
    "tbodyColor": "#ffffff", "tbodySize": 13, "oddRowBg": "#0061C2", "evenRowBg": "#409EFF",
    "showLine": 3, "showIndex": false, "borderColor": "#aaa", "borderType": "",
    "columns": [
      { "colKey": "dev", "title": "设备名称", "type": "text", "width": 0, "id": "c1" },
      { "colKey": "ip", "title": "IP地址", "type": "text", "width": 0, "id": "c2" },
      { "colKey": "status", "title": "状态", "type": "text", "width": 0, "id": "c3" } ] } }
```

### cpt-scroll-list 滚动列表（消息/告警滚动）
dataText：`[{"value":"单条文本"},...]`
```json
{ "cptDataForm": { "dataText": "[{\"value\":\"16:34 张三登录系统\"},{\"value\":\"17:16 订单支付 78 元\"}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "showLine": 3, "color": "#ffffff", "color2": "#ffffff", "fontSize": 13, "interval": 2 },
  "interaction": { "intType": "none" } }
```

### cpt-video-player 视频
dataText：`[{"name":"","value":""}]`（可选数据源）。
```json
{ "attribute": { "loop": true, "autoplay": true, "muted": true, "source": 1, "fileUrl": "",
    "url": "https://视频地址.mp4",
    "filter": { "enable": false, "blur": 0, "brightness": 1, "contrast": 100, "grayscale": 0,
      "hueRotate": 0, "invert": 0, "opacity": 100, "saturate": 100, "sepia": 0 } },
  "interaction": { "intType": "none" },
  "cptDataForm": { "dataText": "[{\"name\":\"视频1\",\"value\":\"\"}]", "dataSource": 1, "pollTime": 0 } }
```

### cpt-audio 音频
```json
{ "cptDataForm": { "dataText": "{\"value\":60,\"audioText\":\"自定义文本告警\"}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "audioSrc": "/audio/Bottle.ogg", "loop": true, "loopCount": 3,
    "alarmConfig": { "operator": ">", "threshold": null } },
  "interaction": { "intType": "none" } }
```

### cpt-weather 天气（心知天气）
无 dataText。纯配置。
```json
{ "attribute": { "cityName": "北京", "refreshFlag": false, "apiKey": "需申请",
    "cityColor": "#ffffff", "temperatureColor1": "#3498db", "temperatureColor2": "#2c3e50",
    "conditionColor": "#666666" }, "interaction": { "intType": "none" } }
```

---

## 3. 指标卡组件

### cpt-num 数值文本
dataText：`{"value":"数值","unit":"单位"}`
```json
{ "cptDataForm": { "dataText": "{\"value\":\"275.39\",\"unit\":\"Kb/s\"}", "dataSource": 1, "pollTime": 0, "apiUrl": "/text" },
  "attribute": { "title": "当月公网流出总流量", "numColor": "#31adfb", "numSize": 20, "numHeight": 30,
    "labelColor": "#dddddd", "labelSize": 13, "fontFamily": "CAI978",
    "alarmConfig": { "operator": ">", "threshold": "", "color": "#f00" } } }
```

### cpt-quota 指标卡（数值 + 图标 + 背景）
dataText：`{"value":"数值"}`
```json
{ "cptDataForm": { "dataText": "{\"value\":\"14039832\"}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "bgColor": ["#0061C2", "#409EFF"], "borderRadius": 4,
    "title": { "text": "指标描述", "fontSize": 18, "color": "#a9a9a9" },
    "num": { "fontSize": 40, "color": "#DEEEFF", "fontFamily": "Arial", "fontWeight": "normal" },
    "icon": { "size": 80, "name": "money", "color": "#ffffff" } } }
```

### cpt-indicator 增长指标（标题 + 数值 + 同比/环比）
dataText：`{"value":数值,"growth":增长率}`
```json
{ "cptDataForm": { "dataText": "{\"value\":15685,\"growth\":0.0125}", "dataSource": 1, "pollTime": 0 },
  "attribute": {
    "icon": { "url": "", "padding": 8 },
    "title": { "fontSize": 14, "color": "#ffffff", "fontFamily": "", "text": "一季度GDP", "fontWeight": "normal" },
    "num": { "fontSize": 18, "color": "#57b1ea", "fontFamily": "lccd", "unit": "万元", "fontWeight": "normal" },
    "grow": { "show": true, "label": "同比", "fontSize": 14, "color": "#7ccc04", "fontFamily": "" },
    "alarmConfig": { "operator": ">", "threshold": "", "textColor": "#f00" } },
  "interaction": { "intType": "none" } }
```

### cpt-rect-num 数字翻牌器
dataText：`{"value":"数值"}`
```json
{ "cptDataForm": { "dataText": "{\"value\":\"1920\"}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "fontSize": 50, "padding": 10, "bgColor": "#409EFF", "color": "#dddddd", "borderRadius": 0,
    "fontFamily": "lccd", "showSeparator": false, "changeType": "all" },
  "interaction": { "intType": "none" } }
```

### cpt-time-run 计时器（倒计时/正计时）
无 dataText。`targetTime` 目标时间，`direction` 1 正计时 / -1 倒计时。
```json
{ "attribute": { "num": { "color": "#31adfb", "fontSize": 50, "fontFamily": "lccd" },
    "label": { "color": "#dddddd", "fontSize": 16, "align": "top" },
    "direction": 1, "targetTime": "2026-12-31 23:59:59", "accuracy": 5, "showYear": true } }
```

### cpt-quota-progress 进度指标（带图标/进度环）
dataText：`{"value":60,"total":100}`
```json
{ "cptDataForm": { "dataText": "{\"value\":60,\"total\":100}", "dataSource": 1, "pollTime": 0 },
  "attribute": {
    "border": { "show": true, "radius": 2, "color": "#00F0FF" },
    "icon": { "url": "", "autoRotate": false },
    "padding": { "top": 10, "right": 10, "bottom": 10, "left": 10 },
    "label": { "totalLabel": "总数", "valueLabel": "已使用", "color": "#dddddd", "fontSize": 13, "fontFamily": "" },
    "value": { "color": "#3c8fe3", "fontSize": 16, "fontFamily": "", "position": "progress" },
    "progress": { "color": ["#f00", "#ff0"], "bgColor": "#12365b", "size": 112 } } }
```

### cpt-link-card 链接卡片
无 dataText。
```json
{ "attribute": { "bgColor": "#ffffff", "textColor": "#000000", "radius": 4, "icon": null,
    "fontSize": 14, "title": "监控视图", "iconSize": 90, "iconBoxLeft": 50 } }
```

### cpt-vue-data-ui (VueUiSparkline) 趋势指标
dataText：`[{"name":"标签","value":数值},...]`
```json
{ "cptDataForm": { "dataText": "[{\"name\":\"1月\",\"value\":12},{\"name\":\"2月\",\"value\":18}]", "dataSource": 1, "pollTime": 0, "apiUrl": "/text" },
  "attribute": { "component": "VueUiSparkline", "refreshKey": "0", "type": "line",
    "style": { "backgroundColor": "rgba(0,0,0,0)", "chartWidth": 290,
      "line": { "color": "#5f8bee", "strokeWidth": 3, "smooth": true },
      "bar": { "borderRadius": 3, "color": "#5f8bee" },
      "zeroLine": { "color": "#505050", "strokeWidth": 1 },
      "dataLabel": { "show": true, "offsetX": 0, "offsetY": 0, "position": "left", "fontSize": 48, "bold": true,
        "color": "#CCCCCC", "roundingValue": 1, "valueType": "latest", "prefix": "", "suffix": "" },
      "title": { "show": true, "textAlign": "left", "color": "#FAFAFA", "fontSize": 18, "bold": true, "text": "趋势指标" },
      "area": { "show": true, "useGradient": true, "opacity": 30, "color": "#5f8bee" } } } }
```

### cpt-rate 评分
dataText：`{"value":3.5}`
```json
{ "cptDataForm": { "dataText": "{\"value\":3.5}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "allowHalf": true, "disabled": true, "color": ["#ED7B2F", "#f5cdb2"], "count": 5, "gap": 4,
    "icon": "star-filled", "showText": false, "size": 30,
    "texts": ["极差", "失望", "一般", "满意", "惊喜"], "defaultValue": 4.5 } }
```

---

## 4. 柱状图 / 折线图类

### cpt-chart-column 基础柱状图（也可作横向/单列/旋风，见第 8 节）
dataText：多系列 `[{"group":"系列名","data":[{"name":"类目","value":数值},...]},...]`
```json
{
  "cptDataForm": { "dataText": "[{\"group\":\"广州机房\",\"data\":[{\"name\":\"Mon\",\"value\":120},{\"name\":\"Tue\",\"value\":200}]},{\"group\":\"深圳机房\",\"data\":[{\"name\":\"Mon\",\"value\":100},{\"name\":\"Tue\",\"value\":120}]}]", "dataSource": 1, "pollTime": 0 },
  "attribute": {
    "direction": "x",
    "color": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"],
    "xAxis": { "show": true, "name": "", "axisLabel": { "show": true, "color": "#eeeeee", "fontSize": 13, "rotate": 0, "interval": "", "formatter": "{value}" },
      "axisLine": { "show": true, "lineStyle": { "color": "#eeeeee" } }, "axisTick": { "show": true },
      "splitLine": { "show": false } },
    "yAxis": { "show": true, "name": "", "type": "value", "axisLabel": { "show": true, "color": "#eeeeee", "fontSize": 13, "formatter": "{value}" },
      "axisLine": { "show": true, "lineStyle": { "color": "#eeeeee" } }, "axisTick": { "show": true },
      "splitLine": { "show": false, "lineStyle": { "color": ["#aaa", "#ddd"] } } },
    "title": { "text": "一天用电量分布", "left": "center", "top": "top",
      "textStyle": { "fontSize": 18, "color": "#cccccc" }, "subtextStyle": { "fontSize": 12, "color": "#aaaaaa" } },
    "legend": { "show": true, "left": "center", "top": "bottom", "orient": "horizontal",
      "textStyle": { "color": "#ffffff", "fontSize": 14 } },
    "grid": { "x": 10, "y": 30, "x2": 30, "y2": 30, "containLabel": true },
    "series": { "showBackground": false, "borderRadius": 0, "barType": "bar",
      "symbol": "path://M0,10 L10,10 C5.5,10 5.5,5 5,0 C4.5,5 4.5,10 0,10 z",
      "label": { "show": true, "position": "top", "color": "#dddddd", "fontSize": 13, "unit": "" }, "barWidth": 18 },
    "barGap": 0.1
  }
}
```
> `direction`: "x"=竖向柱 / "y"=横向柱。横向时交换 x/y 数据含义，并把 label.position 设 "right"、barWidth 调小。
> `series.barType`: "bar" 普通柱 / "pictorialBar" 象形柱；象形柱时 `series.symbol` 选择柱体形状（circle/rect/roundRect/triangle/diamond/arrow/pin 或 `path://...`，枚举见「通用取值枚举」）。

### cpt-chart-line 基础折线图
dataText：同柱状图多系列结构（也用 `[{"group","data":[{"name","value"}]}]`）。
```json
{
  "cptDataForm": { "dataText": "[{\"group\":\"广州机房\",\"data\":[{\"name\":\"Mon\",\"value\":120},{\"name\":\"Tue\",\"value\":200}]}]", "dataSource": 1, "pollTime": 0 },
  "attribute": {
    "smooth": false, "areaShow": false,
    "lineColors": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"],
    "areaColors": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"],
    "xAxis": { "show": true, "name": "", "axisLabel": { "show": true, "color": "#eeeeee", "fontSize": 12, "rotate": 0 },
      "axisLine": { "show": true, "lineStyle": { "color": "#eeeeee" } }, "axisTick": { "show": true },
      "splitLine": { "show": false }, "boundaryGap": true, "formatter": "{value}" },
    "yAxis": { "show": true, "type": "value", "name": "", "axisLabel": { "show": true, "color": "#eeeeee", "fontSize": 13 },
      "axisLine": { "show": true, "lineStyle": { "color": "#eeeeee" } }, "axisTick": { "show": true },
      "splitLine": { "show": false, "lineStyle": { "color": ["#aaa", "#ddd"] } }, "formatter": "{value}" },
    "title": { "text": "一天用电量分布", "left": "center", "top": "top",
      "textStyle": { "fontSize": 18, "color": "#cccccc" }, "subtextStyle": { "fontSize": 12, "color": "#aaaaaa" } },
    "legend": { "show": true, "left": "center", "top": "bottom", "orient": "horizontal",
      "textStyle": { "color": "#ffffff", "fontSize": 14 } },
    "grid": { "x": 10, "y": 30, "x2": 10, "y2": 30 },
    "series": { "label": { "show": true, "position": "top", "color": "#dddddd", "fontSize": 13 } },
    "lineShow": true, "symbolSize": 4
  }
}
```
> 面积图派生：`areaShow:true`；散点折线派生：`lineShow:false, symbolSize:16`。

### cpt-chart-column-3d 3D柱状图
dataText：单系列 `[{"name":"类目","value":数值},...]`
```json
{
  "cptDataForm": { "dataText": "[{\"name\":\"Mon\",\"value\":120},{\"name\":\"Tue\",\"value\":200}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "color": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"],
    "xAxis": { "name": " ", "type": "category", "axisLabel": { "color": "#eeeeee", "fontSize": 13 },
      "axisLine": { "lineStyle": { "color": "#eeeeee" } }, "splitLine": { "show": true, "lineStyle": { "color": "#dddddd" } } },
    "yAxis": { "name": " ", "type": "value", "axisLabel": { "textStyle": { "color": "#eeeeee", "fontSize": 13 } },
      "axisLine": { "lineStyle": { "color": "#eeeeee" } }, "splitLine": { "show": true, "lineStyle": { "color": "#dddddd" } } },
    "title": { "text": "一天用电量分布", "left": "center", "top": "top",
      "textStyle": { "fontSize": 18, "color": "#cccccc" }, "subtextStyle": { "fontSize": 12, "color": "#aaaaaa" } },
    "boxDepth": 20,
    "series": { "label": { "show": true, "color": "#dddddd", "fontSize": 13 } } }
}
```

### cpt-chart-scroll-list 排行榜（横向条形滚动）
dataText：`[{"name":"名称","value":数值},...]`（value 越大排序越前，`sort:"desc"`）
```json
{ "cptDataForm": { "dataText": "[{\"name\":\"北京\",\"value\":120},{\"name\":\"上海\",\"value\":200}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "chartTitle": "销售排行", "titleLeft": "center", "titleTop": "10", "titleTextColor": "#cccccc",
    "xAxisShow": false, "xLineShow": true, "titleSize": 18, "yLabelSize": 13,
    "xLabelColor": "#cccccc", "xLineColor": "#cccccc", "yLabelColor": "#ffffff",
    "yGridLineShow": false, "yTickShow": true, "xTickShow": true, "xGridLineShow": false,
    "barBorderRadius": 5, "barLabelShow": true, "barBgShow": true,
    "barLabelColor": "#cccccc", "barLabelSize": 10, "barLabelPosition": "right", "barLabelUnit": "",
    "barColor": ["#0061C2", "#409EFF"], "barWidth": 8, "sort": "desc",
    "gridX": 10, "gridY": 30, "gridX2": 40, "gridY2": 0,
    "enableScroll": true, "showLine": 5, "scrollType": "one", "scrollTime": 3 } }
```

---

## 5. 饼图 / 环图

### cpt-chart-pie 基础饼图
dataText：`[{"value":数值,"name":"名称"},...]`
```json
{
  "cptDataForm": { "dataText": "[{\"value\":1048,\"name\":\"搜索引擎\"},{\"value\":735,\"name\":\"直接访问\"}]", "dataSource": 1, "pollTime": 0 },
  "attribute": {
    "color": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"],
    "title": { "text": "占比分布", "left": "40", "top": "center", "textStyle": { "fontSize": 18, "color": "#cccccc" } },
    "legend": { "show": true, "orient": "horizontal", "x": "center", "y": "bottom",
      "textStyle": { "color": "#dddddd", "fontSize": 12 } },
    "series": { "roseType": "false", "radius": [0, 60], "center": [50, 50],
      "label": { "show": true, "position": "outside", "fontSize": 13, "color": "#dddddd",
        "formatter": "{b}-{c}({d}%)", "backgroundColor": null, "borderColor": null, "borderWidth": 0, "borderRadius": 0 },
      "itemStyle": { "borderRadius": 0 }, "startAngle": 0, "endAngle": 360 },
    "label": {}
  }
}
```
> 派生：环形图 `series.radius:[40,60]`；玫瑰图 `series.roseType:"radius"`；扇形饼图 `startAngle:180,endAngle:360,center:[50,65],itemStyle.borderRadius:20`。

### cpt-ring-pie 同心环形图
dataText：`[{"value":数值,"name":"名称"},...]`
```json
{ "cptDataForm": { "dataText": "[{\"value\":1048,\"name\":\"搜索引擎\"},{\"value\":735,\"name\":\"直接访问\"}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "borderColor": "#252525", "borderWidth": 0, "padAngle": 4,
    "labelLine": { "length": 10, "length2": 6 },
    "label": { "show": true, "fontSize": 11, "color": "#ffffff", "formatter": "{b}\\n{c}\\n{d}%" },
    "pieColor": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"] } }
```

### cpt-dataV-activeRing 动态环形图
```json
{ "cptDataForm": { "dataText": "[{\"name\":\"周口\",\"value\":55},{\"name\":\"南阳\",\"value\":120}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "lineWidth": 10, "radius": 80, "activeRadius": 60, "showOriginValue": false,
    "color": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"] },
  "interaction": { "intType": "none" } }
```

### cpt-chart-pie-3d 3D饼图
```json
{ "cptDataForm": { "dataText": "[{\"value\":1048,\"name\":\"搜索引擎\"},{\"value\":735,\"name\":\"直接访问\"}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "color": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"],
    "title": { "text": "占比分布", "left": "40", "top": "center", "textStyle": { "fontSize": 18, "color": "#cccccc" } },
    "legend": { "show": true, "orient": "horizontal", "x": "center", "y": "bottom",
      "textStyle": { "color": "#dddddd", "fontSize": 12 } },
    "grid3D": { "show": false, "boxHeight": 12, "top": 0,
      "viewControl": { "distance": 300, "alpha": 25, "beta": 130, "autoRotate": true, "autoRotateSpeed": 20 } },
    "series": { "roseType": false, "radius": 0.5 } } }
```

---

## 6. 水位图 / 仪表盘 / 进度

### cpt-dataV-waterLevel 方形水位图
dataText：`[数值]`（数组，单值）
```json
{ "cptDataForm": { "dataText": "[55]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "colors": ["#0061C2", "#409EFF"], "waveNum": 3, "waveHeight": 40, "waveOpacity": 0.4,
    "formatter": "{value}%", "shape": "rect" } }
```

### cpt-water-prop 水球图
dataText：`{"value":50}`
```json
{ "cptDataForm": { "dataText": "{\"value\":50}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "amplitude": 10, "shape": "circle", "waveNum": 2, "radius": 90,
    "label": { "fontSize": 16, "color": "#abfff9", "insideColor": "#ffffff", "unit": "%" },
    "outline": { "borderDistance": 0, "borderWidth": 6, "borderColorTop": "rgba(69,73,240,0)",
      "borderColorBottom": "rgba(69,73,240,1)", "shadowColor": "#000" },
    "colorTop": "#f46bf5", "colorBottom": "#1ca3e2",
    "backgroundColorTop": "rgba(68,145,253,0)", "backgroundColorBottom": "rgba(68,145,253,1)" },
  "interaction": { "intType": "none" } }
```

### cpt-chart-gauge 仪表盘
dataText：`{"value":22}`
```json
{ "cptDataForm": { "dataText": "{\"value\":22}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "title": "速度", "titleSize": 12, "titleColor": "#dddddd", "titleOffsetTop": "top",
    "min": 0, "max": 100, "detailSize": 25, "detailColor": "#dddddd", "detailOffsetTop": 40,
    "lineWidth": 3, "color1": "#409EFF", "color2": "#31adfb", "color3": "#8FC7FF", "itemColor": "#8FC7FF",
    "pointerOffsetTop": 0, "radius": 100, "startAngle": 225, "endAngle": -45,
    "axisLabel": { "show": true, "distance": 10, "color": "#999999", "fontSize": 12 },
    "axisTick": { "show": true, "length": 5, "distance": 5, "lineStyle": { "width": 1, "color": "#cccccc" } },
    "pointer": { "length": 75, "width": 6, "itemStyle": { "color": "#8FC7FF" } },
    "enableGradual": false } }
```

### cpt-dataV-percentPond 进度池
dataText：`{"value":66}`
```json
{ "cptDataForm": { "dataText": "{\"value\":66}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "borderWidth": 2, "borderRadius": 4, "borderGap": 3, "lineWidth": 3, "lineSpace": 2,
    "localGradient": true, "colors": ["#0061C2", "#409EFF"], "numShow": true } }
```

### cpt-chart-progress-ring 环形进度(组)
dataText：`[{"name":"CPU","value":90},...]`
```json
{ "cptDataForm": { "dataText": "[{\"name\":\"CPU\",\"value\":90},{\"name\":\"内存\",\"value\":60}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "chartWidth": 220, "chartHeight": 200, "title": "速度", "titleSize": 16, "titleColor": "#ffffff",
    "numColor": "#31adfb", "numSize": 50, "format": "{value}%", "progressColor": "#31adfb", "progressBg": "#00000030",
    "roseType": "false", "borderRadius": 5, "progressWidth": 74, "borderColor": "#00000000", "innerRingShow": false,
    "alarmConfig": { "color": "#FF0000", "operator": ">", "threshold": null } } }
```

### cpt-chart-ring 环形刻度图
dataText：`{"value":90}`
```json
{ "cptDataForm": { "dataText": "{\"value\":90}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "title": { "text": "Title", "top": 56, "color": "#f1f1f1", "fontSize": 20 },
    "num": { "top": 38, "color": "#0061C2", "fontSize": 40, "suffix": "%" },
    "bar": { "barWidth": 20, "color1": "#0061C2", "color2": "#31adfb", "background": "rgba(64,158,255,0.1)", "radius1": 10, "radius2": 100 },
    "gauge": { "radius": 85, "splitNumber": 90, "length": 6, "width": 1, "color": "#aaa" },
    "ring": { "color1": "rgba(64,158,255,0.4)", "color2": "rgba(64,158,255,0.3)", "color3": "rgba(64,158,255,0.2)", "color4": "rgba(64,158,255,0.1)" },
    "alarmConfig": { "color1": "#FF0000", "color2": "#701717", "operator": ">", "threshold": null } } }
```

---

## 7. 地图 / 其它图表

### 地图组件通用：地图源与省市 code

四个地图组件（渐变地图 `cpt-chart-map-gc`、飞线地图 `cpt-chart-map-migrate`、3D地图 `cpt-chart-map-3d`、3D飞线地图 `cpt-chart-map-3d-line`）都用 `attribute.provinceCode` + `attribute.cityCode` 决定渲染哪个区域：

- 取值：6 位行政区划 adcode（**字符串**），或 `"china"`（全国）、`"word"`（世界）。
- 生效优先级：`cityCode` 非空用 `cityCode`，否则用 `provinceCode`，再否则 `china`（默认 `provinceCode:'china', cityCode:''`）。
- **geojson 来源**：`"china"` 全国地图 geojson 由**前端内置**；其余省/市（含 `"word"` 世界）地图 geojson 由**后端接口按 code 从 `design_geo_data` 表读取**——产品已内置全国 34 个省级 + 全部地级市 geojson，生成时直接用 adcode 指定即可，无需自备资源。
- 省级 code 为 6 位且以 `0000` 结尾；地级市 code 为 6 位（父级 pointer 指向省级 code）。
- `enableDrillDown`（渐变地图/3D地图支持）：开启后省图可下钻到市，下钻目标由数据里的 `cityCode` 决定。
- **`dataText` 的 `name` 必须与对应地图 geojson 的 `properties.name`（即 `design_geo_data.city_name`）一致**：全国地图用标准省份名（样例「广东」「北京」「南海诸岛」）；省/市地图用该区域下辖的市/区县全称。

省级 adcode 一览（`provinceCode` 可选值）：

```text
china 中国（全国，前端内置） | word 世界（后端内置）
110000 北京市   120000 天津市   130000 河北省   140000 山西省   150000 内蒙古自治区
210000 辽宁省   220000 吉林省   230000 黑龙江省
310000 上海市   320000 江苏省   330000 浙江省   340000 安徽省   350000 福建省
360000 江西省   370000 山东省
410000 河南省   420000 湖北省   430000 湖南省   440000 广东省   450000 广西壮族自治区
460000 海南省
500000 重庆市   510000 四川省   520000 贵州省   530000 云南省   540000 西藏自治区
610000 陕西省   620000 甘肃省   630000 青海省   640000 宁夏回族自治区   650000 新疆维吾尔自治区
710000 台湾省   810000 香港特别行政区   820000 澳门特别行政区
```

地级市 code 示例（`cityCode`，父级为其省级 code）：广州市 `440100`、深圳市 `440300`、珠海市 `440400`（父 440000 广东）；成都市 `510100`（父 510000 四川）；杭州市 `330100`（父 330000 浙江）；苏州市 `320500`、南京市 `320100`（父 320000 江苏）；武汉市 `420100`（父 420000 湖北）；西安市 `610100`（父 610000 陕西）。直辖市下辖用 `110100`(北京)/`310100`(上海)/`500100`(重庆)。完整省市 code 以后端 `design_geo_data` 表为准。

**全国地图（`provinceCode:'china'`）的 dataText `name` 用省份简称**（与前端 china geojson 的 `properties.name` 一致，共 35 个）：

```text
南海诸岛, 北京, 天津, 上海, 重庆, 河北, 河南, 云南, 辽宁, 黑龙江, 湖南, 安徽, 山东,
新疆, 江苏, 浙江, 江西, 湖北, 广西, 甘肃, 山西, 内蒙古, 陕西, 吉林, 福建, 贵州, 广东,
青海, 西藏, 四川, 宁夏, 海南, 台湾, 香港, 澳门
```

> ⚠️ 易混点：`provinceCode / cityCode` 用 **6 位 adcode 数字串**（如 `440000`）；`dataText[].name` 用 **中文简称**（如「广东」）。两者别混用。

### cpt-chart-map-gc 渐变地图（中国地图 choropleth）
dataText：`[{"name":"省/市名","value":数值},...]`（名称须与 china map GeoJSON 的省份名一致，含「北京」「天津」等直辖市及「南海诸岛」）
```json
{ "cptDataForm": { "dataText": "[{\"name\":\"广东\",\"value\":98},{\"name\":\"浙江\",\"value\":104},{\"name\":\"湖北\",\"value\":1052}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "roam": false, "titleText": "肺炎地图", "titleLeft": "center", "titleTop": "top",
    "subtext": "数据纯属虚构", "titleFontSize": 18, "titleColor": "#CCCCCC",
    "subTitleColor": "#aaaaaa", "subTitleFontSize": 13,
    "tipFormatter": "确诊病例<br/>${name}：${value}", "geoLabelColor": "#555555", "geoLabelSize": 10, "borderColor": "#666666",
    "visualMap": { "show": true, "min": 0, "max": 100, "left": 20, "top": "bottom",
      "text": ["高", "低"], "textStyle": { "color": "#dddddd" },
      "pieces": [ { "gte": 100, "label": "> 100 人" }, { "gte": 10, "lt": 100, "label": "10 - 100 人" },
        { "gte": 1, "lt": 10, "label": "1 - 9 人" }, { "gte": 0, "lt": 1, "label": "无" } ] },
    "provinceCode": "china", "cityCode": "", "enableDrillDown": false,
    "color": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"], "areaColor": "#988205",
    "emphasis": { "label": { "show": true, "color": "#ffffff", "fontSize": 14 },
      "itemStyle": { "areaColor": "#797312", "borderColor": "#000000", "borderWidth": 1, "opacity": 0.8 } } } }
```

### cpt-chart-scatter 气泡图
dataText：多系列 `[{"group":"系列","data":[[x,y,size,name],...]},...]`（四元数组）
```json
{ "cptDataForm": { "dataText": "[{\"group\":\"2015年\",\"data\":[[44056,81.8,239,\"Australia\"],[43294,81.7,359,\"Canada\"]]}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "scale": 0.1, "color": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"],
    "title": { "text": "标题", "left": "10", "top": "top", "textStyle": { "fontSize": 18, "color": "#cccccc" } },
    "legend": { "left": "right", "top": "10", "textStyle": { "color": "#dddddd", "fontSize": 13 } },
    "grid": { "left": "8%", "top": "12%", "bottom": 10, "containLabel": true },
    "xAxis": { "axisLabel": { "show": true, "color": "#eeeeee", "fontSize": 13, "rotate": 0, "formatter": "{value}" },
      "axisLine": { "show": true, "lineStyle": { "color": "#eeeeee" } }, "axisTick": { "show": true },
      "splitLine": { "show": true, "lineStyle": { "type": "dashed" } } },
    "yAxis": { "type": "value", "axisLabel": { "show": true, "color": "#eeeeee" },
      "axisLine": { "show": true, "lineStyle": { "color": "#eeeeee", "fontSize": 13, "formatter": "{value}" } },
      "axisTick": { "show": true }, "splitLine": { "show": true, "lineStyle": { "type": "dashed" } }, "scale": true },
    "xTip": "<br/>X：", "yTip": "<br/>Y：", "areaTip": "<br/>面积：" } }
```

### cpt-chart-radar 雷达图
dataText：`[{"data":{"维度key":数值,...},"group":"系列名"},...]`
```json
{ "cptDataForm": { "dataText": "[{\"data\":{\"li\":4200,\"speed\":12000,\"gong\":20000,\"fang\":35000,\"kill\":50000,\"life\":18000},\"group\":\"张三\"}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "colors": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"],
    "title": { "text": "能力分布", "left": "right", "top": "top", "textStyle": { "fontSize": 18, "color": "#cccccc" },
      "subtextStyle": { "fontSize": 12, "color": "#aaaaaa" } },
    "legend": { "show": true, "left": 10, "top": 0, "orient": "horizontal", "textStyle": { "color": "#dddddd", "fontSize": 14 } },
    "radar": { "axisName": { "color": "#dddddd", "fontSize": 14 }, "shape": "polygon",
      "indicator": [ { "name": "力量", "key": "li" }, { "name": "速度", "key": "speed" }, { "name": "攻击", "key": "gong" },
        { "name": "防御", "key": "fang" }, { "name": "伤害", "key": "kill" }, { "name": "生命", "key": "life" } ],
      "splitArea": { "show": true, "areaStyle": { "color": ["rgba(0,196,255,0.1)", "rgba(0,112,176,0.1)"] } },
      "axisLine": { "lineStyle": { "color": "#ffffff", "width": 1, "type": "solid" } } } } }
```
> `indicator[i].key` 必须与 dataText 里 data 对象的 key 一致。

### cpt-chart-funnel 漏斗图
dataText：`[{"value":数值,"name":"名称"},...]`
```json
{ "cptDataForm": { "dataText": "[{\"value\":100,\"name\":\"浏览\"},{\"value\":80,\"name\":\"点击\"},{\"value\":60,\"name\":\"下单\"}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "color": ["#0061C2","#409EFF","#31adfb","#8FC7FF","#DEEEFF","#bbd3fb"],
    "title": { "text": "转化漏斗", "left": "top", "top": "center", "textStyle": { "fontSize": 18, "color": "#cccccc" } },
    "legend": { "show": true, "left": "center", "top": "bottom", "orient": "horizontal", "textStyle": { "color": "#ffffff", "fontSize": 14 } },
    "series": { "orient": "vertical", "gap": 0, "top": 0, "bottom": 30, "left": 10, "right": 10,
      "label": { "show": true, "position": "inside", "color": "#ffffff", "fontSize": 12 },
      "labelLine": { "show": true, "length": 100, "lineStyle": { "width": 1, "type": "solid" } },
      "funnelAlign": "center", "sort": "descending" } } }
```

### cpt-chart-candlestick K线图
dataText：`[{"group":"系列","data":[{"name":"日期","value":[开,收,低,高]},...]},...]`
```json
{ "cptDataForm": { "dataText": "[{\"group\":\"日k\",\"data\":[{\"name\":\"10-08\",\"value\":[20,34,10,38]}]}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "color": ["#009900", "#ff00ff", "#df9e08", "#4677ff"],
    "title": { "text": "日K", "left": "center", "top": "top", "textStyle": { "fontSize": 18, "color": "#cccccc" },
      "subtextStyle": { "fontSize": 12, "color": "#aaaaaa" } },
    "xAxis": { "show": true, "name": "", "axisLabel": { "show": true, "color": "#eeeeee", "fontSize": 13, "rotate": 0 },
      "axisLine": { "show": true, "lineStyle": { "color": "#eeeeee" } }, "axisTick": { "show": true }, "splitLine": { "show": false } },
    "yAxis": { "show": true, "name": "", "type": "value", "axisLabel": { "show": true, "color": "#eeeeee", "fontSize": 13 },
      "axisLine": { "show": true, "lineStyle": { "color": "#eeeeee" } }, "axisTick": { "show": true }, "splitLine": { "show": true } },
    "legend": { "show": true, "left": "center", "top": "bottom", "orient": "horizontal", "textStyle": { "color": "#ffffff", "fontSize": 14 } },
    "grid": { "x": 10, "y": 30, "x2": 10, "y2": 30 } } }
```

### cpt-chart-wordcloud 词云图
dataText：`[{"name":"词","value":权重},...]`（value 越大词越大）
```json
{ "cptDataForm": { "dataText": "[{\"name\":\"数据可视化\",\"value\":888},{\"name\":\"大屏设计器\",\"value\":629}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "color": ["#f65f4e","#f6c603","#31adfb","#5aed18","#09db9a","#0a94f5","#877bed","#ed09dd"],
    "fontWeight": "normal", "gridSize": 4, "shape": "circle", "rotationStep": 45, "sizeRange": [10, 60],
    "drawOutOfBound": false, "maskImage": "",
    "emphasis": { "focus": true, "textStyle": { "textShadowBlur": 10, "textShadowColor": "#ffffff" } } } }
```

### cpt-g6-dom 关系图（拓扑）
无通用 dataText，数据结构复杂（G6 图数据）。生成复杂大屏时慎用；如无把握可跳过此组件。

---

## 8. 派生变体说明（同一组件换 attribute 字段）

这些"组件"在 `options.js` 里复用同 cptKey/cptOptionKey，只是 `attribute` 个别字段不同。生成时写基础组件结构 + 下列差异即可：

| 变体名 | 基础组件 | 差异字段 |
|--------|----------|----------|
| 横向柱状图 | cpt-chart-column | `attribute.direction = "y"`，`series.barWidth = 8`，`series.label.position = "right"` |
| 单列柱状图 | cpt-chart-column | `attribute.legend = null`，`grid.x2 = 10`，`grid.y2 = 10`；dataText 用 `[{"name","value"}]` |
| 横向单列/旋风 | cpt-chart-column | `direction="y"` 等 |
| 百分比堆积柱图 | cpt-chart-column-stack | `attribute.enablePa = "1"`，`series.label.toFixed = 2` |
| 环形图 | cpt-chart-pie | `series.radius = [40, 60]` |
| 玫瑰图 | cpt-chart-pie | `series.roseType = "radius"` |
| 扇形饼图 | cpt-chart-pie | `series.startAngle = 180`，`endAngle = 360`，`center = [50, 65]`，`itemStyle.borderRadius = 20` |
| 面积图 | cpt-chart-line | `attribute.areaShow = true` |
| 散点折线 | cpt-chart-line | `attribute.lineShow = false`，`symbolSize = 16` |
| 圆形雷达图 | cpt-chart-radar | `radar.shape = "circle"` |
| 圆形水位图 | cpt-dataV-waterLevel | `attribute.shape = "round"` |

---

## 9. 统计指标 / 状态 / 时间线

### cpt-icon-quota 统计指标（正常/异常计数）
dataText：`{"normal":数值,"error":数值}`
```json
{ "cptDataForm": { "dataText": "{\"normal\":10,\"error\":20}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "title": "主机", "paddingTop": 60, "titleWidth": 60, "titleLeft": 44, "totalTop": 16, "numSize": 16,
    "bgUrl": null, "progressShow": true, "totalText": "总数", "normalText": "正常", "errorText": "异常", "errorShow": true } }
```

### cpt-icon-ring 线路状态（环形百分比）
dataText：`{"value":90}`
```json
{ "cptDataForm": { "dataText": "{\"value\":90}", "dataSource": 1, "pollTime": 0 },
  "attribute": { "title": "线路", "titleSize": 16, "titleColor": "#ffffff", "numSize": 30,
    "format": "{value}%", "roseType": "false", "progressWidth": 70 },
  "interaction": { "intType": "none" } }
```

### cpt-img-quota 告警图
dataText：`{"value":数值}`；`attribute.img` / `activeImg` 指向图片。
```json
{ "attribute": { "img": null, "activeImg": null, "alarmConfig": { "operator": ">", "threshold": 60 } },
  "interaction": { "eventText": "鼠标聚焦时" },
  "cptDataForm": { "dataText": "{\"value\":66}", "dataSource": 1, "pollTime": 0 } }
```

### cpt-img-state 状态图（按 value 切图）
dataText：`{"value":"0"}`
```json
{ "attribute": { "urls": { "0": "", "1": "", "2": "", "3": "" } },
  "cptDataForm": { "dataText": "{\"value\":\"0\"}", "dataSource": 1, "pollTime": 0, "apiUrl": "/text" } }
```

### cpt-time-line 时间轴
dataText：`[{"name":"日期","value":"事件","color":"可选","icon":"可选"},...]`
```json
{ "cptDataForm": { "dataText": "[{\"name\":\"2025-12-01\",\"value\":\"事件一\"},{\"name\":\"2025-12-02\",\"value\":\"事件二\",\"color\":\"#f00\",\"icon\":\"alarm\"}]", "dataSource": 1, "pollTime": 0 },
  "attribute": { "layout": "horizontal", "labelAlign": "alternate", "color": "#dddddd", "mode": "alternate",
    "reverse": false, "theme": "default", "icon": "circle", "labelSize": 14, "labelColor": "#cccccc",
    "valueSize": 16, "valueColor": "#ffffff", "iconSize": 16 } }
```

### cpt-html-viewer HTML查看器（iframe嵌入 / 富文本渲染）
无 dataText；`attribute.codeType` 决定渲染模式：
- `richText`（默认）：富文本模式，用 `v-html` 直接渲染 HTML 片段，支持文字样式、列表、图片、表格等，适合报告说明/通知公告/富文本内容。
- `html`：iframe 嵌入完整 HTML 页面（`srcdoc`），沙箱允许脚本，适合自定义独立页面或外部代码片段。
```json
{ "attribute": { "refreshKey": "htmlViewer", "codeType": "richText",
    "code": "<h2><span style=\"color: rgb(135, 20, 0);\">通知主题001</span></h2>" },
  "interaction": { "intType": "none" } }
```
> 报告/长页类建议优先使用 `richText` 模式：无需 iframe 沙箱，样式可与页面主题融合，高度自适应内容更自然。

---

## 10. 控件（选项卡/导航/跳转，含多选项交互）

控件类多为**交互组件**，`interaction.multi = true`，dataText 带 `options` 结构：`{"value":"默认选项值","options":[{"label":"选项","value":"值"}]}`。

### cpt-tab 选项卡
```json
{ "cptDataForm": { "dataText": "{\"value\":\"op1\",\"options\":[{\"label\":\"总览\",\"value\":\"op1\"},{\"label\":\"详情\",\"value\":\"op2\"}]}",
    "apiUrl": "/tab", "sql": "", "dataSource": 1, "pollTime": -1 },
  "attribute": { "textColor": "#0061C2", "fontSize": 18, "lineHeight": 30, "autoSwitch": false, "interval": 3,
    "backgroundColor": "#00000000", "highlightFontSize": 20, "highlightColor": "#31adfb", "highlightBgColor": "#00000020",
    "borderRadius": 4, "gap": 10, "borderWidth": 0, "borderColor": "#568dcc", "highlightBorderColor": "#c3daf5" },
  "interaction": { "multi": true } }
```

### cpt-navigator 导航器
```json
{ "cptDataForm": { "dataText": "{\"options\":[{\"label\":\"导航1\",\"value\":\"op1\"},{\"label\":\"导航2\",\"value\":\"op2\"}]}",
    "apiUrl": "/tab", "sql": "", "dataSource": 1, "pollTime": -1 },
  "attribute": { "space": 4, "bgColor": "#242424", "bgColorActive": "#699ef5", "autoSwitch": false, "interval": 3 },
  "interaction": { "multi": true } }
```

### cpt-jumper 跳转（自动轮播跳页）
```json
{ "attribute": { "size": "medium", "layout": "horizontal", "autoSwitch": false, "dataReadySwitch": false,
    "timeout": 5, "trigger": "next" },
  "interaction": { "multi": true },
  "cptDataForm": { "dataText": "{\"value\":\"next\",\"options\":[{\"label\":\"上一页\",\"value\":\"prev\"},{\"label\":\"下一页\",\"value\":\"next\"}]}",
    "apiUrl": "", "sql": "", "dataSource": 1, "pollTime": 0 } }
```

### cpt-select 下拉框 / cpt-input 输入框 / cpt-date-picker 日期选择 / cpt-tree-select 树形选择
交互控件，dataText 带 options（下拉框）或固定结构。生成普通大屏时通常不必使用；如需用，参考：
- cpt-select dataText：`{"value":"op1","options":[{"label":"选项1","value":"op1"},...]}`
- cpt-input dataText：`{"value":"文本"}`
- cpt-date-picker：无 dataText，`attribute` 见默认（label/placeholder/size/mode 等）。
- cpt-tree-select dataText：`{"value":"active","options":[{"label":"节点","value":"值","children":[...]}]}`

---

## 11. 生成时的配色建议

详见 SKILL.md 4.2 节「风格选择指南」中的 7 种配色方案。以下为通用规则：
- 同一大屏内主色统一 1–2 个 + 渐变辅助；KPI 卡片背景可用渐变色对。
- 图表标题统一字号 16–18，正文坐标轴 12–14。
- 使用 `cpt-quota` 时 `bgColor` 可用渐变色数组如 `["#0061C2", "#409EFF"]`。
- 深色大屏的图表 `axisLabel.color` 用 `#eeeeee`，浅色用 `#666666`；`splitLine.lineStyle.color` 深色用 `["#333","#444"]`，浅色用 `["#ddd","#eee"]`。

---

## 12. 合规校验清单（生成每个组件时过一遍）

- [ ] cptKey 在目录中存在；cptOptionKey 与目录标注一致
- [ ] `cptDataForm.dataSource === 1`、`pollTime === 0`（交互组件例外：tab/navigator 用 -1）
- [ ] `cptDataForm.dataText` 是**字符串**，内容能被 `JSON.parse`，且结构匹配该组件
- [ ] `attribute` 里颜色均为具体 hex/rgba（无 `defaultColor[i]` 占位）
- [ ] 无 dataText 的组件（图片/边框/装饰/天气/视频配置等）未声明 cptDataForm（或按目录含 cptDataForm 则带 dataText）
- [ ] 每个组件 `cptOption.interaction` 为**完整默认对象**：非交互组件仅把 `intType` 改为 `"none"`（不省略 `objMap`）；多选项附加 `"multi":true`（目录中的 `"intType":"none"` / `"multi":true` 均为简写，落盘时须展开为完整对象）
- [ ] `id` 唯一；坐标/尺寸在画布内