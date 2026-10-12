---
name: sanhao-motion-video
description: 三好口播动效视频制作系统。把口播稿变成 3:4 竖屏（1080×1440）动效知识分享视频：HyperFrames 渲染 + 中文女声配音 + 三好PPT视觉（米白+墨绿+橙棕+五段彩条）+ 流式追加排版。当用户说"做动效视频"、"知识分享视频"、"把口播稿做成视频"、"复用上次那条视频的流程"、"视频工作流"时触发。不适用：真人数字人口播（走 HeyGen 流程）、纯图文物料。
version: 1.0.0
---

# 三好口播动效视频工作流

把一份口播稿 → 3:4 竖屏动效视频（mp4），配音激情感中文女声、三好品牌视觉、流式追加排版。

## 标准流程（六步）

### Step 1 确认输入与规格
- 必须有：口播稿全文（用户给或从选题系统产出）
- 固定规格：3:4 竖屏 1080×1440、30fps、时长≈稿子朗读时长+10%
- 先出 5-6 秒样片给用户确认风格，**确认后才做全片**（用户明确要求跳过除外）

### Step 2 配音生成（TTS）
**声音优先级**（踩坑排序，2026-10-12 实战验证）：
1. **edge-tts `zh-CN-XiaoyiNeural`**（激情感最好、免费、用户认可过的声线）
   - 分段参数：标题句 rate +24% pitch +6Hz；内容句 +18% ~ +20% pitch +4~5Hz；收尾句 +22% +5Hz
   - ⚠️ **红线：批量前必须告知用户请求量**（每段间隔≥2秒，≤20段/批，否则 IP 被微软 403 长封 30min+）
2. **被限流时的替代**：本地 Kokoro（`npx hyperframes tts -v zf_xiaobei -s 1.25`，需 `HYPERFRAMES_PYTHON=<venv路径>（建议 uv venv --python 3.12 创建）/bin/python`）——情绪平淡但无限流
3. ElevenLabs（用户有 key 时；免费版无中文原生声线，充 $5 才能用中文库声音）

**关键工艺（必做，否则声音拖沓）**：
- edge-tts/Kokoro 每段尾部藏 0.5~0.7s 静音 → 修剪：`ffmpeg -af "silenceremove=start_periods=1:start_threshold=-45dB:start_silence=0.06,areverse,silenceremove=start_periods=1:start_threshold=-45dB:start_silence=0.06,areverse"`
- 句间停顿 0.28~0.3s，用 ffmpeg concat filter 拼接（**勿用 mp3 concat demuxer**，会 dts 损坏）：
  `ffmpeg -i a.mp3 -i b.mp3 -f lavfi -t 0.28 -i anullsrc=r=24000:cl=mono -filter_complex "[0:a][2:a][1:a]concat=n=3:v=0:a=1[out]" -map "[out]"`
- 数字转中文写法再送 TTS（4→四、10→十、2→两）保发音准确
- **逐段实测时长**（ffprobe），记 timing.json：每场景总长 + 场景内每句起点（动效对齐用）

### Step 3 编写 HyperFrames 合成
项目骨架：`npx hyperframes init <名> --non-interactive --example blank` + 复制 package.json/meta.json/hyperframes.json 即可。

三好视觉参数（固定，勿改）：
- 米白底 #F7F5EF；墨绿 #1F4D33；橙棕 #C87A3E；亮橙 #E0A15E；中绿 #2E6B4A；灰绿 #7FA98C
- 顶部五段彩条（等宽 flex）：#1F4D33 / #2E6B4A / #C87A3E / #E0A15E / #7FA98C（三好签名，每屏必须有）
- 标题 "Songti SC" 900（必须 @font-face src:local('Songti SC') 声明，否则 lint error）；正文 "Hiragino Sans GB"
- 白底圆角卡 + 阴影 `0 14px 40px rgba(31,77,51,0.13)`；深底卡片上文字用米黄 rgba(232,213,168,1)
- 装饰：角落淡彩弧圈 + 点阵；页脚<个人品牌落款>（按自己品牌定制）

排版铁律（用户 10-12 定稿）：
- **流式追加**：同一场景内要点卡逐条弹入且保留在屏，绝不一点一屏
- **内容整列水平垂直居中**，禁止左侧堆放右侧留空

动效库（发布会质感，GSAP）：
- 标题逐字弹入：字符拆 span，`fromTo {scale:0.4,y:30} {scale:1,y:0} stagger 0.07~0.09 back.out(1.7)`
- 卡片追加：`fromTo {y:80,scale:0.82} back.out`；序号章：`{scale:0,rotate:±140}→0`
- 数字跳字：tween 对象 onUpdate 更新 textContent；下划线扫描 width 0→n
- 数据条依次 scaleX 生长；强调元素 `to {scale:1.25} yoyo repeat`
- ⚠️ 脉冲类用 `to` 不要 `fromTo`（lint 警告 gsap_repeated_fromto_without_baseline）

HyperFrames 契约硬规则：
- `<audio>` 必须带 id；必须写实测 data-duration（写长了报 clip_media_fit）
- 每条配音独立 track-index（2~8），避免同轨重叠警告
- GSAP 脚本**本地化**到 assets/gsap.min.js（npmmirror 装 gsap 包拷贝），CDN 在 DNS 污染时会超时
- repeat 必须**有限次数**（repeat:-1 警告）
- 动效时间点按 Step 2 的 seg_starts 精确对齐口播

### Step 4 质检门禁
```
npx hyperframes check        # 0 error 才继续（对比度、布局溢出、动效全审）
npx hyperframes snapshot --at <各场景中点>   # contact-sheet 目检
```
- 0 error 但有结构性警告（建议拆子合成/文件过大）可忽略，单体合法
- 装饰出血画布的 panel_out_of_canvas 警告若是设计意图可留

### Step 5 渲染
```
npx hyperframes render --quality high --output renders/final.mp4
```
- 78s 全片约 1.5~5 分钟（硬件 GPU）
- ⚠️ 报 "FFmpeg cannot start"/"Chrome cannot start" 时：先手动测 `/usr/local/bin/ffmpeg -version`，**直接重试即过**（高负载假死，二进制无恙），勿重装

### Step 6 验片交付（全绿才交付）
- ffprobe：1080×1440、时长、h264+aac
- **音轨分段 RMS 抽验**（防叠音/静音，历史翻过车）：每场景中点取 2s 窗 RMS，应全部 2000~4000（单轨水平）；叠音≈单轨 2.7 倍，静音=0
- 关键帧 ffmpeg 抽 JPG 目检（布局、彩条、卡片、文字无溢出）
- `cp` 到 outputs/ + present_files + `npx hyperframes feedback --rating --comment`
- 交付后问用户：是否沉淀/下一步

## 场景结构模板（知识分享类）
1. Hook：三数字大字（时长/数量/成本）+ 打字机标题 + 行动指令
2. 每个工作流/要点：徽章"工作流 0N" + 大标题逐字弹入 + 下划线 + 要点卡流式追加（含数字跳字/评分条/流程链等丰富图形）
3. CTA：利益卡阵列弹入 + 行动指令
4. 结尾：关键词大字块（如"实战"）脉动 + 关注引导 + 品牌行

## 环境依赖（本机已就绪）
- HyperFrames CLI 0.8.149（npx 缓存）；Chrome headless shell ~/.cache/hyperframes/
- Kokoro venv：<venv路径>（建议 uv venv --python 3.12 创建）（uv + python3.12 + kokoro-onnx，tuna 镜像）
- edge-tts 7.2.8（系统与 Kokoro venv 都有）
- ffmpeg/ffprobe /usr/local/bin（evermeet 8.0.1）
- 代理：Clash mixed-port 7890（微软语音服务需好节点；国内直连 IP 也常被拒）

## 历史坑速查
| 症状 | 根因 | 解法 |
|---|---|---|
| 声音重叠混乱 | 多音轨同帧齐播 | 每段独立 track-index + data-start 定位 |
| 后半段无声 | 音轨 start 参数被忽略/写错 | 实测时长写 data-duration，check 门禁验证 |
| 声音拖沓温柔 | TTS 尾部静音未修剪 | silenceremove 双向修剪 + 0.28s 停顿拼接 |
| edge-tts 403 | IP 限流（密集请求触发） | 等 30min~数小时；勿重试轰炸；预防=段间隔≥2s+提前报备 |
| render 报 FFmpeg/Chrome cannot start | 高负载假死 | 直接重试；检查 uptime>50 时等负载降 |
| 合并后时长=首段×N | zsh 里 `fc+="[$i:a]"` 的 `:a` 被当成历史修饰符（转绝对路径），filter 串变成 `[/abs/path/0]` | 一律写 `fc+="[${i}:a]"`，用花括号隔开 |
| trim 脚本把已修剪文件再修剪一遍（出现 `-t-t.wav`） | `for f in voiceover/s*.wav` 通配把派生文件也匹配进去 | 派生物放独立子目录（如 voiceover/final/），或通配收紧为 `s[0-9]p[0-9].wav` |
| 标题里 `<span class="em">` 的字在成片消失 | splitChars 只遍历文本节点，元素子节点被 `el.textContent=""` 清掉 | splitChars 改递归：遇元素节点带 class 下钻，字符 span 上保留原 class |
| 多时刻快照只出一帧 | 重复 `--at` 只有最后一个生效；且快照每次运行会覆盖上一次的 frame-00 | 用逗号分隔一次传：`snapshot --at 5,11,24,45` |
| zsh 脚本里 `$HF -s ...` 全部 FAIL | zsh 不对未加引号变量做分词，整串被当成一个命令名 | 函数里直接写全命令，或数组展开 `${(@)arr}` |
| Kokoro 提速档读完时长偏短（390字仅84s） | `-s 1.22~1.25` 太快，凑不满"约2分钟" | 改 `-s 1.05~1.08`，加场景留白后约 105~110s |
| edge-tts 403（直连+代理都拒） | 早间密集请求的 IP 级封禁可延续数小时，换 Clash 节点也无效 | 别重试轰炸；当天直接切本地 Kokoro 交付，事后再换声 |
| lint font_family error | 命名字体无声明 | @font-face src:local() 声明系统字体 |
| GSAP 加载超时 | jsdelivr DNS 污染 | 本地化 assets/gsap.min.js |

## 验证
成品必须同时满足：check 0 error；snapshot 目检通过；规格 1080×1440/30fps；音轨 RMS 各场景单轨水平；用户听过配音确认风格。
