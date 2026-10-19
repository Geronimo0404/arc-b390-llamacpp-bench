# Intel Arc B390 本地模型实测

使用 Xiaomi Book Pro 14，在 llama.cpp 上运行 [Qwen3.6-35B-A3B](https://huggingface.co/unsloth/Qwen3.6-35B-A3B-GGUF) 模型。

---

## 硬件

|      |                                           |
| ---- | ----------------------------------------- |
| CPU  | Intel Core Ultra X7 358H                  |
| 核显 | Intel Arc B390                            |
| 内存 | 31.5 GiB LPDDR5x 统一内存 / 24.0 GiB 显存 |
| 系统 | Windows 11 build 26200                    |
| 驱动 | Intel Graphics 32.0.101.8860              |

---

## 配置

llama.cpp **b11429**，Vulkan 构建。

`config.ini`：

```ini
[*]
ctx-size           = 131072
cache-type-k       = q8_0
cache-type-v       = q8_0
n-gpu-layers       = 999
parallel           = 1
flash-attn         = on
sleep-idle-seconds = 300

[Qwen3.6 35B A3B:IQ4_XS]
model           = /path/to/Qwen3.6-35B-A3B-UD-IQ4_XS.gguf
mmproj          = /path/to/mmproj-BF16.gguf
reasoning       = off
load-on-startup = 0
```

启动：

```bash
llama-server --models-preset config.ini --models-max 1
```

---

## 基准

`llama-bench -ngl 999 -p 64 -n 32 -r 3`

| 模型                    | 大小      | 参数             | pp64 (t/s)            | tg32 (t/s)          |
| ----------------------- | --------- | ---------------- | --------------------- | ------------------- |
| Qwen3.6-35B-A3B: IQ4_XS | 16.50 GiB | 34.66 B (3B激活) | **382.84 ± 18.75** | **30.02 ± 0.33** |

> pp64 = 提示词处理速度（64 token），tg32 = 生成速度（32 token），单位均为 token/秒。

---

## 上下文与显存

`-ngl 999 -fa on -ctk q8_0 -ctv q8_0`，模型加载后的进程 GPU 占用：

| 上下文  | GPU 占用  |
| ------- | --------- |
| 131,072 | 18,718 MB |

---

## 测试命令

```bash
llama-bench.exe -m <model.gguf> -ngl 999 -p 64 -n 32 -r 3
```

---

MIT
