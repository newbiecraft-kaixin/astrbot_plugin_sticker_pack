# astrbot_plugin_sticker_pack

AstrBot 4.x 插件，把回复里的 `:情绪词:` 换成本地表情包，单独发一条；也提供 `send_meme` 工具，让模型主动发图。

仓库从 [dafeiyu-open](https://github.com/newbiecraft-kaixin/dafeiyu-open) 拆出，代码相同。

## 功能

- 匹配只看文字：拿情绪词和每条表情的“名字 + 含义”算字面分数，分数带内随机抽一张，同一个词每次可能抽到不同的图
- 单独成条：表情不拼在句子里，隔一两秒补发
- 工具调用：`send_meme`，模型要斗图或配图时自己调用
- 收图：群聊、私聊里的图片按规则判断收不收，收下的进入收藏库
- 查重和推荐分开：dHash（阈值 0.90）只在收藏入库时查重，不参与推荐，避免相似图挤掉其他候选
- 新收的图有分数加成
- 否定句不配图，例如“别笑了”

## 安装

1. WebUI → 插件 → 插件市场，搜索 `sticker_pack`
2. 或在“从 URL 安装”里填 `https://github.com/newbiecraft-kaixin/astrbot_plugin_sticker_pack`

## 准备表情库

插件不带图片，需要自己准备：

1. 新建表情包目录，默认 `data/stickers`，可在配置 `pack_dir` 修改
2. 目录里放 `map.txt` 索引，一行一条：

```text
文字名称:表情包意思:相对图片路径
文字名称:相对图片路径
文字名称:意思:face:QQ黄脸ID
```

3. 图片路径相对 `pack_dir` 解析，也支持绝对路径

索引格式在 [main.py](main.py) 顶部有完整说明。

## 配置

配置项在 AstrBot 插件配置页。常用的几项：

| 配置 | 作用 |
| --- | --- |
| `pack_dir` / `map_file` | 表情库目录与索引文件名 |
| `follow_up_probability` | 每条文字回复后补一张表情的概率 |
| `follow_up_cooldown_seconds` / `follow_up_per_minute` | 同一群的最小间隔、每分钟上限 |
| `max_stickers_per_message` | 一条消息最多带几张 |
| `text_match_core_score` / `text_match_min_score` | 整词命中与扫尾的匹配阈值 |
| `fav_*` | 收藏库的收集、审核、每日上限 |

## 不用 AI 自动收录（低配或手动管库）

不想让机器人自己下载、收录群里的图片，可以关掉这部分，改成手动维护：

| 配置 | 改成 | 效果 |
| --- | --- | --- |
| `fav_enabled` | `false` | 群聊和私聊的收图入口直接返回，不下载图片、不建收藏库、不算指纹 |
| `follow_up_probability` | `0` | 不再让模型为每条回复挑图 |
| `fav_vision_check` | `false` | 没有视觉模型时关掉粗筛 |
| `inject_index` | 保持 `false` | 索引不塞进 system prompt，省 token |

关掉之后 `:情绪词:` 的匹配照常可用，只做本地文字计算；`send_meme` 只能从手动库里挑。`tools/` 下有 `purge_static.py`、`fix_gif_ext.py`、`bench_pool.py`、`sync_qq_faces.py`，可以配定时任务维护库。

其他降级项（语音、桌宠、微信桥、模型）见 [dafeiyu-open / docs/DEGRADED.md](https://github.com/newbiecraft-kaixin/dafeiyu-open/blob/main/docs/DEGRADED.md)。

## 搭配的其他插件（可选）

- `astrbot_plugin_basic_tools`：提供超级表情数据（`data/plugin_data/astrbot_plugin_basic_tools/super_faces.json`）
- `data/plugin_data/fy_send_gate.py`：跨插件发送节流，没有就不节流
- `astrbot_plugin_human_sim`：群内拟人回复时会调用本插件

## 已知问题

- 表情库需要自己维护，索引写得越准，推荐越准
- 只在 `aiocqhttp` 适配器下测试过

## 许可与来源

AGPL-3.0，见 [LICENSE](LICENSE)。

本项目是 [dafeiyu-open](https://github.com/newbiecraft-kaixin/dafeiyu-open) 的一部分，单独建仓库为了发布到插件市场。开发时使用 [AstrBot](https://github.com/AstrBotDevs/AstrBot)（AGPL-3.0）的插件接口，没有分发 AstrBot 自身的代码。署名见 [NOTICE.md](NOTICE.md)。
