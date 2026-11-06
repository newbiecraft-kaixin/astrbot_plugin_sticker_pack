# astrbot_plugin_sticker_pack

**让 AI 自己发表情包**：回复里写 `:情绪词:`，插件按语义从本地表情库挑一张，单独成条发出去；AI 也可以主动调用 `send_meme` 工具甩图。

AstrBot 4.x 插件。本仓库从 [dafeiyu-open](https://github.com/newbiecraft-kaixin/dafeiyu-open) 拆出，代码相同。

## 它做什么

- **文字匹配推荐**：AI 写 `:笑死:` `:无语:` 这类情绪词，插件拿它和全库每条表情的「名字 + 含义」算字面分数，分数带内随机抽一张 —— 同一个词每次能抽到不同的图
- **单独成条**：表情不塞在句子里，而是像真人一样隔一两秒补发一条
- **工具调用**：提供 `send_meme` 工具，斗图 / 配图 / 表达情绪时 AI 自己调用
- **收图入库**：群聊 / 私聊里别人发的图，插件按算法判断收不收，收进来的进入收藏库
- **查重用指纹、推荐不用**：dHash 只在收藏入库时用于查重（阈值 0.90），**图片相似度完全不参与推荐**，池子不会被相似图绞窄
- **新鲜度加成**：新收的图有分数加成，收进来就能被抽到
- **否定句保护**：「别笑了」这类字面命中但情绪相反的句子，直接不配图

## 安装

1. AstrBot WebUI → 插件 → 插件市场，搜索 `sticker_pack`；
2. 或在「从 URL 安装」里填：`https://github.com/newbiecraft-kaixin/astrbot_plugin_sticker_pack`

## 准备表情库

插件不带图片，需要自己准备：

1. 建一个表情包目录（默认 `data/stickers`，可在配置 `pack_dir` 改）
2. 目录里放 `map.txt` 索引，一行一条：

```text
文字名称:表情包意思:相对图片路径
文字名称:相对图片路径
文字名称:意思:face:QQ黄脸ID
```

3. 图片路径相对 `pack_dir` 解析，也支持绝对路径

索引格式在 [main.py](main.py) 顶部有完整说明。

## 配置

配置项在 AstrBot 插件配置页（`config.json` 同名）。常调的几个：

| 配置 | 作用 |
| --- | --- |
| `pack_dir` / `map_file` | 表情库目录与索引文件名 |
| `follow_up_probability` | 每条文字回复后补一张表情的概率 |
| `follow_up_cooldown_seconds` / `follow_up_per_minute` | 同一群最小间隔、每分钟上限 |
| `max_stickers_per_message` | 一条消息最多带几张 |
| `text_match_core_score` / `text_match_min_score` | 文本匹配的「整词命中」与「扫尾」阈值 |
| `fav_*` | 收藏库自动收集、审核、每日上限 |

## 不用 AI 自动收录（低配 / 手动管库）

不想让机器人自己下载和收录群友发的图，可以整段关掉，改成纯手动维护：

| 配置 | 改成 | 效果 |
| --- | --- | --- |
| `fav_enabled` | `false` | 群聊/私聊收图入口直接返回，不下载、不建收藏库、不跑指纹 |
| `follow_up_probability` | `0` | 关掉「发完文字让 AI 挑一张」的额外模型调用 |
| `fav_vision_check` | `false` | 没有视觉模型时关掉粗筛 |
| `inject_index` | 保持 `false` | 索引不进 system prompt，省 token |

关掉之后 `:情绪词:` 的匹配仍然可用（纯本地文字相似度），`send_meme` 只能从你手动的 `map.txt` 库里挑。`tools/` 下有 `purge_static.py`、`fix_gif_ext.py`、`bench_pool.py`、`sync_qq_faces.py` 可以配定时任务做库维护。

完整的低配降级对照（语音、桌宠、微信桥、模型档位）见 [dafeiyu-open / docs/DEGRADED.md](https://github.com/newbiecraft-kaixin/dafeiyu-open/blob/main/docs/DEGRADED.md)。

## 可选搭配

- `astrbot_plugin_basic_tools` 提供超级表情数据（`data/plugin_data/astrbot_plugin_basic_tools/super_faces.json`）
- `data/plugin_data/fy_send_gate.py` 跨插件发送节流（缺省时不节流）
- `astrbot_plugin_human_sim` 群内拟人回复时会用到本插件

## 已知限制

- 表情库要自己维护，索引写得越准推荐越准
- 只在 `aiocqhttp` 适配器下测试过

## 许可与来源

AGPL-3.0，见 [LICENSE](LICENSE)。

本项目是 [dafeiyu-open](https://github.com/newbiecraft-kaixin/dafeiyu-open) 的一部分，单独拆仓以便插件市场分发。基于 [AstrBot](https://github.com/AstrBotDevs/AstrBot)（AGPL-3.0）的插件接口开发，未分发 AstrBot 本体代码。完整署名见 [NOTICE.md](NOTICE.md)。
