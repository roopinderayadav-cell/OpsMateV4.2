# OpsMate - AI without any key (local AI + built-in meaning search)

Two separate switches. Use one, both, or neither.

| | Local AI (`AI_PROVIDER = "local"`) | Built-in meaning search (`MEANING_SEARCH = "builtin"`) |
|---|---|---|
| What it does | writes the answer from the SOP passages (like Gemini/Claude) | finds passages by meaning, not only exact words |
| Key needed | no | no |
| Data leaves the machine | no | no (the model is downloaded once, then runs inside OpsMate) |
| Streamlit Community Cloud | **no** (needs 8 GB+ memory) | yes |
| Laptop / company server | yes | yes |

---------------------------------------------------------------------------------------------

## A. Built-in meaning search (easiest, works on Streamlit Cloud too)

1. `requirements.txt` already contains `fastembed==0.9.0` (official package of Qdrant,
   Apache-2.0 licence, from pypi.org).
2. Add one line to Secrets (each entry on ONE line):
   ```
   MEANING_SEARCH = "builtin"
   ```
3. Reboot the app. On the first question OpsMate downloads the model **BAAI/bge-small-en-v1.5**
   (about 70 MB, MIT licence) from Qdrant's official page on huggingface.co
   (fallback: Qdrant's own Google Cloud storage). It is saved in `data/models/`.
4. Admin → Settings → **Build semantic index now** (once; again after a reboot on Cloud).

If the download is blocked (company firewall), OpsMate just keeps using word search and shows a
note in Settings - nothing breaks. Ask IT to allow `huggingface.co` once, or copy the
`data/models/` folder from a machine that has it.

---------------------------------------------------------------------------------------------

## B. Local AI with Ollama (laptop or company server)

### 1. Install Ollama - ONLY from the official site
* Go to **https://ollama.com/download** (type the address yourself; do not use links from emails,
  ads or "mirror" sites). Windows: `OllamaSetup.exe`. Linux server: the install command shown on
  that page. Source code: https://github.com/ollama/ollama (MIT licence).
* Company laptop: ask IT before installing - they may want to install it for you.
* After install Ollama runs in the background and listens only on this computer
  (`http://localhost:11434`).

### 2. Download the models (from Ollama's official library, https://ollama.com/library)
Open Command Prompt / terminal:
```
ollama pull qwen2.5:7b          # answers   (~4.7 GB, Apache-2.0 licence, needs ~8 GB RAM)
ollama pull nomic-embed-text    # meaning search (~270 MB, Apache-2.0)
```
Smaller laptop (8 GB RAM or less):  `ollama pull llama3.2:3b`  (~2 GB, Llama 3.2 Community licence)
then set `LOCAL_MODEL = "llama3.2:3b"`.
Optional - read screenshots: `ollama pull gemma3:4b` (~3.3 GB, Gemma licence) and set
`LOCAL_VISION_MODEL = "gemma3:4b"`.

**Licence check:** before using a model at work, ask your InfoSec / legal team to approve the
licence (Apache-2.0 / MIT are the simplest). The licence is shown on each model's page at
ollama.com/library.

Quick test: `ollama run qwen2.5:7b "say ready"` - it should answer.

### 3. Tell OpsMate to use it
Laptop (Command Prompt, once, then close and reopen):
```
setx AI_PROVIDER "local"
```
or in `.streamlit/secrets.toml` / Secrets:
```
AI_PROVIDER = "local"
LOCAL_MODEL = "qwen2.5:7b"
LOCAL_EMBED_MODEL = "nomic-embed-text"
```
Ollama on another company server:  `OLLAMA_URL = "http://<server-name>:11434"`
(by default Ollama only listens on its own machine; the server admin must open it to the
OpsMate server only, never to the internet).

### 4. Start and check
```
python -m streamlit run app.py
```
Admin → Settings → **Test AI connection** → should show `Local AI (qwen2.5:7b) answered: ready
· meaning search ✅`. Then **Build semantic index now** (once - the old Gemini index can't be
mixed with the local one, OpsMate clears it automatically when the model changes).

---------------------------------------------------------------------------------------------

## What happens when something is wrong

| You see | Reason | Fix |
|---|---|---|
| "Local AI is not running - answering from documents." | Ollama app closed / PC restarted | start Ollama (Windows: Start menu → Ollama). OpsMate checks again after 1 minute |
| "Local model 'x' is not installed - run: ollama pull x" | model name not downloaded | run the command shown |
| "Local AI took too long" | model too big for this PC | use `llama3.2:3b`, or a server with a GPU |
| Screenshots not read | no vision model | set `LOCAL_VISION_MODEL` |
| Voice button missing | voice needs Gemini | type the question |
| "Built-in meaning search could not start" | model download blocked | see section A, step 3 |

In every case OpsMate still answers from the documents (word search), exactly like AI-off mode.

## Safety notes
* No new Python library is needed for local AI - OpsMate talks to Ollama with Python's own
  built-in `urllib`, only to the address in `OLLAMA_URL`, never through a proxy for localhost.
* Only download Ollama from ollama.com and models with `ollama pull` (official library).
  Never run `.exe` files or "model packs" from other websites, shared drives or chats.
* Answer quality of a small local model is lower than Gemini/Claude. Test with your own SOPs
  (20-30 real questions) before using it on the floor. The document-answer safety rules
  (supervisor reply, conflict check, confidence %) work the same.

## What was tested
* Tested: the OpsMate code that talks to Ollama (with a fake Ollama server on the same
  machine), Ollama closed / model missing messages, switching meaning-search models, built-in
  search with a stand-in model, a clean install of requirements.txt (Python 3.13), and all
  earlier answer checks (21 automated tests).
* NOT tested by us: a real model download and real answer quality - do this on your laptop or
  server (step 4 above).
