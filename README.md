# BharatVault

**Intelligent Land Record & Validation System**

Land records in Rajasthan are scattered across different government portals, in different formats, and often as blurry scans or handwritten pages. Checking one plot can mean visiting several offices and comparing documents by hand, which is slow, easy to get wrong, and a good place for fraud to hide.

BharatVault is our attempt to fix that. You upload a scanned record, the system reads it, checks it against rules and other sources, and puts everything into one clean, searchable record linked to a map. A human officer still makes the final call.

Built by **Team ByteBusters** for **Hack for Social Cause 2027** (Governance & Civic Technology, Rajasthan).

---

## What it does

- **Reads scanned documents.** Jamabandi scans, registration deeds, handwritten records and cadastral maps, in Hindi, English and many other languages.
- **Cleans up the image first.** Denoising, de-skewing and contrast fixes with OpenCV so the OCR has a fair chance.
- **Pulls out the important fields.** Owner name, khata, khasra, area, land type and so on, then maps them into one standard format.
- **Checks the record.** Rule-based checks flag missing data, inconsistencies and suspicious patterns, and the record is cross-checked against sources like Apna Khata, e-Panjiyan and Bhu-Naksha.
- **Keeps a human in the loop.** Low-confidence fields go to a reviewer who can correct them. Nothing is final until an authorized officer approves it.
- **Shows it on a map.** Verified records are linked to a Leaflet map with a cadastral overlay, and you can download a PDF summary.

## How it fits together

```
Document upload  ->  Preprocessing (OpenCV)  ->  OCR (PaddleOCR)
      ->  Field extraction (Transformers / LLM)  ->  Validation & cross-check
      ->  Human review & approval  ->  Unified record + map view
```

## Tech stack

| Part | What we use |
|---|---|
| Core | Python |
| Image processing | OpenCV |
| OCR | PaddleOCR (100+ languages) |
| Field extraction & validation | Transformers / LLM |
| Storage | PostgreSQL in the full design, a local `tera-digi.db` file when running on your machine |
| Maps | Leaflet |

---

## Languages

BharatVault is built primarily for Hindi and English. It also supports other languages like Marathi and Bengali, and you can pick which ones to load with the OCR_LANGUAGES setting (see below).

A few honest notes:

- Documents that mix English with a regional language (very common in land records) are fine.
- Accuracy is not the same for every language. Scripts with fewer training examples, and handwriting, will do worse than printed English. That is why the human review step stays in the loop.
- The exact language codes and the full list depend on the PaddleOCR version you have installed, so check the [PaddleOCR docs](https://github.com/PaddlePaddle/PaddleOCR) for the codes available to you.

---

## Getting started

### What you need

- Python 3.9 or newer
- Git
- A few GB of free disk space (the OCR models are not small)

### 1. Clone the repo

```bash
git clone https://github.com/YatinSharma-16/BharatVault
cd BharatVault
```

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

### 4. Set up your environment variables

Copy the example file and fill in your own values:

```bash
cp .env.example .env
```

See the next section for what each variable does.

### 5. Run it

```bash
python app.py
```

Then open http://127.0.0.1:8000 in your browser.

---

## Environment variables

Create a `.env` file in the project root. It should look something like this:

```env
SECRET_KEY=change-this-to-something-long-and-random
DATABASE_PATH=./tera-digi.db
LLM_API_KEY=your-api-key-here
OCR_LANGUAGES=hi,en
PORT=5000
DEBUG=True
```

| Variable | What it's for | Required? |
|---|---|---|
| `SECRET_KEY` | Secret used to sign sessions and protect the app. Use a long random string. | Yes |
| `DATABASE_PATH` | Where the local database file lives. Defaults to `./tera-digi.db`. | No |
| `LLM_API_KEY` | API key for the model that handles field extraction and validation. | Yes |
| `OCR_LANGUAGES` | Comma-separated language codes PaddleOCR should load, e.g. `hi,en` or `hi,en,mr,bn,ta`. Any language PaddleOCR supports works here. More languages means more models to download and a bit more memory. | No |
| `PORT` | Port the app runs on. | No |
| `DEBUG` | Turns on debug mode. Keep it `False` anywhere public. | No |

**Please don't commit your `.env` file.** It has your secrets in it. Make sure `.env` is listed in your `.gitignore`.

---

## Important: the `tera-digi.db` file

When you run BharatVault locally, it creates a file called **`tera-digi.db`**. This is the local database where your records, review status and history are stored, and **the app will not run without it.**

A few things worth knowing:

- It's created automatically the first time you start the app, so you don't need to make it yourself.
- Leave it in the project root (or wherever `DATABASE_PATH` points). If the app can't find it, it won't start.
- Don't delete it unless you're fine losing your local data. If it does get deleted or corrupted, stop the app and start it again so a fresh one is created.
- Treat it like any other local data file. It can contain land record details, so don't share it or push it to a public repo.

---

## Troubleshooting

**The app crashes on startup.**
Check that `tera-digi.db` exists and that `DATABASE_PATH` in your `.env` points to the right place.

**The first OCR run is really slow.**
That's normal. PaddleOCR downloads its language models the first time, so it needs an internet connection and a bit of patience. After that it's much faster.

**Hindi text looks wrong or has missing characters.**
OCR accuracy depends a lot on scan quality. Blurry, faded or skewed scans do worse, and handwritten records are the hardest. This is exactly why we have the human review step. Try a cleaner scan if you can.

**`pip install` fails on a package.**
Update pip first (`python -m pip install --upgrade pip`) and make sure you're inside your virtual environment.

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

OCR accuracy is only a preliminary estimate right now. English comes out best, and regional scripts and handwriting are harder. BharatVault is meant to assist officers and citizens, not replace official verification. Always confirm important records with the relevant government office.
