<div align="center">

# opus-video-skills

<a href="README.md"><img src="https://img.shields.io/badge/English-0E0F0E?style=for-the-badge" alt="English"></a>
<a href="README.zh-CN.md"><img src="https://img.shields.io/badge/简体中文-DDF53D?style=for-the-badge" alt="简体中文"></a>

基于 Claude Opus 5.5 的手绘动画视频制作技能（Claude Code Skill）。

</div>

## 概述

本仓库中的技能能让 Claude Opus 5.5 独立完成手绘动画视频，涵盖分镜、动画、审查、歌词同步与最终编码。所有画面均由代码程序化生成，不依赖任何图像或视频生成模型。每一帧都由时间唯一确定，在无头 Chrome 中渲染，并由 ffmpeg 编码输出。

## 技能

### painted-animation

![painted-animation](skills/painted-animation/docs/xiaozhen-sheet.jpg)

使用 p5.js 与水彩笔刷库 p5.brush 绘制每个镜头。角色依托一套表情库与动画原理进行表演，音乐视频按测得的节拍剪辑，歌词 MV 配有逐字卡拉 OK 字幕。示例为陶喆《小镇姑娘》的 31 秒歌词 MV。[说明文档](skills/painted-animation/README.zh-CN.md)。

## 环境要求

- Claude Code 及 Claude Opus 5.5
- Node.js、Google Chrome、ffmpeg
- Python 3 及 numpy（仅 painted-animation 的节拍检测需要）

## 安装

**以插件方式安装（推荐）。** 在 Claude Code 中执行：

```
/plugin marketplace add tuzhechen2005/opus-video-skills
/plugin install painted-animation@opus-video-skills
```

之后用 `/plugin update` 获取新版本。

**以个人技能方式安装。** 克隆仓库一次，再链接所需的技能：

```bash
git clone https://github.com/tuzhechen2005/opus-video-skills ~/opus-video-skills
ln -s ~/opus-video-skills/skills/painted-animation ~/.claude/skills/painted-animation
```

之后在 `~/opus-video-skills` 中执行 `git pull` 即可更新。

> **从 `painted-animation` 升级。** 本仓库原先是单一的 `painted-animation` 技能，安装方式是直接克隆到 `~/.claude/skills/painted-animation`。该技能现已移至 `skills/painted-animation/`，原有安装方式不再有效。请删除旧的克隆，再按上述任一方式重新安装。

## 使用方法

在 Claude Code 中描述所需视频，系统会自动选用匹配的技能，也可以按名称直接调用：

> 制作一段 15 秒的动画：Clawd 试图抓住一只蝴蝶。

制作音乐 MV 时，请提供歌曲音频和 LRC 歌词文件；故事、情绪、意境、视觉母题和避用内容采用可留空表单。技能会创建项目、提交分镜并逐个镜头审查。未经用户明确选择，禁止开始完整渲染；选择本地后禁止云端全量渲染，并提供准确的本地执行命令。

## 目录结构

| 路径 | 说明 |
|---|---|
| `.claude-plugin/marketplace.json` | 插件市场清单 |
| `skills/painted-animation/` | 水彩动画技能：引擎、指南与示例 |

## 新增风格

新增一种风格，即在 `skills/` 下新建一个目录，包含 `SKILL.md`、可直接运行的 `template/` 以及至少一个完整示例，并在 `.claude-plugin/marketplace.json` 中添加对应条目。现有技能遵循同一套约定：每一帧都是时间的纯函数；`render.mjs` 负责生成关键帧拼图和并行渲染全部帧；最终编码之前，每个镜头都要依据渲染出的画面审查。

## 致谢

painted-animation 的动画引擎与指南改编自 John Heibel 的 [ClaudeAnimationBase](https://github.com/JohnHeibel/ClaudeAnimationBase)（MIT 协议），整体方法参照其 [PDoomVideo](https://github.com/JohnHeibel/PDoomVideo)。本项目使用了 p5.js、p5.brush、Puppeteer 与 ffmpeg。技能及示例由 Claude Opus 5.5 在 Claude Code 中完成。

## 许可协议

MIT，见 [LICENSE](LICENSE)。
