# iKnow - Answers from our SOPs and experts (NORAM · EUROPE · APAC)

(Formerly OpsMate. Only the name on the screens changed - the database, saved answers and
Secrets are the same.)

Agents ask questions in plain words (typos and Hindi/Kannada/Telugu in English letters
are fine) or paste a screenshot, and get one short answer with its SOP source and a
confidence level. When iKnow is not sure it says:
"I do not have answer to this question, please consult with your team supervisor."

## What's new in v4.3 - new name iKnow, backup AI, smarter without AI
- New name **iKnow** on every screen, the login page, chat messages, exports and templates.
- **Backup AI:** Gemini main model -> Gemini second model -> Claude (if key) -> Ollama (if running)
  -> documents. A failed AI is skipped for a few minutes; a note says when the backup answered.
  Settings shows the order and tests every AI separately. New Secret: `AI_BACKUP` (see
  LOCAL_AI_SETUP.md, section C).
- **Smarter without AI:** exact-line look-up in long lists ("BLR airport", "code for Heathrow",
  "LHR", "capital of Denmark", "airlines of Sweden", "card code for Amex"); "what is X" shows the
  line that defines X; an FAQ is ranked first only when it covers every key word of the question;
  "one way", "amex", "visa", "vs" understood; synonyms are never "spell-corrected".
- Gemini requests time out after 60 seconds, so a hanging request never blocks an agent.

## What's new in v3 (upgraded from Compass)
- New look based on the iKnow mock-up: home screen, topic buttons, Recent Questions,
  Important Updates, answer card with confidence, sources with View.
- More accurate: spelling + Hinglish fix, coverage check, re-ranking, optional Gemini
  semantic search. Tested: 26/26 real questions right, 10/10 out-of-scope questions refused.
- Never guesses: low confidence -> the supervisor message (config.FALLBACK_TEXT).
- Conflict check: if two documents give different answers -> shows both + CONFLICT_TEXT.
- Learns by itself: 👍 from 2 people (or 1 admin) saves the answer; 👍/👎 move passages
  up/down for similar questions; 👎 switches off a wrong learned answer; new uploads
  re-check open gaps.
- Knowledge Gaps page: unanswered / conflicting / 👎 questions grouped, with status,
  owner, note, "write the answer now", Excel export.
- Dashboard: KPIs, by region, top topics, trend, most asked, unanswered, feedback,
  Export to Excel (5 sheets).
- Document Management (archive/activate), Announcements, Quick Guides, Settings.
- AI: Gemini (free tier available) or Claude - optional. Voice questions with Gemini.

## Files
| File | What it does |
|---|---|
| `app.py` | Staff screens + menu |
| `admin_pages.py` | Dashboard, Document Management, Review Queue, Knowledge Gaps, Learned Answers, Settings |
| `ui.py` | Colours, header bar, cards |
| `knowledge.py` | Documents, search, 3-layer answering, self-learning, gaps, Excel export |
| `brain.py` | Spelling/Hinglish, topics, confidence, conflict check, short answers |
| `llm.py` | Connection to Gemini, Claude or local AI (Ollama) + built-in meaning search (all optional) |
| `LOCAL_AI_SETUP.md` | How to run AI with no key (Ollama + built-in meaning search) |
| `prompts.py` | Instructions given to the AI - edit to change answer style |
| `config.py` | All settings: name, standard replies, regions, topics, thresholds, AI keys |
| `.streamlit/config.toml` | Colours (blue theme) |
| `sample_docs/` | Demo documents loaded automatically |
| `demo_files/` | A document that deliberately conflicts with the FAQ - upload it to EUROPE to demo the conflict warning |

## Run on a laptop
Double-click `START_APP.bat` (installs libraries the first time), or:
`python -m pip install -r requirements.txt` then `python -m streamlit run app.py`

## Turn on AI (optional)
Streamlit Cloud → App → Settings → Secrets:
```
ADMIN_PASSWORD = "your-admin-password"
GEMINI_API_KEY = "your-gemini-key"
```
Laptop: `setx GEMINI_API_KEY "your-gemini-key"` once, then restart.
Then Settings → "Test AI connection" and "Build semantic index now".
Gemini free tier: Google may use what you send to improve its products - use sample /
non-confidential SOPs on the free tier.

## SME Desk (v4.2) - an expert answers what iKnow could not
* Goes to the **SME Desk**: no answer found, documents disagree, confidence **below 70 %**
  (`SME_REVIEW_BELOW_PCT` in config.py), or an agent marked 👎. The same question asked many
  times shows once ("Asked 5×").
* The SME signs in as **SME** (password `SME_PASSWORD` in Secrets, one line; until it is set SMEs
  use the admin password) and for each question: **Save SME answer** · **iKnow's answer is
  right** · **SOP update needed** (goes to Admin → Knowledge Gaps) · **Not a process question**.
* The answer is saved as a verified answer together with the agents' own wording. iKnow
  searches it first next time (also for similar wording) and shows it as "SME / process team
  answer". Uploaded SOPs and documents are never changed.
* Tab "Answers given by SMEs": search, edit when the process changes, switch off.
* Agents see "sent to an SME" under a no-answer or a below-70 % answer.

## Live chat with an SME (v4.2)
* Agents: menu **iKnow → Chat with SME**, or the button **"Chat live with an SME"** under a
  no-answer / below-70 % answer (the question is filled in). Shows how many SMEs are online.
* SMEs: **SME Desk → Live chat** - switch "I'm available" on, new chats pop up by themselves
  (checked every few seconds), **Pick up** (only one SME gets each chat), reply, and look things up
  in the SOPs on the same screen. SMEs also see "agents waiting" on their home screen.
* When the chat ends the SME ticks the question/answer pairs worth keeping and corrects the
  wording → saved as verified answers that iKnow uses first next time. Greetings / "ok thanks"
  are left out automatically.
* Every chat is kept in full: **SME Desk → Chat history** (search, export to Excel). If nobody
  answered, the question goes to the SME Desk queue so it is not lost.
* Works on Streamlit Cloud and company servers with no extra service (pages refresh every 3-4 s).
  Chats are in the database, so include them in the Settings backup on Streamlit Cloud.

## Card numbers are never stored or shown (v4.2, PCI)
* Typed text (questions, 👎 comments, live chat, SME answers): card numbers become
  `************1234` and CVV / security codes become `***` **before** anything is saved, shown to
  an SME or sent to an AI. Airline ticket numbers (13 digits), phone numbers and PNRs stay readable.
* Screenshots (Ask page and live chat): read on this server with **Tesseract OCR** and every card
  number is covered with a grey box. The agent sees this masked preview before sending. The original
  picture is never written to disk. If Tesseract is missing, screenshots are **refused**.
* SOP uploads: a file with a **real** card number is refused (nothing kept, not even the file) and the
  message says where it is. Public test numbers such as 4111 1111 1111 1111 are allowed and shown
  masked (`FPCCVI************1111/1229`).
* Data saved before this version is masked once automatically at start-up.
* Install Tesseract:
  - Streamlit Cloud: `packages.txt` (contains `tesseract-ocr`) must be in the GitHub repo - done
    automatically on reboot.
  - Linux server: `sudo apt install tesseract-ocr`
  - Windows laptop: installer linked from the official Tesseract page
    https://tesseract-ocr.github.io/tessdoc/Installation.html (UB Mannheim build), then in Secrets
    `TESSERACT_CMD = "C:\\Program Files\\Tesseract-OCR\\tesseract.exe"` (one line).
  - Check: Admin → Settings shows "Card numbers are hidden in text and screenshots".
* Limits: very blurry, rotated or handwritten numbers and photos of a physical card may not be read -
  agents should still crop card details out. Settings → backup files contain only masked data.

## AI with NO key (v4.2) - local AI and built-in meaning search
Full steps: **LOCAL_AI_SETUP.md**. Short version:

| Setting (Secrets or environment) | What it does |
|---|---|
| `AI_PROVIDER = "local"` | answers written by a free AI model on the same laptop/server (Ollama). No key, nothing leaves the network |
| `MEANING_SEARCH = "builtin"` | meaning search with a small model inside iKnow (fastembed). No key, no quota, also works on Streamlit Cloud |
| `LOCAL_MODEL`, `LOCAL_EMBED_MODEL`, `LOCAL_VISION_MODEL`, `OLLAMA_URL` | optional, see LOCAL_AI_SETUP.md |

Local AI does **not** run on Streamlit Community Cloud (too little memory) - run iKnow and
Ollama together on a laptop or company server. If the local AI is closed, iKnow answers from
the documents as before.

## What's new in v4
- Menu bar across the top of every page (iKnow / Admin) and an always-visible **Sign out** button and
  region selector - nothing depends on the side panel any more.
- **LOB tabs** above the chat: NORAM · EUROPE · APAC · QBAY, each with its sub-LOB chips
  (e.g. NORAM: OBT Cancellation, Schedule Change, Ticketing, Refund, Profile Management, Others).
  Questions search the selected LOB (plus GLOBAL documents); the chosen sub-LOB's documents rank first.
  If the LOB has no answer, iKnow searches the other LOBs and says clearly which LOB the answer came from.
  Admins edit sub-LOBs in Settings and tag documents on upload or with ✏️ in Document Management.
- **Confidence %** on every answer (High 80-98 %, Medium 60-79 %), based on match strength, how much of the
  question is covered, and how many documents agree. Hover the ⓘ next to it for the explanation.
- How-to answers come as numbered steps in the order of the SOP.
- **Backup & restore** in Settings (free hosting can erase uploads on restart).
- Warning while the demo admin password is still in use - set ADMIN_PASSWORD in Secrets.

## What's new in v4.1 - pictures, GDS screens and "which one?" questions
- **Pictures in answers**: pictures inside uploaded Word / PDF / PowerPoint files (e.g. a sample Amadeus screen)
  are kept and shown under the answer that comes from the same section. Works without the AI key.
  Documents uploaded before v4.1 get their pictures automatically the first time the app starts.
  (Finding answers *inside* a picture still needs `GEMINI_API_KEY`.)
- **GDS entries look like a terminal**: commands in answers (RTABC123, FXP, TTP/RT, WETR*1, *T …) are shown
  green-on-dark, with a box listing every entry used in the answer.
- **iKnow asks before guessing**: if the steps differ by GDS (Amadeus / Sabre / Galileo …) and the agent did not
  say which, or the best answers come from different sub-LOBs and no sub-LOB is selected, iKnow asks with
  buttons, then answers. "Search all" answers without choosing. Settings: `ASK_CLARIFY`, `CLARIFY_GROUPS` in config.py.
- **Answers even when the AI is out of quota**: if the AI fails, is out of quota or replies "not found" while the
  documents clearly answer the question (strict word-match rule), iKnow shows the document answer with a note.
  After a quota error the AI is skipped for 5 minutes (`AI_PAUSE_SECONDS`), so answers come at once.
- **Home screen**: "Try asking" now follows the selected LOB tab and sub-LOB chip (most-asked questions, then
  questions and section titles from that LOB's documents). The 10 topic buttons were removed (they looked like
  filters but did not change the search). The welcome banner is half the height.
- **Follow-up questions**: "how about Sabre?" after "How to reissue a ticket in Amadeus?" is understood as
  "How to reissue a ticket in Sabre?" (shown under the answer as "follow-up, understood as …").
