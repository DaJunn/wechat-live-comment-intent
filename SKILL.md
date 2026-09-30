---
name: wechat-live-comment-intent
description: >-
  采集视频号「直播回看·数据大屏」左侧「全部评论」的历史评论(直播已结束、不用开播),
  清洗后筛出「购买意向 + 需求提问」类评论,自由抽取观众想要的商品/品类词,
  支持连续多场回放汇总成「选品意向清单」,输出本地 CSV/MD + Obsidian。
  当用户说「采这场回看的评论」「把某主播几场直播的观众意向整理成选品清单」
  「看观众都想买啥」「视频号回看评论选品」「多场评论选品意向」时使用。
  注意:这是「直播结束后回看页」的评论采集;若是「直播进行中」实时弹幕采集,走同项目的
  src/collector.mjs(npm start)。两者共用复盘/分析。
---

# 视频号回看评论 · 选品意向

把一个带货者若干场**已结束**直播的回看评论扒下来,筛出观众明确「想买/在问」的,
汇成选品意向清单。代码复用本机已有项目 `~/Projects/wechat-danmu-collector`(别另起炉灶)。

## 适用 / 不适用
- ✅ 直播**已结束**,进数据大屏「复盘回看」,左侧「全部评论」——本 skill。
- ❌ 直播**进行中**实时弹幕 → 用同项目 `npm start`(src/collector.mjs,playwright 拦接口)。

## 前提
- Chrome 已登录视频号助手(channels.weixin.qq.com)。
- kimi-webbridge daemon 在跑:`~/.kimi-webbridge/bin/kimi-webbridge status`(running+extension_connected)。
- 项目已 `npm install`(回看采集只用 Node 原生 fetch,不需要 playwright;实时采集才需要)。

## 怎么拿 objectId
页面路径:**数据详情 → 数据大屏趋势 → 复盘回看**。
回看页地址栏:`.../dashboardV4/review?objetctId=【这一串就是 objectId】`(注意官方拼写是 objetctId)。
多场就收集多个 objectId。也可直接把整条 review 网址丢给脚本。

## 关键约束:半自动(人手滚 + 脚本同步抓)
回看的「全部评论」是 vue-recycle-scroller 虚拟列表,**懒加载分页只认真实人手滚轮**——
实测合成 WheelEvent、改 scrollTop、CDP `Input.dispatchMouseEvent(mouseWheel)`、点击聚焦、
键盘 PageDown、拉视频进度/倍速播放,**全都触发不了加载**(列表冻在首批 ~16 条);只有用户
用鼠标真滚才会一批批刷出来。且虚拟列表 DOM 里只留约 16 行,滚过的会被回收。
所以采集必须:**脚本高频轮询 DOM 抓 + 用户同步用鼠标把评论从头滚到底**,按 data-index 去重收全。
(这条已多轮实测确认,别再尝试全自动滚动。)

## 标准流程(一次一场)

```bash
cd ~/Projects/wechat-danmu-collector

# 1) 起抓取窗口(秒数给够你滚到底,如 180);脚本会自动导航到该场回看页
node src/capture-while-scroll.mjs <objectId 或 review网址> 180
#    → 脚本一跑起来,立刻去浏览器把左侧「全部评论」从最顶【慢慢滚到底】,
#      滚到不再冒新评论;滚完可在终端按 Ctrl+C 提前落盘 → replays/<objectId>.jsonl
#    多场就对每场各跑一次(各自滚一遍)

# 2) 出选品意向(多场汇总),写本地 CSV/MD + Obsidian
node src/select-intent.mjs replays/*.jsonl --md
```

懒人:双击 `回看选品.command`,粘 objectId、给秒数,按提示滚,自动出报告。

## 口径(已定,改口径就改脚本顶部词典)
- **过滤口径**:分析前先去掉纯表情/空评论、1-2 字无意义短句、短句空话、完全重复内容/疑似刷屏;原始 jsonl 仍保留全量评论方便复核。
- **意向范围**:购买意向(我要/怎么买/链接/上车/多少钱…) + 需求提问(有没有/求推荐/哪个好/含问号…)。词典在 `src/select-intent.mjs` 顶部 `BUY`/`NEED`,可自由增删。
- **商品归类**:纯自由抽取——不依赖本场讲解列表,从评论里剥意图词后抽残留中文词作候选,**含噪声,以原句为准、人工复核**。
- **输出**:`reports/选品意向-原句-*.csv`、`reports/选品清单-候选-*.csv`、`reports/选品意向-*.md`,并写 Obsidian
  `经营运营/直播评论选品意向/`(库路径可用环境变量 `OBSIDIAN_VAULT` 覆盖)。

## 技术要点(改版维护点)
- 数据大屏是 **wujie 微前端**,评论在同源 iframe(`empty.html`)里——脚本自动钻 iframe.contentDocument。
- 评论列表是 **vue-recycle-scroller 虚拟滚动**(`.comment__list`),只渲染约 16 行;脚本高频(~250ms)轮询渲染中的行,按 `data-index`(全量真实序号)去重累积,人滚到哪收到哪。
- 单条评论:`.message-username-desc`(昵称) / `.message-content`(正文,表情转 `[名]`) / 行尾 `HH:MM`(直播内时刻) / 标签 `买过N单`·`粉丝`·`回复`。
- 评论不在任何单一接口里一次性返回(实测 `get_live_dashboard_basic_info` 等都不带全量),是播放器随回看逐步吐 + 滚动懒加载,所以走 DOM 抓而非接口。
- 平台改版若采不到:先 `kimi-webbridge status`;再看选择器(`.comment__list` / `.review-comment-item` / `.message-content`)是否被改名。

## 边界
- 只读 DOM,不发言、不操作中控台、不破解接口,账号风险最低。
- 不编造数据;评论里 `**` 是平台对昵称的脱敏,原样保留。
- 闲聊场/社群场可能抽不到选品词(正常,说明那几场没明确商品意向)。
