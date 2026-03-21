# Automatic Number Plate Recognition

A real-time ANPR project built with Python, OpenCV, and Tesseract OCR.

This system captures frames from a webcam, detects plate-shaped regions, corrects perspective, extracts text from the plate, validates the result against known patterns, and saves confirmed detections to disk.

## Why This Project Stands Out

Instead of stopping at simple OCR on a static image, this project follows a complete computer vision pipeline:

- live camera capture
- contour-based plate detection
- perspective alignment
- OCR preprocessing
- text extraction with Tesseract
- format validation
- temporal confirmation across multiple frames
- CSV logging plus image capture for confirmed plates

That makes it a practical end-to-end demo rather than just a single-script experiment.

## Demo Pipeline

The recognition flow is:

1. Capture a frame from the webcam.
2. Detect plate-like rectangular candidates using edges and contours.
3. Warp the detected plate into a flat aligned image.
4. Preprocess the aligned crop for OCR.
5. Read characters using Tesseract.
6. Validate the extracted text using known plate patterns.
7. Confirm the plate only after repeated matching across recent frames.
8. Save the confirmed result to `data/plates.csv` and `data/captures/`.

## Current Tech Stack

- Python
- OpenCV
- NumPy
- pytesseract
- imutils
- Pillow

## Project Structure

```text
plate-recognition/
|-- README.md
|-- Quickstart.md
|-- requirements.txt
|-- main_verbose.py
|-- data/
|   |-- captures/
|-- screenshots/
|-- src/
|   |-- main.py
|   |-- camera.py
|   |-- detect.py
|   |-- align.py
|   |-- ocr.py
|   |-- validate.py
|   |-- temporal.py
|   |-- storage.py
|-- venv/
```

## What Each File Does

- `src/main.py`: runs the main live ANPR pipeline and displays the OpenCV windows
- `src/camera.py`: opens the webcam and sets capture resolution
- `src/detect.py`: finds plate-like regions using grayscale conversion, filtering, edges, morphology, and rotated rectangles
- `src/align.py`: applies a four-point perspective transform and resizes the aligned plate
- `src/ocr.py`: preprocesses the plate image and reads text using Tesseract
- `src/validate.py`: validates OCR output against supported plate patterns
- `src/temporal.py`: reduces false positives by confirming repeated detections across recent frames
- `src/storage.py`: saves confirmed results to CSV and prevents duplicate saves within a short time window
- `main_verbose.py`: an older verbose/debug script that is not aligned with the current functional pipeline

## How Detection Works

The plate detector in `src/detect.py` follows a lightweight classical computer vision approach:

- convert frame to grayscale
- preserve edges with bilateral filtering
- run Canny edge detection
- use morphological closing to merge plate characters into a stronger plate-shaped blob
- inspect the largest contours
- estimate rotated bounding boxes
- keep candidates with plate-like aspect ratios and sufficient area

This is a good fit for a school project because it is explainable, easy to debug, and does not require training a model.

## OCR and Validation

After alignment, the plate image is:

- converted to grayscale
- smoothed with bilateral filtering
- binarized using Otsu thresholding

Tesseract is then called with a character whitelist of uppercase letters and digits. The extracted result is cleaned and matched against known validation patterns such as:

- `ABC123`
- `ABC123D`
- `AB123CD`

## Confirmation Logic

OCR on live video can be noisy. To avoid saving false positives, the system does not save a plate immediately after a single read.

Instead, `TemporalConfirm` stores recent detections and confirms a plate only when it appears repeatedly within the recent history. In the current implementation:

- history size: `10`
- confirmation threshold: `3`

This makes the output more stable in real-world use.

## Output

When a plate is confirmed, the system saves:

- the plate number
- the timestamp
- the image path

to:

```text
data/plates.csv
```

It also saves the aligned plate image into:

```text
data/captures/
```

Debug screenshots can also be written to:

```text
screenshots/
```

## Installation

### 1. Create and activate a virtual environment

Windows PowerShell:

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

### 2. Install Python packages

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 3. Install Tesseract OCR

On Windows, install Tesseract and make sure the executable exists at:

```text
C:\Program Files\Tesseract-OCR\tesseract.exe
```

The current project is configured to use that path in `src/ocr.py`.

If your Tesseract installation is in a different location, update:

```python
pytesseract.pytesseract.tesseract_cmd = r"C:\Program Files\Tesseract-OCR\tesseract.exe"
```

## How To Run

From the project root:

```powershell
python .\src\main.py
```

## Controls

- `q`: quit the application
- `s`: save the current debug screenshots

## Screens You Will See

The app can display:

- `Detection`: live frame with detected candidate outlines and OCR status
- `Aligned Plate`: perspective-corrected plate crop
- `OCR Input`: the thresholded image passed to Tesseract

## Example Use Case

This project can be used as:

- a computer vision course project
- a final-year demonstration project
- a base for parking gate automation experiments
- a starting point for building a smarter ANPR system with deep learning later

## Known Limitations

This implementation is strong as a classical CV prototype, but it still has some practical limitations:

- it depends heavily on lighting and camera angle
- it uses contour heuristics rather than a trained detector
- OCR quality drops when the plate is blurry, dirty, tilted, or far away
- the current OCR path is configured specifically for Windows
- `main_verbose.py` appears to reference class-based components that are not present in the current `src/` code

## Ideas For Future Improvement

- add automatic OS-specific Tesseract path detection
- support multiple plate formats and regions
- add OCR confidence scoring to saved output
- export bounding boxes and recognition metadata
- replace heuristic detection with a trained detector such as YOLO
- add a test script for static sample images
- build a small dashboard for browsing saved detections

## Notes

This repository looks like it started from a cloned template, but the current codebase already has a real working pipeline and sample outputs. The README now reflects the code that actually exists in this project today.

## License

This project is intended for educational and learning purposes.
