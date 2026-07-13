# C5_TTS API 接口详细说明文档

> **版本：v3.0.0 | 模型：CosyVoice2-0.5B | 采样率：24000Hz**
>
> **东北大学 NEU NLP | 小牛翻译 NiuTrans.com**

---

## 目录

1. [概述](#一概述)
2. [快速上手](#二快速上手)
3. [接口1：synthesize() — 单句合成](#三接口1synthesize--单句合成)
4. [接口2：batch_synthesize() — 批量合成](#四接口2batch_synthesize--批量合成)
5. [接口3：synthesize_stream() — 流式合成](#五接口3synthesize_stream--流式合成)
6. [接口4：list_voices() — 列出音色](#六接口4list_voices--列出音色)
7. [接口5：get_module_info() — 模块信息](#七接口5get_module_info--模块信息)
8. [接口6：generate_voice_with_control() — 高级合成](#八接口6generate_voice_with_control--高级合成)
9. [接口7：run_pipeline_from_json() — JSON管线](#九接口7run_pipeline_from_json--json管线)
10. [便捷函数](#十便捷函数)
11. [数据结构速查表](#十一数据结构速查表)
12. [完整参数速查表](#十二完整参数速查表)
13. [错误码说明](#十三错误码说明)
14. [预留接口](#十四预留接口)

---

## 一、概述

### 1.1 这是什么

`c5_api.py` 是 C5_TTS 模块的 **Python API 封装**。它把你已有的 `c5_tts.py` 命令行功能包装成了 Python 类和函数，让你可以在 Python 代码中直接调用 TTS 合成，而不需要通过命令行。

### 1.2 什么时候用

| 场景 | 用什么 |
|------|--------|
| 在终端里手动跑一条合成 | `python c5_tts.py --text "你好"` |
| C3/C4给你JSON文件，批量合成 | `python c5_tts.py --dataset xxx.json` |
| 在自己写的Python脚本里调用TTS | `from c5_api import C5TTSAPI` |
| 自动化流水线（Shell管道） | `echo '{"text":"你好"}' \| python c5_api.py --json-stdin` |
| 想看模块有哪些能力 | `python c5_api.py --module-info` |

### 1.3 导入方式

```python
import sys
sys.path.insert(0, "C5_TTS/code")   # 确保能找到c5_api.py

from c5_api import C5TTSAPI, synthesize_and_return_path
```

### 1.4 生命周期

```
初始化 C5TTSAPI()  → 首次调用自动加载模型(~27s) → 调用各接口 → 进程结束释放
```

> **注意：** 模型加载约需27秒（GPU），只需加载一次。实例化后的 `api` 对象可重复调用所有接口，不会重复加载。

---

## 二、快速上手

```python
from c5_api import C5TTSAPI

# 1. 创建实例（模型首次调用时自动加载）
api = C5TTSAPI()

# 2. 合成一句话
resp = api.synthesize("你好世界")
print(f"音频文件: {resp.wav_path}")
print(f"时长: {resp.audio_duration_sec}s")
print(f"合成耗时: {resp.synth_time_sec}s")
print(f"RTF(实时率): {resp.rtf}")  # <1 表示比实时播放快

# 3. 带情感的合成
resp = api.synthesize("太棒了！我们成功了！", emotion="happy")
```

---

## 三、接口1：synthesize() — 单句合成

### 3.1 功能

输入一段文本，输出一个WAV音频文件。这是最核心、最常用的接口。

### 3.2 方法签名

```python
def synthesize(
    self,
    text: str,                  # 必填：待合成文本
    output: str = None,         # 输出WAV路径（默认自动生成）
    lang: str = "zh",           # 语言：zh/en/ja/yue/ko/mix
    speed: float = 1.0,         # 语速倍率：0.5~2.0
    emotion: str = None,        # 情感：happy/sad/angry/gentle/calm/excited/tired
    instruct: str = None,       # 自定义指令（优先级>emotion）
    voice_ref: str = None,      # 参考音频路径（音色克隆）
    voice_id: str = None,       # 已注册音色ID
    stream: bool = False        # 是否流式输出
) -> SynthesizeResponse
```

### 3.3 参数详解

#### text（必填）
待合成的文本字符串。支持的语言：

| lang值 | 语言 | 示例文本 |
|--------|------|---------|
| `"zh"` | 中文 | `"今天天气很好，我们一起去公园散步吧。"` |
| `"en"` | 英文 | `"The weather is really nice today."` |
| `"ja"` | 日语 | `"今日の天気はとても良いです。"` |
| `"yue"` | 粤语 | `"今日天气好好，我哋一齐去公园散步啦。"` |
| `"ko"` | 韩语 | `"오늘 날씨가 정말 좋네요."` |
| `"mix"` | 中英混合 | `"今天的meeting我们讨论一下AI的roadmap。"` |

#### output（可选）
输出WAV文件的路径。不传则自动生成到 `C5_TTS/outputs/api/` 目录下，文件名格式：`synth_{lang}_{文本MD5前8位}.wav`

```python
# 自动路径
api.synthesize("你好")  # → outputs/api/synth_zh_a1b2c3d4.wav

# 指定路径
api.synthesize("你好", output="/home/user/my_voice.wav")
```

#### lang（可选，默认"zh"）
指定文本的语言模式。不同语言走不同的CosyVoice内部推理路径：
- `"zh"` → `inference_zero_shot`（中文原生，效果最佳）
- `"en"/"ja"/"yue"/"ko"` → `inference_cross_lingual`（跨语言合成，保持参考音频音色）
- `"mix"` → `inference_cross_lingual`（中英混合，不加语言token）

#### speed（可选，默认1.0）
语速倍率，范围0.5~2.0：

| 值 | 效果 |
|----|------|
| 0.5 | 半速（很慢） |
| 0.7 | 慢速 |
| 0.85 | 稍慢 |
| 1.0 | 正常 |
| 1.2 | 稍快 |
| 1.5 | 快速 |
| 2.0 | 双倍速（很快） |

```python
api.synthesize("你好", speed=1.5)  # 1.5倍速
```

#### emotion（可选）
情感标签，7种可选：

| 值 | 中文含义 | 效果 |
|----|---------|------|
| `"happy"` | 开心 | 语调上扬，声音明亮 |
| `"sad"` | 悲伤 | 语速缓慢，声音沉重 |
| `"angry"` | 愤怒 | 语气坚定，声音高亢 |
| `"gentle"` | 温柔 | 声音柔和，娓娓道来 |
| `"calm"` | 平静 | 客观冷静，声音平稳 |
| `"excited"` | 激动 | 语速快，声音洪亮 |
| `"tired"` | 疲惫 | 语速慢，声音松弛 |

```python
api.synthesize("今天的项目终于完成了", emotion="happy")
api.synthesize("这次的结果让人失望", emotion="sad")
api.synthesize("这简直是不可接受的错误", emotion="angry")
```

> **原理**：emotion标签被映射为一段自然语言指令（如"用开心愉悦的语气朗读，充满快乐和活力"），通过CosyVoice的 `inference_instruct2` 接口控制生成效果。

#### instruct（可选，优先级高于emotion）
自定义的instruct2指令文本，可以实现比固定情感标签更精细的控制。

```python
# 比emotion更灵活
api.synthesize(
    "大家好",
    instruct="用悲伤的中年男声朗读，语速缓慢，语气低沉"
)

# 同时传emotion和instruct时，instruct生效
api.synthesize(
    "你好",
    emotion="happy",  # ← 被忽略
    instruct="用非常严肃的语气朗读"  # ← 实际生效
)
```

#### voice_ref（可选）
参考音频文件路径（.wav格式）。提供3-10秒的清晰人声，CosyVoice会提取这段音频的说话人音色，合成语音时会模仿这个音色。

```python
# 克隆某人的声音
api.synthesize("你好，欢迎使用语音合成系统", voice_ref="/data/speaker_male.wav")
```

> **注意**：
> - 推荐3-10秒，太短(<1s)克隆效果差
> - 音频中尽量只有一个人说话
> - 背景噪音越小越好

#### voice_id（可选）
已预先注册的音色ID。需要先用命令行注册：

```bash
# 先注册音色
python c5_tts.py --register-voice 中年男 --register-wav /data/male.wav --register-text "参考文本"

# 然后代码中使用
api.synthesize("今天的新闻有...", voice_id="中年男")
```

#### stream（可选，默认False）
是否启用流式合成。设为 `True` 时，生成过程中边合成边写入文件，适合长文本。

### 3.4 返回值：SynthesizeResponse

```python
resp: SynthesizeResponse = api.synthesize("你好")

# resp 有以下属性：
resp.success            # bool:  是否成功
resp.wav_path           # str:   输出WAV文件的绝对路径
resp.audio_duration_sec # float: 生成音频的时长（秒）
resp.synth_time_sec     # float: 合成耗时（秒）
resp.rtf                # float: 实时率 = synth_time / audio_duration
                        #        <1 表示比实时播放快，越小越快
resp.error              # str:   错误信息（成功时为空字符串）
resp.request            # dict:  回显的请求参数

# 也可以直接转JSON
resp.to_dict()
# → {
#     "success": true,
#     "wav_path": ".../outputs/api/synth_zh_abc12345.wav",
#     "audio_duration_sec": 2.18,
#     "synth_time_sec": 1.42,
#     "rtf": 0.651,
#     "error": "",
#     "request": {"text": "你好", "lang": "zh", "speed": 1.0, ...}
#   }
```

### 3.5 完整使用示例

```python
from c5_api import C5TTSAPI

api = C5TTSAPI()

# 示例1：最简调用
resp = api.synthesize("你好世界")
if resp.success:
    print(f"✅ 生成成功: {resp.wav_path}")

# 示例2：英文 + 快语速
resp = api.synthesize("Hello world, this is a test.", lang="en", speed=1.5)

# 示例3：开心情感
resp = api.synthesize("太棒了！今天放假！", emotion="happy")

# 示例4：自定义指令 + 指定输出路径
resp = api.synthesize(
    "各位听众晚上好，欢迎收听今天的晚间新闻。",
    output="/home/user/news_output.wav",
    instruct="用专业的新闻播音语气朗读，语速适中，吐字清晰"
)

# 示例5：指定音色克隆
resp = api.synthesize(
    "你好，我是你的AI助手",
    voice_ref="/data/my_boss_voice.wav"
)

# 示例6：检查失败情况
resp = api.synthesize("")  # 空文本
if not resp.success:
    print(f"❌ 失败: {resp.error}")  # → "输入文本为空"
```

---

## 四、接口2：batch_synthesize() — 批量合成

### 4.1 功能

读取一个JSON文件，逐条提取文本合成语音。专为对接 C3/C4 模块设计。

### 4.2 方法签名

```python
def batch_synthesize(
    self,
    dataset_path: str,         # 必填：JSON文件路径
    output_dir: str = None,    # 输出目录（默认 outputs/batch/）
    lang: str = "zh",          # 语言
    speed: float = 1.0,        # 语速
    voice_ref: str = None,     # 统一参考音频（所有条目共用）
    emotion: str = None,       # 统一情感
    instruct: str = None       # 统一指令
) -> BatchSynthesizeResponse
```

### 4.3 JSON输入格式

JSON文件是一个列表，每个元素是一条记录。自动识别以下字段（按优先级）：

| 优先级 | 字段名 | 来源 |
|--------|--------|------|
| 1（最高） | `translation_zh` | C3级联翻译结果 |
| 2 | `e2e_translation_zh` | C4端到端翻译结果 |
| 3 | `text` | 通用文本字段 |

```json
[
    {
        "id": "sample_001",
        "translation_zh": "今天天气很好，我们一起去公园散步吧。"
    },
    {
        "id": "sample_002",
        "text": "人工智能正在改变我们的生活方式。"
    },
    {
        "id": "sample_003",
        "e2e_translation_zh": "深度学习是机器学习的一个分支。"
    }
]
```

### 4.4 输出

每条生成 `s2s_{id}.wav` 文件到 `output_dir`，同时在 `output_dir` 下保存 `batch_summary.json`。

### 4.5 返回值：BatchSynthesizeResponse

```python
resp: BatchSynthesizeResponse = api.batch_synthesize("cascade_results.json")

resp.success          # bool: 是否成功
resp.total_items      # int:  总条目数
resp.total_time_sec   # float: 批量总耗时
resp.avg_time_sec     # float: 平均每条耗时
resp.results          # list: 每条详细结果
resp.error            # str:  错误信息

# results中每条：
# {
#     "id": "sample_001",
#     "text": "今天天气很好...",
#     "wav": "s2s_sample_001.wav",
#     "audio_sec": 3.52,
#     "synth_sec": 2.18
# }

resp.to_dict()  # 转JSON
```

### 4.6 使用示例

```python
api = C5TTSAPI()

# 对接C3翻译结果
resp = api.batch_synthesize("../C3_cascade/outputs/cascade_results.json")
print(f"合成 {resp.total_items} 条，总耗时 {resp.total_time_sec}s")

# 带统一情感
resp = api.batch_synthesize(
    "cascade_results.json",
    emotion="gentle",    # 所有条目用温柔语气
    lang="zh"
)
```

---

## 五、接口3：synthesize_stream() — 流式合成

### 5.1 功能

边合成边返回音频块（Python生成器），适合需要低延迟、边生成边播放的实时场景。

### 5.2 方法签名

```python
def synthesize_stream(
    self,
    text: str,                 # 必填：待合成文本
    lang: str = "zh",          # 语言
    emotion: str = None,       # 情感
    voice_ref: str = None      # 参考音频
) -> Generator[bytes, None, None]
```

### 5.3 返回值

一个生成器，每次 `yield` 返回一个 `bytes` 对象（PCM音频数据块）。

### 5.4 使用示例

```python
api = C5TTSAPI()

# 逐个音频块处理（如实时播放）
for chunk in api.synthesize_stream("你好世界欢迎来到人工智能时代"):
    # chunk 是 bytes，可以直接写入播放缓冲区
    audio_buffer.write(chunk)
    print(f"收到音频块: {len(chunk)} bytes")
```

> **注意**：流式模式当前通过临时文件中转。未来 v3.2 版本计划通过预留的 `streaming_websocket()` 接口实现真正的实时流推送。

---

## 六、接口4：list_voices() — 列出音色

### 6.1 功能

列出当前所有可用的音色/说话人，包括模型内置的和用户在 voice_bank 中注册的。

### 6.2 方法签名

```python
def list_voices(self) -> List[Dict]
```

### 6.3 返回值

```python
voices = api.list_voices()
# → [
#     {"name": "中文女", "spk_id": "中文女", "source": "model"},
#     {"name": "中文男", "spk_id": "中文男", "source": "model"},
#     {"name": "中年男", "spk_id": "中年男", "source": "voice_bank", "wav": "..."},
#   ]
```

- `source: "model"` → CosyVoice内置的预注册说话人
- `source: "voice_bank"` → 你自己注册的音色

### 6.4 使用示例

```python
api = C5TTSAPI()
voices = api.list_voices()

print(f"可用音色共 {len(voices)} 个：")
for v in voices:
    print(f"  - {v['name']} (来源: {v['source']})")

# 用列出的音色ID合成
api.synthesize("你好", voice_id="中文女")
```

---

## 七、接口5：get_module_info() — 模块信息

### 7.1 功能

返回 C5_TTS 模块的完整能力描述，包括版本、支持的参数范围、能力清单、预留接口列表。适合验收展示和文档生成。

### 7.2 方法签名

```python
def get_module_info(self) -> ModuleInfo
```

### 7.3 返回值：ModuleInfo

```python
info = api.get_module_info()
d = info.to_dict()
```

返回字段：

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
    "基础合成": "单句文本→WAV语音文件",
    "批量合成": "批量文本→批量语音",
    "多语种": "支持中文/英文/日语/粤语/韩语/中英混合",
    "语速控制": "0.5x~2.0x原生调速",
    "音色控制": "通过参考音频实现音色切换",
    "情感控制": "7种情感+强度梯度",
    "说话人克隆": "Zero-Shot音色复刻",
    "流式合成": "边生成边输出音频块",
    "实验评估": "RTF/F0/可视化"
  },
  "reserved_interfaces": {
    "register_custom_voice": "动态注册自定义音色（预计v3.1）",
    "fine_tune_voice": "微调音色模型（预计v3.2）",
    "voice_conversion": "语音转换（预计v4.0）",
    "text_frontend_normalize": "文本前端归一化（预计v3.1）",
    "streaming_websocket": "WebSocket流式推送（预计v3.2）"
  },
  "input_format": { ... },
  "output_format": { ... }
}
```

### 7.4 使用示例

```python
import json
api = C5TTSAPI()
info = api.get_module_info()
print(json.dumps(info.to_dict(), ensure_ascii=False, indent=2))
```

也可以通过命令行查看：
```bash
python c5_api.py --module-info
```

---

## 八、接口6：generate_voice_with_control() — 高级合成

### 8.1 功能

`synthesize()` 的别名，语义上更强调"精细控制"。功能和参数完全一致，只是名字更直观。

### 8.2 方法签名

```python
def generate_voice_with_control(
    self,
    text: str,
    output: str = None,
    speed: float = 1.0,
    emotion: str = None,
    voice_ref: str = None,
    voice_id: str = None,
    instruct: str = None
) -> SynthesizeResponse
```

### 8.3 使用示例

```python
api = C5TTSAPI()

# 一句话搞定所有控制参数
resp = api.generate_voice_with_control(
    "今天的项目演示非常成功，感谢大家的配合",
    emotion="happy",
    speed=1.2,
    voice_id="中文女",
    output="/home/user/demo.wav"
)
```

---

## 九、接口7：run_pipeline_from_json() — JSON管线

### 9.1 功能

用一个JSON文件描述多条合成任务，一次性批量执行。每条任务可以有不同的参数（语言、情感、音色等）。适合自动化批量生成多个不同风格的文件。

### 9.2 方法签名

```python
def run_pipeline_from_json(
    self,
    json_path: str,            # 必填：管线JSON文件路径
    output_dir: str = None     # 统一输出目录（可被单条覆盖）
) -> Dict
```

### 9.3 JSON管线格式

```json
{
    "pipeline_name": "验收演示批量生成",
    "global_params": {
        "default_speed": 1.0,
        "default_lang": "zh"
    },
    "tasks": [
        {
            "id": "zh_happy",
            "text": "今天是个好日子，阳光明媚，心情愉悦",
            "lang": "zh",
            "emotion": "happy"
        },
        {
            "id": "zh_sad",
            "text": "这次失败了，心情非常沉重",
            "emotion": "sad"
        },
        {
            "id": "en_fast",
            "text": "The quick brown fox jumps over the lazy dog",
            "lang": "en",
            "speed": 1.5
        },
        {
            "id": "ja_calm",
            "text": "今日の天気は穏やかです",
            "lang": "ja",
            "emotion": "calm",
            "output": "/home/user/japanese_output.wav"
        }
    ]
}
```

每条任务的字段：

| 字段 | 必填 | 说明 |
|------|------|------|
| `id` | ✅ | 任务标识，用于命名输出文件 |
| `text` | ✅ | 待合成文本 |
| `lang` | ❌ | 语言（默认从global_params取） |
| `speed` | ❌ | 语速（默认从global_params取） |
| `emotion` | ❌ | 情感 |
| `voice_ref` | ❌ | 参考音频 |
| `voice_id` | ❌ | 音色ID |
| `instruct` | ❌ | 自定义指令 |
| `output` | ❌ | 指定输出路径（覆盖统一输出目录） |

### 9.4 返回值

```python
result = api.run_pipeline_from_json("tasks.json")

# {
#   "success": True,
#   "pipeline_name": "验收演示批量生成",
#   "total": 4,
#   "completed": 4,
#   "failed": 0,
#   "results": [
#       {"id": "zh_happy", "success": True, "wav_path": "...", "rtf": 0.62},
#       {"id": "zh_sad",   "success": True, "wav_path": "...", "rtf": 0.65},
#       {"id": "en_fast",  "success": True, "wav_path": "...", "rtf": 0.58},
#       {"id": "ja_calm",  "success": True, "wav_path": "...", "rtf": 0.71}
#   ]
# }
```

### 9.5 使用示例

```python
api = C5TTSAPI()

result = api.run_pipeline_from_json("my_tasks.json")

print(f"完成 {result['completed']}/{result['total']} 条")
if result['failed'] > 0:
    for r in result['results']:
        if not r['success']:
            print(f"  ❌ {r['id']}: {r['error']}")
```

---

## 十、便捷函数

### 10.1 synthesize_and_return_path()

最简单的调用方式：传入文本，返回WAV文件路径。

```python
from c5_api import synthesize_and_return_path

wav_path = synthesize_and_return_path("你好世界")
print(wav_path)  # → "C5_TTS/outputs/api/synth_zh_abc12345.wav"

# 带参数
wav_path = synthesize_and_return_path(
    "Hello world",
    lang="en",
    speed=1.2,
    emotion="happy"
)
```

> **适用场景**：在C3/C4的Python脚本中快速调用，不需要自己管理API实例。

---

## 十一、数据结构速查表

### SynthesizeResponse（单句合成返回）

| 字段 | 类型 | 说明 |
|------|------|------|
| `success` | `bool` | 是否成功 |
| `wav_path` | `str` | 输出WAV文件绝对路径 |
| `audio_duration_sec` | `float` | 音频时长（秒） |
| `synth_time_sec` | `float` | 合成耗时（秒） |
| `rtf` | `float` | 实时率，<1表示比实时快 |
| `error` | `str` | 错误信息（成功时为空） |
| `request` | `dict` | 回显的请求参数 |

### BatchSynthesizeResponse（批量合成返回）

| 字段 | 类型 | 说明 |
|------|------|------|
| `success` | `bool` | 是否成功 |
| `total_items` | `int` | 总条目数 |
| `total_time_sec` | `float` | 批量总耗时 |
| `avg_time_sec` | `float` | 平均每条耗时 |
| `results` | `list` | 每条结果列表 |
| `error` | `str` | 错误信息 |

---

## 十二、完整参数速查表

| 参数 | 类型 | 默认值 | 可选值/范围 | 出现于接口 |
|------|------|--------|-----------|----------|
| `text` | `str` | — | 任意文本 | 1,3,6,便捷函数 |
| `output` | `str` | `None` | 文件路径 | 1,6 |
| `lang` | `str` | `"zh"` | zh/en/ja/yue/ko/mix | 1,2,3,便捷函数 |
| `speed` | `float` | `1.0` | 0.5~2.0 | 1,2,6,便捷函数 |
| `emotion` | `str` | `None` | happy/sad/angry/gentle/calm/excited/tired | 1,2,3,6,便捷函数 |
| `instruct` | `str` | `None` | 任意中文描述 | 1,2,6 |
| `voice_ref` | `str` | `None` | .wav文件路径 | 1,2,3,6 |
| `voice_id` | `str` | `None` | 已注册音色ID | 1,6 |
| `stream` | `bool` | `False` | True/False | 1 |
| `dataset_path` | `str` | — | JSON文件路径 | 2 |
| `output_dir` | `str` | `None` | 目录路径 | 2 |
| `json_path` | `str` | — | 管线JSON路径 | 7 |

---

## 十三、错误码说明

所有接口返回的响应都有 `success` 字段。失败时 `error` 字段会有具体描述。

| 错误场景 | success | error示例 |
|---------|---------|----------|
| 文本为空 | `False` | `"输入文本为空"` |
| 模型目录不存在 | 抛异常 | `FileNotFoundError: 模型目录不存在...` |
| JSON文件不存在 | `False` | `"数据集文件不存在: /path/to/file"` |
| JSON解析失败 | `False` | `"JSON解析失败: ..."` |
| 合成过程异常 | `False` | 具体异常信息 |

```python
resp = api.synthesize("")
if not resp.success:
    print(f"失败原因: {resp.error}")
```

---

## 十四、预留接口

以下接口已在代码结构中预留，规划了完整的接口签名和实现方案，将在后续版本中实现。

### 14.1 register_custom_voice()（v3.1）
动态注册自定义音色，支持标签检索。

### 14.2 fine_tune_voice()（v3.2）
基于少量数据LoRA微调说话人音色。

### 14.3 voice_conversion()（v4.0）
音色A→音色B的语音转换（Voice Conversion）。

### 14.4 text_frontend_normalize()（v3.1）
文本前端归一化（数字/日期/缩写→中文读法）。

### 14.5 streaming_websocket()（v3.2）
WebSocket流式TTS推送服务。

> 每个预留接口的详细签名、参数说明、实现方案见：`C5_TTS/docx/C5_TTS_验收文档.md` 第四章。

---

> **C5_TTS v3.0.0** | 东北大学 NEU NLP × 小牛翻译 NiuTrans.com