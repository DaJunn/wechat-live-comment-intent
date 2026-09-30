# 视频号回看评论选品意向 Skill

从已结束直播的评论中筛出购买意向和需求提问，保留原句并生成待复核的候选商品清单，支持指定多场汇总。

**本仓库只有操作说明。** 需要另备 `wechat-danmu-collector` 项目，默认目录为 `~/Projects/wechat-danmu-collector`；安装 skill 不会安装采集器。

## 安装 Skill

```bash
git clone https://github.com/DaJunn/wechat-live-comment-intent.git \
  ~/.agents/skills/wechat-live-comment-intent
```

## 怎么用

- 「采这场回看的评论，我来滚动，帮我整理观众想买什么。」
- 「只分析这两场 JSONL，输出候选品类和原句证据。」

新采集需要 Node.js、已登录的视频号 Chrome、Kimi WebBridge，并由用户慢慢滚动评论。已有 JSONL 可直接本地分析。该流程不自动采集直播中的实时弹幕，也不保证评论全量覆盖。

## 输出与注意事项

默认输出意向原句和候选商品两份 CSV。候选词来自规则抽取，需结合原句复核，未命中不代表无需求。

加 `--md` 会同时写本地 Markdown 和 Obsidian；一键入口还会分析目录内全部 JSONL。操作前需核对场次范围与写入目标，采集后需验证文件条数和覆盖缺口。

完整步骤、命令与限制见 [SKILL.md](SKILL.md)。
