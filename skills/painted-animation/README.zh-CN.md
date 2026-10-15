<div align="center">

# painted-animation

<a href="README.md"><img src="https://img.shields.io/badge/English-2B2233?style=for-the-badge" alt="English"></a>
<a href="README.zh-CN.md"><img src="https://img.shields.io/badge/简体中文-D97757?style=for-the-badge" alt="简体中文"></a>

基于 Claude Opus 5.5 的手绘风格动画与歌词 MV 制作技能（Claude Code Skill）。
属于 [opus-video-skills](../../README.zh-CN.md) 合集。

</div>

![小镇姑娘](docs/xiaozhen-sheet.jpg)

## 概述

本技能使 Claude Opus 5.5 能够独立完成动画视频的制作，涵盖分镜设计、角色动画、镜头运动、转场、音乐同步与卡拉 OK 字幕。所有画面均由代码程序化绘制，不依赖任何图像生成模型。

该方法源自 John Heibel 的两个项目：[PDoomVideo](https://github.com/JohnHeibel/PDoomVideo) 和 [ClaudeAnimationBase](https://github.com/JohnHeibel/ClaudeAnimationBase)。前者是一部主要由 Opus 5.5 自主完成的 156 秒音乐视频，后者是在其基础上整理的通用动画工具包。本技能将二者整合为统一的工作流程，并在此基础上增加了歌词 MV 流程，包括节拍检测、音频剪辑与卡拉 OK 字幕。

## 技术原理

1. **素材收集与分镜。** 制作音乐 MV 时，Claude 会先索取歌曲音频和 LRC 歌词文件，再提供故事、情绪、意境、视觉母题和避用内容五项可留空表单。创意字段均可不填。随后编写镜头表，确定场景、配色、角色情绪线与转场方式。每个镜头都列出观众需要依次理解的信息，并为每条信息分配时间。
2. **绘制。** 每个镜头是一个 JavaScript 函数，使用 p5.js 与水彩笔刷库 [p5.brush](https://github.com/acamposuribe/p5.brush) 绘制完整画面。每一帧都由时间唯一确定。
3. **审查。** 少量关键帧拼图、短逐帧序列与局部特写可以在云端渲染，由 Claude 检查画面并修改代码，直至每个镜头符合规范。
4. **输出。** Claude 禁止未经询问直接开始完整渲染，必须让用户明确选择本地或云端。选择本地后禁止云端全量渲染，并改为提供准确的本地执行命令。各帧并行渲染后，由 ffmpeg 编码为 MP4；如提供音频，一并合入。

## 主要特性

- **角色一致性。** 每个角色由单一绘制函数定义，在所有镜头中造型保持一致。
- **逐帧精确的时间控制。** 画面由时间计算得出，事件可精确对齐到节拍乃至单个唱词。
- **局部修改。** 修改某一元素只需调整相应代码并重新渲染受影响的片段，其余部分不受影响。
- **内置动画原理。** 引擎以可复用函数的形式提供预备动作、挤压与拉伸、跟随动作及表情过渡。
- **自动审查。** 渲染画面会依据明确的检查清单逐项核对，涵盖可读性、节奏、接触关系、转场与色彩。

## 示例：陶喆《小镇姑娘》

<p align="center"><img src="docs/koi-gag.gif" width="520" alt="大经理 → 大锦鲤"></p>

本例为该曲一段主歌的 31 秒歌词 MV。输入包括：歌词；参考整首歌上下文的要求；体现歌迷将"大经理"谐音为"大锦鲤"这一梗的要求；原曲音频；LRC 时间轴。

- **场景。** 全片设定在同一座小镇火车站，首尾呼应：开头为一年前女主角乘火车离开，结尾为男主角登上火车离去。
- **视觉线索。** 女主角头上的小花贯穿她的各个形态：车窗中、电视里、锦鲤与星星。
- **节奏。** 经测定，歌曲速度为 154 BPM，每句 8 拍，全部动作均按此节拍剪辑。
- **谐音梗。** 唱到"经理"时，电视画面中的人物变为锦鲤，字幕中的"经理"同时被划去并替换为"锦鲤"；随后锦鲤跃出屏幕，化作下一句中"闪亮的星星"。

分镜与场景代码见 [examples/xiaozhen](examples/xiaozhen/)。因版权原因，仓库中不包含音频。

## 环境要求

- Claude Code 及 Claude Opus 5.5
- Node.js、Google Chrome、ffmpeg
- Python 3 及 numpy（仅节拍检测需要）

## 安装

参见[合集说明](../../README.zh-CN.md#安装)。以插件方式安装：

```
/plugin marketplace add tuzhechen2005/opus-video-skills
/plugin install painted-animation@opus-video-skills
```

## 使用方法

在 Claude Code 中描述所需视频，例如：

> 制作一段 15 秒的动画：Clawd 试图抓住一只蝴蝶。

> 为这首歌制作歌词 MV。（附音频文件、LRC 文件，并说明歌曲的大致故事和意境）

也可通过 `/painted-animation` 直接调用。技能会创建项目、提交分镜、逐个镜头制作并审查，并在完整渲染前询问应在本地还是云端执行。渲染耗时取决于显卡性能；在集成显卡上，水彩填充会明显变慢。

## 目录结构

| 路径 | 说明 |
|---|---|
| `SKILL.md` | Claude 遵循的工作流程与规则 |
| `template/` | 动画引擎：角色、笔刷、镜头、转场、卡拉 OK、渲染器 |
| `scripts/new_project.sh` | 项目初始化与环境检查 |
| `scripts/beat_grid.py` | 节拍与相位检测，并将 LRC 歌词对齐到节拍 |
| `references/music-video.md` | 音乐视频与长篇制作规范 |
| `examples/xiaozhen/` | 上述示例的分镜与场景代码 |

## 致谢

动画引擎与动画指南改编自 John Heibel 的 [ClaudeAnimationBase](https://github.com/JohnHeibel/ClaudeAnimationBase)（MIT 协议，见 [template/LICENSE](template/LICENSE)），整体方法参照其 [PDoomVideo](https://github.com/JohnHeibel/PDoomVideo)。本项目使用了 p5.js、p5.brush、Puppeteer 与 ffmpeg。技能本身及示例由 Claude Opus 5.5 在 Claude Code 中完成。

## 许可协议

MIT，见仓库的 [LICENSE](../../LICENSE)。
