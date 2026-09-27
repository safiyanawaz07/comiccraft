# 💥 ComicCraft — AI Comic Story Creator

Turn a one-line story idea into a finished, illustrated comic, then download it as a PDF.

ComicCraft uses **Google Gemini** to plan the story and write the dialogue, and **Stable Diffusion**
(plus other free image AIs) to draw every panel.

TEAM MEMBERS:
1.Sai varshini(team ledear)
2.Sai Medha
3.Safiya Nawaz
4.Roshini
## Features

- Story, characters and dialogue written by Google Gemini
- A comic title, 6 panels, narration captions and speech bubbles
- Artwork in the style you pick (anime, cartoon, watercolor, pixel art, ...), drawn by free AI image services with automatic fallback: FLUX.1 → Stable Diffusion 3 → Pollinations.ai → AI Horde
- The main character is described the same way in every image prompt, so they look alike from panel to panel
- Panels are drawn in parallel, so a comic is ready in about 1–2 minutes
- A PDF with a cover page and one page per panel
- A gallery of every comic you've made
- A "How it works" page to use in presentations
- Friendly error messages. If Gemini's free quota runs out, the app switches to a backup model.
- A JSON API (`POST /generate-comic/json`) with interactive docs at `/docs`

## Project structure

```
ComicCraft/
├── app/
│   ├── main.py            # FastAPI app entry point
│   ├── routes.py          # Web pages + API endpoints
│   ├── config.py          # Settings, paths, API keys
│   ├── setup_keys.py      # Asks for API keys and writes .env
│   ├── gemini_client.py   # Shared Gemini helper (retries, model fallback)
│   ├── gemini_flash.py    # Step 1: story outline
│   ├── gemini_pro.py      # Step 2: narration + dialogue
│   ├── image_generator.py # Step 3: panel artwork (Hugging Face)
│   ├── layout_builder.py  # Step 4: combine text + images
│   └── exporters.py       # Step 5: PDF export
├── templates/             # HTML pages (Jinja2)
├── static/
│   ├── css/style.css
│   ├── fonts/             # DejaVu fonts for the PDFs, so non-English letters show correctly
│   ├── panels/            # generated images (created at runtime)
│   └── exports/           # generated PDFs (created at runtime)
├── requirements.txt
├── .env.example           # template for your API keys
├── run.bat                # one-click start (Windows)
└── run.sh                 # one-click start (macOS / Linux)
```

## Setup

### 1. Install Python

Install **Python 3.10 or newer** from https://www.python.org/downloads/.
On Windows, tick **"Add Python to PATH"** during installation.

Works on **Windows 10/11**, macOS and Linux. Any old laptop is fine, because the AI runs in the cloud,
but it needs an internet connection. Windows 7/8 isn't supported, because newer Python versions don't run on them.

### 2. Get two free API keys

| Key | Where to get it |
|---|---|
| `GEMINI_API_KEY` | https://aistudio.google.com/apikey → **Create API key** |
| `HF_API_KEY` | https://huggingface.co/settings/tokens → **Create new token** (type: *Read*) |

### 3. Run it

**Windows:** double-click `run.bat`.
**macOS / Linux:** run `./run.sh` in a terminal.

The first run installs everything, then asks you to paste your two keys (right-click pastes in the Windows terminal).
They're saved to a `.env` file, and the app opens at **http://127.0.0.1:8000**.

To change your keys later, delete `.env` and run the script again, or run `python -m app.setup_keys`.

<details>
<summary>Manual setup (without the scripts)</summary>

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate      macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
python -m app.setup_keys    # asks for your keys and saves them to .env
python -m uvicorn app.main:app --reload
```
</details>

## Pages

| URL | What it does |
|---|---|
| `/` | Create a comic |
| `/gallery` | All comics made so far |
| `/about` | How it works + tech stack |
| `/docs` | Interactive API docs (FastAPI) |
| `/health` | Status check. Shows whether the API keys are set. |

## Troubleshooting

| Problem | Fix |
|---|---|
| "Setup needed" banner on the homepage | Add the missing key to `.env` and restart the app |
| "Gemini's free-tier usage limit was reached" | Free keys allow a limited number of requests per day. Wait a minute, try again tomorrow, or use another key. |
| Panels show a placeholder instead of artwork | All four image services failed. Check your internet connection and try again in a few minutes. |
| Images have a small "pollinations.ai" watermark | The first two services were out of free quota. It resets daily (FLUX) or monthly (Hugging Face). |
| Comics are slow | Add `NUM_PANELS=4` to `.env` for shorter comics |
| `python` is not recognized (Windows) | Reinstall Python and tick **Add Python to PATH** |
| Port 8000 already in use | Close the other app, or run `uvicorn app.main:app --port 8001` |

## Uploading to GitHub

> ⚠️ **Never upload your `.env` file.** It holds your secret API keys. Anyone who sees them can use up your quota.

### Option A: GitHub Desktop (recommended, no commands)

GitHub Desktop reads `.gitignore`, so it automatically leaves out `.env`, `.venv` and your generated comics.

1. Create a free account at https://github.com/signup
2. Install **GitHub Desktop** from https://desktop.github.com and sign in
3. **File → Add local repository →** choose the `ComicCraft` folder
4. It will say "this directory does not appear to be a Git repository". Click **create a repository**, then **Create repository**
5. Check the list of files on the left. `.env` must **not** be in it.
6. Click **Publish repository**. Untick "Keep this code private" if you want it public, then click **Publish**.

To upload changes later: write a short summary, click **Commit to main**, then **Push origin**.

### Option B: Upload in the browser

The browser upload does **not** read `.gitignore`, so remove these first:

- Delete `.env`, or move it out of the folder
- Delete the `.venv` folder, if you've run the app already
- Empty `static/panels/` and `static/exports/`, but keep the `.gitkeep` files

Then:

1. On GitHub click **+ → New repository**, name it `ComicCraft`, then click **Create repository**
2. Click **uploading an existing file**
3. Drag in **everything inside** the `ComicCraft` folder (not the folder itself)
4. Click **Commit changes**

### Option C: Command line (if Git is installed)

```bash
cd ComicCraft
git init
git add .
git status            # make sure .env is NOT listed
git commit -m "ComicCraft: AI comic story creator"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/ComicCraft.git
git push -u origin main
```

### If a key was ever uploaded by mistake

Deleting the file from GitHub isn't enough. Create a new key and delete the old one at
https://aistudio.google.com/apikey and https://huggingface.co/settings/tokens, then update your local `.env`.

## Credits

- Text generation: [Google Gemini](https://ai.google.dev/)
- Image generation: [FLUX.1-schnell](https://huggingface.co/spaces/black-forest-labs/FLUX.1-schnell), [Stable Diffusion 3](https://huggingface.co/stabilityai/stable-diffusion-3-medium-diffusers), [Pollinations.ai](https://pollinations.ai), [AI Horde](https://aihorde.net)
- PDF font: [DejaVu Fonts](https://dejavu-fonts.github.io/) (free license, see `static/fonts/DejaVu-LICENSE.txt`)
