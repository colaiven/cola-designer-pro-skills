---
name: create-custom-component
description: 在 Cola Designer Pro 项目中新增自定义可视化组件（渲染组件 + attrs 默认配置 + 属性表单 + 注册 + color.js 主题配色登记）。当用户提出"新增组件 / 创建组件 / 自定义组件 / 加一个图表 / 做一个小部件 / 实现一个 xxx 组件"等诉求时触发。也可交给 codex 等其它 agent 直接按本文件执行。
---

# 创建自定义可视化组件

本 Skill 面向 cola design pro 前端仓库（Vue3 纯 JS + TDesign）。目标是让 Claude / Codex 仅凭本文件即可按项目规范，端到端地新增一个可视化组件并正确接入主题、动态数据、交互体系。

> 涉及的核心源码（动手前先打开对齐）：
> - 组件三件套机制：`src/components/register-cpt.js`、`src/components/register-option.js`、`src/components/options.js`、`src/main.js`
> - 主题配色：`src/components/color.js`
> - 动态数据：`src/utils/refresh-cpt-data.js`
> - 交互默认结构：`src/views/designer/model/default-action-obj.js`
> - 配置表单 helper 组件：`src/components/designer/`（e-collapse / e-color-group / e-icon-select / e-shape-select / e-chart-position / gallery）
> - 设计器/预览渲染与接入点：`src/views/designer/index.vue`、`src/views/designer/report-editor.vue`、`src/views/preview/index.vue`、`src/views/preview/report-view.vue`

（文档约定已在本 Skill 内联说明，无需额外查阅外部文档。）

---

## 0. 必须遵守的执行顺序（不要跳步）

1. **整理需求**：搞清楚用户要什么组件、外观长什么样、哪些东西要可配置、是否读取数据。
2. **查重**：检索现有组件，判断是否已有相似组件。
3. **抽取配置项**：把"可配置的东西"拆成 attribute（样式/外观）与 cptDataForm（动态数据内容）两部分，产出配置项清单。
4. **确认动态数据**：询问用户哪些字段放入 `cptDataForm.dataText`。
5. **向用户确认**：把「需求 + 配置项清单 + 是否用动态数据 + 组件分组 + cpt-key + 初始宽高」一次性整理好，**等用户确认后才写代码**。
6. **生成代码**：按第 2 节三件套 + 注册 + color.js 登记，逐文件产出。

> ⚠️ 第 5 步是硬性要求：**需求和配置项没有确认之前，不执行代码生成。**

---

## 1. 查重与"优先走配置项"原则

创建前先搜索现有组件，判断是否已存在相似组件。

搜索方式：
- 在 `src/components/options.js` 里遍历所有 group/children 的 `name` 和 `cptKey`，看分类与用途是否接近。
- 在 `src/components/**/cpt-*.vue` 里 grep 组件的渲染功能关键词。
- 参考项目维护的更新日志（changelog）里的历史"新增组件"列表，确认是否曾经做过。

判断结果处理：
- **已有组件能通过改配置项（attribute）实现**：优先"新增一个 options.js 的子项（变体）"而非新建组件。做法见下文 4.3「派生变体」。此时**告知用户**"该需求可用现有组件 cpt-xxx 通过配置项实现，无需新增组件"。
- **差异过大 / 改动现有组件风险高**：才新增组件。

---

## 2. 组件三件套机制（核心约定，必须三处登记）

每个可视化组件由三部分组成，新增一个组件需要 3 个文件 + 2 处注册 + 1 处目录 + （可选）1 处配色登记：

| # | 文件 | 作用 |
|---|------|------|
| 1 | `src/components/<类别>/cpt-xxx.vue` | **渲染组件**，画布/预览上真正渲染 DOM 的组件 |
| 2 | `src/components/<类别>/attrs/cpt-xxx-option.js` | **默认配置**，导出 `{ attribute, cptDataForm?, interaction? }` |
| 3 | `src/components/<类别>/options/cpt-xxx-option.vue` | **属性表单**，右侧配置栏"属性"页渲染的表单 |
| 4 | `src/components/register-cpt.js` | 注册渲染组件（全局组件） |
| 5 | `src/components/register-option.js` | 注册属性表单（全局组件） |
| 6 | `src/components/options.js` | 左侧组件目录登记入口 |
| 7 | `src/components/color.js` | 若含颜色配置，在 `colorFields` 登记主题配色 |

类别目录（`<类别>`）取值：`echarts/`、`element/`、`dataV/`、`vuedataui/`、`g6/`、`three/`、`other/`。根据组件实现技术选一个（echarts 图表放 echarts，普通 DOM 组件放 element，DataV 装饰放 dataV，vue-data-ui 放 vuedataui）。

`main.js` 会自动把注册对象里的 key 中下划线替换为连字符注册为全局组件：
```js
for (const key in cpt) { app.component(key.replaceAll('_','-'), cpt[key]) }
```
因此：
- 文件名、cptKey 用 **kebab-case**：`cpt-xxx`。
- `register-cpt.js` / `register-option.js` 里的 import 变量名用 **下划线**：`cpt_xxx`、`cpt_xxx_option`，二者经替换后拼出 `cpt-xxx`、`cpt-xxx-option`。

---

## 3. 硬性规则清单（逐条遵守）

### 3.1 命名
- 配置项字段名：**英文、驼峰（camelCase）**，简单直观描述作用。例如 `textColor`、`bgColor`、`fontSize`、`borderRadius`、`barWidth`、`showTitle`、`rotateSpeed`。
- 组件中文名、配置表单里每个 label 的中文：**≤ 5 个汉字**（如「文本」「静态表格」「滚动列表」）。
- `cptKey` 必须与 `src/components/options.js` 及 `register-cpt.js` 中已存在的 key 全部不同，且避免与原生/TDesign 标签名冲突（input、image、select 等——现有已占用）。用大小写/下划线检查。

### 3.2 组件形态
- 新组件一律使用 **Vue3 Composition API（`<script setup>`）**，不要写 Options API。
- 查看 `package.json` 评估是否已有可复用的第三方库；**不允许新增第三方依赖**（除非用户明确指定）。可用依赖：echarts、echarts-gl、echarts-liquidfill、echarts-wordcloud、@kjgl77/datav-vue3、vue-data-ui、@antv/g6、three、tdesign-vue-next、uuid。

### 3.3 初始尺寸
- 在 options.js 入口里**显式写 `width` / `height`**（不写会落到默认 400×300）。
- 由 AI 评估合理值，范围：**宽 100–600px，高 20–500px**。

### 3.4 图标与分组
- 默认图标：`icon:'default'`（对应 `src/assets/icon/components/default.svg`）。
- 分组由 AI 依据功能判断，放入 `options.js` 已存在的分组（基础组件、控件、指标卡、柱状图、饼/环图、水位图、折线/折柱图、k线/趋势、仪表盘/进度、散点图、雷达图、关系图、漏斗图、地图、3D、其它）。**不允许新建分组**。
- 生成完成后，在回复里明确告知用户"组件放在了哪个分组下"。

### 3.5 配置表单规范
- 用 `e-collapse`（已全局注册，无需 import）按功能把表单项分组；第一组常用 `:expand="true"` 默认展开，其余 `false`。
- 单个颜色配置：`t-color-picker`（`format="HEX"`，需透明时 `format="RGBA"` + `enableAlpha`）。
- **多个颜色/色板配置**（如饼图配色）：用 `e-color-group`。
- 图标配置：`e-icon-select`；echarts 自定义形状（symbol）配置：`e-shape-select`。
- 图片配置：`gallery`（图片素材库）。
- 位置配置（左/中/右 或 上/中/下 + 自定义数值）：`e-chart-position`。
- 富文本编辑：使用 `@wangeditor/editor-for-vue` 组件（已新增依赖），支持深色主题适配。
- helper 组件除 `e-collapse` 外，都需要在表单组件里 `import`（见第 5 节 API 速查）。

### 3.6 动态数据
- **默认静态数据源**：`cptDataForm.dataSource = 1`。
- `cptDataForm.dataText` **必须是 JSON 格式的字符串**。
- 若用户反馈组件不需要动态数据源，则**不在 attrs 里声明 `cptDataForm`**（右侧便不会出现"数据"tab）。
- 判断哪些字段进 `cptDataForm.dataText`（数据/内容），哪些进 `attribute`（样式/外观）——详见第 6 节。

---

## 4. 文件生成模板

### 4.1 渲染组件 `src/components/<类别>/cpt-xxx.vue`

props 约定（设计器传入 `:option/:width/:height/:show/:design`，预览页不传 `design` 或传 `false`）：
`option`（组件 cptOption）、`width`（px）、`height`（px）、`show`（是否显示）、`design`（是否设计态）。

**（A）带动态数据的模板（最常用，Composition API）：**
```vue
<template>
  <div style="width: 100%;height: 100%;box-sizing: border-box"
       :style="{ color: props.option.attribute.textColor, fontSize: props.option.attribute.fontSize + 'px' }"
       @click="clickHandler">
    {{ cptData.value }}
  </div>
</template>

<script setup>
import { v1 as uuidv1 } from 'uuid'
import { getDataJson, pollingRefresh } from '@/utils/refresh-cpt-data'
import { reactive, onMounted } from 'vue'

const props = defineProps({
  width: Number,
  height: Number,
  option: Object,
  show: Boolean,
  design: Boolean,
})
const emit = defineEmits(['clickHandler'])

// uuid 必须，用于清除轮询定时器
const uuid = uuidv1()
// 组件接收数据的反应式容器，字段名与 dataText 的 JSON key 对应
const cptData = reactive({ value: '' })

// refreshCptData 固定写法，必须
function refreshCptData(cptDataForm) {
  pollingRefresh(uuid, cptDataForm, () => loadData(cptDataForm))
}
// 自定义数据处理：res 即 dataText 的 JSON 或接口/SQL/数据集/ws 返回结果
function loadData(cptDataForm) {
  getDataJson(cptDataForm).then(res => {
    cptData.value = res.value
  })
}

// 初始化数据。echarts 类组件必须放在 onMounted 中 init 后再调！
onMounted(() => refreshCptData(props.option.cptDataForm))

// 暴露 refreshCptData，必须
defineExpose({ refreshCptData })

// 点击交互：把 interaction 和取值抛给预览页 cptClickHandler
function clickHandler() {
  emit('clickHandler', props.option.interaction, 'default', cptData.value)
}
</script>
```

**（B）纯静态组件（无 cptDataForm）：**
```vue
<template>
  <div style="width:100%;height:100%"
       :style="{ color: props.option.attribute.color }">
    {{ props.option.attribute.text }}
  </div>
</template>
<script setup>
const props = defineProps({ width:Number, height:Number, option:Object, show:Boolean, design:Boolean })
</script>
```

**（C）echarts 类组件要点：**
```vue
<template>
  <div :id="uuid" :style="{ width: width+'px', height: height+'px' }"/>
</template>
<script setup>
import { v1 as uuidv1 } from 'uuid'
import { getDataJson, pollingRefresh } from '@/utils/refresh-cpt-data'
import * as echarts from 'echarts'          // 或 app.config.globalProperties.$echarts
import { onMounted, watch } from 'vue'

const props = defineProps({ width:Number, height:Number, option:Object, show:Boolean })
const uuid = uuidv1()
let chart = null, cptData = []
const emit = defineEmits(['clickHandler'])

onMounted(() => {
  chart = echarts.init(document.getElementById(uuid))   // 必须 mount 后 init
  refreshCptData(props.option.cptDataForm)
})
// 属性变化重建图表
watch(() => props.option.attribute, attr => loadChart(attr), { deep: true })
// 尺寸变化 resize
watch(() => [props.width, props.height], () => chart && chart.resize({ width: props.width, height: props.height }))

function refreshCptData(cptDataForm){ pollingRefresh(uuid, cptDataForm, () => loadData(cptDataForm)) }
function loadData(cptDataForm){
  getDataJson(cptDataForm).then(res => { cptData = res; loadChart(props.option.attribute) })
}
function loadChart(attr){ /* 用 attr + cptData 组装 echarts option 后 chart.setOption(...)；
   chart.on('click', params => emit('clickHandler', props.option.interaction, 'default', params.name)) */ }
defineExpose({ refreshCptData })
</script>
```

> 提示：
> - 若不支持交互（像 iframe/天气那种纯展示），attrs 里加 `interaction:{ intType:'none' }`，渲染组件可不 emit clickHandler。
> - 对"视图不刷新"的第三方组件，可按需 `watch(() => props.option.attribute, ..., {deep:true})` 深度监听（参考进度池/地图组件）。
> - 兼容历史配置（可选）：setup 里把 attrs 默认值深拷贝后 `Object.assign(defaultAttr, props.option.attribute)` 回填缺省字段，避免旧数据缺字段导致渲染报错。

### 4.2 默认配置 `src/components/<类别>/attrs/cpt-xxx-option.js`

```js
// 需要主题色时，可从 color.js 引入默认色板（其 default 导出为色板数组，标注"废弃/待处理"，仅作初始默认值）
// import defaultColor from '@/components/color'

export default {
  attribute: {
    textColor: '#fff',
    fontSize: 16,
    // ... 其它样式/外观配置项
  },
  // 需要动态数据时才声明 cptDataForm（整体可省略）
  cptDataForm: {
    dataText: '{"value":"默认文本"}',  // 必须是 JSON 字符串
    dataSource: 1,                     // 默认静态
    pollTime: 0,                       // 0 不轮询；-1 表示组件不支持轮询
    apiUrl: '/design/test',            // 可省略，拖入时设计器会补
    sql: '',                           // 可省略
  },
  // 不支持交互时显式关闭；支持交互则省略（由设计器合并 defaultActionObj）
  // interaction: { intType: 'none' },
}
```

`cptDataForm` 字段速查：

| 字段 | 说明 | 备注 |
|------|------|------|
| dataText | 静态数据 JSON 字符串 | 必填（有 cptDataForm 时） |
| dataSource | 1静态 / 2API / 3SQL / 4数据集 / 5websocket | 默认 1 |
| pollTime | 轮询秒数；0 关；-1 不支持轮询 | |
| apiUrl | API 地址（dataSource=2） | 拖入时自动补 `/design/test` |
| sql | SQL（dataSource=3） | |
| datasourceId | 数据源 id（dataSource=3） | |
| datasetId / datasetType | 数据集（dataSource=4） | |
| socketDataKey | websocket 取值 key（dataSource=5） | |

### 4.3 options.js 左侧目录登记

在 `src/components/options.js` 顶部 import 你的 attrs，再在对应分组的 `children` 数组里加一条：

```js
import cpt_xxx_option from '@/components/<类别>/attrs/cpt-xxx-option'
// ...
{
  name: '组件中文名',        // ≤5 汉字
  icon: 'default',           // 默认图标
  cptKey: 'cpt-xxx',         // 全局唯一
  cptOptionKey: 'cpt-xxx-option', // 可选，默认 = cptKey + '-option'
  width: 200, height: 80,    // AI 评估，宽 100-600 / 高 20-500
  option: cpt_xxx_option,    // 或内联 { attribute:{...}, cptDataForm:{...} }
}
```

**派生变体（不新建组件，只加配置项）**：用 `JSON.parse(JSON.stringify(baseOption))` 深拷贝已有 attrs，改动个别字段后作为新子项；若共用同一属性表单，可复用 cptKey，仅换 `option`；若需要不同属性表单，则同一 cptKey 配不同 `cptOptionKey`（参考 cpt-vue-data-ui 系列）。派生变体若使用不同 cptOptionKey，主题配色里 colorFields 的 key 也要用 cptOptionKey。

### 4.4 注册

`src/components/register-cpt.js`（渲染组件）：
```js
import cpt_xxx from '@/components/<类别>/cpt-xxx'
// export default {} 里加一行：
cpt_xxx,
```

`src/components/register-option.js`（属性表单）：
```js
import cpt_xxx_option from '@/components/<类别>/options/cpt-xxx-option'
// export default {} 里加一行：
cpt_xxx_option,
```

---

## 5. 属性表单 helper 组件 API 速查

属性表单 `src/components/<类别>/options/cpt-xxx-option.vue`：
```vue
<template>
  <t-form labelWidth="90px">
    <e-collapse title="标题" :expand="true">
      <t-form-item label="文字"><t-input size="small" v-model="attribute.text"/></t-form-item>
    </e-collapse>
    <e-collapse title="配色" :expand="false">
      <t-form-item label="配色"><e-color-group v-model="attribute.color" :max="20"/></t-form-item>
    </e-collapse>
  </t-form>
</template>

<script setup>
import EColorGroup from '@/components/designer/e-color-group.vue'
const props = defineProps({ attribute: Object, cptDataForm: Object })
// 模板里直接 v-model 的 attribute 即 currentCpt.cptOption.attribute（响应式对象）
</script>
```

各 helper 组件（除 `e-collapse` 全局注册外，其余需 import）：

| 组件 | import 路径 | 关键 props | 事件 | 用法 |
|------|-------------|-----------|------|------|
| `e-collapse` | 无需 import（已全局注册） | `title`(String)、`expand`(Boolean,默认 false) | — | `<e-collapse title="边框" :expand="false">…</e-collapse>` |
| `e-color-group` | `@/components/designer/e-color-group.vue` | `modelValue`(Array)、`min`(默认1)、`max`(默认10)、`format`('HEX')、`gradient`(默认false)、`absolute`、`enableAlpha` | `update:modelValue`、`change` | `<e-color-group v-model="attribute.color" :max="20" :gradient="false"/>` |
| `e-icon-select` | `@/components/designer/e-icon-select.vue` | `modelValue`(String,默认'api') | `update:modelValue` | `<e-icon-select v-model="attribute.icon"/>` |
| `e-shape-select` | `@/components/designer/e-shape-select.vue` | `modelValue`(String,默认'circle')、`base64`(Boolean) | `update:modelValue` | `<e-shape-select v-model="attribute.series.symbol"/>`（值如 circle/rect/roundRect/triangle/diamond/arrow/pin 或 path://...） |
| `e-chart-position` | `@/components/designer/e-chart-position.vue` | `modelValue`(默认'center')、`direction`('x'/'y') | `update:modelValue` | `<e-chart-position v-model="attribute.title.left" :direction="'x'"/>` |
| `gallery` | `@/components/designer/gallery` | `readOnly`(Boolean) | `confirmCheck(filePath)` | 见下 |

gallery 图片素材库用法（在表单组件里）：
```vue
<template>
  <t-form labelWidth="80px">
    <t-form-item label="图片">
      <div @click="showGallery" style="width:168px;height:160px;cursor:pointer;">
        <t-image style="width:100%;height:100%" :src="attribute.url ? fileUrl + attribute.url : ''" fit="fill"/>
      </div>
    </t-form-item>
  </t-form>
  <gallery ref="gallery" @confirmCheck="confirmCheck"/>
</template>
<script setup>
import { ref } from 'vue'
import Gallery from '@/components/designer/gallery'     // 或 gallery.vue
import { fileUrl } from '/env'
const props = defineProps({ attribute: Object })
const gallery = ref(null)
function showGallery(){ gallery.value.opened() }
function confirmCheck(filePath){ props.attribute.url = filePath }
</script>
```
> 渲染组件里图片的 src 用 `fileUrl + option.attribute.url`（`import { fileUrl } from '/env'`）。

---

## 6. 动态数据（cptDataForm.dataText）与配置项拆分

**拆分配置项的原则**：把「会随数据变化的内容」放进 `dataText` 的 JSON；把「样式/外观/行为开关」放进 `attribute`。

参考例子：
- `cpt-text`：文本内容 → `dataText` 的 `{ "value": "..." }`；字体/颜色/行高/对齐/边框 → `attribute`。
- `cpt-chart-pie`：饼图数据 → `dataText` 的 `[{"value":1048,"name":"搜索引擎"}, ...]`；标题/图例/标签/配色/圆心半径 → `attribute`。

**固定步骤**：
1. 列出组件所有可变内容。
2. 判断哪些属于"内容/数据"（可能来自 API/SQL/数据集/ws），推荐放进 `dataText`。
3. **主动询问用户**："以下字段建议放入数据模块（dataText），是否同意？还有没有要额外放入动态数据的字段？"
4. 若用户说不需要动态数据，则不声明 `cptDataForm`。
5. `dataText` 必须是合法 JSON 字符串（默认给一份可渲染的示例数据，能 `JSON.parse`）。

---

## 7. color.js 主题颜色字段登记（重要）

切主题时，设计器/预览会调用 `updateThemeOption(cptKey, cptOptionKey, option, themeIndex, textColor)`（见 `color.js` 底部的实现，以及 `header-bar.vue` 的 `changeTheme`/`changeTextColor`）。逻辑：

1. 用 `cptKey` 查 `colorFields`，查不到再用 `cptOptionKey` 查；都查不到则原样返回（不随主题变化）。
2. `fill`：主题色生效字段。每项 `{ field, colorIndex }`：
   - `field`：`option.attribute` 里的颜色字段**点路径**（内部通过 `getByPath`/`setByPath` 安全取值，支持 `a.b.c` 及数组下标 `yAxis[0].axisLine.lineStyle.color`）。
   - `colorIndex`：`themeGroup[themeIndex].colors` 的下标，缺省 0。
   - 若该字段当前值是**数组**（如饼图配色 `color` 是色板），会在运行时按 `oldColor.length` 对主题色数组 `slice` 截取，保持色板长度一致。
3. `text`：字体/文本色字段（字符串数组），切换字体色时统一覆盖为 `textColor`。

登记模板：
```js
// color.js 的 colorFields 里加：
'cpt-xxx': {
  fill: [
    { field: 'bgColor', colorIndex: 0 },          // 单色主题色
    { field: 'color' },                            // 色板数组（默认 colorIndex 0）
    { field: 'bar.color1', colorIndex: 1 },        // 嵌套路径 + 指定下标
  ],
  text: ['color', 'title.textStyle.color'],        // 随主题字体色
},
```

**登记规则**：
- 组件含"主题色"性质的颜色字段（底色、主色、色板、柱色、描边色等）→ 进 `fill`。
- 组件含"文字/标签/坐标轴文本"颜色字段 → 进 `text`。
- **key 用 `cptKey`**；仅当同一个渲染组件被多个 `cptOptionKey` 复用（派生多套 theme 语义）时，key 改用 `cptOptionKey`（参考 `color.js` 里 `cpt-dataui-sparkline-option` 等 vuedataui 系列）。此时新增的子项也要对应登记一条 cptOptionKey 形式的 key。
- 纯图片/iframe/视频这类不受主题影响的组件，可不登记。
- 嵌套字段路径要写全（如 `title.textStyle.color`、`series.color`），与你 attrs 里的结构严格一致；写错会导致路径取值返回 undefined，运行时仅 console.warn，不影响其它组件（但该字段不生效）。

---

## 8. 交互 interaction（可选）

- 交互默认结构见 `src/views/designer/model/default-action-obj.js`：`intType`（display 显隐 / dataset 更新数据源参数 / drill 下钻 / redirect 跳转）、`objMap`（多选项组件的 key 映射）、`paramType/paramName/paramValue` 等。
- 设计器拖入组件时，若 attrs 未声明 `interaction`，会自动 merge 完整 `defaultActionObj`；若声明了，则以其为底合并。
- 不希望交互的组件：attrs 里加 `interaction:{ intType:'none' }`。
- 希望可点击交互的组件：渲染组件里 `emit('clickHandler', props.option.interaction, objKey, val)`；objKey 默认 `'default'`，选项卡/下拉框类多选项组件传选项值。预览页 `cptClickHandler` 会据此执行显示隐藏/下钻/跳转/更新数据集参数。

---

## 9. 完成后的自检清单（提交给用户前逐项过一遍）

- [ ] 中文名与表单 label ≤ 5 汉字
- [ ] cptKey 全局唯一，与已有组件不冲突；文件名/变量名符合 kebab/下划线约定
- [ ] 三处登记齐全：`options.js`（目录）、`register-cpt.js`、`register-option.js`；attrs 文件已建
- [ ] 显式写了 `width`(100–600) / `height`(20–500)
- [ ] `icon:'default'`
- [ ] 放在已有分组，未新建分组；已告知用户所在分组
- [ ] 未新增第三方依赖（除非用户指定）
- [ ] 使用 `<script setup>` Composition API
- [ ] 有动态数据的话：`cptDataForm.dataText` 是合法 JSON 字符串，`dataSource=1`；渲染组件有 `uuid` + `pollingRefresh` + `getDataJson` + `defineExpose({refreshCptData})`
- [ ] 无动态数据的话：attrs 未声明 `cptDataForm`
- [ ] 颜色字段已在 `color.js` 的 `colorFields` 里登记（含正确点路径与 colorIndex）
- [ ] 表单按功能用 `e-collapse` 分组；多色用 `e-color-group`，图标用 `e-icon-select`，echarts 形状用 `e-shape-select`，图片用 `gallery`，位置用 `e-chart-position`
- [ ] 交互（按压需求）已处理：不支持的加 `intType:'none'`，支持的 emit clickHandler
- [ ] 用意在"告诉我放在哪个分组"的收尾说明

---

## 10. 常见坑

1. **不带 cptDataForm 的组件**：不要在渲染组件里调用 `getDataJson`/`pollingRefresh`，否则报错（config-bar 的"数据"tab 也不会出现）。
2. **echarts init 时机**：必须 `onMounted` 后 `echarts.init(document.getElementById(uuid))`，再调 `refreshCptData`；直接用模板字符串 id，避免重复 init。
3. **uuid 缺失**：带轮询的组件没有 `uuid`，`pollingRefresh` 无法按组件清除定时器，轮询会串台。
4. **dataText 忘转义**：JS 里写成 `'{"value":"文本"}'`，注意内外引号；含中文没问题，但必须能被 `JSON.parse`。
5. **colorFields 路径写错**：字段路径要对应 attrs 结构；`fill` 数组字段默认会被截断为主题色数组长度。
6. **表单 label 过长**：超过 5 汉字可能撑破布局，用 2–5 字短词。
7. **忘记在 register 里登记**：组件在画布上 `Message.error('组件未实现')` 或属性表单空白，通常是 register-cpt/register-option 漏加。