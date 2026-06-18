# Tag Graph Links（标签连线）

一个 Obsidian 插件：让**拥有相同标签**的笔记在关系图谱里自动连线。
比如文章 A 和文章 B 都有 `#色调` 标签，那么图谱里 A 和 B 之间就会出现一条线。
这条线可以单独设置**颜色**，和普通的双向链接区分开。

全局图谱（Graph View）和局部图谱（Local Graph）都生效。图谱左上角的控制面板里有一个链接图标，点开即可快速切换显示/隐藏连线、就地修改连线颜色，无需进设置页。

## 效果对比

| 插件关闭 | 插件开启 |
|---------|---------|
| <img width="441" height="312" alt="插件前" src="https://github.com/user-attachments/assets/ffee1ff4-0071-49f7-9ce7-f35faee3ef8f" /> | <img width="339" height="252" alt="插件后" src="https://github.com/user-attachments/assets/0e7f4308-4e95-4955-a13f-84d2db4c4b35" /> |

「一隅矣：孤芳」和「旅行人像1」共享 `#色调` 标签，开启后两者之间出现一条红色连线。

> 提示：在 GitHub 网页编辑 README 时，把光标放到上面表格对应的单元格文字上，直接把图片文件拖进编辑框，GitHub 会自动上传并生成图片链接，替换掉「把…拖到这里」这行字即可。

---

## ⚠️ 必读：技术现实

Obsidian 官方插件 API **不支持**往关系图谱里加自定义边、也不支持给单条边单独上色。
本插件靠 hook（拦截）Obsidian 的**内部、未公开接口**来实现，这意味着：

- **连线功能**（让共享标签的笔记连起来）是社区验证过的成熟做法，比较稳。
- **颜色 / 粗细**依赖更深的渲染细节，是「尽力而为」。在某些 Obsidian 版本上字段名可能对不上，导致连线画出来了但颜色没变。遇到这种情况，用插件自带的「诊断」命令把渲染器结构打印出来，按你的版本校准即可（见下文）。
- Obsidian 大版本升级后，插件可能需要跟着改。

如果只是想要「共享标签的笔记产生关联」，这个功能很稳；颜色是锦上添花。

---

## 一、准备开发环境（从零开始）

### 1. 安装 Node.js

去 https://nodejs.org 下载 **LTS 版本**（macOS 选 .pkg，一路下一步）。
装完打开「终端」（Terminal）验证：

```bash
node --version   # 应该显示 v18 或更高
npm --version
```

### 2. 把这个项目放到合适的位置

把整个 `tag-graph-links` 文件夹放到你电脑上任意目录（比如 `~/Documents/`）。
建议**先在这里开发构建**，再把产物拷进 Obsidian（见第三步）。

### 3. 安装依赖

在终端里 `cd` 进项目文件夹，然后：

```bash
cd 路径/tag-graph-links
npm install
```

这会下载 esbuild、typescript、obsidian 类型定义等开发依赖（只在开发时用）。

### 4. 构建插件

```bash
npm run build
```

成功后项目根目录会生成一个 **`main.js`**。这是 Obsidian 实际加载的文件。

> 开发调试时可以用 `npm run dev`，它会进入监听模式，改完 `main.ts` 自动重新构建。

---

## 二、装进 Obsidian

一个 Obsidian 插件，运行时只需要 3 个文件：

```
main.js        ← npm run build 生成的
manifest.json  ← 插件信息
styles.css     ← 样式（本插件基本用不到，但要在）
```

### 步骤

1. 找到你的库（vault）文件夹，进入隐藏目录 `.obsidian/plugins/`
   （如果没有 `plugins` 文件夹就新建一个）。
2. 在里面新建一个文件夹 `tag-graph-links`。
3. 把上面 3 个文件拷进去，最终结构：

   ```
   你的库/.obsidian/plugins/tag-graph-links/
   ├── main.js
   ├── manifest.json
   └── styles.css
   ```

4. 打开 Obsidian → 设置 → 第三方插件（Community plugins）→ 关闭「安全模式」（如果还开着）。
5. 在「已安装插件」列表里找到 **Tag Graph Links**，打开开关。

> macOS 下 `.obsidian` 是隐藏文件夹，在访达里按 `Cmd + Shift + .` 显示隐藏文件。

---

## 三、使用

1. 打开关系图谱（左侧边栏图标，或命令面板搜「图谱」）。
2. 给若干笔记打上相同标签，比如都写 `#色调`。
3. 这些笔记之间就会出现标签连线。

### 图谱里的快捷开关

图谱左上角的控制面板（有「中心力」「连线力」那些滑块的面板）里会多出一个**链接图标**。点它会弹出一个小面板，上下两行：

- **标签连线**：开关，控制连线显示/隐藏。
- **颜色**：取色器，直接改连线颜色，图谱实时生效。

这个面板和设置页里的选项是双向同步的，改哪边都一样。

### 命令面板（Cmd/Ctrl + P）里可用的命令

- **刷新标签连线** —— 改了设置后图谱没变化时手动刷新。
- **开启 / 关闭标签连线** —— 快速开关。
- **诊断：打印图谱渲染器结构到控制台** —— 见下文校准。

### 设置项（设置 → Tag Graph Links）

- **启用标签连线**：总开关。
- **连线颜色**：标签边颜色。
- **单标签最大连线数**：性能保护。某个标签下笔记太多时，两两连线数量会爆炸（n 篇笔记 = n×(n-1)/2 条边）。超过这个阈值的标签会被整体跳过。默认 200。
- **忽略的标签**：列在这里的标签不参与连线，逗号分隔。适合排除 `#笔记`、`#待办` 这种几乎每篇都有的标签。
- **调试日志**：在开发者控制台输出注入了多少边、跳过了哪些标签。

---

## 四、如果连线出现了但颜色 / 粗细没生效

这说明你的 Obsidian 版本里，渲染器内部字段名和插件的假设不一致。校准方法：

1. 打开一个关系图谱。
2. 命令面板运行「**诊断：打印图谱渲染器结构到控制台**」。
3. 按 `Cmd/Ctrl + Shift + I` 打开开发者控制台，找到 `===== Tag Graph Links 诊断 =====` 那段。
4. 把里面打印的 `renderer keys`、`第一条 link 的字段`、`link.line 类型` 等信息贴出来。

根据这些信息，`main.ts` 里 `applyEdgeStyles()` 方法中对 `link.line.tint` 的处理就能改成你版本对应的字段，重新 `npm run build` 即可。

---

## 六、开源到 GitHub（网页拖拽上传）

不用装任何工具，全程在浏览器里完成。

### 1. 新建一个空仓库

1. 登录 GitHub，右上角 `+` → **New repository**。
2. Repository name 填 `tag-graph-links`（或你喜欢的名字）。
3. 选 **Public**（公开才算开源）。
4. **不要**勾选 "Add a README"、"Add .gitignore"、"Choose a license"——因为这些文件项目里已经有了，勾了反而会冲突。
5. 点 **Create repository**。

### 2. 把文件拖进去

建好后页面上有一行链接 **"uploading an existing file"**，点它，进入上传页（或直接打开 `你的仓库地址/upload`）。

然后把项目文件夹里的文件拖进网页。**注意以下两点：**

**✅ 要上传这些文件：**

```
main.ts            ← 源码
manifest.json
package.json
tsconfig.json
esbuild.config.mjs
styles.css
versions.json
README.md
LICENSE
.gitignore
main.js            ← 构建产物，建议一起传，方便别人直接下载用
```

**❌ 不要上传：**

- `node_modules/` 文件夹（几百兆的依赖，体积大且没必要，`.gitignore` 已经标了忽略它）
- `data.json`（如果有，是你的本地设置）

> 小技巧：在访达/文件管理器里进入项目文件夹，框选**除 `node_modules` 外**的所有文件，一次性拖到网页的上传区即可。GitHub 网页上传不支持拖文件夹，但你这个项目是扁平结构，直接拖文件没问题。

### 3. 提交

拖完后页面下方有个 "Commit changes" 区域，填一句说明（比如 `首次提交`），点 **Commit changes**。完成，仓库就建好了。

### 4. （可选）改一下作者信息

`LICENSE` 文件里有一行 `Copyright (c) 2026 你的名字`，`manifest.json` 里 `"author": "you"`，可以在 GitHub 网页上点开这两个文件、点铅笔图标直接改成你的名字，再 commit。

### 5. （可选）以后想让别人在 Obsidian 里一键安装

如果想把它提交到 Obsidian 官方社区插件市场（让所有人能在 Obsidian 里搜到安装），需要额外走官方审核流程，门槛较高。先放着，等你想做的时候我再带你走。现在这个公开仓库，别人已经可以手动下载 `main.js`、`manifest.json`、`styles.css` 来用了。

---

## 七、文件说明

| 文件 | 作用 |
|------|------|
| `main.ts` | 插件源码（你改这个） |
| `manifest.json` | 插件元信息（id、名称、版本） |
| `package.json` | npm 依赖与构建脚本 |
| `tsconfig.json` | TypeScript 配置 |
| `esbuild.config.mjs` | 打包配置 |
| `styles.css` | 样式占位 |
| `versions.json` | 版本兼容表 |
| `LICENSE` | 开源许可证（MIT） |
| `main.js` | **构建产物**，Obsidian 实际加载的文件 |

---

## 工作原理（给好奇的你）

1. 插件扫描全库每篇笔记的标签（正文 `#标签` 和 frontmatter `tags` 都算），建立「标签 → 笔记列表」索引。
2. 拦截图谱渲染器的 `setData(data)` 方法。`data.nodes` 是以笔记路径为 key 的对象，每个节点的 `links` 字段记录它连到哪些节点。插件在这里把「共享同一标签」的笔记两两写进彼此的 `links`，于是图谱就画出了这些边。
3. 记下哪些边是「标签边」。渲染器每帧重画连线，插件用 `requestAnimationFrame` 持续把这些标签边重新上色，盖过默认样式。

MIT License.
