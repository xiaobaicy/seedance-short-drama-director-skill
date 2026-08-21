# Seedance 2.0 中文微短剧导演 Skill

一套面向 **Seedance 2.0 中文真人微短剧** 的导演、分镜、表演、连续性和生成Prompt编译系统。

它不是简单的“提示词扩写器”，而是把剧情转换为可以审核、生成、验收和连续制作的完整执行方案：

- 真人演员的眼神、微表情、呼吸、手势与身体重心
- 多人镜头中谁先动、谁后动、谁保持静止
- 景别、机位、焦段感、运镜速度、起点与终点
- 首帧生图Prompt和Seedance视频Prompt
- 角色、服装、道具、空间、视线、声音连续性
- 上一镜尾帧到下一镜首帧的镜间接缝设计
- 动作复杂度评分、自动拆镜和失败诊断

## 适用场景

- 古装、宫斗、权谋、仙侠
- 都市、霸总、豪门、逆袭、悬疑
- 真人短剧、剧情广告、带货剧情、品牌短片
- 多镜连续生成、续拍、首尾帧衔接
- 人脸漂移、眼神乱飘、手部异常、动作不执行、多人抢戏、运镜跳变等返修

不适用于非Seedance模型的专用参数配置，也不用于纯图片提示词任务。

## 核心工作流

```text
剧情意图
  ↓
导演判断与复杂度评分
  ↓
动作时间轴 + 镜头执行参数
  ↓
首帧生图Prompt + Seedance视频Prompt
  ↓
生成并验收本镜
  ↓
提取真实稳定尾帧
  ↓
更新镜间接缝卡和下一镜首帧
```

多镜项目遵守一条关键规则：

> N个生成Clip必须有N−1张镜间接缝卡；真实稳定尾帧高于事先计划尾帧。

因此，它不会把一部短剧处理成互不相关的若干条Prompt。

## 安装

### Windows PowerShell

```powershell
git clone https://github.com/xiaobaicy/seedance-short-drama-director-skill "$env:USERPROFILE\.codex\skills\seedance-short-drama-director"
```

### macOS / Linux

```bash
git clone https://github.com/xiaobaicy/seedance-short-drama-director-skill ~/.codex/skills/seedance-short-drama-director
```

安装后重新打开Codex任务，或开始一个新任务，让Skill重新进入可用列表。

### 更新

进入本地Skill目录后执行：

```bash
git pull
```

## 调用方法

明确调用：

```text
$seedance-short-drama-director 优化这套85秒中文真人微短剧分镜。
```

严格生产版：

```text
$seedance-short-drama-director 严格优化这套分镜，输出动作时间轴、镜头执行参数、首帧生图Prompt、Seedance视频Prompt，并为所有相邻镜头生成镜间接缝卡。
```

只要可复制Prompt：

```text
$seedance-short-drama-director 隐藏导演分析，只输出每镜首帧生图Prompt、Seedance视频Prompt和相邻镜头接缝。
```

失败诊断：

```text
$seedance-short-drama-director 诊断这条生成视频里人脸漂移、眼神乱飘和动作不执行的问题；每次只修改一个主变量。
```

## 每镜输出内容

| 内容 | 用途 | 是否直接提交给生成模型 |
|---|---|---|
| 导演判断 | 明确本镜剧情功能与注意力中心 | 否 |
| 复杂度评分 | 判断是否过载、是否需要拆镜 | 否 |
| 动作时间轴 | 审核人物动作顺序与静止要求 | 已编入视频Prompt |
| 镜头执行参数 | 审核景别、机位、焦段感、运镜和终点 | 已编入视频Prompt |
| 首帧生图Prompt | 生成本镜第0秒静态起点 | 提交给生图工具 |
| Seedance视频Prompt | 驱动人物动作、镜头和声音 | 提交给Seedance |
| 镜间接缝卡 | 检查上一尾帧和下一首帧 | 用于校正下一镜Prompt |
| 验收标准 | 选择可进入连续性正史的take | 否 |

## 镜间接缝系统

相邻镜头会明确记录：

- 怎么切：原位续拍、停稳切镜、动作匹配、正反打、视线匹配、道具匹配、声音桥等
- 上一镜如何结束
- 下一镜第0秒如何开始
- 必须继承的人物、手势、视线、道具、光线和声音
- 允许改变的景别、机位和构图范围
- 跨镜动作从哪个相位继续
- 已经完成、不得重演的动作和台词
- 不能出现的人物换边、动作复位、道具瞬移和声音重启

上一镜生成后，必须观察最后0.5–1秒。如果最后一帧变形、模糊或处在转场中，应选择更早的稳定帧并修剪尾部，不能把坏帧继续传给下一镜。

## 目录结构

```text
seedance-short-drama-director-skill/
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
├── references/
│   ├── performance-vocab.md
│   ├── emotion-library.md
│   ├── camera-library.md
│   ├── short-drama-recipes.md
│   ├── complexity-scoring.md
│   ├── continuity-system.md
│   ├── shot-seam-system.md
│   ├── generation-handoff.md
│   └── failure-atlas.md
└── examples/
    └── imperial-wine-85s-zh.md
```

## 参考模块

### 表演词库

`performance-vocab.md`包含300条可拍摄表演表达，覆盖：

- 眼神与视线
- 眼部、眼睑与眉部
- 嘴部、下颌、头部与颈部
- 手势、身体与重心
- 呼吸、声音节奏、人际互动和动作收势

### 情绪与潜台词库

`emotion-library.md`包含50种状态。每种提供可见表演拆解、L1–L4强度、推荐镜头、禁忌写法和示例Prompt。

### 镜头语言库

`camera-library.md`覆盖景别、机位、焦段感、推拉摇移跟环绕、速度、起点、终点、情绪映射与常见失败。

### 中文微短剧配方

`short-drama-recipes.md`覆盖宫斗、霸总、逆袭、悬疑、仙侠、古装权谋、甜宠、家庭伦理、职场、校园、带货剧情和品牌广告等结构。

### 连续性与镜间接缝

`continuity-system.md`负责记住跨镜状态；`shot-seam-system.md`负责把上一镜自然切到下一镜。两者共同锁定角色、服装、道具、左右关系、视线、动作相位、摄影机和声音。

### 复杂度与故障诊断

`complexity-scoring.md`负责风险评分和自动拆镜；`failure-atlas.md`负责定位人脸、眼神、手部、动作、运镜、参考素材和连续性故障，并限制每次只改一个主要变量。

## 完整案例

[`examples/imperial-wine-85s-zh.md`](examples/imperial-wine-85s-zh.md) 是一套85秒宫廷剧情广告案例，包含：

- 20个实际生成Clip
- 20套动作时间轴
- 20套镜头执行参数
- 20条首帧生图Prompt
- 20条Seedance视频Prompt
- 19张镜间接缝卡

## 设计原则

1. 描述摄像机能看到或听到的行为，不堆抽象情绪词。
2. 短镜头默认一个焦点人物、一个主动作、一个情绪变化和一个主运镜。
3. 多人镜头写清谁先动、谁后动、谁保持静止。
4. 动作有起点、触发、过程、收势和可观察终点。
5. 产品文字英雄镜与旋转、开瓶、手部遮挡分开生成。
6. 只有已验收take进入连续性正史。
7. 计划状态不能覆盖生成视频中已经看见的真实状态。

## 灵感与原创说明

本Skill综合研究了以下社区项目的导演化、镜头设计、Prompt组织和连续生成思路：

- [Emily2040/seedance-2.0](https://github.com/Emily2040/seedance-2.0)
- [Woodfantasy/Seedance2.0-ShotDesign-Skills](https://github.com/Woodfantasy/Seedance2.0-ShotDesign-Skills)
- [MapleShaw/seedance2.0-prompt-skill](https://github.com/MapleShaw/seedance2.0-prompt-skill)
- [dexhunter/seedance2-skill](https://github.com/dexhunter/seedance2-skill)

仓库内容为面向中文真人微短剧的原创整合与增强，并非上述仓库原文复制。

## 说明

- 本项目不是Seedance、字节跳动或OpenAI官方项目。
- 社区经验不代表模型内部机制或官方能力保证。
- 平台界面、参数、时长与声音能力可能变化，实际制作以当前入口为准。
- 使用真人肖像、声音、品牌、产品和版权角色前，请确认拥有相应授权。

