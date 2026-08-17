# K线逐根回放 · 技术选型验证 Demo

## 项目目标
金手指双盲测试系统的K线回放功能技术选型验证。用户点击"下一根"逐条揭示K线（不是自动播放），
需要支持成交量/MACD/KDJ/BOLL/成交额等副图指标的显示切换，手机/平板/电脑三端自适应（含手机横竖屏），
支持明暗主题。目标是从以下三个候选中选出最合适的图表库：

- **方案一：klinecharts** (v10)
- **方案二：ECharts** (v6，多 grid 堆叠实现多副图)
- **方案三：Lightweight Charts**（TradingView 出品，v5，原生多 pane）

## 如何运行
纯前端单文件 Demo，无需构建：

```bash
cd claudespace
python3 -m http.server 8791
# 浏览器打开 http://localhost:8791/index.html
```

三个库通过运行时动态 `<script>` 加载（npmmirror → jsdelivr → unpkg，ECharts 额外有 cdnjs），
**不是**打包进本地文件，所以本地测试仍需要正常访问这些 CDN 的网络环境。这也是这份代码要挪到本地
用真实网络测试的主要原因——云端沙箱环境完全没有出网权限（curl/npm 全部 403），只能靠一个走
Anthropic 代理的 WebFetch 工具查资料，没法真正跑起来验证。

## 文件说明
- `index.html` —— 完整 Demo（数据模拟、三套图表引擎、副图指标切换、主题、响应式布局、诊断日志全在这一个文件里）
- `K线逐根回放技术选型方案.docx` —— 配套技术方案文档（对比表格、指标切换难度评分、踩坑记录、已知限制、下一步建议），后续会持续更新

## 当前已知问题（需要在本地用真实网络复验）
1. **klinecharts 反复加载失败**：已定位并修复过一次 CDN 路径问题（v10 正确路径是
   `dist/umd/klinecharts.min.js`，不是 `dist/klinecharts.min.js`），并把 v9 时代的
   `applyNewData()` 全部改成 v10 的 `setDataLoader({getBars, subscribeBar, unsubscribeBar})`
   模型。但用户最近一次反馈是**三个引擎同时加载失败**（此前 ECharts 明明刚成功过一次），
   这个和代码改动对不上，怀疑是网络环境/CDN 可达性的问题，需要在本地真实网络下重新验证，
   而不是继续在沙箱里盲改代码。
2. **真实历史数据拉取失败**：尝试从 `stooq.com` 拉 CSV（AAPL），报错 "Load failed"，大概率是
   该接口对任意来源有 CORS 限制。目前 Demo 会自动降级用模拟数据，不影响核心功能验证，但如果
   需要真实数据体验，这块还没解决。
3. **移动端横竖屏**：CSS 已经写了对应的 media query（`max-width:640px` 竖屏 / `max-height:500px
   and orientation:landscape`），但还没有在真实手机设备上实测过。

## 建议下一步（如果用 Codex CLI 协助）
- 优先在本地起服务器实测三个引擎能否稳定加载（不同网络环境/时间点多测几次），确认是否还有真实
  bug，还是纯粹网络抖动
- 如果 klinecharts / stooq 问题仍然存在，可以直接在 `index.html` 里对应的
  `LIB_SOURCES` / `fetchRealHistoricalData` 附近排查网络请求详情（浏览器 DevTools Network 面板
  会比云端沙箱里的模拟更可靠）
- 三个引擎 + 副图指标切换的评分标准写在 `index.html` 底部的 `.findings` 卡片里，以及 docx 文档
  的"副图指标显示/隐藏难度评分"章节，如果实测后结论有变化，两处都要同步更新
- 真机测试手机横竖屏、平板、桌面三种视口下的布局
