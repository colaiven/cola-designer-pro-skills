# Cola Designer Pro Skills

Provides **Claude Code** (and other AI agents such as Codex) with two skills for working with [Cola Designer Pro](https://cdesign.fun/), a visual large-screen / report designer:

1. **`create-custom-component`** — **add a new custom visualization component** to the Cola Designer Pro frontend repository (render component + `attrs` defaults + property form + registration + theme color registration), done end-to-end in one pass. This skill is only applicable to users who have purchased the commercial edition of cola-designer-pro and have access to its source code.
2. **`generate-design`** — **generate an importable large-screen / report design file (`.cd`)** directly from a requirement, producing a valid file that can be dropped straight into the designer. This skill is applicable to **all users**. Official website: [Cola Designer Pro](https://cdesign.fun/)

| Skill                     | Capability                                                                                       | Intended for                                                    | Deliverable                                   |
| ------------------------- | ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------- | --------------------------------------------- |
| `create-custom-component` | Add a custom visualization component per project conventions (Vue3 + TDesign, wired into theme / dynamic data / interaction) | **Frontend developers** / maintainers of the cola-designer-pro repo | Component source code (several file edits & registrations) |
| `generate-design`         | Turn natural-language requirements into a valid `.cd` large-screen / report design file          | **Large-screen users** / anyone who wants to produce a dashboard quickly | `<title>.cd` (directly importable into the designer) |

> Official product website: [https://cdesign.fun/](https://cdesign.fun/) (try the designer and templates online).
> Both skills are **documentation-only** (Markdown, no external dependencies, no API key), ready to use out of the box.

---

## Directory Structure

```
cola-designer-pro-skills/
├── README.md                          # This file (Chinese)
├── README_EN.md                       # This file (English)
├── create-custom-component/
│   └── SKILL.md                       # Add a custom visualization component (full spec + code templates + checklist)
└── generate-design/
    ├── SKILL.md                       # Generate a large-screen .cd design file (structure, layout algorithm, data rules)
    └── references/
        └── component-catalog.md      # Component catalog & config reference (consult when generating .cd)
```

---

## 🚀 Install (one-liner)

Send the following message to the Claude running on your machine (Claude Code must be installed):

> Please install the skills from this repository: https://gitee.com/colaiven/cola-designer-pro-skills
> Follow the "AI Install Instructions" section of the repo README.

Claude will clone the repo, copy the skills, and they work immediately after installation.

> 💡 **For users in China**: prefer the Gitee mirror (faster, no VPN needed): `https://gitee.com/colaiven/cola-designer-pro-skills`
> For international / GitHub users: `https://github.com/colaiven/cola-designer-pro-skills`

---

## 🤖 AI Install Instructions

(This section is written for the Claude performing the installation. For manual/human installation, see the next section.)

### 1. Clone the repository to a temp directory

```bash
# For users in China (faster, no VPN needed):
git clone https://gitee.com/colaiven/cola-designer-pro-skills.git /tmp/cdp-skills

# For international / GitHub users:
# git clone https://github.com/colaiven/cola-designer-pro-skills.git /tmp/cdp-skills
```

If no git environment is available, download the zip and extract it.

### 2. Copy the skills to the target location (choose per use case, or install both)

A skill's `SKILL.md` only needs to be placed somewhere Claude Code can discover it — no other configuration is required. Recommended split by purpose:

- **`generate-design` (generate .cd design files)**: general-purpose — install to the **user-level** directory for global use:

  ```bash
  mkdir -p ~/.claude/skills
  cp -r /tmp/cdp-skills/generate-design ~/.claude/skills/
  ```

- **`create-custom-component` (add components)**: only meaningful for the cola-designer-pro **frontend repository** — install to that repo's **project-level** directory:

  ```bash
  # assuming the frontend repo root is /path/to/cola-designer-pro
  mkdir -p /path/to/cola-designer-pro/.claude/skills
  cp -r /tmp/cdp-skills/create-custom-component /path/to/cola-designer-pro/.claude/skills/
  ```

  > Note: `generate-design` requires its `references/component-catalog.md` alongside it — copy the whole directory (including the `references/` subdirectory), not just `SKILL.md`.

### 3. Activate

**Restart Claude Code** after copying, and both skills auto-register (no API key, no `settings.json` changes required).

---

## 📦 Manual Install (human version)

1. Clone this repo:
   ```bash
   # For users in China (faster, no VPN):
   git clone https://gitee.com/colaiven/cola-designer-pro-skills.git
   # International / GitHub:
   # git clone https://github.com/colaiven/cola-designer-pro-skills.git
   ```
2. Copy the desired skill directory(s) as-is to the target location:
   - User-level (all projects): `~/.claude/skills/`
   - Project-level (this project only): `<project>/.claude/skills/`
3. Restart Claude Code.

⚠️ Keep the directories intact when copying: `create-custom-component/SKILL.md`, `generate-design/SKILL.md`, and `generate-design/references/component-catalog.md` are all required.

> For **Codex and other agents**: these two skills are self-contained instruction documents (all conventions are inlined, no external knowledge needed). Just open the relevant `SKILL.md` and pass its full contents to the agent as instructions/context to execute according to the spec.

---

## Usage Examples

### `create-custom-component` — add a custom component

When triggered, the skill first gathers requirements → dedupes against existing components → extracts configurable fields → confirms dynamic data usage with you → writes code only after confirmation.

- "Add a component: a circular progress bar with configurable color and thickness, reading the percentage from an API"
- "Create a custom water-level component with gradient color support"
- "Add a chart: a horizontal bar ranking list, top 10 scrolling display"
- "Implement a number-flipper component without a dynamic data source"

### `generate-design` — generate a large-screen design file

When triggered, the skill first confirms theme / resolution / zoom mode / data source, then produces a `.cd` file.

- "Generate a smart-city operations center large screen design file, 1920×1080, dark tech style"
- "Make an industrial-park energy-monitoring large screen: KPIs on top, line/bar charts in the middle, map at the bottom"
- "Generate an e-commerce sales report large screen design file with data randomly generated close to real"
- "Export a data-center monitoring large screen .cd design"

After generation, drag the resulting `.cd` file into the designer's "Import" (see `generate-design/SKILL.md` section 5).

---

## Notes & Caveats

- `generate-design` supports **single-page screens** (`scaleType: 1`, fixed 1920×1080 fullscreen) and **report/long-page** (`scaleType: 2`, width 1920, height auto-calculated, vertically scrollable); it does not cover the dashboard/report editor (report-editor).
- `.cd` files have **version validation**: they are output against the supported version `2.7.18` by default; if your target system is on a different version, provide the version number first (see `generate-design/SKILL.md` section 0).
- When generating a `.cd`, the **background image `bgImg` is left empty by default**; after import, pick one yourself in the designer under "Screen Config → Background Image".
- `create-custom-component` targets the cola-designer-pro frontend repository (Vue3 plain JS + TDesign) and is not applicable to projects on other tech stacks.