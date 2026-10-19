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

llama.cpp  b11429，Vulkan 构建。以下 `config.ini` 用于 `llama-server` 路由模式提供服务。它不参与下面的基准测试。

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

```bash
llama-bench -m Qwen3.6-35B-A3B-UD-IQ4_XS.gguf -ngl 999 -p 64 -n 32 -r 3 \
            -fa on -ctk q8_0 -ctv q8_0
```

| 模型                    | 大小      | 参数             | pp64 (t/s)                | tg32 (t/s)              |
| ----------------------- | --------- | ---------------- | ------------------------- | ----------------------- |
| Qwen3.6-35B-A3B: IQ4_XS | 16.50 GiB | 34.66 B (3B激活) | **378.91 ± 11.69** | **29.43 ± 0.17** |

- `pp64` = 处理 64 token 提示词的速度，`tg32` = 生成 32 token 的速度
- pp64 和 tg32 分别测量 64 token 的提示词处理速度与 32 token 的生成速度。本次未设置额外的上下文深度，结果不代表长上下文性能，也不代表填满 131,072 token 时的表现。

---

## 上下文与显存

`-ngl 999 -c 131072 -fa on -ctk q8_0 -ctv q8_0`，模型加载完成后读取：

| 上下文  | GPU 占用  |
| ------- | --------- |
| 131,072 | 18,718 MB |

---

[MIT](LICENSE)
