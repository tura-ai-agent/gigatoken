# Gigatoken

[English](README.md) · [简体中文](README.zh-CN.md) · **日本語**

<div align="center">

HuggingFace tokenizers より約 1000 倍高速な、ドロップイン置換です。

*テキストデータを GB/s でトークナイズ！*

![GPT-2 の高速化](https://raw.githubusercontent.com/marcelroed/gigatoken/main/assets/throughput_owt_train_gpt-2.svg)

HF tokenizers と tiktoken は、どちらもすでにマルチスレッド Rust で動作している点にご注意ください。
</div>

## Gigatoken とは？
Gigatoken は、言語モデリング向けの最速トークナイザーです。
幅広い CPU ハードウェアと、一般的に使われるほぼすべてのトークナイザーをサポートします。
トークナイザーと CPU ごとの詳細なスループットは、[ベンチマーク](#ベンチマーク)を参照してください。

## インストール
```bash
pip install gigatoken
```

## 使い方
Gigatoken は独自 API のほか、HuggingFace Tokenizers または Tiktoken との互換モードでも利用できます。

### 互換モード（最も簡単）
```python
import gigatoken as gt

# 既存の HuggingFace tokenizers の使い方から最小限の変更（互換モード）
hf_tokenizer = ...
tokenizer = gt.Tokenizer(hf_tokenizer).as_hf()

# tokenizer は hf_tokenizer と同じ場面で利用可能
tokens = tokenizer.encode_batch(["This is a test string", "And here is another"])

# または tiktoken と組み合わせる
tiktokenizer = ...
tokenizer = gt.Tokenizer(tiktokenizer).as_tiktoken()

# 既存の tiktoken トークナイザーと同様に動作
tokens = tokenizer.encode_batch(["This is a test string", "And here is another"])
```

この設定で HuggingFace Tokenizers とまったく同じ出力になるよう多大な労力が注がれていますが、そのため無視できない性能コストが生じます。
それでも全般的に大幅な高速化を期待できますが、Gigatoken API で得られる約 1000 倍には届きません。

### Gigatoken API（最速）
```python
import gigatoken as gt

tokenizer = gt.Tokenizer("Qwen/Qwen3-8B")  # HF モデル名を指定可能
file_source = gt.TextFileSource(["owt_train.txt"], separator=b"<|endoftext|>")
tokens = tokenizer.encode_files(file_source)
```

Gigatoken API を使うと、Rust 実装がデータを直接読み取り、最大限に並列化しながら可能な限りオーバーヘッドを省けます。
ただし、この API に Python のデータ構造を渡す場合は、Python から読み取るオーバーヘッドが引き続き発生します。

<!-- benchmarks:start -->
## ベンチマーク

<details>
<summary><b>owt_train.txt（11.9 GB）のエンコードスループット — AMD EPYC 9565 72 コアプロセッサ × 2 ソケット（144 コア）</b></summary>

| トークナイザー | gigatoken | HF tokenizers | tiktoken | 対 HF | 対 tiktoken |
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
<summary><b>owt_train.txt（11.9 GB）のエンコードスループット — Apple M4 Max（16 コア）</b></summary>

| トークナイザー | gigatoken | HF tokenizers | tiktoken | 対 HF | 対 tiktoken |
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
<summary><b>owt_train.txt（11.9 GB）のエンコードスループット — AMD Ryzen 7 9800X3D 8 コアプロセッサ（16 コア）</b></summary>

| トークナイザー | gigatoken | HF tokenizers | tiktoken | 対 HF | 対 tiktoken |
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
<summary><b>ベンチマークの詳細</b></summary>

OWT（openwebtext）は、CommonCrawl 文書から抽出した後のテキストをおおむね代表しているため採用しました。
Gigatoken はファイル全体を事前分割せずにエンコードするため、分割境界の検出と自動並列化において他のトークナイザーより多くの処理を行います。
HuggingFace tokenizers（`encode_batch_fast`）は先頭 100 MB、tiktoken（`encode_ordinary_batch`）は先頭 1 GB を使用し、どちらも `<|endoftext|>` で事前分割します。
比較対象のどちらもキャッシュを行わず、処理中の速度がおおむね一定であるため、公平な比較です。
現在、tiktoken の列は公式にサポートされるトークナイザーについてのみ記載しています。

最も遅い行は SentencePiece ベースのトークナイザーで、Gigatoken ではまだ十分に最適化されていません。

各行は、語彙・マージ・プレトークナイザーが同一の独立したトークナイザーを表し、代表的なモデルリポジトリで測定しています。
利用中のトークナイザーが表にない場合、既存のいずれかを基にしている可能性があります。
例：

- **Llama 3 / 3.1 / 3.2** — Llama 3 / 3.1 / 3.2、DeepSeek-R1-Distill-Llama、Hermes 3、Saiga、およびその他の Llama-3 ファインチューニングモデル
- **Llama 3.3** — Llama 3.3、Llama-3.1-Nemotron-Nano-VL、SmolLM3、Kanana 1.5、jina-embeddings-v5、Ultravox
- **Qwen 2 / 2.5** — Qwen 2 と 2.5（Coder と VL を含む）、Qwen3-Coder、Qwen3-VL、DeepSeek-R1 の Qwen 蒸留モデル、MiMo V2.5、MiniCPM-o 2.6、InternVL3
- **Qwen 3** — Qwen 3（Embedding と Reranker を含む）、Qwen2.5-Omni、Qwen3-VL-Embedding、MiMo V2.5 Pro、jina-reranker-m0、pplx-embed、MOSS-TTS、Zeta
- **DeepSeek V3 / R1 / V4** — DeepSeek V3 / V3.1 / V3.2、R1、V4 Flash と Pro、DeepSeek-VL2
- **GLM 4** — GLM 4.1V、4.5、4.7
- **GLM 5** — GLM 5 / 5.2 と GLM-4.7-Flash
- **Nemotron 3** — Nemotron 3 Nano、Super、Ultra
- **Kimi K2** — Kimi K2 / K2.5 / K2.6 / K2.7、Kimi-Linear、Kimi-VL、Moonlight
- **Phi-4-mini** — Phi-4-mini と Phi-4-multimodal
- **TinyLlama / Phi-3 (Llama 2)** — TinyLlama、Phi-3-mini、Phi-3.5-mini、Phi-3.5-vision（Llama 2 の語彙）
- **Gemma 3** — Gemma 3（270M–27B）と EmbeddingGemma
- **Gemma 4** — Gemma 4（Dense、MoE、E シリーズ）と DiffusionGemma

</details>
<!-- benchmarks:end -->


## よくある質問
### Q：特定の CPU とトークナイザーだけに過剰最適化したのですか？ なぜこれほど高速なのですか？
いいえ、これらすべての組み合わせに徹底的な最適化を施しました！
結果は CPU（最新の x86 と ARM）間でも、個々のトークナイザー間でも非常に一貫しています。

主な改善点は、通常は正規表現エンジンに委ねられる実装（プレトークナイズ）を SIMD、分岐の削減などで徹底的に最適化したことと、プレトークン対応表のキャッシュ（以前に現れた単語のエンコード済みトークンを効率よく検索）の大幅な最適化です。
キャッシュが急速に増大し、プレトークンの分布が非常にロングテールであるため、この領域のキャッシュは非常に難しい問題です。

Python とのやり取りを減らし、スレッド間通信を避けることでも性能が向上しています。


### Q：自分のトークナイザーが対応しているか、すぐ確認するには？
何もインストールせずに試せます。次のコマンドは、指定した HuggingFace モデルリポジトリのトークナイズを検証し、所要時間を測定します：

```bash
# データをダウンロード
wget https://huggingface.co/datasets/stanford-cs336/owt-sample/resolve/main/owt_train.txt.gz  # 一例です
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
EPYC CPU で測定した速度なら、[Common Crawl 全体](https://arxiv.org/pdf/2211.04325)（しばしばインターネット全体と見なされる 130 兆トークン）を 6.5 時間弱でトークナイズできます。

この例では[このデータセット](https://huggingface.co/datasets/stanford-cs336/owt-sample)の train サンプルを使用します。CLI はデフォルトでファイルの先頭 100 MB を抽出し、検証と HF との比較に使います。
各オプションのヘルプは `uvx gigatoken bench --help` で確認できます。
macOS では正確な測定値を得るためにコマンドを 2 回実行する必要がある場合があります。初回実行時には必ずセキュリティスキャンが行われ、Rust コードが遅くなるためです。


### Q：不一致や遅いユースケースを見つけました。これは想定内ですか？
おそらく違います。かなり幅広くテストしていますが、すべてのユースケースは手元にありません。見つけた問題は [GitHub Issue](https://github.com/marcelroed/gigatoken/issues) で報告してください。できるだけ早く対応します。


<!--
## Gigatoken の仕組み

Gigatoken は、いくつかの観察から生まれました：
* 現在のトークナイザーでは、処理時間の大半がプレトークナイズに費やされる ->

Gigatoken のプレトークナイザーは、1 スレッドあたり >2 GB/s で動作します

アルゴリズムとシステムの変更により、Gigatoken は他のライブラリより高速です。
主な利点の一つは、ほぼすべてのトークナイザーがプレトークナイズに使う正規表現を、まったく同じ処理を行う独自実装に置き換えたことです。
これは他の実装における深刻なボトルネックであり、本ライブラリの非常に高い速度を支える大きな要因です。
さらに Gigatoken は並行データ構造を使い、より多くの箇所でマルチプロセスを活用します。
\* この節の参考速度はすべて M4 Pro CPU で測定
-->


## 引用
研究で Gigatoken を使用する場合は、次の形式で引用してください：

```bibtex
@software{roed2026gigatoken,
  author = {Marcel R{\o}d},
  title = {{G}igatoken: SIMD and Cache Hierarchies for 1000x Faster Byte-Pair Encoding Tokenization on Modern CPUs},
  url = {https://github.com/marcelroed/gigatoken},
  year = {2026},
}
```

## 既知の問題
* Python の反復処理は Rust で行いますが ABI3 を使うため、バージョン固有の CPython 内部 API より低速です。将来は Python の各バージョンに特化してこのオーバーヘッドを削減する予定です。初期実験では、オーバーヘッドが支配的なケースで 2 倍高速になることが確認されています。
* Gigatoken API には、まだファイルシンクが実装されていません。
* WordPiece はまだサポートされていません。
* SentencePiece ベースのトークナイズは、一般的な BPE トークナイザーほど最適化されていません。SentencePiece を使うのは主に Google のモデルや BERT 系モデルであるため、現在の優先度は低くなっています。
* Windows では十分にテストされていないため、現時点では WSL の利用を推奨します。

---

<details>
<summary>AI 利用に関する開示</summary>
このコードベースの大部分は AI を使わずに手作業で作成されています（プロジェクトの Git 履歴からも確認できます）。
プロジェクトの最終段階では、AI を次の作業の補助に使用しました：

* ユーザー向け API の実装
* 互換性の拡大。たとえば、より多くのトークナイザーに対応するためのプレトークナイザー実装の一般化と移植、パディング・切り詰め・Unicode 正規化など優先度の低い機能
* AVX512 / AVX2 / NEON 間での SIMD 戦略の移植
* 最終プロファイリング、および分岐の排除とプレトークンキャッシュ階層の改善による最後の約 4 倍の性能向上
* リファクタリングとコード再利用
</details>
