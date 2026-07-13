# C5 TTS 文本转语音模块

> **东北大学 NEU NLP | 小牛翻译 NiuTrans.com**
>
> 模型：CosyVoice2-0.5B | 采样率：24000Hz

---

## 一、模块定位

C5是整套实训的输出收尾模块：**接收文本 → 输出语音**。

工作流很简单：
```
C3/C4 输出 cascade_results.json
        ↓
  python c5_tts.py --dataset cascade_results.json
        ↓
  outputs/s2s_*.wav（语音文件）
```

---

## 二、环境配置

```bash
# 1. 独立conda环境（Python 3.10，与其他模块隔离）
conda create -n cosyvoice python=3.10 -y && conda activate cosyvoice

# 2. 克隆CosyVoice仓库（务必加 --recursive）
git clone --recursive https://github.com/FunAudioLLM/CosyVoice.git

# 3. 安装依赖（setuptools必须<81，否则whisper编译失败）
pip install "setuptools<81"
cd CosyVoice && pip install -r requirements.txt

# 4. 下载模型权重（modelscope: iic/CosyVoice2-0.5B）

# 5. 设置环境变量
export COSYVOICE_REPO=/你的路径/CosyVoice
export COSYVOICE_MODEL=/你的路径/CosyVoice2-0.5B
```

---

## 三、基本使用

### 3.1 单句合成

```bash
cd C5_TTS/code

python c5_tts.py --text "今天天气很好" -o output.wav          # 中文
python c5_tts.py --text "Hello world" --lang en -o en.wav      # 英文
python c5_tts.py --text "今天的meeting讨论AI" --lang mix       # 中英混合
python c5_tts.py --text "你好" --speed 1.5 -o fast.wav         # 调语速
python c5_tts.py --text "太棒了" --emotion happy               # 带情感
```

### 3.2 批量合成（对接C3/C4）

```bash
# C3/C4同学把翻译结果给你一个JSON → 你跑下面这行命令就行了
python c5_tts.py --dataset ../C3_cascade/outputs/cascade_results.json

# JSON里只需要有这些字段之一就行（按优先级）：
#   translation_zh      → C3级联翻译结果（优先用这个）
#   e2e_translation_zh  → C4端到端翻译结果
#   text                → 通用文本字段
```

生成文件：`outputs/lang_zh.wav`、`outputs/lang_en.wav`、`outputs/s2s_*.wav`、`outputs/c5_summary.json`

---

## 四、进阶功能

### 4.1 一键运行全部

```bash
python c5_tts.py --all-advanced
```

生成以下输出（全部在 `outputs/` 下）：

| 输出目录 | 内容 | 对应进阶任务 |
|---------|------|------------|
| `multilingual/` | 中/英/日/粤/韩/混合 6语种音频 | ① 多语种 |
| `speed_timbre/` | 5档语速 + 多音色对比 | ② 语速音色 |
| `emotion/` | 7种情感 + 4级强度梯度 | ③ 情感控制 |
| `speaker_clone/` | Zero-Shot克隆 + 时长对比 | ④ 说话人 |
| `benchmark/` | 多模型横向对比 | ⑤ 横向对比 |
| `report/` | 可视化图表（PNG） | — |

### 4.2 单独运行某项

```bash
python c5_tts.py --multilingual-test    # 进阶①: 多语种
python c5_tts.py --speed-test           # 进阶②: 语速5档
python c5_tts.py --timbre-test          # 进阶②: 音色对比
python c5_tts.py --emotion-test         # 进阶③: 7种情感
python c5_tts.py --emotion-intensity    # 进阶③: 情感强度
python c5_tts.py --speaker-test         # 进阶④: 说话人克隆
python c5_tts.py --benchmark            # 进阶⑤: 横向对比
```

---

## 五、进阶功能详细说明

### 5.1 多语种（6种）

| 语言 | 代码 | 示例 |
|------|------|------|
| 中文 | zh | 人工智能正在改变我们的生活方式 |
| 英文 | en | Artificial intelligence is transforming... |
| 日语 | ja | 人工知能は私たちの生活様式を変革し... |
| 粤语 | yue | 人工智能改变紧我哋嘅生活方式... |
| 韩语 | ko | 인공지능은 우리의 생활 방식을 변화시키고... |
| 中英混合 | mix | 今天的meeting讨论AI的roadmap |

### 5.2 语速控制（5档）

| 档位 | Speed值 |
|------|--------|
| very_slow | 0.70 |
| slow | 0.85 |
| normal | 1.00 |
| fast | 1.20 |
| very_fast | 1.50 |

### 5.3 情感控制（7种）

| 情感 | 效果 |
|------|------|
| happy | 开心愉悦，语调上扬 |
| sad | 悲伤低沉，语速缓慢 |
| angry | 严肃愤怒，语气坚定 |
| gentle | 温柔亲切，娓娓道来 |
| calm | 平静中性，客观播报 |
| excited | 激动兴奋，声音洪亮 |
| tired | 慵懒疲惫，有气无力 |

每种情感支持4级强度（Lv.1 ~ Lv.4），通过 `--emotion-intensity` 测试。

### 5.4 说话人控制（Zero-Shot克隆）

- 给3秒以上参考音频即可复刻音色
- 支持跨性别克隆、多说话人切换
- `--voice-ref /path/to/audio.wav` 指定参考音频
- `--voice-id 中年男` 使用已注册音色（先 `--register-voice` 注册）

---

## 六、输入输出规范

### 输入（你接收什么）

| 方式 | 示例 |
|------|------|
| 直接文本 | `--text "你好世界"` |
| JSON文件 | `--dataset cascade_results.json`（C3/C4输出） |
| 参考音频 | `--voice-ref speaker.wav`（音色克隆） |

### 输出（你产出什么）

| 产物 | 格式 |
|------|------|
| 语音文件 | WAV, 24kHz, 16bit, 单声道 |
| 统计文件 | JSON（含每条RTF、耗时、音频时长） |
| 可视化图表 | PNG（多语种/情感/语速/说话人对比图） |

---

## 七、代码结构

```
C5_TTS/
├── README.md                    # 本文档
├── code/
│   ├── c5_tts.py               # 主入口（基础+进阶全部功能）
│   ├── multilingual.py         # 进阶① 多语种
│   ├── speed_timbre.py         # 进阶② 语速+音色
│   ├── emotion_control.py      # 进阶③ 情感控制
│   ├── speaker_clone.py        # 进阶④ 说话人克隆
│   ├── benchmark.py            # 进阶⑤ 横向对比
│   ├── visualize_results.py    # 可视化（生成7张PNG图表）
│   ├── c5_api.py               # 可选的Python封装（供需要函数调用的场景）
│   └── voice_bank/             # 音色库
├── docx/
│   └── readme.md               # 实训说明（原始需求文档）
└── outputs/                    # 所有输出产物
```

---

## 八、关于"对外接口"

readme要求"提供接口给组内其他模块调用"。你的接口就是 **CLI命令行参数**：

```
C3/C4同学 → 给你 cascade_results.json
              ↓
你不需要写任何新代码，直接跑：
   python c5_tts.py --dataset cascade_results.json
              ↓
自动读取JSON中每条翻译文本 → 逐条合成语音 → 保存 outputs/s2s_*.wav
```

**`--dataset` 参数就是你对外提供的接口。** 上游只要按约定格式输出JSON，你就能直接消费。

`c5_api.py` 是同一个功能的Python封装版本（可选），供需要 `import c5_tts` 在代码中调用的场景使用，不是必选项。

---

## 九、验收清单

| 要求 | 状态 | 怎么验证 |
|------|------|---------|
| 基础任务1: 跑通TTS | ✅ | `python c5_tts.py --text "你好" -o out.wav` |
| 基础任务2: 批量合成 | ✅ | `python c5_tts.py --dataset xxx.json` |
| 基础任务3: 中英双语 | ✅ | 自动生成 `lang_zh.wav` + `lang_en.wav` |
| 基础任务4: 对接C3 | ✅ | `--dataset` 读C3翻译结果 |
| 基础任务5: RTF统计 | ✅ | `c5_summary.json` |
| 进阶①: 多语种 | ✅ | `--multilingual-test` |
| 进阶②: 语速音色 | ✅ | `--speed-test` + `--timbre-test` |
| 进阶③: 情感控制 | ✅ | `--emotion-test` |
| 进阶④: 说话人 | ✅ | `--speaker-test` |
| 独立运行 | ✅ | 所有命令可独立执行 |
| 独立演示 | ✅ | 命令行 + 可视化图表 |
| 有实验对比 | ✅ | 进阶实验RTF/F0/相似度分析 |

---

## 十、常见问题

**Q: 找不到COSYVOICE_REPO？**
A: 确认已 `export COSYVOICE_REPO=/你的路径/CosyVoice`

**Q: setuptools报错？**
A: `pip install "setuptools<81"`，新版setuptools≥81会导致whisper编译失败

**Q: GPU显存不够？**
A: CosyVoice2-0.5B约需2-3GB显存，不够可用CPU推理（会慢很多）

**Q: 怎么换音色？**
A: `--voice-ref /path/to/speaker.wav`（给3-10秒参考音频即可）

---

> **C5_TTS v3.0.0** | 东北大学 NEU NLP × 小牛翻译 NiuTrans.com