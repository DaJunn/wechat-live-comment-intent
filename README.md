# wechat-live-comment-intent

**回看评论挖选品意向** —— 清洗评论、筛出购买意向与需求提问，汇总成选品意向清单

让 AI agent（DSH / Codex / Claude Code 等）使用。

## 解决什么问题

之前想知道「观众到底想买什么」，只能凭印象，或者一条条翻评论。

现在采集直播回看的全部评论（直播已结束、不用开播），清洗后筛出「购买意向 + 需求提问」类评论，自由抽取观众想要的商品 / 品类词，支持连续多场汇总成**选品意向清单**。

## 前置依赖

1. **kimi-webbridge daemon** 在跑：
   ```bash
   ~/.kimi-webbridge/bin/kimi-webbridge status   # 要 running:true + extension_connected:true
   ```
   没装：`curl -fsSL https://cdn.kimi.com/webbridge/install.sh | bash`
2. **Chrome 已登录`channels.weixin.qq.com`（视频号直播回看·数据大屏）**。本系列只复用你自己已打开的标签页，**不代登录**。

## 安装

```bash
git clone https://github.com/DaJunn/wechat-live-comment-intent.git \
  ~/.agents/skills/wechat-live-comment-intent
```

## 触发方式

对 agent 说：「采这场回看的评论」「把某主播几场直播的观众意向整理成选品清单」「看观众都想买啥」「视频号回看评论选品」。


## 说明

- ⚠️ **评论列表是懒加载**，键盘 PageDown / 调倍速都触发不了加载。需要**用户用鼠标把评论从头滚到底**，脚本同步轮询抓取并按 data-index 去重
- 这是「直播结束后回看页」的评论；直播进行中的实时弹幕采集走同项目 `src/collector.mjs`

## 相关

- 完整技能合集见飞书文档《AI减负视频号运营技能合集》
- 更多 skill：https://github.com/DaJunn
