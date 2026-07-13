# C5_TTS 命令使用大全

> 基于 CosyVoice2-0.5B 的语音合成工具，支持音色克隆、语速调节、情感控制
> 用法：`python c5_tts.py [参数]`

---

## 目录

1. [基础合成](#1-基础合成)
2. [音色克隆（说话人控制）](#2-音色克隆说话人控制)
3. [音色库管理](#3-音色库管理)
4. [语速控制](#4-语速控制)
5. [情感控制](#5-情感控制)
6. [语言模式](#6-语言模式)
7. [进阶测试（一键运行）](#7-进阶测试一键运行)
8. [批量合成](#8-批量合成)
9. [常见组合场景](#9-常见组合场景)
10. [参数速查表](#10-参数速查表)

---

## 1. 基础合成

### 1.1 单句合成（最基础）

```bash
cd /root/siton-tmp/lmy/C5_TTS/code

# 基本用法
python c5_tts.py --text "你好世界" -o output.wav

# 不指定 -o，默认保存到 ../outputs/output.wav
python c5_tts.py --text "你好世界"
```

### 1.2 流式合成（边生成边播放）

```bash
python c5_tts.py --text "这是一段流式合成的语音" --stream -o stream.wav
```

### 1.3 指定输出路径

```bash
# 保存到指定目录
python c5_tts.py --text "你好" -o /root/siton-tmp/lmy/C5_TTS/test/output.wav
```

---

## 2. 音色克隆（说话人控制）

### 2.1 直接用参考音频克隆音色（不注册，一步到位）

```bash
python c5_tts.py --text "这是克隆后的声音" --voice /root/siton-tmp/lmy/C5_TTS/test/1.wav -o output.wav
```

**说明：**
- `--voice` / `--voice-ref` 后面跟参考音频路径
- 不需要知道参考音频里说了什么，程序自动用 cross_lingual 模式提取音色
- 参考音频建议 3~10 秒效果最佳

### 2.2 先注册音色，再使用（适合反复使用同一音色）

```bash
# 第1步：注册音色（如果不知道1.wav的内容，--register-text 留空）
python c5_tts.py --register-voice my_speaker_1 --register-wav /root/siton-tmp/lmy/C5_TTS/test/1.wav --register-text ""

# 第2步：用已注册音色朗读
python c5_tts.py --text "你好，欢迎使用智能语音合成系统。" --voice-id my_speaker_1 -o output.wav
```

### 2.3 如果你知道参考音频的文字内容（效果更好）

```bash
# 注册时填写准确文字
python c5_tts.py --register-voice 张三 --register-wav /path/to/zhangsan.wav --register-text "我是张三，很高兴认识你"

# 使用
python c5_tts.py --text "今天天气真好" --voice-id 张三 -o output.wav
```

### 2.4 查看所有已注册音色

```bash
python c5_tts.py --list-voices
```

---

## 3. 音色库管理

### 3.1 注册新音色

```bash
python c5_tts.py --register-voice 中年男 --register-wav /path/to/male.wav --register-text "参考音频的文字"
```

`--register-text` 填写规则：

| 情况 | 操作 | 说明 |
|------|------|------|
| 知道音频内容 | 填写准确文字 | 走 zero_shot 模式，效果最佳 |
| 不知道音频内容 | 留空 `""` 或省略 | 自动走 cross_lingual 模式 |
| 音频内容是外语 | 留空即可 | cross_lingual 跨语言 |

### 3.2 音色 + 情感 组合

```bash
# 使用已注册音色 + 情感
python c5_tts.py --text "今天是个好日子" --voice-id 张三 --emotion happy -o output.wav

# 使用已注册音色 + 自定义指令
python c5_tts.py --text "那天之后我再也没见过他" --voice-id 张三 --instruct "用悲伤的语气朗读" -o output.wav
```

### 3.3 音色 + 语速 组合

```bash
python c5_tts.py --text "快速朗读这段文字" --voice-id 张三 --speed 1.5 -o fast.wav
```

---

## 4. 语速控制

### 4.1 使用 --speed 参数（原生支持，推荐）

```bash
# 不同档位的语速
python c5_tts.py --text "今天天气真不错" --speed 0.7 -o very_slow.wav    # 非常慢
python c5_tts.py --text "今天天气真不错" --speed 0.85 -o slow.wav        # 慢速
python c5_tts.py --text "今天天气真不错" --speed 1.0 -o normal.wav       # 正常（默认）
python c5_tts.py --text "今天天气真不错" --speed 1.2 -o fast.wav         # 快速
python c5_tts.py --text "今天天气真不错" --speed 1.5 -o very_fast.wav    # 非常快
```

取值范围：`0.5`（极慢）~ `2.0`（极快）

### 4.2 使用 --instruct 控制语速（自然语言指令）

```bash
python c5_tts.py --text "今天天气真不错" --instruct "用非常慢的语速朗读，每个字都拖长" -o slow.wav
```

### 4.3 语速 + 音色 组合

```bash
python c5_tts.py --text "今天天气真不错" --voice /path/to/ref.wav --speed 1.3 -o fast_clone.wav
```

---

## 5. 情感控制

### 5.1 7种预设情感

```bash
# 开心
python c5_tts.py --text "太棒了，终于成功了！" --emotion happy -o happy.wav

# 悲伤
python c5_tts.py --text "失去了才知道珍惜" --emotion sad -o sad.wav

# 愤怒
python c5_tts.py --text "你怎么能这样做！" --emotion angry -o angry.wav

# 温柔
python c5_tts.py --text "晚安，做个好梦" --emotion gentle -o gentle.wav

# 平静（默认中性）
python c5_tts.py --text "据气象台报道，明天晴转多云" --emotion calm -o calm.wav

# 激动
python c5_tts.py --text "我们赢了！太不可思议了！" --emotion excited -o excited.wav

# 疲惫
python c5_tts.py --text "好累啊，今天工作太多了" --emotion tired -o tired.wav
```

### 5.2 使用自定义指令（比 --emotion 更灵活）

```bash
# 可以自由组合情感描述
python c5_tts.py --text "你好" --instruct "用温柔亲切的语气，像妈妈对孩子说话一样" -o gentle.wav
python c5_tts.py --text "你好" --instruct "用严肃低沉的声音，像领导讲话一样" -o serious.wav
python c5_tts.py --text "你好" --instruct "用欢快活泼的语气，像幼儿园老师一样" -o lively.wav
```

### 5.3 情感 + 音色 组合

```bash
# 使用参考音频 + 情感
python c5_tts.py --text "今天真开心" --voice /path/to/ref.wav --emotion happy -o happy_clone.wav

# 使用已注册音色 + 情感
python c5_tts.py --text "今天真开心" --voice-id 张三 --emotion happy -o happy_zhangsan.wav
```

### 5.4 情感 + 语速 组合

```bash
# 悲伤 + 慢速
python c5_tts.py --text "永别了，我的爱人" --emotion sad --speed 0.7 -o sad_slow.wav

# 激动 + 快速
python c5_tts.py --text "我们成功了！太棒了！" --emotion excited --speed 1.3 -o excited_fast.wav
```

---

## 6. 语言模式

### 6.1 中文

```bash
python c5_tts.py --text "人工智能正在改变我们的生活方式" --lang zh -o zh.wav
```

### 6.2 英文

```bash
python c5_tts.py --text "Artificial intelligence is transforming our daily lives." --lang en -o en.wav
```

### 6.3 中英混合

```bash
python c5_tts.py --text "今天的meeting讨论AI的roadmap，我们需要尽快完成deadline" --lang mix -o mix.wav
```

### 6.4 多语种进阶测试

```bash
python c5_tts.py --multilingual-test
```
自动测试中文、英文、日语、粤语、韩语、中英混合共6种语言。

---

## 7. 进阶测试（一键运行）

### 7.1 全部进阶测试

```bash
python c5_tts.py --all-advanced
```
一键运行：多语种 + 语速 + 音色 + 情感 + 说话人克隆 + 横向对比

### 7.2 单独运行某项

```bash
# 进阶① 多语种测试
python c5_tts.py --multilingual-test

# 进阶② 语速测试（5档速度对比）
python c5_tts.py --speed-test

# 进阶② 音色测试（多参考音频对比）
python c5_tts.py --timbre-test

# 进阶③ 情感测试（7种情感）
python c5_tts.py --emotion-test

# 进阶③ 情感强度测试（4级强度梯度）
python c5_tts.py --emotion-intensity

# 进阶④ 说话人克隆测试（Zero-Shot克隆）
python c5_tts.py --speaker-test

# 进阶⑤ 模型横向对比
python c5_tts.py --benchmark
```

### 7.3 指定参考音频目录（用于音色/说话人测试）

```bash
python c5_tts.py --timbre-test --ref-audio-dir /path/to/audio/dir
python c5_tts.py --speaker-test --ref-audio-dir /path/to/audio/dir
```

---

## 8. 批量合成

### 8.1 从JSON批量合成（对接C3/C4）

```bash
python c5_tts.py --dataset /path/to/cascade_results.json
```

JSON格式要求（只需包含以下任一字段）：
```json
[
  {
    "id": "001",
    "translation_zh": "你好世界",       // 优先级最高
    "e2e_translation_zh": "你好世界",   // 第二优先级
    "text": "你好世界"                   // 兜底
  }
]
```

### 8.2 批量合成 + 音色克隆

```bash
# 全部使用同一个参考音频
python c5_tts.py --dataset /path/to/data.json --voice /path/to/ref.wav
```

### 8.3 批量合成 + 情感

```bash
python c5_tts.py --dataset /path/to/data.json --emotion happy
```

---

## 9. 常见组合场景

### 场景1：用1.wav的音色，朗读一句话（你的场景）

```bash
python c5_tts.py --text "今天天气很好，适合出去散步。" --voice /root/siton-tmp/lmy/C5_TTS/test/1.wav -o /root/siton-tmp/lmy/C5_TTS/test/output.wav
```

### 场景2：注册多个音色，切换使用

```bash
# 注册
python c5_tts.py --register-voice speaker_A --register-wav /path/to/A.wav --register-text ""
python c5_tts.py --register-voice speaker_B --register-wav /path/to/B.wav --register-text ""

# 切换使用
python c5_tts.py --text "你好" --voice-id speaker_A -o a.wav
python c5_tts.py --text "你好" --voice-id speaker_B -o b.wav
```

### 场景3：情感 + 音色 + 语速 三合一

```bash
python c5_tts.py --text "太棒了，今天真是美好的一天！" --voice /path/to/ref.wav --speed 1.2 --emotion happy -o all_in_one.wav
```

### 场景4：长篇文本朗读

```bash
python c5_tts.py --text "人工智能（Artificial Intelligence，简称AI）是计算机科学的一个分支，它企图了解智能的实质，并生产出一种新的能以人类智能相似的方式做出反应的智能机器。" --voice /path/to/ref.wav --speed 1.0 -o long_text.wav
```

### 场景5：制作不同风格的语音样本

```bash
# 新闻播报风格
python c5_tts.py --text "据新华社报道，今年国民经济运行总体平稳" --instruct "用新闻播报的语气，客观冷静，发音清晰" -o news.wav

# 儿童故事风格
python c5_tts.py --text "从前有一只小兔子，它非常喜欢在森林里玩耍" --instruct "用温柔亲切的语气，像给小朋友讲故事一样" -o story.wav

# 广告营销风格
python c5_tts.py --text "限时抢购！不要错过这个千载难逢的机会！" --instruct "用富有激情和感染力的语气，像广告配音一样" -o ad.wav
```

### 场景6：语速渐变对比

```bash
# 生成一组不同语速的音频用于对比
python c5_tts.py --text "大家好，欢迎收听今天的节目" --speed 0.7 -o speed07.wav
python c5_tts.py --text "大家好，欢迎收听今天的节目" --speed 1.0 -o speed10.wav
python c5_tts.py --text "大家好，欢迎收听今天的节目" --speed 1.5 -o speed15.wav
```

### 场景7：跨语言克隆（用中文参考音频读英文）

```bash
# --voice 会自动用 cross_lingual 模式，无需关心参考音频的语言
python c5_tts.py --text "Hello, welcome to our intelligent speech synthesis system." --voice /path/to/chinese_speaker.wav -o zh_to_en.wav
```

---

## 10. 参数速查表

### 10.1 基础参数

| 参数 | 别名 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `--text` | — | str | 无 | 要合成的文本 |
| `-o` | `--output` | str | `../outputs/output.wav` | 输出WAV路径 |
| `--speed` | — | float | `1.0` | 语速倍率 (0.5~2.0) |
| `--lang` | — | str | `zh` | 语言: zh / en / mix |
| `--stream` | — | flag | 关闭 | 流式合成 |
| `--dataset` | — | str | — | 批量合成的JSON路径 |

### 10.2 音色参数

| 参数 | 别名 | 类型 | 说明 |
|------|------|------|------|
| `--voice` | `--voice-ref` | str | 参考音频路径（直接克隆，不注册） |
| `--voice-id` | — | str | 已注册音色ID |
| `--register-voice` | — | str | 注册新音色名称 |
| `--register-wav` | — | str | 注册用的参考音频路径 |
| `--register-text` | — | str | 参考音频的文字内容（留空=跨语言模式） |
| `--list-voices` | — | flag | 列出所有已注册音色 |

### 10.3 情感参数

| 参数 | 类型 | 可选值 | 说明 |
|------|------|--------|------|
| `--emotion` | str | happy / sad / angry / gentle / calm / excited / tired | 预设情感 |
| `--instruct` | str | 任意文本 | 自定义指令（覆盖 --emotion） |

### 10.4 进阶测试参数

| 参数 | 说明 |
|------|------|
| `--multilingual-test` | 多语种测试（6种） |
| `--speed-test` | 语速控制测试（5档） |
| `--timbre-test` | 音色控制测试 |
| `--emotion-test` | 情感控制测试（7种） |
| `--emotion-intensity` | 情感强度梯度测试（4级） |
| `--speaker-test` | 说话人克隆测试 |
| `--benchmark` | TTS模型横向对比 |
| `--all-advanced` | 全部进阶测试 |
| `--ref-audio-dir` | 参考音频目录 |

---

## 附录：环境变量

| 变量 | 说明 | 当前值 |
|------|------|--------|
| `COSYVOICE_REPO` | CosyVoice 仓库路径 | `/root/siton-tmp/lmy/CosyVoice` |
| `COSYVOICE_MODEL` | 模型权重路径 | `/root/siton-tmp/lmy/CosyVoice2-0.5B` |
| `AUDIO_DATA_DIR` | 音频数据目录 | — |

## 附录：路径约定

| 路径 | 说明 |
|------|------|
| `C5_TTS/code/` | 脚本所在目录（cd到这里执行） |
| `C5_TTS/outputs/` | 默认输出目录 |
| `C5_TTS/code/voice_bank/` | 音色库存储目录 |

---

> 所有命令均在 `cd /root/siton-tmp/lmy/C5_TTS/code` 之后执行
> 模型加载耗时约 10~15 秒（首次）