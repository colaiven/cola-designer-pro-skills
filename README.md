# Cola Designer Pro Skills

给 **Claude Code**（以及 Codex 等 AI agent）提供两个与 [Cola Designer Pro](https://cdesign.fun/)（一款可视化大屏 / 报表设计器）打交道的技能：

1. **`create-custom-component`** —— 在 Cola Designer Pro 前端仓库里**新增一个自定义可视化组件**（渲染组件 + attrs 默认配置 + 属性表单 + 注册 + 主题配色登记），端到端一次性搞定。此技能只适用于购买了cola-designer-pro商业版并拥有源代码的用户。
2. **`generate-design`** —— 根据需求**直接生成一份可导入的大屏 / 报表设计文件（`.cd`）**，产出合法、可直接拖进设计器的成品。此技能适用**所有用户**，官网地址：[Cola Designer Pro](https://cdesign.fun/)

| Skill                     | 能力                                                                             | 适用对象                                         | 产出                            |
| ------------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------- |
| `create-custom-component` | 按项目规范新增自定义可视化组件（Vue3 + TDesign，接入主题 / 动态数据 / 交互体系） | **前端开发者** / 维护 cola-designer-pro 仓库的人 | 组件源码（几处文件与注册改动）  |
| `generate-design`         | 将自然语言需求转成合法 `.cd` 大屏 / 报表设计文件                                 | **大屏使用者** / 任何想快速出大屏的人            | `<标题>.cd`（可直接导入设计器） |

> 官方产品地址：[https://cdesign.fun/](https://cdesign.fun/)（可在线体验设计器与模板）。
> 两个 Skill 均为**纯文档型 Skill**（Markdown，无外部依赖、无需 API Key），开箱即用。

---

## 目录结构

```
cola-designer-pro-skills/
├── README.md                          # 本文件
├── create-custom-component/
│   └── SKILL.md                       # 新增自定义可视化组件（完整规范 + 代码模板 + 自检清单）
└── generate-design/
    ├── SKILL.md                       # 生成大屏 .cd 设计文件（结构、布局算法、数据规则）
    └── references/
        └── component-catalog.md       # 组件目录与配置项参考（.cd 生成时查阅）
```

---

## 🚀 安装（一句话）

把下面这句话发给你电脑上的 Claude（需已安装 Claude Code）：

> 请安装这个仓库里的 skills：https://gitee.com/colaiven/cola-designer-pro-skills
> 按仓库 README 的「AI 安装指令」一节执行。

Claude 会自动克隆仓库、复制 skills，装完即可直接使用。

> 💡 **国内用户优先使用 Gitee 仓库**（速度快、无需梯子）：`https://gitee.com/colaiven/cola-designer-pro-skills`
> 海外用户或 GitHub 用户可使用：`https://github.com/colaiven/cola-designer-pro-skills`

---

## 🤖 AI 安装指令

（本节写给执行安装的 Claude。人类手动安装见下一节。）

### 1. 克隆仓库到临时目录

```bash
# 国内用户优先使用 Gitee（速度快、无需梯子）
git clone https://gitee.com/colaiven/cola-designer-pro-skills.git /tmp/cdp-skills

# 海外用户或 GitHub 用户可使用：
# git clone https://github.com/colaiven/cola-designer-pro-skills.git /tmp/cdp-skills
```

无 git 环境则下载 zip 解压。

### 2. 复制 skills 到目标位置（按用途二选一或都装）

Skills 的 `SKILL.md` 只需被放到 Claude Code 能发现的位置即可，无需其它配置。推荐按用途分开放：

- **`generate-design`（生成 .cd 设计文件）**：通用能力，装到**用户级**目录，全局可用：

  ```bash
  mkdir -p ~/.claude/skills
  cp -r /tmp/cdp-skills/generate-design ~/.claude/skills/
  ```

- **`create-custom-component`（新增组件）**：只对 cola-designer-pro **前端仓库**有意义，装到该仓库的**项目级**目录：

  ```bash
  # 假设前端仓库根目录为 /path/to/cola-designer-pro
  mkdir -p /path/to/cola-designer-pro/.claude/skills
  cp -r /tmp/cdp-skills/create-custom-component /path/to/cola-designer-pro/.claude/skills/
  ```

  > 注意：`generate-design` 需要同时携带 `references/component-catalog.md`，请把整个目录（含 `references/` 子目录）一起复制，不要只复制 `SKILL.md`。

### 3. 生效

复制完成后**重启 Claude Code**，两个 Skill 即会自动注册（无需 API Key、无需改 settings.json）。

---

## 📦 手动安装（人类版）

1. 克隆本仓库：
   ```bash
   # 国内优先
   git clone https://gitee.com/colaiven/cola-designer-pro-skills.git
   # 海外 / GitHub
   # git clone https://github.com/colaiven/cola-designer-pro-skills.git
   ```
2. 把需要的 skill 目录原样复制到目标位置：
   - 用户级（对所有项目生效）：`~/.claude/skills/`
   - 项目级（仅该项目生效）：`<项目>/.claude/skills/`
3. 重启 Claude Code。

⚠️ 复制时保持目录完整：`create-custom-component/SKILL.md`、`generate-design/SKILL.md`、`generate-design/references/component-catalog.md` 一个都不能少。

> 给 **Codex 等其它 agent** 用：这两个 Skill 本身就是自洽的指令文档（已内联全部规范，无需外部知识）。直接打开对应 `SKILL.md`，把整篇内容作为指令/上下文交给 agent 即可按规范执行。

---

## 用法示例

### `create-custom-component` —— 新增一个自定义组件

触发后 Skill 会先整理需求 → 查重 → 抽配置项 → 与你确认是否用动态数据 → 确认后才写代码。

- 「新增一个组件：环形进度条，颜色和粗细可配置，从 API 读取百分比」
- 「自定义一个水位图组件，支持渐变配色」
- 「加一个图表：横向条形排行榜，顶部 10 名滚动显示」
- 「实现一个数字翻牌器组件，不带动态数据源」

### `generate-design` —— 生成一张大屏设计文件

触发后 Skill 会先确认主题 / 分辨率 / 缩放模式 / 数据来源，再产出 `.cd` 文件。

- 「生成一张智慧城市运营中心大屏的设计文件，1920×1080，深色科技风」
- 「做一个园区能耗监控大屏，顶部 KPI、中间折线/柱图、底部地图」
- 「生成一个电商销售报表大屏的设计文件，数据用随机贴近真实的」
- 「导出一份机房监控大屏的 .cd 设计」

生成后把得到的 `.cd` 文件拖进设计器「导入」即可（详见 `generate-design/SKILL.md` 第 5 节）。

---

## 说明与注意

- `generate-design` 支持**单页大屏**（`scaleType: 1`，固定 1920×1080 全屏铺满）和**报告/长页**（`scaleType: 2`，宽度 1920、高度按内容自动计算可滚动），不涉及仪表盘编辑器（report-editor）。
- `.cd` 文件有**版本校验**：默认按支持版本 `2.7.18` 输出，若目标系统是其它版本，需先告知版本号（详见 `generate-design/SKILL.md` 第 0 节）。
- 生成 `.cd` 时**背景图 `bgImg` 默认置空**，导入后可在设计器「大屏配置 → 背景图片」自行选择。
- `create-custom-component` 面向 cola-designer-pro 前端仓库（Vue3 纯 JS + TDesign），不适用于其它技术栈项目。