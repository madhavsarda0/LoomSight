# WeaveCAD — Greige Fabric Technical Sheet to CAD Converter

A full-stack web application that converts greige weaving fabric technical sheets (PDF, JPEG, Excel, DOC) into weave pattern CAD images (PNG/SVG).

## Features

- **Multi-format upload**: PDF, JPEG, PNG, Excel (.xls/.xlsx), Word (.doc/.docx), CSV, TXT
- **Auto text extraction**: Extracts specifications from uploaded documents using OCR and parsers
- **Smart parsing**: Auto-detects EPI, PPI, yarn counts, weave type, and width from technical text
- **Weave pattern generation**: Supports Plain, Twill, Satin, Basket, and Huckaback weaves
- **Dual view modes**:
  - **Point paper**: Classic CAD grid view for loom programming
  - **Fabric simulation**: Realistic thread interlacing visualization
- **Export**: Download patterns as high-resolution PNG or vector SVG
- **Technical summary**: Auto-calculates total warp ends, repeat size, and construction details

## Supported Weave Types

| Weave | Code | Repeat |
|-------|------|--------|
| Plain 1/1 | `plain` | 2×2 |
| Twill 1/2 | `twill12` | 3×3 |
| Twill 2/1 | `twill21` | 3×3 |
| Twill 2/2 | `twill22` | 4×4 |
| Twill 3/1 | `twill31` | 4×4 |
| Twill 1/3 | `twill13` | 4×4 |
| Satin 5-harness | `satin5` | 5×5 |
| Satin 8-harness | `satin8` | 8×8 |
| Basket 2/2 | `basket22` | 4×4 |
| Basket 3/3 | `basket33` | 6×6 |
| Huckaback | `huck` | 6×6 |

## Installation

### 1. Clone / download the project

```bash
cd weave-cad-app
```

### 2. Create a virtual environment (recommended)

```bash
python -m venv venv
source venv/bin/activate   # Linux/Mac
venv\Scripts\activate    # Windows
```

### 3. Install Python dependencies

```bash
pip install -r requirements.txt
```

### 4. Install Tesseract OCR (optional, for image/PDF OCR)

**Ubuntu/Debian:**
```bash
sudo apt-get install tesseract-ocr poppler-utils
```

**macOS:**
```bash
brew install tesseract poppler
```

**Windows:**
Download and install from [https://github.com/UB-Mannheim/tesseract/wiki](https://github.com/UB-Mannheim/tesseract/wiki)

### 5. Run the application

```bash
python app.py
```

Open your browser to: **http://localhost:5000**

## Architecture

```
weave-cad-app/
├── app.py                 # Flask backend (API + routes)
├── requirements.txt       # Python dependencies
├── README.md             # This file
├── uploads/              # Temporary file storage
├── templates/
│   └── index.html        # Single-page frontend
├── static/
│   ├── css/              # Styles (inline in index.html)
│   └── js/               # Scripts (inline in index.html)
```

### Backend API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Main application page |
| `/api/upload` | POST | Upload file, extract text, parse specs |
| `/api/parse` | POST | Parse pasted text for specifications |
| `/api/generate` | POST | Generate weave pattern (PNG + SVG) |
| `/api/download/<format>` | POST | Download PNG or SVG file |

### Text Extraction Pipeline

1. **PDF**: `pdfplumber` extracts text → fallback to `pytesseract` OCR if scanned
2. **Images**: `pytesseract` OCR
3. **Excel**: `openpyxl` / `pandas` reads all sheets
4. **DOCX**: `python-docx` reads paragraphs and tables
5. **DOC**: `antiword` command-line tool (or convert to DOCX)
6. **TXT/CSV**: Direct text read

### Parsing Engine

Regex-based parser detects:
- `EPI` / `Ends per inch` / `Warp density`
- `PPI` / `Picks per inch` / `Weft density`
- Construction format: `80x72/40x40`
- Width in inches
- Yarn counts (warp/weft)
- Weave type keywords (plain, twill, satin, basket, huckaback)

### Weave Pattern Generation

- **Point paper**: Binary grid matrix rendered with PIL
- **Fabric simulation**: Thread-over-thread visualization with shading and shadows
- **SVG export**: Pure vector output for CAD/CAM systems

## Usage Workflow

1. **Upload** a technical sheet (PDF, image, Excel, Word) or paste extracted text
2. **Review** auto-detected specifications (EPI, PPI, weave, width, yarn counts)
3. **Adjust** parameters manually if needed
4. **Select** view mode (Point paper or Fabric simulation)
5. **Generate** the weave pattern
6. **Download** as PNG (raster) or SVG (vector CAD)

## Example Input Text

```
Greige Fabric Specification
Construction: 80x72/40x40
Weave: Plain 1/1
Width: 63 inches
Warp: 40 Ne
Weft: 40 Ne
EPI: 80
PPI: 72
```

The parser will automatically extract all parameters from this text.

## Docker (Optional)

```dockerfile
FROM python:3.11-slim

RUN apt-get update && apt-get install -y \
    tesseract-ocr poppler-utils \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

```bash
docker build -t weave-cad .
docker run -p 5000:5000 weave-cad
```

## License

MIT License — free for commercial and personal use.

## Notes

- For scanned PDFs without embedded text, install `pdf2image` + `poppler` + `tesseract` for OCR support
- Old `.doc` files require `antiword` or conversion to `.docx`
- The application runs in debug mode by default; set `debug=False` for production
