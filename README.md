# BharatVault

**Intelligent Land Record & Validation System**

Land records in Rajasthan are scattered across different government portals, in different formats, and often as blurry scans or handwritten pages. Checking one plot can mean visiting several offices and comparing documents by hand, which is slow, easy to get wrong, and a good place for fraud to hide.

BharatVault is our attempt to fix that. You upload a scanned record, the system reads it, checks it against rules, and puts everything into one clean, searchable record linked to a map. A human officer still makes the final call.

Built by **Team ByteBusters** for **Hack for Social Cause 2027** (Governance & Civic Technology, Rajasthan).

---

## What it does

- **Reads scanned documents.** Jamabandi scans, registration deeds and other land records, mainly in Hindi and English.
- **Cleans up the image first.** Denoising and contrast fixes with OpenCV so the OCR has a fair chance.
- **Pulls out the important fields.** Owner name, khasra, khata, area, land use and so on, in both the original language and English.
- **Checks the record.** Rule-based checks flag missing data, inconsistencies and suspicious patterns, and a timeline view follows the chain of ownership.
- **Keeps a human in the loop.** Citizens can submit documents, clerks and revenue officers review them, and admins manage users and audit logs. Nothing is final until an officer approves it.
- **Shows it on a map.** Records are linked to a Leaflet map built on OpenStreetMap.

## How it fits together

```
Document upload  ->  Preprocessing (OpenCV)  ->  OCR (PaddleOCR)
      ->  Field extraction (local rules, optional Gemini)  ->  Validation
      ->  Human review & approval  ->  Unified record + map view
```

## Tech stack

| Part | What we use |
|---|---|
| Backend | Python, FastAPI, Uvicorn |
| Login | JWT tokens (python-jose) |
| Image & PDF handling | OpenCV, Pillow, PyMuPDF |
| OCR | PaddleOCR (Tesseract as an optional fallback) |
| Field extraction | Built-in local rules, with optional Google Gemini |
| Storage | SQLite file (`terra_digitize.db`) in this build, PostgreSQL in the full design |
| Frontend | Single `index.html` page with Leaflet and OpenStreetMap |

---

## Languages

BharatVault is built primarily for **Hindi and English**, and that's what the OCR and field extraction are tuned for. Support for other languages like **Marathi** and **Bengali** is planned. PaddleOCR can load other language models through the `PADDLE_OCR_LANG` setting, but we haven't tuned the extraction rules for them yet.

Records that mix Hindi and English on the same page are fine. Accuracy isn't the same for every document, and handwriting is harder than print. That's why the human review step stays in the loop.

---

## Getting started

### What you need

- Python 3.9 or newer
- Git
- An internet connection the first time you run it (OCR models get downloaded) and whenever you use the map
- A few GB of free disk space

### 1. Clone the repo

```bash
git clone https://github.com/YatinSharma-16/BharatVault.git
cd BharatVault
cd "Source Code"
```

Everything below is run from inside the `Source Code` folder, next to `main.py` and `index.html`.

### 2. Make a virtual environment

```bash
python -m venv venv

# macOS / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### 3. Install the dependencies

```bash
pip install -r requirements.txt
```

This installs PaddleOCR, so it can take a few minutes.

### 4. Start the server

```bash
uvicorn main:app --reload
```

### 5. Open the site

Go to **http://127.0.0.1:8000** in your browser.

To log in the first time, use the default admin account:

- **Username:** `admin`
- **Password:** `admin123`

Please change that password before sharing the app with anyone (see [Before you deploy](#before-you-deploy)).

---

## Environment variables

Good news: you don't need any to get started. BharatVault runs fine with the defaults.

If you want to change something, set the variable in your terminal **before** starting the server. The app doesn't read a `.env` file.

```bash
# macOS / Linux
export GEMINI_API_KEY=your-key-here

# Windows PowerShell
$env:GEMINI_API_KEY="your-key-here"

# Windows Command Prompt
set GEMINI_API_KEY=your-key-here
```

| Variable | What it's for | Default |
|---|---|---|
| `GEMINI_API_KEY` | Optional. If set, Google Gemini helps read documents and normalise the fields. If not, the built-in local extraction is used. | not set |
| `OCR_ENGINE` | Which OCR engine to use: `paddle` or `tesseract`. | `paddle` |
| `PADDLE_OCR_LANG` | Language model PaddleOCR loads. If it isn't available, the app falls back to `hi` and then `en`. | `devanagari` |
| `TESSERACT_LANG` | Language codes for Tesseract. Only used when `OCR_ENGINE=tesseract`. | `hin+eng` |
| `MAX_OCR_PAGES` | Maximum number of pages read from a single PDF. | `6` |
| `PADDLE_DOC_ORIENTATION` | Set to `1` to detect and fix page rotation. | `0` |
| `PADDLE_ENABLE_MKLDNN` | Set to `1` to turn on Intel MKLDNN acceleration. It's off by default because it can crash on some Windows setups. | `0` |

**Never put real API keys in the code or commit them to GitHub.**

### Using Tesseract instead of PaddleOCR (optional)

1. Install the Tesseract program on your computer, with Hindi language data.
2. In `requirements.txt`, remove the `#` in front of `pytesseract` and `pdf2image`, then run `pip install -r requirements.txt` again.
3. Set `OCR_ENGINE=tesseract` and start the server.

---

## Important: the `terra_digitize.db` file

The first time you start BharatVault, it creates a database file called **`terra_digitize.db`**. This is where users, records, review status and audit logs are stored, and the app depends on it to run locally. It also creates an `uploads/` folder for the documents people upload.

A few things worth knowing:

- It's created automatically, so you don't need to make it yourself.
- The path is relative, so **always start the server from the same folder** (`Source Code`). If you start it from somewhere else, you'll get a brand new empty database and it will look like your data and logins vanished.
- If you delete it, you start fresh: all records and users are gone, and only the default admin account comes back.
- It holds land record details and login data, so don't share it. Keep `terra_digitize.db` and `uploads/` out of Git by adding them to your `.gitignore`.

---

## Troubleshooting

**I see "index.html not found next to main.py".**
You started the server from the wrong folder. Go into `Source Code` and run `uvicorn main:app --reload` again.

**My data or login disappeared.**
Most likely you started the server from a different folder, so a new `terra_digitize.db` was created. Go back to the original folder.

**The first OCR run is really slow.**
That's normal. PaddleOCR downloads its models the first time, so it needs internet and a bit of patience. After that it's much faster.

**PaddleOCR crashes on Windows.**
Make sure `PADDLE_ENABLE_MKLDNN` is not set to `1`.

**The map is blank.**
The map loads Leaflet and OpenStreetMap tiles from the internet, so check your connection.

**Hindi text looks wrong or has missing characters.**
OCR accuracy depends a lot on scan quality. Blurry, faded or skewed scans do worse, and handwritten records are the hardest. Try a cleaner scan if you can, and use the review step to correct the fields.

**`pip install` fails on a package.**
Update pip first (`python -m pip install --upgrade pip`) and make sure you're inside your virtual environment.

---

## Before you deploy

BharatVault is currently set up for local use and demos. Before putting it on a public server:

- Change the default `admin` / `admin123` login.
- Replace the `SECRET_KEY` that's hardcoded in `main.py` with your own secret.
- Restrict the CORS settings in `main.py`, which currently allow requests from any origin.

---

## Data sources

BharatVault is designed around official Rajasthan sources:

- Apna Khata (e-Dharti): khasra, khata, jamabandi
- e-Panjiyan: property registration and deeds
- Bhu-Naksha (Rajasthan State GIS): cadastral maps
- Urban Local Bodies (JDA, UITs), RIICO and the Department of Revenue

Legal grounding includes the Rajasthan Land Revenue Act 1956, the Registration Act 1908 and the Information Technology Act 2000.

## Team

| Name | Role |
|---|---|
| **Daksh Saxena** (Team Lead) | Backend, OCR pipeline and validation engine, project coordination |
| **Yatin Sharma** | Frontend, GIS / map integration, UI and UX |
| **Jayesh Kumawat** | Research, data sourcing, testing and documentation |

Registration ID: `HSC|RJ|00091`

## A note on accuracy

OCR accuracy is only a preliminary estimate right now. English comes out best, and Hindi and handwriting are harder. BharatVault is meant to assist officers and citizens, not replace official verification. Always confirm important records with the relevant government office.

---

Made with a lot of chai in Rajasthan. If you find a bug or have an idea, open an issue. We'd love to hear it.