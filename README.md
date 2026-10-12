# sanhao-motion-video 🎬

> 三好口播动效视频制作 Skill —— 把一份口播稿变成 **3:4 竖屏动效知识分享视频**

一个 [Agent Skills](https://agentskills.io) 开放格式的技能包：让任何支持 SKILL.md 的智能体（千问办公 / Claude Code / Codex CLI 等）自动完成

**配音生成 → 品牌视觉合成 → 质检门禁 → 渲染 → 验片交付** 的全流程。

## 成片效果

| Hook 开场（数字计数器+逐字弹入） | 工作流场景（流式追加要点卡） |
|:---:|:---:|
| ![Hook](docs/scene-hook.jpg) | ![Workflow](docs/scene-workflow.jpg) |

| AI 复盘场景（数据条生长动效） | CTA 场景（利益卡阵列） |
|:---:|:---:|
| ![Review](docs/scene-review.jpg) | ![CTA](docs/scene-cta.jpg) |

*规格：1080×1440 · 30fps · H.264+AAC · 约 70 秒 · 配音激情感中文女声*

## 核心特性

- 🎙️ **配音方案三级备选**：edge-tts 激情女声（免费）→ 本地 Kokoro（无限流）→ Azure 官方免费层（同款声音根治限流）
- ✂️ **广播级配音工艺**：尾静音双向修剪 + 0.28s 精确句间停顿 + 分段变速变调（钩子提速、收尾留白）
- 🎨 **品牌视觉系统**：米白底 + 墨绿宋体大标题 + 橙棕强调 + 顶部五段彩条签名 + 白底圆角卡片
- 📐 **流式追加排版**：要点卡随口播逐条弹入且保留在屏，内容整列居中——杜绝"一点一屏"和半屏留白
- 🎬 **发布会级动效库**：标题逐字弹入、序号章旋转盖章、数字跳字、评分条渐填、数据条生长、下划线扫描
- ✅ **自动质检门禁**：lint/布局溢出/WCAG 对比度/动效 seek 一键全审 + 音轨分段 RMS 验片（防叠音防静音断档）

## 快速开始

```bash
# 1. 把 sanhao-motion-video/ 文件夹放进你的智能体技能目录
#    QwenWork: ~/.qwenworkcn/skills/  |  Claude Code: ~/.claude/skills/  |  Codex: ~/.codex/skills/

# 2. 环境准备（一次性）
brew install ffmpeg          # 视频编码
pip install edge-tts         # 免费中文配音（注意限流，段间隔≥2秒）

# 3. 对你的智能体说：
#    "用这份口播稿做一条动效视频"
```

智能体会自动：分段配音 → 修剪拼接 → 生成 HyperFrames HTML 合成 → 质检 → 渲染 MP4 → 验片交付。

详细的环境依赖、配音参数配方和踩坑速查表见 [SKILL.md](SKILL.md)。

## 六步工作流

```
口播稿 ──► ① 确认规格（先出5秒样片）──► ② 配音生成（分段+修剪+停顿拼接）
       ──► ③ 编写合成（品牌视觉+动效对齐口播）──► ④ 质检门禁（check 0 error）
       ──► ⑤ 渲染（--quality high）──► ⑥ 验片（规格+RMS+抽帧目检）──► 交付
```

## 适用与不适用

| ✅ 适用 | ❌ 不适用 |
|---|---|
| 知识分享/干货口播视频 | 真人数字人口播（走 HeyGen） |
| 短视频获客/自媒体内容 | 纯图文物料 |
| 多场景多要点讲解 | 10 秒内纯动效短片 |

## License

MIT · 作者 [sanhao-ai](https://github.com/sanhao-ai)
