# Medical Data Extraction – FastAPI Server

A Python-based medical document extraction project that converts scanned prescription and patient-detail PDFs into structured data.

## Overview

The application follows this pipeline:

```text
PDF
 ↓
PDF → Image (pdf2image + Poppler)
 ↓
Image preprocessing (OpenCV)
 ↓
OCR (Tesseract + pytesseract)
 ↓
Raw text
 ↓
Regex-based parsing
 ↓
Structured medical data
 ↓
FastAPI endpoint
```

The project supports two document types:

- **Prescription**
- **Patient Details**

## Technologies

- Python
- FastAPI
- Uvicorn
- pdf2image
- Poppler
- Tesseract OCR
- pytesseract
- OpenCV
- NumPy
- Pillow
- Regular Expressions (`re`)
- Pytest

## Project Structure

```text
medical_data_extraction/
│
├── src/
│   ├── main.py
│   ├── extractor.py
│   ├── parser_generic.py
│   ├── parser_patient_details.py
│   ├── parser_prescription.py
│   └── util.py
│
├── tests/
│   ├── test_patient_details_parser.py
│   └── test_prescription_parser.py
│
├── resources/
│   ├── patient_details/
│   └── prescription/
│
├── notebooks/
│   ├── cv_concepts.ipynb
│   ├── pd_parser.ipynb
│   └── prescription_parser.ipynb
│
└── requirements.txt
```

## Components

### `main.py`

Creates the FastAPI application and exposes the API endpoint:

```text
POST /extract_from_doc
```

The endpoint accepts:

- `file_format` – document type (`prescription` or `patient_details`)
- `file` – uploaded PDF

It temporarily saves the uploaded file, sends it to the extraction pipeline, returns the extracted data, and removes the temporary file.

### `extractor.py`

Acts as the main extraction pipeline.

It:

1. Converts the PDF into images.
2. Preprocesses the image.
3. Uses Tesseract OCR to extract text.
4. Selects the appropriate parser.
5. Returns structured data.

```text
PDF → Image → Preprocessing → OCR → Parser → Structured Data
```

### `util.py`

Contains image preprocessing logic using OpenCV.

The preprocessing includes:

- converting the image to grayscale
- resizing the image
- adaptive thresholding

The purpose is to improve OCR accuracy when scanned documents contain shadows, noise, or uneven backgrounds.

### `parser_generic.py`

Defines an abstract base parser:

```python
class MedicalDocParser(...)
```

It provides the common structure for document parsers and requires subclasses to implement:

```python
parse()
```

This allows different document types to follow a common parser interface.

### `parser_patient_details.py`

Extracts patient-related information using regular expressions, including:

- patient name
- phone number
- medical problems
- Hepatitis B vaccination status

### `parser_prescription.py`

Extracts prescription information, including:

- patient name
- patient address
- medicines
- directions
- refills

A dictionary of regex patterns is used so that fields can be extracted through a common `get_field()` method.

## API Usage

Start the FastAPI application:

```bash
uvicorn main:app --reload
```

Or run `main.py` directly if the `__main__` block is used.

The API is available locally at:

```text
http://127.0.0.1:8000
```

Interactive API documentation:

```text
http://127.0.0.1:8000/docs
```

## Example Request

Endpoint:

```text
POST /extract_from_doc
```

Form data:

```text
file_format = prescription
file = <PDF file>
```

The server processes the document and returns structured JSON-like data.

Example:

```json
{
  "patient_name": "Marta Sharapova",
  "patient_address": "9 tennis court, new Russia, DC",
  "medicines": "Prednisone 20 mg\nLialda 2.4 gram",
  "directions": "Prednisone, Taper 5 mg every 3 days",
  "refills": "3"
}
```

## Testing

The project uses **Pytest** for automated testing.

Tests cover:

- patient name extraction
- phone number extraction
- medical problems
- vaccination status
- prescription fields
- empty input handling
- complete parser output

Run tests with:

```bash
pytest
```

## Installation

Install the Python dependencies:

```bash
python -m pip install -r requirements.txt
```

Poppler and Tesseract OCR must also be installed separately because they are external programs rather than Python packages.

### Poppler

`pdf2image` uses Poppler to convert PDF pages into images.

### Tesseract

`pytesseract` provides the Python interface, while Tesseract performs the actual OCR.

Update the paths in `extractor.py` to match the local installation:

```python
POPPLER_PATH = r"<path-to-poppler>\Library\bin"

pytesseract.pytesseract.tesseract_cmd = r"<path-to-tesseract>\tesseract.exe"
```

## Key Concepts Demonstrated

- REST API development with FastAPI
- File upload handling
- PDF-to-image conversion
- Image preprocessing
- OCR
- Regular-expression-based information extraction
- Object-oriented parser design
- Abstract base classes
- API validation
- Automated testing with Pytest
- Error handling
- Temporary file management

## Limitations

This is a learning/project implementation and uses rule-based extraction. OCR quality and regex accuracy can vary depending on document layout, scan quality, handwriting, fonts, and unexpected formatting.

The current extraction pipeline processes the first PDF page in `extractor.py`. A production implementation would need additional handling for multi-page documents, more robust parsing, configurable paths, stronger validation, and broader document variations.

## Learning Outcome

The project demonstrates an end-to-end workflow for turning unstructured scanned medical documents into structured information and exposing the extraction functionality through a web API.
