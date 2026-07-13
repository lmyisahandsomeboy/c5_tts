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

# 代码架构详解与API使用方式

## 十一、代码文件结构总览

```
C5_TTS/code/
├── c5_tts.py              # ★ 主入口脚本 — 基础合成+批量合成+全部进阶功能调度
├── c5_api.py              # ★ 统一对外API模块 — 为C1~C4及其他外部系统提供标准化接口
├── multilingual.py        # 进阶① 多语种支持验证（zh/en/ja/yue/ko/mix）
├── speed_timbre.py        # 进阶② 语速控制 + 音色注册与切换
├── emotion_control.py     # 进阶③ 7种情感控制 + 强度梯度 + 声学特征分析(F0/语速/能量)
├── speaker_clone.py       # 进阶④ Zero-Shot说话人克隆 + 跨性别 + 时长影响分析
├── benchmark.py           # 进阶⑤ CosyVoice2 vs IndexTTS2 横向对比
├── visualize_results.py   # 可视化 — 生成RTF柱状图、声学特征雷达图、情感对比图等
├── voice_bank/            # 音色库目录（存放自定义注册音色的JSON配置）
├── outputs/               # 输出目录（基础合成、批量合成、各进阶测试的音频+JSON结果）
└── log/                   # 运行日志目录（runtime.log，JSON Lines格式）
```

---

## 十二、核心模块详解

### 12.1 c5_tts.py — 主入口脚本（683行）

**定位**：C5_TTS模块的总控脚本，整合了基础合成、批量合成以及全部6条进阶功能的调度逻辑。既可独立命令行运行，也是 `c5_api.py` 的底层引擎。

#### 关键函数/组件一览

| 函数/变量 | 说明 |
|---|---|
| `COSYVOICE_REPO` / `MODEL_DIR` | 环境变量读取CosyVoice仓库路径和模型权重目录；带默认回退值和启动校验 |
| `load_model(model_dir, verbose)` | 加载CosyVoice2模型，返回`AutoModel`实例，打印采样率和加载耗时（~27s GPU） |
| `synth_and_save(gen, out_path, sr)` | 消费生成器、拼接音频张量、保存WAV；返回(耗时, 路径, 音频时长) |
| `basic_synthesize(...)` | **核心合成函数**：文本→WAV，支持11个参数（文本/语言/语速/情感/指令/参考音频/音色ID/流式等），按优先级路由到不同的推理接口 |
| `batch_synthesize(...)` | 批量合成：解析C3/C4的JSON结果，逐条提取文本字段（`translation_zh`→`e2e_translation_zh`→...），自动匹配情感标签和参考音频路径 |
| `run_multilingual_test()` | 调用`multilingual.py`的多语种测试 |
| `run_speed_test()` | 调用`speed_timbre.py`的语速测试（5档预设：0.5x~2.0x） |
| `run_timbre_test()` | 调用`speed_timbre.py`的音色测试（从参考音频目录采样多音色对比） |
| `run_emotion_test()` | 调用`emotion_control.py`的7种情感验证 |
| `run_emotion_intensity_test()` | 调用`emotion_control.py`的情感强度梯度测试（4级强度） |
| `run_speaker_clone_test()` | 调用`speaker_clone.py`的说话人克隆（时长影响+跨性别对比） |
| `run_benchmark()` | 调用`benchmark.py`的模型横向对比（CosyVoice2 vs IndexTTS2） |
| `run_all_advanced()` | 一键串行运行上述全部进阶测试，带异常隔离（单个失败不影响后续） |
| `main()` | argparse CLI入口，覆盖全部参数的帮助文档和分支路由 |

#### basic_synthesize 推理路由逻辑

`basic_synthesize()` 根据传入参数的不同组合，自动选择CosyVoice2最合适的推理接口：

```
输入参数解析
  ├── voice_ref存在 & 有效路径  → inference_cross_lingual (音色克隆优先)
  ├── instruct存在              → inference_instruct2    (自定义指令模式)
  ├── emotion存在               → inference_instruct2    (情感标签→指令映射)
  ├── voice_id在spk2info中      → inference_zero_shot    (注册音色)
  ├── lang=="en"               → inference_cross_lingual (添加<|en|>token)
  ├── lang=="mix"              → inference_cross_lingual (中英混合不加token)
  └── 默认(中文)                → inference_zero_shot    (常规零样本)
```

#### CLI完整参数列表

| 参数组 | 参数 | 类型 | 说明 |
|---|---|---|---|
| **基础** | `--text` | str | 直接输入合成文本（单句模式） |
| | `--dataset` | str | C3/C4 JSON结果路径（默认自动查找C3级联结果） |
| | `-o / --output` | str | 输出WAV路径 |
| | `--speed` | float | 语速倍率（0.5~2.0，默认1.0） |
| | `--lang` | str | 语言模式：zh / en / mix |
| | `--voice / --voice-ref` | str | 参考音频路径（音色克隆） |
| | `--voice-id` | str | 已注册音色ID |
| | `--stream` | flag | 启用流式合成 |
| | `--emotion` | str | 情感：happy/sad/angry/gentle/calm/excited/tired |
| | `--instruct` | str | 自定义instruct2指令 |
| **进阶** | `--multilingual-test` | flag | ① 多语种 |
| | `--speed-test` | flag | ② 语速 |
| | `--timbre-test` | flag | ② 音色 |
| | `--emotion-test` | flag | ③ 情感 |
| | `--emotion-intensity` | flag | ③ 情感强度梯度 |
| | `--speaker-test` | flag | ④ 说话人克隆 |
| | `--benchmark` | flag | ⑤ 横向对比 |
| | `--all-advanced` | flag | 一键全部进阶 |
| | `--ref-audio-dir` | str | 参考音频目录 |
| **音色库** | `--list-voices` | flag | 列出已注册音色 |
| | `--register-voice` | str | 注册新音色名称 |
| | `--register-wav` | str | 注册音色的参考音频 |
| | `--register-text` | str | 注册音色的参考文本 |

#### 使用示例

```bash
# 环境配置
conda activate cosyvoice
export COSYVOICE_REPO=/path/to/CosyVoice
export COSYVOICE_MODEL=/path/to/CosyVoice2-0.5B

# 单句合成（中文默认音色）
python c5_tts.py --text "今天天气很好，我们一起去公园散步吧。" -o hello.wav

# 英文跨语言合成
python c5_tts.py --text "The weather is really nice today." --lang en -o hello_en.wav

# 语速调节（1.5倍速）
python c5_tts.py --text "你好世界" --speed 1.5 -o fast.wav

# 情感合成（开心语气）
python c5_tts.py --text "太棒了！我们成功了！" --emotion happy -o happy.wav

# 自定义指令合成
python c5_tts.py --text "各位观众晚上好" --instruct "用新闻主播的专业播音腔朗读，庄重大气" -o broadcast.wav

# 音色克隆（指定参考音频）
python c5_tts.py --text "欢迎使用语音合成系统" --voice /path/to/ref_speaker.wav -o cloned.wav

# 流式合成
python c5_tts.py --text "这是一段很长的文本用于测试流式合成效果" --stream -o stream.wav

# 批量合成（对接C3级联翻译结果）
python c5_tts.py --dataset ../../C3_cascade/outputs/c3_predictions.json -o ./batch_output/

# 批量合成（对接C4端到端翻译结果）
python c5_tts.py --dataset ../../C4_end2end/outputs -o ./batch_output/

# 一键运行全部进阶测试
python c5_tts.py --all-advanced

# 音色管理
python c5_tts.py --list-voices
python c5_tts.py --register-voice "中年男" --register-wav /path/to/speaker.wav --register-text "参考文本"
```

---

### 12.2 c5_api.py — 统一对外API模块（942行）

**定位**：C5_TTS的标准化对外接口层，为组内C1~C4模块以及外部系统提供7个明确接口。支持三种调用方式：Python函数调用、命令行JSON stdin/stdout管道、JSON文件驱动管线。

#### C5TTSAPI 类 — 7个核心接口

| 接口 | 方法 | 输入 | 输出 | 用途 |
|---|---|---|---|---|
| **接口1** | `synthesize(text, ...)` | 文本 + 控制参数 | `SynthesizeResponse` | 单句文本→WAV |
| **接口2** | `batch_synthesize(dataset_path, ...)` | C3/C4 JSON路径 | `BatchSynthesizeResponse` | 批量文本→批量WAV |
| **接口3** | `synthesize_stream(text, ...)` | 文本 | `Generator[bytes]` | 流式合成（逐块返回PCM） |
| **接口4** | `list_voices()` | 无 | `List[Dict]` | 列出所有可用音色 |
| **接口5** | `get_module_info()` | 无 | `ModuleInfo` | 模块元信息（版本/能力/参数范围） |
| **接口6** | `generate_voice_with_control(...)` | 文本 + 全控制参数 | `SynthesizeResponse` | 高级一站式合成（synthesize增强别名） |
| **接口7** | `run_pipeline_from_json(json_path)` | JSON管线文件 | `Dict` | JSON驱动的批量合成管线 |

#### 典型调用示例

**示例1：Python函数调用（供C3/C4模块集成）**

```python
from C5_TTS.code.c5_api import C5TTSAPI

# 初始化（自动加载模型，约27s）
api = C5TTSAPI()

# 接口1 — 基础合成
resp = api.synthesize("今天天气很好，我们一起去公园散步吧。")
print(f"音频: {resp.wav_path}, 时长: {resp.audio_duration_sec}s, RTF: {resp.rtf}")

# 接口1 — 带情感和语速的高级合成
resp = api.synthesize("太棒了！我们成功了！", emotion="happy", speed=1.2)
print(f"输出: {resp.wav_path}, RTF: {resp.rtf}")

# 接口6 — 高级一站式合成
resp = api.generate_voice_with_control(
    "今天的项目进展顺利",
    emotion="gentle", speed=1.1, voice_id="中年男"
)

# 接口2 — 批量合成（对接C3级联结果）
resp = api.batch_synthesize(
    "../../C3_cascade/outputs/c3_predictions.json",
    emotion="calm"
)
print(f"合成{resp.total_items}条，平均{resp.avg_time_sec:.2f}s/条")

# 接口4 — 查看可用音色
voices = api.list_voices()
for v in voices:
    print(f"  {v['name']} (来源: {v['source']})")

# 接口5 — 获取模块信息（供验收文档使用）
import json
info = api.get_module_info()
print(json.dumps(info.to_dict(), ensure_ascii=False, indent=2))
```

**示例2：最简集成（专为C3/C4模块设计的极简接口）**

```python
from C5_TTS.code.c5_api import synthesize_and_return_path

# 一行调用：传入翻译文本，返回WAV文件路径
wav_path = synthesize_and_return_path(c3_translation_text, lang="zh")
print(f"语音文件已生成: {wav_path}")
```

**示例3：命令行JSON stdin/stdout管道模式**

```bash
# 单句合成（管道输入JSON，输出JSON结果）
echo '{"text": "你好世界", "lang": "zh", "emotion": "gentle"}' | python c5_api.py --json-stdin

# 批量合成（管道输入JSON）
echo '{"dataset_path": "../../C3_cascade/outputs/c3_predictions.json", "lang": "zh"}' | python c5_api.py --json-stdin

# 查看模块信息
python c5_api.py --module-info

# 列出可用音色
python c5_api.py --list-voices

# JSON驱动批量管线
python c5_api.py --pipeline pipeline_tasks.json
```

**示例4：JSON管线文件格式（接口7）**

```json
{
    "pipeline_name": "c5_s2s_pipeline",
    "tasks": [
        {
            "id": "sample_001",
            "text": "今天天气很好",
            "lang": "zh",
            "speed": 1.0,
            "emotion": "gentle",
            "voice_ref": "/path/to/ref.wav",
            "output": "/path/to/sample_001.wav"
        },
        {
            "id": "sample_002",
            "text": "The weather is nice",
            "lang": "en",
            "speed": 1.2,
            "emotion": "happy"
        }
    ],
    "global_params": {
        "default_speed": 1.0,
        "default_lang": "zh"
    }
}
```

#### API响应数据结构

**SynthesizeResponse（接口1/6的返回值）**：

```json
{
    "success": true,
    "wav_path": "/path/to/output.wav",
    "audio_duration_sec": 3.52,
    "synth_time_sec": 1.85,
    "rtf": 0.526,
    "error": "",
    "request": {"text": "你好世界", "lang": "zh", "speed": 1.0}
}
```

**BatchSynthesizeResponse（接口2的返回值）**：

```json
{
    "success": true,
    "total_items": 50,
    "total_time_sec": 95.3,
    "avg_time_sec": 1.906,
    "results": [
        {"id": "sample_001", "wav": "s2s_sample_001.wav", "audio_sec": 3.5, "synth_sec": 1.8},
        {"id": "sample_002", "wav": "s2s_sample_002.wav", "audio_sec": 4.2, "synth_sec": 2.1}
    ],
    "error": ""
}
```

**ModuleInfo（接口5的返回值）**：

```json
{
    "module_name": "C5_TTS_API",
    "version": "3.0.0",
    "model": "CosyVoice2-0.5B",
    "sample_rate": 24000,
    "supported_languages": ["zh", "en", "ja", "yue", "ko", "mix"],
    "supported_emotions": ["happy", "sad", "angry", "gentle", "calm", "excited", "tired"],
    "supported_speed_range": "0.5 ~ 2.0",
    "capabilities": {
        "基础合成": "单句文本→WAV语音文件，支持中英双语",
        "批量合成": "批量文本→批量语音（对接C3/C4输出JSON）",
        "多语种": "支持中文/英文/日语/粤语/韩语/中英混合",
        "语速控制": "0.5x~2.0x原生调速",
        "音色控制": "通过参考音频实现音色切换，支持音色库注册",
        "情感控制": "7种情感(happy/sad/angry/gentle/calm/excited/tired)，支持强度梯度",
        "说话人克隆": "Zero-Shot说话人音色复刻，支持跨性别",
        "流式合成": "边生成边输出音频块，适配实时对话场景"
    },
    "input_format": {
        "function_call": "Python函数调用，传入文本/参数",
        "json_stdin": "通过stdin传入JSON请求",
        "command_line": "命令行参数",
        "json_file": "JSON文件路径（批量模式）"
    },
    "output_format": {
        "audio": "24kHz WAV单声道音频文件",
        "json": "JSON格式结果（含路径、耗时、RTF）",
        "stream": "流式音频块生成器"
    }
}
```

---

### 12.3 multilingual.py — 多语种支持验证（160行）

**核心类**：`MultilingualTester`

| 方法 | 说明 |
|---|---|
| `__init__(cosyvoice_model, prompt_wav, ...)` | 初始化测试器，绑定模型和参考音频 |
| `synthesize_lang(lang, text, output_name)` | 单个语种合成：中文走`inference_zero_shot`，mix走`inference_cross_lingual`，其他语言加语言token走跨语言接口 |
| `test_all_languages(langs, repeat)` | 批量测试所有语种×N次重复，输出JSON汇总 |
| `evaluate(results)` | 按语种统计平均RTF和音频时长 |

**支持语种**：zh(中文) / en(英文) / ja(日语) / yue(粤语) / ko(韩语) / mix(中英混合)

**语言token映射**（用于`cross_lingual`接口）：
```python
LANG_TOKEN = {
    "zh": "<|zh|>", "en": "<|en|>",
    "ja": "<|ja|>", "yue": "<|yue|>", "ko": "<|ko|>",
}
```

---

### 12.4 emotion_control.py — 情感控制验证（236行）

**核心类**：`EmotionController`

**7种情感指令预设**（`EMOTION_INSTRUCTIONS`）：

| 情感 | 指令描述 |
|---|---|
| happy | 用开心愉悦的语气朗读，充满快乐和活力，语调上扬，声音明亮 |
| sad | 用悲伤低沉的语气朗读，语速缓慢，声音沉重压抑，充满哀愁 |
| angry | 用严肃愤怒的语气朗读，语气坚定有力，语速偏快，声音高亢 |
| gentle | 用温柔亲切的语气朗读，像朋友聊天一样娓娓道来，声音柔和 |
| calm | 用平静中性的语气朗读，客观冷静地播报，声音平稳 |
| excited | 用激动兴奋的语气朗读，充满热情，语速快，声音洪亮 |
| tired | 用慵懒疲惫的语气朗读，语速慢，声音松弛无力，有气无力 |

**情感强度梯度**（以sad为例）：

| 级别 | 标签 | 描述 |
|---|---|---|
| 1 | sad_1 | 略带一丝忧伤，语气稍微低沉 |
| 2 | sad_2 | 有些难过，声音明显沉重 |
| 3 | sad_3 | 非常悲伤，语速缓慢，充满哀愁 |
| 4 | sad_4 | 极度悲痛，几乎要哭泣，声音颤抖 |

**核心函数**：`analyze_acoustic_features(audio_np, sr)` — 提取F0基频、语速(时长)、能量(RMS)等声学特征，用于情感效果量化分析。

---

### 12.5 speed_timbre.py — 语速与音色控制

**核心类**：`SpeedController` / `TimbreController`

| 功能 | 说明 |
|---|---|
| `run_speed_test()` | 5档语速预设测试（0.5x / 0.75x / 1.0x / 1.5x / 2.0x），同一文本不同语速对比 |
| `run_timbre_test()` | 从参考音频目录采样多音色，同一文本不同音色输出对比 |
| `TimbreController.register_voice(name, wav, text)` | 注册自定义音色到voice_bank |
| `TimbreController.list_registered_voices()` | 列出所有已注册音色 |

---

### 12.6 speaker_clone.py — Zero-Shot说话人克隆

**核心类**：`SpeakerCloneTester`

| 方法 | 说明 |
|---|---|
| `test_duration_impact(wav, text, output_dir)` | 测试参考音频时长对克隆质量的影响（1s/3s/5s/10s分段测试） |
| `test_cross_gender(ref_wavs, output_dir)` | 跨性别说话人克隆测试（用男声参考合成女声文本，反之亦然） |
| `test_multi_speaker(ref_wavs, output_dir)` | 多说话人批量克隆（最多8个不同说话人对比） |

---

### 12.7 benchmark.py — 多模型横向对比

**核心函数**：`run_benchmark(cosyvoice_model, prompt_wav, prompt_text, output_dir)`

对比维度：
- **合成速度**：RTF实时率、平均每条耗时
- **音频质量**：MOS-like主观评分框架
- **显存占用**：GPU内存使用量
- **功能覆盖**：情感/语速/多语种/音色克隆支持度

对比模型：CosyVoice2-0.5B（本实训主推）vs IndexTTS2（进阶参考）

---

### 12.8 visualize_results.py — 结果可视化

生成6类图表（输出到 `report/` 目录）：
1. **RTF实时率柱状图** — 各模型/语种RTF对比（`chart_multilingual.png`）
2. **情感声学特征雷达图** — F0/语速/能量多维对比（`chart_emotion.png`）
3. **说话人克隆相似度对比图**（`chart_speaker_clone.png`）
4. **语速-合成耗时散点图**（`chart_speed.png`）
5. **批量合成耗时分布直方图**
6. **综合仪表盘汇总图**（`dashboard_summary.png`）

---

## 十三、运行日志与输出规范

| 输出项 | 路径 | 格式 |
|---|---|---|
| 基础合成音频 | `outputs/` 目录下 `.wav` 文件 | 24kHz 16bit 单声道WAV |
| 批量合成音频 | `outputs/batch/s2s_{id}.wav` | 同上 |
| 批量汇总JSON | `outputs/batch/batch_summary.json` | JSON，含每条耗时/RTF |
| 运行日志 | `log/runtime.log` | JSON Lines，每条一行 |
| 进阶测试结果 | `outputs/multilingual/` / `emotion/` / `speaker_clone/` / `benchmark/` / `speed_timbre/` | 各子目录含WAV + summary JSON |
| 可视化图表 | `report/` 目录 | PNG图表（chart_*.png） |

---

> **C5_TTS v3.0.0** | 东北大学 NEU NLP × 小牛翻译 NiuTrans.com