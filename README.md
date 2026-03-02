# O-RAG — Offline Retrieval-Augmented Generation for Android

<p align="center">
  <b>A fully offline AI assistant for Android — no internet, no cloud, no data leaks.</b><br/>
  Chat freely or ask questions about your own PDF/TXT documents, entirely on-device.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Android-brightgreen" />
  <img src="https://img.shields.io/badge/Python-3.11-blue" />
  <img src="https://img.shields.io/badge/Model-Qwen_2.5_1.5B-orange" />
  <img src="https://img.shields.io/badge/Retrieval-Hybrid_BM25%2BTF--IDF%2BSemantic-purple" />
  <img src="https://img.shields.io/badge/License-MIT-lightgrey" />
</p>

---

## ✨ What It Does

| Feature | Description |
|---------|-------------|
| 💬 **Offline Chat** | Talk to a local AI (Qwen 2.5 1.5B) — no internet needed |
| 📄 **Document Q&A** | Upload a PDF or TXT, ask anything about it |
| 🔒 **100% Private** | All processing on-device — no data sent anywhere |
| ⚡ **Hybrid Retrieval** | BM25 + TF-IDF + Semantic embeddings for best accuracy |
| 📱 **Android Native** | Runs as a foreground service — stays alive in background |

---

## 🧠 How It Works

```
 You upload a PDF
        │
        ▼
 Chunker splits it into ~80-word overlapping windows
        │
        ▼
 SQLite stores chunks + BM25/TF-IDF vectors
        │
        ▼
 Nomic Embed computes semantic vectors (background thread)
        │
 You ask a question
        │
        ▼
 Hybrid Retriever: BM25 (30%) + TF-IDF (20%) + Semantic (50%)
        │
        ▼
 Top-2 chunks + your question → Qwen 2.5 1.5B
        │
        ▼
 Grounded, streamed answer — strictly from your document
```

---

## 🏗️ Project Structure

```
ORAG/
├── main.py                    # App entry point (Kivy + Android service)
├── cli.py                     # Desktop CLI for testing (no UI needed)
├── requirements.txt           # Python dependencies
├── buildozer.spec             # Android build configuration
│
├── rag/
│   ├── chunker.py             # Text extraction (PDF/TXT) + chunking + TF-IDF
│   ├── db.py                  # SQLite CRUD (documents + chunks)
│   ├── retriever.py           # Hybrid BM25 + TF-IDF + Semantic retrieval
│   ├── llm.py                 # LLM backend (llama-cpp-python / Ollama / llama-server)
│   ├── pipeline.py            # RAG orchestrator + chat history compression
│   └── downloader.py          # HuggingFace model download + APK asset extraction
│
├── ui/
│   └── screens/
│       └── chat_screen.py     # Complete Kivy UI (chat bubbles, file picker, streaming)
│
├── service/
│   └── main.py                # Android foreground service (keeps llama-server alive)
│
├── compressed/
│   ├── compressed_qwen.py     # GGUF quantization utility for Qwen
│   └── compressed_nomic.py    # GGUF quantization utility for Nomic
│
├── evaluation/
│   ├── rag_industry_eval.ipynb         # Industry-grade RAG evaluation (8 queries, 10 metrics)
│   ├── embedding_comparison.ipynb      # BM25 vs TF-IDF vs Semantic comparison
│   ├── benchmark_hf_quantized_run.ipynb # Model speed & quality benchmark
│   └── employee_handbook.txt           # Benchmark document
│
└── .github/workflows/
    ├── build_apk.yml           # CI: full APK (~1.3 GB, models bundled)
    └── build_apk_slim.yml      # CI: slim APK (~120 MB, models downloaded on launch)
```

---

## 🤖 AI Models

| Role | Model | Size | Source |
|------|-------|------|--------|
| **Generation** | Qwen 2.5 1.5B Instruct Q4_K_M | ~1.1 GB | [`Qwen/Qwen2.5-1.5B-Instruct-GGUF`](https://huggingface.co/Qwen/Qwen2.5-1.5B-Instruct-GGUF) |
| **Embedding** | Nomic Embed Text v1.5 Q4_K_M | ~80 MB | [`nomic-ai/nomic-embed-text-v1.5-GGUF`](https://huggingface.co/nomic-ai/nomic-embed-text-v1.5-GGUF) |

Models are auto-downloaded from HuggingFace on first launch (slim APK) or bundled inside the APK (full build).

---

## 📊 Evaluation Results

Tested on 8 benchmark queries over `employee_handbook.txt`:

| Metric | Score |
|--------|-------|
| **Context Recall** | 1.000 |
| **Hit Rate** | 1.000 |
| **MRR (Mean Reciprocal Rank)** | 1.000 |
| **NDCG@5** | 1.000 |
| **Avg. Retrieval Latency** | ~21 ms |
| **Generation Speed** | ~14.8 tokens/s (CPU) |

> Run `evaluation/rag_industry_eval.ipynb` to reproduce these results.

---

## 🖥️ Desktop Setup & Testing (Windows)

### 1. Create Virtual Environment

```powershell
cd C:\path\to\ORAG
python -m venv venv
.\venv\Scripts\activate
```

### 2. Install Dependencies

```powershell
pip install pymupdf huggingface-hub kivy tqdm requests certifi pandas seaborn plotly jupyter
pip install "https://github.com/abetlen/llama-cpp-python/releases/download/v0.3.2/llama_cpp_python-0.3.2-cp312-cp312-win_amd64.whl"
```

### 3. Download llama-server Binary (Windows CPU)

Download [`llama-b8185-bin-win-cpu-x64.zip`](https://github.com/ggml-org/llama.cpp/releases/tag/b8185) and extract all files into `llamacpp_bin/`.

### 4. Download Models

Models are auto-downloaded to `~/models/` on first run, or manually:
```python
from huggingface_hub import hf_hub_download
hf_hub_download("Qwen/Qwen2.5-1.5B-Instruct-GGUF", "qwen2.5-1.5b-instruct-q4_k_m.gguf", local_dir="~/models")
hf_hub_download("nomic-ai/nomic-embed-text-v1.5-GGUF", "nomic-embed-text-v1.5.Q4_K_M.gguf", local_dir="~/models")
```

### 5. Test via CLI

```powershell
python cli.py
```

### 6. Test via Web UI

```powershell
cd llamacpp_bin
.\llama-server.exe --model %USERPROFILE%\models\qwen2.5-1.5b-instruct-q4_k_m.gguf --port 8082 --ctx-size 2048
```
Then open → **http://127.0.0.1:8082**

### 7. Run Evaluation Notebooks

```powershell
.\venv\Scripts\jupyter notebook evaluation/
```

---

## 📱 Building the Android APK

### Prerequisites

- Ubuntu 22.04 (or GitHub Actions — recommended)
- Java 17, Android SDK 34, Android NDK 25b
- Python 3.11

### Build (GitHub Actions — recommended)

Push to GitHub and the workflow auto-builds. Two flavors:

| Workflow | APK Size | Models |
|----------|----------|--------|
| `build_apk.yml` | ~1.3 GB | Bundled — no internet needed at all |
| `build_apk_slim.yml` | ~120 MB | Downloaded on first launch |

The APK appears as a GitHub Actions artifact after each build.

### Build Locally

```bash
pip install buildozer
buildozer android debug
```

---

## 🔬 Retrieval — Technical Detail

The hybrid retriever fuses three signals:

```
Score = 0.30 × BM25_norm  +  0.20 × TF-IDF_norm  +  0.50 × Semantic_norm
```

| Method | Description |
|--------|-------------|
| **BM25** | Okapi BM25, k₁=1.5, b=0.75 — keyword relevance |
| **TF-IDF Cosine** | Sparse cosine similarity with smoothed IDF |
| **Semantic** | 768-dim cosine (Nomic Embed Text v1.5) |

Falls back to BM25 + TF-IDF (50/50) if the embedding model isn't loaded.

---

## ⚙️ LLM Backend Priority

The app tries three backends in order:

1. **`llama-cpp-python`** — in-process, fastest (requires C++ build or pre-built wheel)
2. **`Ollama`** — if `localhost:11434` responds
3. **`llama-server`** — bundled executable, works everywhere (default on Android)

---

## 🔧 Chunking Parameters

| Parameter | Value | Reason |
|-----------|-------|--------|
| `CHUNK_SIZE` | 80 words | Fits Nomic's 128-token context window |
| `CHUNK_OVERLAP` | 15 words | Preserves sentences across chunk boundaries |
| Top-K retrieved | 2 | ~200 tokens of context — fits in 768-token budget |
| Max reply tokens | 320 | ~240 words per answer |

---

## 📦 Dependencies

```
kivy>=2.3.0
pymupdf>=1.24.0
llama-cpp-python>=0.2.57
huggingface-hub>=0.23.0
```

> `llama-cpp-python` will be compiled from source unless a pre-built wheel is used (see setup instructions above).

---

## ⚠️ Known Limitations

- Scanned/image PDFs are not supported (no OCR)
- Conversation history resets on app restart
- Only the last ~30 chunks receive semantic embeddings for large documents
- Cold start takes 10–20s while the model loads into memory
- ARM64 only (no x86 Android support)

---

## 🔮 Future Work

- Multi-document cross-retrieval with source attribution
- Cross-encoder re-ranking for improved precision
- OCR integration (Tesseract) for scanned PDFs
- Persistent conversation sessions
- NPU/NNAPI acceleration for faster inference

---

## 👩‍💻 Author

**Yerramsetty Sai Venkata Suchita**  
GitHub: [github.com/yerramsettysuchita](https://github.com/yerramsettysuchita)

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

## 📚 References

1. Lewis et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*. NeurIPS.
2. Robertson & Zaragoza (2009). *The Probabilistic Relevance Framework: BM25 and Beyond*.
3. Qwen Team (2024). *Qwen2.5 Technical Report*. Alibaba Group.
4. Nussbaum et al. (2024). *Nomic Embed: Training a Reproducible Long Context Text Embedder*. arXiv:2402.01613.
5. Es et al. (2023). *RAGAS: Automated Evaluation of Retrieval Augmented Generation*. arXiv:2309.15217.
