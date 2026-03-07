---
title: 语音识别：PyAudio、SoundDevice、Vosk、openai-whisper、Argos-Translate、FunASR（Python）
date: 2026-01-05 08:16:00
tags:
  - "语音识别"
  - "Whisper"
  - "Vosk"
  - "PyAudio"
  - "SoundDevice"
  - "FunASR"
categories:
  - "语音识别"
---

﻿@[TOC](语音识别：PyAudio、SoundDevice、Vosk、openai-whisper、Argos-Translate、FunASR（Python）)
# PyAudio
PyAudio 是 Python 中一个强大的**跨平台音频 I/O 库**。它基于 PortAudio 库（一个免费、跨平台、开源的音频 I/O 库），为 Python 提供了录制和播放音频流的接口。
## 核心功能

1.  **音频录制（输入）：** 从麦克风、音频接口或其他输入设备捕获音频数据。
2.  **音频播放（输出）：** 将音频数据发送到扬声器、耳机或其他输出设备。
3.  **实时音频流处理：** 核心优势在于处理音频数据流，允许在数据流入（录制）或流出（播放）时进行实时处理（如效果处理、分析、语音识别等）。
4.  **低延迟操作：** 设计目标之一是尽量减少音频输入和输出之间的延迟，这对实时交互应用（如乐器、语音聊天）至关重要。
5.  **跨平台：** 支持 Windows, macOS, Linux。

## 核心概念

1.  **`pyaudio.PyAudio` 类：**
    *   这是主要的入口点。你需要先创建一个 `PyAudio` 实例来访问音频功能。
    *   它负责管理 PortAudio 资源、查询设备信息和创建流。
    *   **重要方法：**
        *   `__init__()`: 构造函数，通常不需要显式参数。
        *   `get_device_count()`: 返回系统中可用的音频设备数量。
        *   `get_device_info_by_index(index)`: 获取指定索引设备的详细信息（字典形式），包含设备名、最大输入/输出通道数、默认采样率等。
        *   `get_default_input_device_info()`: 获取系统默认输入设备的信息。
        *   `get_default_output_device_info()`: 获取系统默认输出设备的信息。
        *   `open(...)`: **最关键的方法！** 用于打开一个新的音频流（输入、输出或同时）。返回一个 `pyaudio.Stream` 对象。参数众多，下面详解。
        *   `terminate()`: **非常重要！** 当你完成所有音频操作后，必须调用此方法来正确释放 PortAudio 分配的资源。通常放在 `finally` 块或使用上下文管理器（`with` 语句）来确保调用。

2.  **`pyaudio.Stream` 类：**
    *   代表一个活动的音频流（输入流、输出流或双工流）。
    *   通过 `PyAudio.open()` 创建。
    *   **重要方法：**
        *   `start_stream()`: 启动流。流开始使用回调函数或等待读写操作。
        *   `stop_stream()`: 暂停流，但不关闭它。可以稍后重新启动。
        *   `close()`: 关闭流并释放相关资源。应在流不再需要时调用。
        *   `is_active()`: 如果流正在运行（已启动且未停止/关闭），则返回 True。
        *   `is_stopped()`: 如果流被明确停止或未启动，则返回 True。
        *   `read(num_frames, exception_on_overflow=True)`: **(主要用于输入流或阻塞回调模式)** 从输入流中读取指定数量的音频帧（二进制数据）。`exception_on_overflow` 控制缓冲区溢出时是否引发异常。
        *   `write(frames, num_frames=None, exception_on_underflow=True)`: **(主要用于输出流或阻塞回调模式)** 将二进制音频数据（`frames`）写入输出流。`num_frames` 通常可以省略（由 `frames` 长度推断），`exception_on_underflow` 控制缓冲区欠载时是否引发异常。
    *   **工作模式：**
        *   **回调模式 (Callback Mode - 更常见且高效)：** 在 `open()` 时指定一个 `callback` 函数。每当流需要新的数据进行播放（输出）或有新的数据可供处理（输入）时，PortAudio 会自动调用这个函数。函数需要在有限时间内返回数据（输出）或处理数据（输入）。**这是处理实时音频的首选模式。**
        *   **阻塞模式 (Blocking Mode)：** 在 `open()` 时不指定 `callback`。你需要使用 `read()`（输入）或 `write()`（输出）方法在主线程中显式地、**阻塞地**读取或写入数据。主线程会在等待数据可用（输入）或缓冲区空间可用（输出）时被阻塞。适用于简单的、非实时的操作。

### `open()` 方法关键参数详解

```python
stream = p.open(
    format=FORMAT,                # 音频数据格式 (e.g., pyaudio.paInt16, pyaudio.paFloat32)
    channels=CHANNELS,            # 声道数 (1=单声道, 2=立体声)
    rate=RATE,                    # 采样率 (每秒样本数, e.g., 44100, 48000)
    input=INPUT,                  # 是否打开输入流 (True/False)
    output=OUTPUT,                # 是否打开输出流 (True/False)
    input_device_index=INPUT_DEVICE_INDEX,  # 可选，输入设备索引 (None 使用默认)
    output_device_index=OUTPUT_DEVICE_INDEX, # 可选，输出设备索引 (None 使用默认)
    frames_per_buffer=CHUNK,      # 每个缓冲区/块包含的帧数。影响延迟和CPU负载。
    stream_callback=CALLBACK,     # 可选，回调函数（用于回调模式）
    start=False                   # 可选，是否在打开后立即启动流 (默认为 False)
)
```

*   **`format`:** 定义音频数据的表示形式。常用值：
    *   `pyaudio.paInt16`: 16 位有符号整数（最常用，CD 质量）。
    *   `pyaudio.paInt24`: 24 位有符号整数（更高精度）。
    *   `pyaudio.paInt32`: 32 位有符号整数。
    *   `pyaudio.paFloat32`: 32 位浮点数（范围 -1.0 到 1.0，适合处理）。
*   **`channels`:** 1 表示单声道，2 表示立体声，更高表示多声道（需要硬件支持）。
*   **`rate`:** 采样率（Hz）。常见值：8000（电话）、16000、22050、44100（CD）、48000（专业音频）、96000、192000。必须与设备支持的采样率匹配。
*   **`input`/`output`:** 布尔值，指示流是用于输入（录音）、输出（播放）还是两者（双工）。
*   **`input_device_index`/`output_device_index`:** 指定要使用的设备索引（通过 `get_device_info_by_index` 获取）。`None` 表示使用默认设备。
*   **`frames_per_buffer` (CHUNK):** **极其重要！** 定义每次回调调用处理的帧数，或每次 `read()`/`write()` 操作传输的帧数。
    *   **帧 (Frame):** 所有通道在 *同一时间点* 的样本集合。对于立体声（2 声道），1 帧 = 2 个样本（左声道一个，右声道一个）。
    *   **块/缓冲区 (Chunk/Buffer):** 包含连续多帧数据的单位。`frames_per_buffer` 定义了每个缓冲区包含多少帧。
    *   **影响：** 较小的值 → 更低延迟，但更频繁的回调/IO 操作 → 更高 CPU 负载，更容易出现缓冲区欠载/溢出错误。较大的值 → 更高延迟，更少的回调/IO → 更低 CPU 负载，更稳定。需要根据应用在延迟和稳定性之间权衡（如实时效果常用 256, 512, 1024）。
*   **`stream_callback`:** **回调模式的核心。** 指定一个函数，当流需要数据（输出）或有新数据（输入）时自动调用。函数必须具有特定签名：`callback(in_data, frame_count, time_info, status_flags)`。在输出流中，函数需要返回 `(out_data, pyaudio.paContinue)`（或其他状态码）。在输入流中，函数接收数据 `in_data`。
*   **`start`:** 设为 `True` 会使流在 `open()` 后立即开始。通常保持 `False`，显式调用 `start_stream()`。

### 回调函数签名

```python
def callback(in_data: Optional[bytes],  # 输入数据 (输入流或双工流时有效，否则为None)
             frame_count: int,           # 期望返回/提供的帧数 (通常等于 frames_per_buffer)
             time_info: dict,            # 时间信息字典 (包含输入/输出缓冲时间戳)
             status_flags: int) -> (Optional[bytes], int): # 返回值: (输出数据, 状态码)
```

*   **对于纯输入流：** `in_data` 包含新捕获的音频数据（二进制）。函数应处理数据（或保存），并返回 `(None, pyaudio.paContinue)`（或其他状态码）。
*   **对于纯输出流：** `in_data` 为 `None`。函数应生成 `frame_count` 帧的音频数据（二进制），并返回 `(out_data, pyaudio.paContinue)`。
*   **对于双工流：** `in_data` 包含输入数据。函数应处理输入并生成输出数据，返回 `(out_data, pyaudio.paContinue)`。
*   **状态码：**
    *   `pyaudio.paContinue`: 流应继续。
    *   `pyaudio.paComplete`: 流正常结束（不再有数据）。
    *   `pyaudio.paAbort`: 流应立即中止（发生错误）。

### 典型工作流程

1.  **创建 PyAudio 实例：** `p = pyaudio.PyAudio()`
2.  **(可选) 查询设备信息：** 使用 `p.get_device_count()`, `p.get_device_info_by_index()` 等选择非默认设备。
3.  **打开流：**
    *   **回调模式：** `stream = p.open(..., stream_callback=my_callback_function)`
    *   **阻塞模式：** `stream = p.open(..., input=True, output=False) # 例如纯输入`
4.  **启动流：** `stream.start_stream()`
5.  **处理音频：**
    *   **回调模式：** 你的 `my_callback_function` 会被自动调用处理数据流。
    *   **阻塞模式：**
        *   **输入：** 循环调用 `data = stream.read(CHUNK)`，然后处理 `data`。
        *   **输出：** 循环生成音频数据 `data`，然后调用 `stream.write(data)`。
6.  **停止流：** `stream.stop_stream()`
7.  **关闭流：** `stream.close()`
8.  **终止 PyAudio：** `p.terminate()` **(绝对不能忘记这一步！)**

### 重要注意事项和最佳实践

1.  **延迟与块大小 (`frames_per_buffer`):** 仔细选择块大小。实时应用（如乐器、语音聊天）需要小值（如 256, 512），批处理或播放录制好的文件可以用大值（如 1024, 2048）。测试是关键。
2.  **缓冲区欠载/溢出：**
    *   **欠载 (Underflow):** 输出流：CPU 来不及提供足够数据，导致播放中断（咔嗒声/爆音）。通常因回调函数太慢或块大小太小导致。
    *   **溢出 (Overflow):** 输入流：CPU 来不及处理捕获的数据，导致部分音频数据丢失。
    *   **处理：** 增加 `frames_per_buffer`（增加延迟），优化回调函数代码（提高效率），检查系统负载。PyAudio 可以在这些情况下抛出异常（取决于 `read`/`write` 的参数）。
3.  **数据类型转换：** PyAudio 处理的是原始的二进制数据（字节串）。要使用 `numpy` 或 `struct` 等库进行处理（如应用效果、可视化），你需要将这些字节串转换为数值数组（如 `np.frombuffer(data, dtype=np.int16)`），处理后再转换回字节串（如 `processed_data.astype(np.int16).tobytes()`）。
4.  **线程安全：** PortAudio 回调函数在专门的音频线程中执行。如果你的程序是多线程的，需要确保共享数据访问的线程安全（使用锁等同步机制）。避免在回调函数中执行耗时操作（文件 I/O、复杂计算、网络请求）。
5.  **设备兼容性与采样率：** 不是所有采样率和格式都被所有设备支持。使用 `get_device_info_by_index()` 检查设备支持的能力，或尝试打开流时处理异常。有时需要重新采样或转换格式。

### 简单代码示例

1.  **录制音频到 WAV 文件 (阻塞模式):**

```python
import pyaudio
import wave

CHUNK = 1024
FORMAT = pyaudio.paInt16
CHANNELS = 1
RATE = 44100
RECORD_SECONDS = 5
WAVE_OUTPUT_FILENAME = "output.wav"

p = pyaudio.PyAudio()

stream = p.open(format=FORMAT,
                channels=CHANNELS,
                rate=RATE,
                input=True,
                frames_per_buffer=CHUNK)

print("* recording")

frames = []

for i in range(0, int(RATE / CHUNK * RECORD_SECONDS)):
    data = stream.read(CHUNK)
    frames.append(data)

print("* done recording")

stream.stop_stream()
stream.close()
p.terminate()

wf = wave.open(WAVE_OUTPUT_FILENAME, 'wb')
wf.setnchannels(CHANNELS)
wf.setsampwidth(p.get_sample_size(FORMAT))
wf.setframerate(RATE)
wf.writeframes(b''.join(frames))
wf.close()
```

2.  **播放 WAV 文件 (阻塞模式):**

```python
import pyaudio
import wave

CHUNK = 1024

wf = wave.open('output.wav', 'rb')  # 用上面录制的文件

p = pyaudio.PyAudio()

stream = p.open(format=p.get_format_from_width(wf.getsampwidth()),
                channels=wf.getnchannels(),
                rate=wf.getframerate(),
                output=True)

data = wf.readframes(CHUNK)

while data:
    stream.write(data)
    data = wf.readframes(CHUNK)

stream.stop_stream()
stream.close()
p.terminate()
```

3.  **实时回显 (输入->立即播放) (回调模式):**

```python
import pyaudio

CHUNK = 512
FORMAT = pyaudio.paInt16
CHANNELS = 1
RATE = 44100

def callback(in_data, frame_count, time_info, status):
    # 直接将输入数据作为输出数据返回 (回显)
    return (in_data, pyaudio.paContinue)

p = pyaudio.PyAudio()

# 打开双工流
stream = p.open(format=FORMAT,
                channels=CHANNELS,
                rate=RATE,
                input=True,
                output=True,
                frames_per_buffer=CHUNK,
                stream_callback=callback)

stream.start_stream()

print("Echoing. Press Enter to stop...")
input()  # 等待用户按回车

stream.stop_stream()
stream.close()
p.terminate()
```

---
# SoundDevice
## 核心类与函数
### 1. `sounddevice.play()` - 播放音频数组
```python
sounddevice.play(data, samplerate=None, blocksize=0, loop=False, mapping=None, 
                clipping=None, latency='low', extra_settings=None, device=None, 
                dtype=None)
```
- **`data`**: 音频数据（NumPy 数组，形状为 `(samples, channels)`）。
- **`samplerate`**: 采样率（Hz），默认使用设备默认值（通常 44100）。
- **`blocksize`**: 音频块大小（样本数），0 表示自动选择。
- **`loop`**: 是否循环播放（`True`/`False`）。
- **`device`**: 输出设备 ID 或名称（通过 `query_devices()` 获取）。
- **`dtype`**: 输出数据类型（如 `'float32'`, `'int16'`），默认与输入一致。

### 2. `sounddevice.rec()` - 录制音频
```python
sounddevice.rec(frames, samplerate=None, channels=None, dtype='float32', 
               blocking=False, **kwargs)
```
- **`frames`**: 录制的样本数（长度）。
- **`channels`**: 录制通道数（默认 1）。
- **`blocking`**: 是否阻塞程序直到录制完成（`True`/`False`）。
- **返回值**: 录制的音频数组（NumPy 数组）。

### 3. `InputStream` / `OutputStream` - 高级流控制
```python
# 输入流（录制）
stream_in = sounddevice.InputStream(
    samplerate=44100, 
    channels=1,
    device='microphone',
    callback=record_callback
)

# 输出流（播放）
stream_out = sounddevice.OutputStream(
    samplerate=44100, 
    channels=2,
    device='speakers',
    callback=play_callback
)
```
- **`callback`**: 回调函数，处理实时音频数据（原型：`callback(indata, outdata, frames, time, status)`）。
- **`blocksize`**: 每次回调处理的帧数（影响延迟）。
- **`latency`**: 延迟设置（`'low'`, `'high'` 或秒数）。

---
#### 1. `device` 参数
- **作用**: 指定输入/输出设备。
- **获取设备列表**:
  ```python
  import sounddevice as sd
  print(sd.query_devices())  # 列出所有设备
  print(sd.default.device)   # 查看默认设备
  ```
#### 2. `samplerate` 参数
- 必须与音频数据匹配。常见值：44100 Hz（CD质量）、48000 Hz。
- 设备支持的值可通过 `sd.query_devices(device_id)['default_samplerate']` 查询。

#### 3.`blocksize` 参数
- 较小的值 → 更低延迟，但可能引发音频中断。
- 较大的值 → 更稳定，但延迟更高。
- 推荐：0（自动选择）或 256/512。

#### 4. `latency` 参数
- `'low'`: 最小延迟（可能不稳定）。
- `'high'`: 稳定但延迟较高。
- 数值：如 `0.1`（100毫秒）。

#### 5. `dtype` 参数
- 控制音频数据类型：
  - `'float32'`（范围 [-1.0, 1.0]）
  - `'int16'`（范围 [-32768, 32767]）
- 录制时指定 `dtype='int16'` 可节省内存。

---

### 完整示例
#### 录制并播放音频（10秒）
```python
import sounddevice as sd
import numpy as np

# 录制音频
duration = 10  # 秒
fs = 44100     # 采样率
recording = sd.rec(int(duration * fs), samplerate=fs, channels=1)
sd.wait()  # 等待录制完成

# 播放录制的音频
sd.play(recording, samplerate=fs)
sd.wait()
```

#### 实时音频处理（回环示例）
```python
def callback(indata, outdata, frames, time, status):
    if status:
        print("Error:", status)
    outdata[:] = indata  # 将输入直接复制到输出（实时回环）

# 创建双向流
with sd.Stream(
    device=(sd.default.input_device, sd.default.output_device),
    samplerate=44100,
    channels=1,
    callback=callback
):
    print("实时回环中... 按 Enter 停止")
    input()
```
---
## RawInputStream
`sounddevice.RawInputStream` 是 `sounddevice` 库中用于处理原始音频输入的低级接口，特别适用于需要直接访问未处理音频字节的场景（如音频编码、网络传输等）。
### RawInputStream 核心参数详解
```python
class sounddevice.RawInputStream(
    samplerate=None,
    blocksize=None,
    device=None,
    channels=None,
    dtype='float32',
    latency=None,
    extra_settings=None,
    callback=None,
    finished_callback=None,
    clip_off=None,
    dither_off=None,
    never_drop_input=None,
    prime_output_buffers_using_stream_callback=None,
    **kwargs
)
```

#### 关键参数说明
1. **`samplerate`** (int, 可选)  
   - 采样率（Hz），默认使用设备默认值
   - **示例**: `samplerate=48000`

2. **`blocksize`** (int, 可选)  
   - 每次回调处理的帧数（样本数）
   - 较小值 → 低延迟但CPU负担重
   - 较大值 → 高延迟但稳定
   - **推荐**: `0`（自动选择）或 `256/512/1024`

3. **`device`** (int/str, 可选)  
   - 输入设备ID或名称
   - 获取设备列表: `sd.query_devices()`
   - **示例**: `device='麦克风阵列'` 或 `device=3`

4. **`channels`** (int, 可选)  
   - 音频通道数（默认使用设备最大通道数）
   - **示例**: `channels=2`（立体声）

5. **`dtype`** (str, 必需)  
   - 音频数据格式（**原始字节格式**）
   - 支持类型: `'float32'`, `'int32'`, `'int16'`, `'int8'`, `'uint8'`
   - **注意**: 不同于普通InputStream，这里数据以原始字节形式传递

6. **`latency`** (str/float, 可选)  
   - 延迟设置：`'low'`（低延迟）、`'high'`（高稳定性）或秒数
   - **示例**: `latency=0.05`（50ms延迟）

7. **`callback`** (callable, 必需)  
   - 核心回调函数，原型：
     ```python
     def callback(indata: bytes, frames: int, time: CData, status: CallbackFlags) -> None
     ```
   - `indata`: 原始音频字节（**非NumPy数组**）
   - `frames`: 本次回调的帧数
   - `time`: 时间戳信息
   - `status`: 错误状态标志

---

### 与普通 InputStream 的区别
| 特性                | RawInputStream                     | InputStream               |
|---------------------|------------------------------------|---------------------------|
| **数据格式**        | 原始字节 (`bytes`)                 | NumPy数组                 |
| **内存开销**        | 更低（无格式转换）                 | 稍高                      |
| **使用场景**        | 编码/网络传输/硬件交互             | 实时处理/分析             |
| **数据处理**        | 需手动解析字节                     | 直接使用数组              |
| **性能**            | 更高（避免格式转换开销）           | 稍低                      |

---

### 完整使用示例
#### 1. 原始音频录制（保存为WAV）
```python
import sounddevice as sd
import wave

# 配置参数
CHUNK = 1024
FORMAT = 'int16'  # 原始格式
CHANNELS = 1
RATE = 44100
RECORD_SECONDS = 5

# 创建WAV文件
wf = wave.open("raw_audio.wav", 'wb')
wf.setnchannels(CHANNELS)
wf.setsampwidth(2)  # 16-bit = 2 bytes
wf.setframerate(RATE)

def callback(indata: bytes, frames, time, status):
    """接收原始字节并写入文件"""
    wf.writeframes(indata)

# 创建原始输入流
stream = sd.RawInputStream(
    samplerate=RATE,
    blocksize=CHUNK,
    dtype=FORMAT,
    channels=CHANNELS,
    callback=callback
)

# 开始录制
print("Recording raw audio...")
stream.start()
sd.sleep(int(RECORD_SECONDS * 1000))
stream.stop()
wf.close()
print("Done")
```
#### 2. 实时原始音频处理（字节转数组）
```python
import sounddevice as sd
import numpy as np

def callback(indata: bytes, frames, time, status):
    """将原始字节转为NumPy数组处理"""
    if status:
        print("Error:", status)
    
    # 字节 → NumPy数组 (int16格式)
    audio_array = np.frombuffer(indata, dtype=np.int16)
    
    # 示例处理：计算分贝值
    rms = np.sqrt(np.mean(audio_array**2))
    db = 20 * np.log10(rms / 32768)  # 16-bit范围[-32768, 32767]
    print(f"当前音量: {db:.1f} dB")

# 创建原始输入流
stream = sd.RawInputStream(
    samplerate=48000,
    blocksize=512,
    dtype='int16',
    callback=callback
)

# 开始处理
stream.start()
input("按Enter停止...")
stream.stop()
```

---

### 高级技巧与注意事项
1. **字节解析技巧**  
   根据 `dtype` 解析字节：
   ```python
   # float32 (4字节/样本)
   data = np.frombuffer(indata, dtype=np.float32)
   
   # int16 (2字节/样本)
   data = np.frombuffer(indata, dtype=np.int16)
   ```

2. **避免回调阻塞**  
   - 回调中避免耗时操作（如文件写入）
   - 使用队列将数据传递到后台线程：
     ```python
     from queue import Queue
     audio_queue = Queue()
     
     def callback(indata, ...):
         audio_queue.put(indata)
     ```

3. **错误处理**  
   检查 `status` 参数：
   ```python
   def callback(indata, frames, time, status):
       if status.input_overflow:
           print("输入溢出！增加blocksize")
       if status:
           print("PortAudio错误:", status)
   ```

4. **低延迟优化**  
   - 设置 `latency='low'`
   - 使用专用音频驱动：ASIO (Windows), JACK (Linux)
   - 减少 `blocksize`（但需测试稳定性）

---

### 常见问题解决
**Q1: 为什么回调收到的数据长度不对？**  
- 检查 `dtype` 设置：每个样本字节数 × 通道数 × 帧数应等于 `len(indata)`
- 示例：`int16`立体声，1024帧 → 2字节 × 2通道 × 1024 = 4096字节

**Q2: 如何实现零拷贝处理？**  
- 直接操作字节对象（避免 `np.frombuffer` 复制）：
  ```python
  def callback(indata, ...):
      # 直接处理字节 (示例：反转字节序)
      swapped = bytes(reversed(indata))
  ```

**Q3: 出现 `PortAudioError: Unanticipated host error`？**  
- 降低采样率或增加 `blocksize`
- 关闭其他占用音频设备的程序
- 更新音频驱动程序

---
# Vosk
Vosk 是一个高效的离线语音识别库，支持实时流式处理和多种语言（包括中文）。以下是其核心参数详解及使用示例，结合 Python 实现：
## 一、Vosk 核心参数详解
### 1. 模型加载参数 (`Model` 类)
- **`model_path`** (str)  ：模型解压后的目录路径。  
### 2. 识别器参数 (`KaldiRecognizer` 类)
- **`model`** (Model 对象)  
  加载的语音识别模型实例。
- **`sample_rate`** (int)  
  音频采样率（单位：Hz），**必须与输入音频一致**，通常为 `16000` 或 `8000`。  
- **`show_words`** (bool, 可选)  
  是否在结果中返回每个单词的时间戳和置信度（默认 `False`）。  
  ```python
  from vosk import KaldiRecognizer
  rec = KaldiRecognizer(model, 16000, show_words=True)
  ```
### 3. 音频处理参数
- **音频格式要求**  
  - 单声道（mono）、16-bit PCM、采样率匹配 `sample_rate`。
  - 若音频格式不符，需用 `ffmpeg` 转换：  
    ```python
    process = subprocess.Popen([
        'ffmpeg', '-i', 'input.mp3',  # 输入文件
        '-ar', '16000',               # 采样率
        '-ac', '1',                   # 单声道
        '-f', 's16le', '-'            # 输出格式为16-bit PCM
    ], stdout=subprocess.PIPE)
    ```
### 4. 识别结果获取方法
- **`AcceptWaveform(data: bytes)`**  
  传入音频数据块（字节流），返回 `True` 当有完整句子识别完成。  
- **`Result()`**  
  返回完整句子的 JSON 结果（包含 `"text"` 字段）。  
- **`PartialResult()`**  
  返回中间识别结果（实时流处理时常用）。  
- **`FinalResult()`**  
  返回最终识别结果并重置识别器。  

---
### 二、完整代码示例
#### 1. 实时麦克风语音识别
```python
import sounddevice as sd
from vosk import Model, KaldiRecognizer
import queue

# 初始化模型和识别器
model = Model("model")  # 模型路径
rec = KaldiRecognizer(model, 16000)
audio_queue = queue.Queue()

def audio_callback(indata, frames, time, status):
    audio_queue.put(bytes(indata))  # 原始音频字节流

# 开始录音
with sd.RawInputStream(samplerate=16000, dtype='int16', callback=audio_callback):
    print("实时识别中... 按 Ctrl+C 停止")
    while True:
        data = audio_queue.get()
        if rec.AcceptWaveform(data):
            result = rec.Result()
            print("完整结果:", result)  # 输出完整句子
        else:
            partial = rec.PartialResult()
            print("中间结果:", partial)  # 实时输出部分识别
```

#### 2. 音频文件识别（结合ffmpeg）
```python
from vosk import Model, KaldiRecognizer
import subprocess
import json

model = Model("model")
rec = KaldiRecognizer(model, 16000)

# 通过ffmpeg转换音频格式
process = subprocess.Popen([
    'ffmpeg', '-i', 'input.mp3',
    '-ar', '16000', '-ac', '1', '-f', 's16le', '-'
], stdout=subprocess.PIPE)

results = []
while True:
    data = process.stdout.read(4000)
    if not data: 
        break
    if rec.AcceptWaveform(data):
        result = json.loads(rec.Result())
        results.append(result["text"])

# 获取最终结果
final = json.loads(rec.FinalResult())
results.append(final["text"])
print("识别结果:", " ".join(results))
```

---
### 三、参数配置技巧
#### 1. 优化识别准确率
- **模型选择**：  
  - 小模型（~50MB）：适合移动端（如 `vosk-model-small-cn-0.22`）。  
  - 大模型（~1.3GB）：适合服务器（如 `vosk-model-cn-0.15`）。  
- **词汇表定制**：  
  通过 `Rec.SetWords(True)` 启用单词级输出，动态扩展专业词汇。

#### 2. 实时流处理优化
- **块大小 (`blocksize`)**：  
  较小的值（如 `4000` 字节）降低延迟，但增加CPU负载。  
- **部分结果频率**：  
  调用 `PartialResult()` 控制实时反馈的粒度。

#### 3. 输出结果处理
- **提取有效文本**：  
  ```python
  def extract_text(json_str):
      result = json.loads(json_str)
      return result.get("text", "")
  ```
- **保存时间戳**（需 `show_words=True`）：  
  ```json
  {
    "text": "你好世界",
    "result": [{"conf": 0.95, "start": 1.2, "end": 1.5, "word": "你好"}, ...]
  }
  ```

---

# OpenAI Whisper
##  一、模型选择与加载参数
Whisper 提供多种模型尺寸，在速度与精度间权衡：

| **模型类型** | **参数规模** | **显存占用** | **相对速度** | **适用场景**               | 
|--------------|--------------|--------------|--------------|----------------------------|
| `tiny`       | 39M          | ~1GB         | 32x          | 移动端/实时低延迟          | 
| `base`       | 74M          | ~1GB         | 16x          | 平衡场景（推荐默认）       | 
| `small`      | 244M         | ~2GB         | 6x           | 高精度转录                 | 
| `medium`     | 769M         | ~5GB         | 2x           | 专业场景（医疗/学术）      | 
| `large-v3`   | 1550M        | ~10GB        | 1x           | 极致精度（多语言/复杂口音）|

- **首次加载**：自动下载模型至 `~/.cache/whisper/`，可通过 `model_dir` 指定本地路径：
  ```python
  model = whisper.load_model("small", download_root="/path/to/models")
  ```

---
## 二、转录核心参数详解
`transcribe()` 是核心方法，关键参数如下：

```python
result = model.transcribe(
    audio="input.wav",      # 音频路径或NP数组
    language="zh",          # 强制指定语言（zh/en/es等）
    temperature=0.2,        # 采样随机性（0-1，低=确定性强）
    fp16=False,             # 关闭FP16加速（CPU环境必设）
    verbose=True,           # 打印实时进度
    word_timestamps=True,   # 返回词级时间戳
    initial_prompt="以下是普通话："  # 提供上下文提示
)
```
- `task="transcribe"`：将语音转录为原始语言（默认值）。

- `task="translate"`：将语音翻译成英语（无论原始语言是什么）。
### 参数效果对比表：
| **参数**            | **推荐值**       | **影响说明**                                                                 |
|---------------------|------------------|----------------------------------------------------------------------------|
| `language`          | `None` (自动检测) | 显式设置（如`"zh"`)可提升非英语识别精度15%+                    |
| `temperature`       | `0.0` (贪婪解码) | >0 增加多样性，但可能引入错误词；长文本建议设为0 |
| `word_timestamps`   | `True`           | 生成逐词时间戳（适配字幕场景），输出包含 `segments[i]['words']` |
| `initial_prompt`    | 领域关键词       | 提示模型专注特定领域（如医疗术语），错误率降低可达10%           |

---
## 三、高级使用与性能优化
### 1. 长音频分段处理策略
Whisper 默认处理30秒片段，长音频需拆解：
```python
audio = whisper.load_audio("long.mp3")
segments = []
for i in range(0, len(audio), 30 * 16000):  # 30秒步进
    chunk = audio[i:i+30*16000]
    result = model.transcribe(chunk)
    segments.extend(result["segments"])
```
- **为何分段**：避免显存溢出，且模型对短音频优化更好

### 2. 输出后处理技巧
- **术语替换**：针对行业术语纠错  
  ```python
  term_map = {"心机梗塞": "心肌梗死", "糖料病": "糖尿病"}
  text = result["text"]
  for wrong, correct in term_map.items():
      text = text.replace(wrong, correct)
  ```
- **时间戳对齐**：合并连续片段  
  ```python
  merged = []
  current_seg = segments[0]
  for seg in segments[1:]:
      if seg['start'] - current_seg['end'] < 0.5:  # 间隔小于0.5秒合并
          current_seg['text'] += seg['text']
          current_seg['end'] = seg['end']
      else:
          merged.append(current_seg)
  ```

### 3. 速度优化方案
- **硬件加速**：
  - GPU用户：开启 `fp16=True`（默认启用）
  - CPU优化：用 `whisper-ctranslate2` 替代库，提速4倍
- **批处理**：避免重复加载模型  
  ```python
  model = whisper.load_model("base")  # 全局加载一次
  files = ["a.wav", "b.wav"]
  for f in files:
      model.transcribe(f)
  ```

---

##  四、典型应用场景代码
### 1. 会议录音转结构化纪要
```python
import whisper
from openai import OpenAI

# 语音转文本
model = whisper.load_model("medium")
result = model.transcribe("meeting.mp3", word_timestamps=True)

# 调用GPT-4总结重点
client = OpenAI(api_key="YOUR_KEY")
response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=[{"role": "user", "content": f"总结会议要点：{result['text']}"}]
)
print(response.choices[0].message.content)
```

### 2. 带时间轴的字幕生成
```python
result = model.transcribe("video.mp4", word_timestamps=True)
with open("subtitle.srt", "w") as f:
    for i, seg in enumerate(result["segments"]):
        start = seg["start"]
        end = seg["end"]
        text = seg["text"]
        f.write(f"{i+1}\n{start:.3f} --> {end:.3f}\n{text}\n\n")
```

 - **中文识别不准**
   - 显式设置 `language="zh"`
   - 添加提示词：`initial_prompt="以下是普通话对话"`
---
# Argos-Translate
## 一、安装与语言包管理参数
### 1. 环境配置与包管理
- **`update_package_index()`**  
  更新可用的语言包列表（需联网）。首次使用前必须调用，否则无法获取最新模型。  
  ```python
  import argostranslate.package
  argostranslate.package.update_package_index()  # 更新包索引
  ```

- **`get_available_packages()`**  
  获取所有可用语言包（返回对象列表）。每个包包含 `from_code`、`to_code`、`package_name` 等属性。  
  ```python
  available_packages = argostranslate.package.get_available_packages()
  ```

- **`install_from_path(path)`**  
  安装本地下载的语言包（`.argosmodel` 文件）。  
  ```python
  package_to_install = next(filter(lambda x: x.from_code=="en" and x.to_code=="zh", available_packages))
  argostranslate.package.install_from_path(package_to_install.download())
  ```

### 2. 自定义存储路径
  默认缓存路径在 `~/.cache/argos-translate`，可通过环境变量修改：  
  ```python
  import os
  os.environ["XDG_CACHE_HOME"] = "./custom_cache"  # 自定义缓存目录
  os.environ["XDG_DATA_HOME"] = "./custom_data"    # 自定义数据目录
  ```
  **适用场景**：避免占用系统盘空间，或分布式部署时统一管理路径。

---

## 二、核心翻译参数
### 1. 基础翻译函数
- **`translate(text, from_code, to_code)`**  
  - `text`: 待翻译文本（字符串）  
  - `from_code`: 源语言代码（如 `"en"`）  
  - `to_code`: 目标语言代码（如 `"zh"`）  
  **返回**：翻译后的字符串。  
  ```python
  translated_text = argostranslate.translate.translate("Hello World", "en", "zh")
  print(translated_text)  # 输出：你好，世界
  ```

## 三、高级功能参数
### 1. 中间语言自动转换
  当直接语言对（如 `ja→zh`）未安装时，自动通过中间语言（如英语 `en`）桥接：  
  ```python
  # 只需安装 ja→en 和 en→zh 包，即可实现 ja→zh 翻译
  translated_text = translate("こんにちは", "ja", "zh")  # 输出：你好
  ```
  **注意**：转换层级增加可能降低翻译质量。

### 2. GPU 加速
  通过环境变量启用 CUDA 加速（需安装 PyTorch GPU 版）：  
  ```python
  os.environ["ARGOS_DEVICE_TYPE"] = "cuda"  # 或 "auto"
  ```
  **效果**：大型模型（如 `en→zh`）翻译速度提升 3-5 倍。

---

## 四、应用场景参数配置
### 1. Web API 服务（Flask 示例）
  ```python
  from flask import Flask, request, jsonify
  import argostranslate.translate as translate

  app = Flask(__name__)

  @app.route('/translate', methods=['POST'])
  def api_translate():
      data = request.json
      text = data["text"]
      from_code = data["from_code"]
      to_code = data["to_code"]
      try:
          result = translate.translate(text, from_code, to_code)
          return jsonify({"translation": result})
      except Exception as e:
          return jsonify({"error": str(e)}), 500
  ```
  **关键参数**：  
  - `text`: 用户输入的文本  
  - `from_code`/`to_code`: 动态指定的语言对。

### 2. 批量文件翻译
  ```python
  from argostranslate import translate, package

  def translate_file(input_path, output_path, from_code, to_code):
      with open(input_path, "r") as f:
          text = f.read()
      translated = translate.translate(text, from_code, to_code)
      with open(output_path, "w") as f:
          f.write(translated)

  translate_file("doc_en.txt", "doc_zh.txt", "en", "zh")
  ```
  **适用场景**：本地化文档、多语言报告生成。

---
## get_translation_from_codes
在 Argos Translate 中，`get_translation_from_codes()` 是**核心翻译对象创建函数**，它根据语言代码创建可复用的翻译器对象。
### 函数定义与参数详解
```python
from argostranslate.translate import get_translation_from_codes

# 创建翻译器对象
translator = get_translation_from_codes(
    from_code="en",    # 源语言代码 (ISO 639-1)
    to_code="zh",      # 目标语言代码 (ISO 639-1)
    max_cache_size=100 # 缓存大小 (可选)
)
```

### 参数说明表
| **参数**        | **类型** | **必选** | **默认值** | **说明**                                                                 |
|-----------------|----------|----------|------------|--------------------------------------------------------------------------|
| `from_code`     | `str`    | ✅        | -          | 源语言代码 (如 `"en"`=英语, `"zh"`=中文)                                  |
| `to_code`       | `str`    | ✅        | -          | 目标语言代码 (如 `"es"`=西班牙语, `"fr"`=法语)                            |
| `max_cache_size`| `int`    | ❌        | `0`        | 翻译缓存大小（0=禁用缓存）                                                |
| `priority`      | `int`    | ❌        | `0`        | 翻译路径优先级（仅当存在多条路径时生效）                                   |

---
### 核心功能与优势
#### 1. 复用翻译器提升效率
```python
# 创建翻译器（只需一次）
en_to_zh = get_translation_from_codes("en", "zh")

# 多次复用（避免重复加载模型）
text1 = en_to_zh.translate("Hello World")  # 你好，世界
text2 = en_to_zh.translate("Good morning") # 早上好
```
- **性能提升**：复用翻译器比每次调用 `translate()` 快 5-10 倍
- **内存优化**：避免重复加载模型（大型模型如 `zh→en` 约 500MB）

#### 2. 智能中间语言桥接
当直接语言对未安装时，自动寻找最优路径：
```python
# 未安装 ja→zh 但安装了 ja→en 和 en→zh
jp_to_cn = get_translation_from_codes("ja", "zh")
result = jp_to_cn.translate("ありがとう")  # 输出：谢谢
```
- **路径选择逻辑**：优先选择最短路径（最少中间语言）
- **手动指定路径**：
  ```python
  # 强制通过法语中转
  translator = get_translation_from_codes("de", "ru", priority=1)
  ```

#### 3. 缓存机制加速重复内容
```python
# 启用缓存（存储最近100条翻译）
translator = get_translation_from_codes("en", "es", max_cache_size=100)

# 首次翻译（较慢）
translator.translate("Hello")  # => Hola

# 重复内容直接读缓存（极速）
translator.translate("Hello")  # 从缓存返回"Hola"
```
- **适用场景**：处理大量重复文本（如日志文件、模板内容）

---

### 高级使用技巧
#### 1. 批量翻译优化
```python
translator = get_translation_from_codes("fr", "de")

# 一次性提交多个句子
sentences = ["Bonjour", "Comment ça va?", "Merci"]
results = [translator.translate(s) for s in sentences]

print(results) 
# ["Hallo", "Wie geht es dir?", "Danke"]
```

#### 2. 动态语言切换
```python
class MultiTranslator:
    def __init__(self):
        self.cache = {}
    
    def translate(self, text, from_code, to_code):
        # 创建或复用翻译器
        key = f"{from_code}-{to_code}"
        if key not in self.cache:
            self.cache[key] = get_translation_from_codes(from_code, to_code)
        return self.cache[key].translate(text)

# 使用示例
mt = MultiTranslator()
print(mt.translate("Hello", "en", "zh"))  # 你好
print(mt.translate("Hola", "es", "ja"))   # こんにちは
```

#### 3. 错误处理与调试
```python
try:
    # 尝试创建不存在的语言对翻译器
    translator = get_translation_from_codes("xx", "yy")
except Exception as e:
    print(f"错误: {str(e)}")
    # 输出: "No path from 'xx' to 'yy'"

# 检查可用路径
from argostranslate.translate import get_installed_languages
langs = {l.code: l for l in get_installed_languages()}
if langs["en"].has_translation_to(langs["zh"]):
    print("en→zh 翻译可用")
```

---
### 完整工作流程示例
```python
from argostranslate.package import update_package_index, install_package
from argostranslate.translate import get_translation_from_codes

# 1. 更新包索引
update_package_index()

# 2. 安装所需语言包
packages = get_available_packages()
en_to_zh = next(pkg for pkg in packages if pkg.from_code=="en" and pkg.to_code=="zh")
install_package(en_to_zh)

# 3. 创建翻译器
translator = get_translation_from_codes("en", "zh", max_cache_size=50)

# 4. 批量处理文本
documents = [
    "Artificial Intelligence is changing the world.",
    "Machine learning algorithms improve over time.",
    "Neural networks mimic the human brain."
]

translated_docs = [translator.translate(doc) for doc in documents]

# 输出结果
for en, zh in zip(documents, translated_docs):
    print(f"EN: {en}\nZH: {zh}\n")
```
# FunASR
##  AutoModel
1. **`model`** (必选)  

   - **说明**：主 ASR 模型名称或本地路径，支持预训练模型（如 `paraformer-zh`、`iic/SenseVoiceSmall`）。  

   - **示例**：  

     ```python
     model = AutoModel(model="paraformer-zh")  # 中文通用模型
     model = AutoModel(model="iic/SenseVoiceSmall")  # 轻量级多语种模型
     ```

   - **注意**：首次使用会从 ModelScope 自动下载模型。

2. **`vad_model`**  

   - **说明**：语音活动检测（VAD）模型，用于分割音频中的语音段，如 `fsmn-vad`。  

3. **`punc_model`**  

   - **说明**：标点恢复模型（如 `ct-punc`），为识别文本添加标点。

4. **`model_revision`** / **`vad_model_revision`**  
	- **说明**：模型版本号（如 `v2.0.4`），避免版本不兼容问题。英文模型需指定 `model_revision="v2.0.3"`。

5. **`disable_update`**  
	- **说明**：设为 `True` 禁用模型自动更新，确保本地模型稳定性。

6.  **`hub`**  
	- **说明**：模型下载源，默认为 `"ms"`（ModelScope）。可切换至 Hugging Face 、本地等。
# 参考

 1. [https://python-sounddevice.readthedocs.io/en/0.5.1/](https://python-sounddevice.readthedocs.io/en/0.5.1/)
 2. [https://people.csail.mit.edu/hubert/pyaudio/docs/](https://people.csail.mit.edu/hubert/pyaudio/docs/)
 3. [https://ffmpeg.org/](https://ffmpeg.org/)
 4. [https://alphacephei.com/vosk/](https://alphacephei.com/vosk/)
 5. [https://openai.com/index/whisper/](https://openai.com/index/whisper/)
 6. [https://www.argosopentech.com/argospm/index/](https://www.argosopentech.com/argospm/index/)
 7. [https://github.com/modelscope/FunASR](https://github.com/modelscope/FunASR)
 8. [https://github.com/modelscope/FunASR/blob/main/runtime/docs/SDK_advanced_guide_online_zh.md](https://github.com/modelscope/FunASR/blob/main/runtime/docs/SDK_advanced_guide_online_zh.md)
![在这里插入图片描述](/img/b1bd309e8fe04c548d1af4ca07616ee5.png)

