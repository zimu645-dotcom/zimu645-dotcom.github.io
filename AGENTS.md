# zimu645-dotcom.github.io — 数据看板站点维护指南

> 本文档是给接手本仓库的 AI 智能体 / 开发者看的**完整上下文**。
> 任何智能体进入本目录即自动读取。**读懂它 = 知道怎么改、怎么验、怎么发布。**

---

## 1. 项目是什么

一个部署在 **GitHub Pages** 的**纯静态数据看板门户**，属于「zimu645-dotcom」个人账号。

- 线上地址：**https://zimu645-dotcom.github.io/**
- 站点性质：电商数据分析可视化大屏合集（家纺行业）
- 技术栈：**纯静态 HTML + CSS + JavaScript + ECharts(CDN)**，无任何构建工具、无后端
- 部署方式：推送到 `main` 分支 → GitHub Pages 自动部署（Projects/legacy，从仓库根目录直接发布）

**一句话：改文件 → git push → 等 Pages 重新构建 → 线上生效。没有第 4 步。**

---

## 2. 站点地图（7 个页面）

| 文件 | 页面 | 说明 |
|------|------|------|
| `index.html` | 门户首页 | 个人介绍 + 6 张数据大屏入口卡片（门面，最常改） |
| `last-month.html` | 电商短板指标对比 | ECharts 柱状图：当前值 vs 行业参考值 |
| `national-day.html` | 国庆营销实时大屏 | 顶部=优化前/实时优化/行业平均动态对比图 + 实时成交、KPI、排名、明细（LIVE） |
| `pareto.html` | 帕累托模型 | 年龄群体消费贡献占比 柱状+折线+80%参考线 |
| `project-results.html` | 项目成果总览 | 转化率对比分析(优化前/实时优化/行业平均动态图) + 直播成效、成果亮点 |
| `malaysia-live.html` | 马来西亚 TikTok 直播 | 实时在线、商品点击率趋势、流量结构（LIVE） |
| `rfm.html` | RFM 八象限三维模型 | 满屏 SVG 三维立方体，无 ECharts |

> **文件名一律用英文小写短名。** 严禁中文件名（会变 URL 编码乱码，`上月数据.html` 是错误的示范）。
> rfm.html 是全屏 SVG（`overflow:hidden`），改布局要单独处理，见 §6。

---

## 3. 统一导航栏规范（重要）

所有页面顶部有一套**相同的固定导航栏**，`class="hz-nav"`，通过 JS 按当前路径自动高亮。

- 品牌区：`<a class="hz-brand" href="index.html">📊 数据看板</a>`
- 导航项：`.hz-links` 内多个 `<a>`，指向各页面
- 高亮逻辑：`script id="hz-nav-js"` 读取 `location.pathname`，给匹配的链接加 `.active`
- 导航是 `position:fixed` 覆盖在页面顶部，**页面 body 必须加 `padding-top:76px`（rfm 页面为 56px）**，否则内容被遮挡

**新增页面时，必须**：
1. 复制导航栏到新页面（`class="hz-nav"`，放 `<body>` 之后）
2. 在 `index.html` 的卡片区 + 所有页面的导航 `.hz-links` 里加上对应入口链接
3. 给新页面 body 加 `padding-top:76px`

---

## 4. 设计风格指南

- **调性**：深色数据大屏 / 科技感。背景深蓝黑 `#0a0e17` 系（各页面 `#08111d`~`#121a30` 之间）
- **点缀色**：青 `#22d3ee`、蓝 `#4f8cf7`、紫 `#a78bfa`，渐变标题 `linear-gradient(120deg,#f8fafc,#67e8f9,#c4b5fd)`
- **卡片**：半透明毛玻璃 `rgba(17,24,39,.72)` + 圆角 `18px` + 细边框 `rgba(148,163,184,.14)` + hover 上浮
- **导航栏**：毛玻璃深色 `rgba(8,12,22,.9)` + `backdrop-filter:blur(14px)` + 底部渐变细线，当前页 `.active` 高亮
- **字号**：整体偏大，标题醒目，信息清晰不拥挤
- **原则**：界面精简，不堆多余按钮；新增元素风格必须与现有深色大屏统一

---

## 5. ⚠️ 环境与陷阱（务必先读，否则必踩坑）

### 5.1 git 全局代理是坏值（本机头号大坑）
- 本机 `git config --global http.proxy` 是 **`http://your-proxy:port`**（无效占位值）
- **任何 `git clone / push / pull` 前，必须先清掉：**
  ```bash
  git config --global --unset http.proxy
  git config --global --unset https.proxy
  ```
- 清掉后直连 github.com 其实是通的（HTTP 200），无需代理

### 5.2 github.com 间歇性被墙
- 直连多数时候通，但偶尔超时。失败就重试 1~2 次，或用内置浏览器访问。
- `gh` CLI 已登录（账号 `zimu645-dotcom`），走 API 通道较稳定。

### 5.3 GitHub Pages 部署有延迟（1~3 分钟）
- push 后**不会立即生效**，需等部署完成。查状态：
  ```bash
  gh api repos/zimu645-dotcom/zimu645-dotcom.github.io/pages/builds/latest --jq '.status'
  ```
- `status` 从 `building` → `built` 才算部署完。**没 `built` 之前验证到的是旧内容，别慌。**

### 5.4 jq 不认中文字段名
- `--jq '{大小: .size}'` 会报错，必须用英文 key：`--jq '{size: .size}'`

### 5.5 CDN 缓存
- 线上有 CDN 缓存，改完立即 curl 可能读到旧版。等 `built` 后验证，必要时加时间戳参数破缓存。

### 5.6 仓库名是全名
- 仓库是 **`zimu645-dotcom.github.io`**（不是 `zimu645-dotcom`），API 路径必须写全，否则 404。

---

## 6. rfm.html 特殊说明

- 全屏 SVG，`html,body{height:100%;overflow:hidden}`
- 注入导航后 body 用 `padding-top:56px;box-sizing:border-box`，SVG 用 `height:calc(100vh - 56px)!important`
- 改它时注意 SVG `viewBox='0 0 1600 900' preserveAspectRatio='xMidYMid meet'`
- 无 ECharts，纯手绘 SVG，改样式要动 SVG 内部 `<style>`

---

## 7. 标准操作流程（改 → 验 → 发）

对**任何修改**，按此流程，缺一不可：

```bash
# 1. 清代理（每次操作前）
git config --global --unset http.proxy 2>/dev/null
git config --global --unset https.proxy 2>/dev/null

# 2. 进入工作区
cd "D:/html各可视化大屏/zimu645-dotcom.github.io"   # 本地工作区(含中文路径,必须加引号)

# 3. 改文件（HTML/CSS/JS）
# 改完【必须先本地验证】

# 4. 提交推送
git add -A
git commit -m "feat: 说明改动"
git push

# 5. 等 Pages 部署完
while :; do
  st=$(gh api repos/zimu645-dotcom/zimu645-dotcom.github.io/pages/builds/latest --jq '.status')
  [ "$st" = "built" ] && break
  sleep 10
done

# 6. 线上验证
curl -s -o /dev/null -w "%{http_code}\n" https://zimu645-dotcom.github.io/<改动的文件>
```

---

## 8. 交付验收标准（必须自己做到，别等用户发现 bug）★

1. **本地验证**：改前改后用浏览器打开 `file:///...`，检查：
   - 图表/内容正常渲染，无空白、无 undefined、无乱码
   - 导航栏存在，当前页正确 `.active` 高亮
   - 内容不被导航遮挡（body 有 `padding-top`）
   - 浏览器控制台无 JS 报错
2. **推送后等 `built`**，再线上 curl / 浏览器确认新内容生效
3. **端到端走一遍**：从门户首页点击入口 → 能跳转到目标页面 → 导航能互相切换
4. 自己全流程测完再交给用户，**别让用户来发现 bug 再返工**

---

## 9. 设计偏好（用户是 boss，务必遵守）

- **字号偏大**，信息清晰，别用爬虫似的 11px 小字
- **界面精简**，不堆功能性冗余按钮
- 新增视觉元素必须与现有深色数据大屏统一（配色、圆角、毛玻璃、渐变）
- 交付前**自己加活**：主动全局自测（含所有交互），一次到位，避免反复返工

---

## 10. 常用命令速查

```bash
gh auth status                      # 确认登录
gh api repos/zimu645-dotcom/zimu645-dotcom.github.io   # 仓库信息
gh api "repos/zimu645-dotcom/zimu645-dotcom.github.io/pages" --jq '{branch, build_type}'  # Pages配置
gh api "repos/zimu645-dotcom/zimu645-dotcom.github.io/git/trees/HEAD?recursive=1" --jq '.tree[].path'  # 文件列表
curl -s -o /dev/null -w "%{http_code}\n" https://zimu645-dotcom.github.io/   # 线上存活检测
```

---

## 11. 动态对比图表（优化前 / 实时优化 / 行业平均）模式与坑

已在 **national-day.html**（国庆营销·实时优化）、**project-results.html**（项目成果·国内渠道）实现。样式：横向分组柱状图，3 个 series。

**数据结构**：每指标 `{ name, before(优化前,固定), target(目标上限), ref(行业平均), current(实时值) }`，`current` 初始 = `before`。

**动态逻辑**：`setInterval(refresh, 1500)`，refresh 里 `current = min(current + inc, target)`（只增不减，逐秒逼近目标）。`inc` 按量级保底：`≥10 → 0.1`；`1~10 → 0.02`；`<1 → 0.005`。

**更新图表（★最关键，别踩）**：
```js
// ✅ 正确：更新第2个 series(实时优化)
myChart.setOption({ series: [{}, { data: COMPARE_DATA.map(d => d.current) }] });
// ❌ 错误：`[{data}]` 会默认合并到 series[0](优化前)，导致两系列数据颠倒！
myChart.setOption({ series: [{ data: ... }] });
```
series 顺序：0=优化前(灰 `rgba(160,190,235,0.32)`)、1=实时优化(青渐变 `#00d4ff→#4a7cf7`)、2=行业平均(金 `rgba(240,192,64,0.82)`)。

**⚠️ 标签重叠坑**：3 个值接近且都较小（如京东主图点击率 1.5/1.6/2.2）时，柱子贴太近→右侧数值标签重叠。解法：
- 每个 series 加 `barGap: '70%'`，category 轴加 `barCategoryGap: '45%'`，拉开柱距
- `grid.right` 加大（如 118），给长标签留空间

**⚠️ 文件名规范事故（吃过亏）**：曾有人把 `national-day.html` 重命名为中文 `国内营销数据.html` 推上线，导致 `/national-day.html` **404**（所有导航/门户都链向英文名）。教训：**严守英文小写短名**（§2），杜绝中文文件名。若线上出现中文名文件导致 404，用 `git mv 中文名.html 英文名.html` 改回并重推。

---

**读完这份文档，你应该能独立完成：改任意页面、加新大屏页面、改导航、调整风格、安全发布并自测。**
