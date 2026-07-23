# Gigatoken

[English](README.md) · **简体中文** · [日本語](README.ja.md)

<div align="center">

速度约为 HuggingFace tokenizers 的 1000 倍，可直接替换使用。

*以 GB/s 的速度对文本数据进行分词！*

![GPT-2 加速效果](https://raw.githubusercontent.com/marcelroed/gigatoken/main/assets/throughput_owt_train_gpt-2.svg)

请注意，HF tokenizers 和 tiktoken 本身都已经在使用多线程 Rust！
</div>

## 什么是 Gigatoken？
Gigatoken 是面向语言建模的最快分词器。
它支持广泛的 CPU 硬件以及几乎所有常用分词器。
有关不同分词器和 CPU 的详细吞吐量数据，请参阅[基准测试](#基准测试)。

## 安装
```bash
pip install gigatoken
```

## 使用方法
Gigatoken 既可以使用自身 API，也可以通过兼容模式与 HuggingFace Tokenizers 或 Tiktoken 配合使用。

### 兼容模式（最简单）
```python
import gigatoken as gt

# 只需对现有 HuggingFace tokenizers 用法做最小改动（兼容模式）
hf_tokenizer = ...
tokenizer = gt.Tokenizer(hf_tokenizer).as_hf()

# tokenizer 可在与 hf_tokenizer 相同的场景中使用
tokens = tokenizer.encode_batch(["This is a test string", "And here is another"])

# 或者与 tiktoken 配合使用
tiktokenizer = ...
tokenizer = gt.Tokenizer(tiktokenizer).as_tiktoken()

# 现在可像现有 tiktoken 分词器一样工作
tokens = tokenizer.encode_batch(["This is a test string", "And here is another"])
```

项目投入了大量工作，以确保此模式下的输出与 HuggingFace Tokenizers 完全一致，但这会带来不可忽略的性能成本。
整体性能仍会快得多，只是达不到使用 Gigatoken API 时约 1000 倍的提升。

### Gigatoken API（最快）
```python
import gigatoken as gt

tokenizer = gt.Tokenizer("Qwen/Qwen3-8B")  # 接受 HF 模型名称
file_source = gt.TextFileSource(["owt_train.txt"], separator=b"<|endoftext|>")
tokens = tokenizer.encode_files(file_source)
```

使用 Gigatoken API 时，Rust 实现可以直接读取数据，在实现最大并行度的同时尽可能省去额外开销。
请注意，通过此 API 传入 Python 数据结构仍会产生从 Python 读取数据的开销。

<!-- benchmarks:start -->
## 基准测试

<details>
<summary><b>owt_train.txt（11.9 GB）编码吞吐量 — AMD EPYC 9565 72 核处理器 × 2 路（144 核）</b></summary>

| 分词器 | gigatoken | HF tokenizers | tiktoken | 相比 HF | 相比 tiktoken |
|---|---:|---:|---:|---:|---:|
| GPT-2 | 24.53 GB/s | 24.8 MB/s | 36.0 MB/s | 989× | 681× |
| Phi-4 | 24.00 GB/s | 29.9 MB/s | — | 801× | — |
| GPT-OSS | 23.96 GB/s | 49.7 MB/s | 42.8 MB/s | 482× | 560× |
| OLMo 2 / 3 | 23.06 GB/s | 27.7 MB/s | — | 833× | — |
| Nemotron 3 | 22.79 GB/s | 49.4 MB/s | — | 462× | — |
| Qwen 3 | 22.16 GB/s | 34.2 MB/s | — | 648× | — |
| Llama 3 / 3.1 / 3.2 | 22.15 GB/s | 48.5 MB/s | — | 457× | — |
| GLM 5 | 20.97 GB/s | 74.8 MB/s | — | 280× | — |
| Llama 3.3 | 20.82 GB/s | 48.3 MB/s | — | 431× | — |
| Llama 4 | 20.77 GB/s | 72.7 MB/s | — | 286× | — |
| GLM 4 | 20.61 GB/s | 72.3 MB/s | — | 285× | — |
| Phi-4-mini | 20.05 GB/s | 27.6 MB/s | — | 726× | — |
| DeepSeek V3 / R1 / V4 | 19.69 GB/s | 26.2 MB/s | — | 750× | — |
| Qwen 2 / 2.5 | 19.12 GB/s | 27.7 MB/s | — | 691× | — |
| Kimi K2 | 18.85 GB/s | — | — | — | — |
| Qwen 3.5 / 3.6 | 15.49 GB/s | 27.7 MB/s | — | 558× | — |
| Gemma 4 | 4.82 GB/s | 334.1 MB/s | — | 14× | — |
| ModernBERT | 4.18 GB/s | 26.9 MB/s | — | 155× | — |
| Mistral 7B v0.3 | 3.57 GB/s | 354.7 MB/s | — | 10× | — |
| TinyLlama / Phi-3 (Llama 2) | 3.48 GB/s | 323.6 MB/s | — | 11× | — |
| CodeLlama | 3.47 GB/s | 347.4 MB/s | — | 10.0× | — |
| Gemma 3 | 3.43 GB/s | 357.2 MB/s | — | 9.6× | — |
| Gemma 1 | 2.51 GB/s | 342.2 MB/s | — | 7.3× | — |

</details>
<details>
<summary><b>owt_train.txt（11.9 GB）编码吞吐量 — Apple M4 Max（16 核）</b></summary>

| 分词器 | gigatoken | HF tokenizers | tiktoken | 相比 HF | 相比 tiktoken |
|---|---:|---:|---:|---:|---:|
| GPT-2 | 8.79 GB/s | 6.9 MB/s | 62.8 MB/s | 1,268× | 140× |
| Nemotron 3 | 7.82 GB/s | 10.9 MB/s | — | 715× | — |
| Phi-4 | 7.76 GB/s | 7.7 MB/s | — | 1,012× | — |
| Llama 3 / 3.1 / 3.2 | 7.60 GB/s | 11.2 MB/s | — | 676× | — |
| OLMo 2 / 3 | 7.56 GB/s | 5.8 MB/s | — | 1,299× | — |
| Llama 3.3 | 7.50 GB/s | 15.7 MB/s | — | 479× | — |
| Phi-4-mini | 6.97 GB/s | 7.2 MB/s | — | 964× | — |
| Kimi K2 | 6.88 GB/s | — | — | — | — |
| Llama 4 | 6.81 GB/s | 11.6 MB/s | — | 590× | — |
| Qwen 2 / 2.5 | 6.37 GB/s | 5.8 MB/s | — | 1,105× | — |
| Qwen 3 | 6.36 GB/s | 6.9 MB/s | — | 918× | — |
| Qwen 3.5 / 3.6 | 6.31 GB/s | 6.3 MB/s | — | 994× | — |
| GPT-OSS | 6.20 GB/s | 20.2 MB/s | 87.2 MB/s | 306× | 71× |
| GLM 4 | 6.17 GB/s | 15.8 MB/s | — | 392× | — |
| DeepSeek V3 / R1 / V4 | 5.68 GB/s | 7.2 MB/s | — | 788× | — |
| GLM 5 | 5.55 GB/s | 12.2 MB/s | — | 456× | — |
| ModernBERT | 2.64 GB/s | 5.8 MB/s | — | 452× | — |
| Mistral 7B v0.3 | 1.99 GB/s | 95.1 MB/s | — | 21× | — |
| Gemma 4 | 1.82 GB/s | 85.2 MB/s | — | 21× | — |
| CodeLlama | 1.73 GB/s | 80.2 MB/s | — | 22× | — |
| TinyLlama / Phi-3 (Llama 2) | 1.69 GB/s | 80.1 MB/s | — | 21× | — |
| Gemma 1 | 1.42 GB/s | 85.7 MB/s | — | 17× | — |
| Gemma 3 | 1.38 GB/s | 82.2 MB/s | — | 17× | — |

</details>
<details>
<summary><b>owt_train.txt（11.9 GB）编码吞吐量 — AMD Ryzen 7 9800X3D 8 核处理器（16 核）</b></summary>

| 分词器 | gigatoken | HF tokenizers | tiktoken | 相比 HF | 相比 tiktoken |
|---|---:|---:|---:|---:|---:|
| GPT-2 | 6.27 GB/s | 59.0 MB/s | 92.1 MB/s | 106× | 68× |
| Phi-4 | 6.09 GB/s | 55.4 MB/s | — | 110× | — |
| OLMo 2 / 3 | 6.06 GB/s | 55.4 MB/s | — | 109× | — |
| Phi-4-mini | 5.80 GB/s | 54.6 MB/s | — | 106× | — |
| GPT-OSS | 5.68 GB/s | 79.6 MB/s | 112.7 MB/s | 71× | 50× |
| Qwen 3 | 5.34 GB/s | 54.4 MB/s | — | 98× | — |
| Qwen 2 / 2.5 | 5.30 GB/s | 51.7 MB/s | — | 103× | — |
| Llama 3.3 | 5.26 GB/s | 79.9 MB/s | — | 66× | — |
| Llama 3 / 3.1 / 3.2 | 5.24 GB/s | 79.5 MB/s | — | 66× | — |
| Kimi K2 | 5.23 GB/s | — | — | — | — |
| Qwen 3.5 / 3.6 | 5.22 GB/s | 51.6 MB/s | — | 101× | — |
| Nemotron 3 | 5.20 GB/s | 79.0 MB/s | — | 66× | — |
| GLM 5 | 5.05 GB/s | 79.5 MB/s | — | 63× | — |
| GLM 4 | 5.04 GB/s | 79.5 MB/s | — | 63× | — |
| Llama 4 | 5.03 GB/s | 78.2 MB/s | — | 64× | — |
| DeepSeek V3 / R1 / V4 | 4.21 GB/s | 51.6 MB/s | — | 82× | — |
| ModernBERT | 2.84 GB/s | 52.1 MB/s | — | 54× | — |
| Mistral 7B v0.3 | 1.47 GB/s | 91.6 MB/s | — | 16× | — |
| Gemma 4 | 1.45 GB/s | 78.8 MB/s | — | 18× | — |
| CodeLlama | 1.38 GB/s | 85.2 MB/s | — | 16× | — |
| TinyLlama / Phi-3 (Llama 2) | 1.37 GB/s | 84.9 MB/s | — | 16× | — |
| Gemma 1 | 1.14 GB/s | 84.9 MB/s | — | 13× | — |
| Gemma 3 | 1.12 GB/s | 83.0 MB/s | — | 13× | — |

</details>
<details>
<summary><b>基准测试详情</b></summary>

选择 OWT（openwebtext）是因为它大致代表了从 CommonCrawl 文档提取后得到的文本。
Gigatoken 会对整个文件进行未预切分编码，因此在查找切分边界和自动并行化方面比其他分词器承担更多工作。
HuggingFace tokenizers（`encode_batch_fast`）取前 100 MB，tiktoken（`encode_ordinary_batch`）取前 1 GB；两者都预先按 `<|endoftext|>` 切分。
这种比较是公平的，因为两个对比分词器都不做缓存，处理过程中的速度大致均匀。
目前仅为 tiktoken 官方支持的分词器填写 tiktoken 数据行。

最慢的几行是基于 SentencePiece 的分词器；Gigatoken 对它们的优化尚不充分。

每一行代表一个独立分词器（词表、合并规则和预分词器相同），并在一个代表性模型仓库上测量。
如果表中没有你的分词器，它很可能基于其中某个现有分词器。
例如：

- **Llama 3 / 3.1 / 3.2** — Llama 3 / 3.1 / 3.2、DeepSeek-R1-Distill-Llama、Hermes 3、Saiga 以及其他 Llama-3 微调模型
- **Llama 3.3** — Llama 3.3、Llama-3.1-Nemotron-Nano-VL、SmolLM3、Kanana 1.5、jina-embeddings-v5、Ultravox
- **Qwen 2 / 2.5** — Qwen 2 和 2.5（包括 Coder 与 VL）、Qwen3-Coder、Qwen3-VL、DeepSeek-R1 Qwen 蒸馏模型、MiMo V2.5、MiniCPM-o 2.6、InternVL3
- **Qwen 3** — Qwen 3（包括 Embedding 与 Reranker）、Qwen2.5-Omni、Qwen3-VL-Embedding、MiMo V2.5 Pro、jina-reranker-m0、pplx-embed、MOSS-TTS、Zeta
- **DeepSeek V3 / R1 / V4** — DeepSeek V3 / V3.1 / V3.2、R1、V4 Flash 与 Pro、DeepSeek-VL2
- **GLM 4** — GLM 4.1V、4.5 和 4.7
- **GLM 5** — GLM 5 / 5.2 和 GLM-4.7-Flash
- **Nemotron 3** — Nemotron 3 Nano、Super 和 Ultra
- **Kimi K2** — Kimi K2 / K2.5 / K2.6 / K2.7、Kimi-Linear、Kimi-VL、Moonlight
- **Phi-4-mini** — Phi-4-mini 和 Phi-4-multimodal
- **TinyLlama / Phi-3 (Llama 2)** — TinyLlama、Phi-3-mini、Phi-3.5-mini 和 Phi-3.5-vision（使用 Llama 2 词表）
- **Gemma 3** — Gemma 3（270M–27B）和 EmbeddingGemma
- **Gemma 4** — Gemma 4（稠密、MoE 和 E 系列）以及 DiffusionGemma

</details>
<!-- benchmarks:end -->


## 常见问题
### 问：你是不是只针对某个特定 CPU 和分词器做了过度优化？为什么这么快？
不是，我对所有这些组合都做了极致优化！
在不同 CPU（现代 x86 和 ARM）以及不同具体分词器上，结果都非常一致。

主要提升来自：使用 SIMD、减少分支等技巧，对通常交由正则表达式引擎完成的实现（预分词）进行深度优化；同时大幅优化预分词映射的缓存（若某个词此前出现过，便高效查找其编码后的 token）。
缓存会迅速增长，而预分词项的分布又具有很长的尾部，因此缓存是该领域非常困难的问题。

减少与 Python 的交互以及避免线程间通信，也带来了一部分提升。


### 问：怎样快速检查我的分词器是否受支持？
无需安装任何东西即可试用！下面的命令会针对给定的 HuggingFace 模型仓库验证分词结果并测量耗时：

```bash
# 下载数据
wget https://huggingface.co/datasets/stanford-cs336/owt-sample/resolve/main/owt_train.txt.gz  # 仅作示例！
gunzip owt_train.txt.gz
```

```bash
uvx --with tokenizers gigatoken bench 'openai-community/gpt2' owt_train.txt \
    --validate --doc-separator "<|endoftext|>"
```
```bash
      cpu: Apple M4 Max, 16 cores
gigatoken:    1.432 s |   11920.51 MB at  8327.05 MB/s |  2701.65 Mtok at 1887.23 Mtok/s
       hf:   16.250 s |     100.00 MB at     6.15 MB/s |    22.76 Mtok at    1.40 Mtok/s
gigatoken is 1353.13x faster than hf
validation OK: 20401 documents match
```

```bash
      cpu: AMD EPYC 9565 72-Core Processor, 144 cores, 2 sockets
gigatoken:    0.486 s |   11920.51 MB at 24532.45 MB/s |  2701.65 Mtok at 5564.94 Mtok/s
       hf:    4.033 s |     100.00 MB at    24.80 MB/s |    22.76 Mtok at    5.63 Mtok/s
gigatoken is 989.21x faster than hf
validation OK: 20401 documents match
```
按照 EPYC CPU 上测得的速度，你可以在不到 6.5 小时内对 [Common Crawl 的全部内容](https://arxiv.org/pdf/2211.04325)（通常被视为整个互联网，共 130 万亿个 token）完成分词！

此示例使用[这个数据集](https://huggingface.co/datasets/stanford-cs336/owt-sample)的训练样本；默认情况下，CLI 会截取文件的前 100 MB，用于验证并与 HF 对比。
可通过 `uvx gigatoken bench --help` 查看这些选项的帮助。
在 macOS 上可能需要运行命令两次才能获得准确读数，因为第一次运行总会执行安全扫描，从而拖慢 Rust 代码。


### 问：我发现结果不一致或某个用例很慢，这是预期行为吗？
很可能不是！虽然已经进行了相当广泛的测试，但我无法覆盖所有用例；请通过 [GitHub Issue](https://github.com/marcelroed/gigatoken/issues) 报告发现的问题，以便我尽快处理。


<!--
## Gigatoken 如何工作？

Gigatoken 源于以下几点观察：
* 当前分词器的大部分时间都花在预分词上 ->

Gigatoken 实现的预分词器吞吐量可达每线程 >2 GB/s

得益于算法和系统层面的改进，Gigatoken 比其他库更快。
Gigatoken 的一个关键优势，是用对同一方法的自定义实现替代几乎所有分词器用于预分词的正则表达式。
这在其他实现中是严重瓶颈，也是本库达到极高速度的重要原因。
此外，Gigatoken 使用并发数据结构，从而在更多环节利用多进程。
\* 本节所有参考速度均在 M4 Pro CPU 上测得
-->


## 引用
如果你在研究中使用 Gigatoken，请按以下格式引用：

```bibtex
@software{roed2026gigatoken,
  author = {Marcel R{\o}d},
  title = {{G}igatoken: SIMD and Cache Hierarchies for 1000x Faster Byte-Pair Encoding Tokenization on Modern CPUs},
  url = {https://github.com/marcelroed/gigatoken},
  year = {2026},
}
```

## 已知问题
* Python 迭代由 Rust 处理，但使用 ABI3；它比使用特定 CPython 版本的内部 API 更慢。未来计划针对各 Python 版本进行专门优化，以降低这部分开销。早期实验显示，在受开销限制的场景中速度可提升 2 倍。
* Gigatoken API 尚未实现文件接收端。
* 尚不支持 WordPiece。
* 基于 SentencePiece 的分词尚未达到常见 BPE 分词器的优化程度。由于 SentencePiece 主要用于 Google 模型或 BERT 类模型，目前优先级较低。
* Windows 上的测试还不充分，因此目前建议优先使用 WSL。

---

<details>
<summary>AI 使用披露</summary>
此代码库的大部分内容均由人工编写，未使用 AI（从项目的 Git 历史中也可以看出）。
在项目最后阶段，AI 用于协助：

* 实现面向用户的 API
* 扩大兼容范围，例如泛化并移植预分词器实现以支持更多分词器，以及补齐填充、截断、Unicode 规范化等次要功能
* 在 AVX512 / AVX2 / NEON 之间移植 SIMD 策略
* 完成最终性能分析，并通过消除分支、改进预分词缓存层次结构取得最后约 4 倍的性能提升
* 重构与代码复用
</details>
