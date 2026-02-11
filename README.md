### duolingo_vocab_scraper

Scrape your Duolingo vocabulary list from the Words / Practice Hub page into a local JSONL file using Playwright connected over CDP to an existing Chromium session.

---

### What this script does

- **Connects to an existing browser**: Uses Playwright to attach to a running Chromium instance via CDP at `http://127.0.0.1:9222`.
- **Navigates to the Duolingo words page**: `https://www.duolingo.com/practice-hub/words`.
- **Extracts word + translation pairs**: Scrapes list items that contain an `h3` (word) and a `p` (translation), independent of Duolingo’s randomized CSS class names.
- **Loads additional pages of words**: Repeatedly scrolls/clicks “More” / “Show more” style buttons until no new words appear or the expected count is reached.
- **Writes results to disk**: Saves lines of JSON to `data/duolingo_words.jsonl`, one word/translation pair per line.

---

### Requirements

- **Python**: 3.9+ recommended.
- **Playwright for Python**:
  - `pip install playwright`
  - Then, if you haven’t already, install browser binaries:

```bash
playwright install
```

- **Chromium-based browser**:
  - Must be started with **remote debugging** enabled on port `9222`.
  - You must be **logged into Duolingo** in that browser profile.

---

### Starting your browser with remote debugging

Example for Brave on macOS (adjust path as needed):

```bash
"/Applications/Brave Browser.app/Contents/MacOS/Brave Browser" \
  --remote-debugging-port=9222
```

Example for Chromium on macOS (adjust path as needed):

```bash
"/Applications/Chromium.app/Contents/MacOS/Chromium" \
  --remote-debugging-port=9222
```

Then log into Duolingo in that window (if you are not already), and optionally open the Words / Practice Hub page:

```text
https://www.duolingo.com/practice-hub/words
```

The script will attempt to reuse an existing tab with that URL if it finds one; otherwise it will open a new tab.

If your remote debugging endpoint or port differs, update `CDP_ENDPOINT` near the top of `duolingo_vocab_scraper.py`.

---

### Installation

From the project root:

```bash
python -m venv .venv
source .venv/bin/activate  # on Windows: .venv\Scripts\activate
pip install playwright
playwright install
```

---

### Running the scraper

1. **Start Chromium with remote debugging** (as shown above) and ensure:
   - You are logged into Duolingo.
   - The Words / Practice Hub page can load normally in that browser.
2. **From this project directory**, run:

```bash
python duolingo_vocab_scraper.py
```

The script will:

- Connect to the browser at `CDP_ENDPOINT`.
- Ensure the words list is visible.
- Iterate through visible items, trying to click “More” style controls and scroll to load additional words.
- Stop when:
  - The expected count (parsed from the `h2` heading containing “words”) is reached, **or**
  - It stops seeing new words for several rounds.

On success, you’ll see a message like:

```text
Wrote 427 items to data/duolingo_words.jsonl
```

---

### Output format

- **Directory**: `data/`
- **File**: `duolingo_words.jsonl`
- **Format**: one JSON object per line:

```json
{"word": "hola", "translation": "hello"}
{"word": "adiós", "translation": "goodbye"}
```

This format is convenient for downstream processing (e.g. importing into a database, Anki deck generation, or further analysis).

---

### Customization

- **Change output path**:
  - Edit `OUT_DIR` and `OUT_PATH` near the top of `duolingo_vocab_scraper.py`.
- **Change target URL**:
  - Update the `URL` constant if Duolingo moves or renames the words page.
- **Adjust “More” button heuristics**:
  - The function `click_more_if_possible` tries several selectors and text patterns (`"More"`, `"Show more"`, `"Load more"`, etc.).
  - If Duolingo changes the UI, update the selector list or logic there.

---

### Troubleshooting

- **The script exits with “Couldn't find the word list”**:
  - Make sure:
    - The Chromium window started with `--remote-debugging-port=9222`.
    - You are logged into Duolingo in that profile.
    - The Words page is accessible and fully loaded.
- **No “More” button detected / fewer words than expected**:
  - UI or text labels may have changed.
  - Check the page manually for the button label and adjust `click_more_if_possible`.
- **Connection refused to `http://127.0.0.1:9222`**:
  - Confirm the browser is running with remote debugging enabled.
  - If you use a different port or host, update `CDP_ENDPOINT`.

