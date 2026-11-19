# 架构审视 HTML 报告格式

> miemou §6 架构深化的配套报告格式，完整迁移自开源 skill improve-codebase-architecture
> 的 HTML-REPORT.md。

架构审视渲染为单个自包含 HTML 文件，写入项目内 `.miemou/`（不入库）。Tailwind 与 Mermaid
都走 CDN。Mermaid 可靠处理图状图示；手工 div 与内联 SVG 处理更编辑性的视觉（质量图、
剖面图）。两者混用：不要什么都靠 Mermaid，会显得模板化。

## 脚手架

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="utf-8" />
    <title>{{仓库名}} 架构审视</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* Tailwind 盖不干净的小型自定义层：
         虚线接缝线、手绘感箭头等 */
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## 页头

仓库名、日期、紧凑图例：实线框 = 模块，虚线 = 接缝，红箭头 = 泄漏，加粗深色框 = 深模块。
不写引言段落，直接进候选。

## 候选卡片

图承担主要重量。文字稀疏、平实，术语直接用 `references/codebase-design.md` 的词汇表，
不加仪式感。

每个候选一个 `<article>`：

- **标题**：短，点出深化动作（如「收敛订单受理管道」）。
- **徽章行**：建议强度（`强` = 翡翠色，`值得探索` = 琥珀色，`投机` = 石板灰），
  外加依赖分类标签（`进程内`、`本地可替换`、`端口与适配器`、`mock`）。
- **文件**：等宽字体清单，`font-mono text-sm`。
- **前后对比图**：核心。两栏并排。见下方模式。
- **问题**：一句话。哪里疼。
- **方案**：一句话。改什么。
- **收益**：条目，每条 ≤6 个词。如「测试只打一个接口」「定价逻辑不再泄漏」「删 4 个浅包装」。
- **ADR 提示框**（如适用）：琥珀底色一行。

不写解释段落。图需要一段话才能看懂，就重画图。

## 图示模式

按候选挑合适的模式，混着用，别让每张图长得一样。多样本身就是目的。

### Mermaid 图（依赖 / 调用流的主力）

重点是「X 调 Y 调 Z，看这一团糟」时用 Mermaid `flowchart` / `graph`。包在 Tailwind 卡片里
别显得突兀。用 classDef 把泄漏边标红、深模块标深。时序图适合「前：6 次往返；后：1 次」。

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.leak.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```

### 手工盒箭头（Mermaid 布局不听话时）

模块用带边框的 `<div>`，箭头用绝对定位在相对容器上的内联 SVG `<line>` / `<path>`。
想让「后」图呈现一个粗边框深模块 + 灰掉的内脏时用这个——Mermaid 渲染不出那个分量。

### 剖面图（适合分层浅）

水平条带堆叠（`h-12 border-l-4`）展示调用穿过的层。前：6 个薄层各啥也不干。
后：1 条厚条带标着合并后的职责。

### 质量图（适合「接口和实现一样宽」）

每个模块两个矩形：接口表面积一个，实现一个。前：接口矩形几乎和实现矩形一样高（浅）。
后：接口矩形矮，实现矩形高（深）。

### 调用图坍缩

前：函数调用树渲染成嵌套盒子。后：同一棵树坍缩成一个盒子，成为内部的调用以褪色显示在盒内。

## 样式指南

- 偏编辑感，不是企业仪表盘。留白大方。标题可用衬线（`font-serif` 与 stone/slate 很搭）。
- 克制用色：一个强调色（翡翠或靛蓝），红给泄漏，琥珀给警告。
- 图高约 320px，前后对比并排放下不用滚动。
- 图内模块标签用 `text-xs uppercase tracking-wider`，读起来是示意图不是 UI。
- 脚本只有 Tailwind CDN 与 Mermaid ESM。其余全静态：无应用代码，无 Mermaid 渲染之外的交互。

## 首选推荐节

一张更大的卡片。候选名、一句为什么、锚链接到它的卡片。就这些。

## 语气

平实、精炼，但架构名词动词全部来自 `references/codebase-design.md` 的词汇表。
精炼不是漂移的借口。

**必须用**：模块、接口、实现、深度、深、浅、接缝、适配器、杠杆、局部性。

**绝不替换成**：组件、服务、单元（说模块时）· API、签名（说接口时）· 边界（说接缝时）·
层、包装（指模块时）。

**符合风格的措辞**：

- 「订单受理模块是浅的：接口几乎追平实现。」
- 「定价跨接缝泄漏。」
- 「深化：一个接口，一处测试。」
- 「两个适配器撑起接缝：生产走 HTTP，测试走内存。」

**收益条目**用词汇表里的词说收益：「局部性：bug 集中到一个模块」「杠杆：一个接口，N 个
调用点」「接口收缩；实现吞掉包装」。不写「更好维护」「代码更干净」——这些词不在词汇表里，
配不上位置。

不模棱两可，不清嗓子，不写「值得注意的是……」。一句话能变条目就变条目；条目能删就删。
词汇表里没有的词，先找一个有的，再考虑造新词。
