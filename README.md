
# Smart Document Scanner with OCR

Turn a phone photo of a document into a clean, flat scan with its text extracted.

Group project (team of 2) for the Digital Image Processing Lab, Iqra University, Karachi.

---

## What it does

Upload a photo or use your camera, and the app will:

- Find the four corners of the page and let you drag them to adjust
- Fix the perspective so the page becomes flat and straight
- Remove shadows, clean up noise, and whiten the paper
- Read the text using OCR
- Pull out fields like name, ID number, and date of birth
- Find and crop any photos printed on the document

Everything runs on your own machine. Nothing is uploaded or stored anywhere.

---

## Results

| What was measured | Result |
|---|---|
| Documents found in benchmark scenes | 14 out of 14 |
| Corner accuracy | 5 to 18 pixels on a 3000x4000 frame |
| Mean OCR word confidence | 94.3% |
| Time from crop to results | about 3.5 seconds |

Tested on white A4 paper, a green Pakistani ID card, a book page on patterned cloth, a white page on a white desk, and pages running off the edge of the frame.

---

## How it works

This project uses **classical image processing only**, no trained neural networks. That was a deliberate choice so every step can be shown and measured instead of hidden inside a model.

**Finding the page.** The standard tutorial method of grayscale, blur, Canny, and largest four point contour worked on clean photos and failed on almost everything else. So instead, candidate shapes are generated from colour masks, Canny edges, and a Hough line transform, then each one is scored on four things: how well each side sits on a real edge, the colour step from inside to outside the page, whether the inside looks like paper or like patterned cloth, and the overall shape. The score takes the worst of the four sides, so one bad side rejects the whole candidate.

**Cleaning the image.** The most important step turned out to be background flattening. The paper is estimated using a morphological closing that removes the text, then each channel is divided by that estimate. This removes uneven lighting and makes the paper close to white across the whole page. After that comes denoising, contrast stretching, and sharpening.

**Reading the text.** Photo regions are masked out, the image is rescaled so letters are about thirty pixels tall, then deskewed and passed to Tesseract. Low confidence words are filtered out afterwards.

---

## Tech used

**Backend:** Python, FastAPI, OpenCV, Tesseract 5
**Frontend:** React, TypeScript, Vite

---

## Running it

You need Python 3.10 or newer, Node.js, and Tesseract 5 installed.

Backend:
```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --port 8000
```

Frontend:
```bash
cd frontend
npm install
npm run dev
```

Open the HTTPS address shown in the terminal. HTTPS is required because browsers block camera access on plain HTTP.

---

## What it does not do

- Handwriting
- Languages other than English
- Multi page PDFs

The benchmark uses generated test scenes rather than a labelled set of real photos, so the numbers above show the system works but are not a formal evaluation. The field parser was built around Pakistani ID card labels and will return nothing for document types it has not seen.
