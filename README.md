# FaceAI Command Center

Real-time face enrollment and live recognition system with a React dashboard and FastAPI backend.

## What this project does

- Enrolls a person from multiple guided camera captures (pose-based flow).
- Extracts face embeddings using InsightFace.
- Stores embeddings in FAISS for fast similarity search.
- Runs live recognition from webcam frames.
- Shows detection history, confidence, and live stats in the frontend.
- Optionally records enrollment metadata in PostgreSQL.

## Tech stack

### Frontend

- React 19 + Vite 7
- Tailwind CSS
- Framer Motion
- React Router

### Backend

- FastAPI
- InsightFace (buffalo_l)
- ONNX Runtime (CUDA + CPU providers configured)
- OpenCV
- FAISS (inner-product index for cosine similarity)
- NumPy
- Optional PostgreSQL via psycopg

## Project structure

```text
Face_ai/
  app/
    main.py                  # FastAPI app + routes
    core/
      recognizer.py          # InsightFace wrapper
      vector_db.py           # FAISS index and labels persistence
      matcher.py             # Similarity matching logic
      camera.py              # OpenCV camera helper
      database.py            # Optional PostgreSQL metadata storage
    services/
      enrollment.py          # Enrollment pipeline (multi-image)
      pipeline.py            # OpenCV real-time pipeline (desktop)
  src/
    pages/
      EnrollmentPage.jsx
      LiveDetectionPage.jsx
    hooks/
      useCamera.js
      useEnrollmentFlow.js
      useFaceDetection.js
    services/
      apiBase.js
      enrollmentService.js
      faceApi.js
  data/
    faiss.index              # Saved vector index
    labels.pkl               # Label mapping
```

## Prerequisites

- Python 3.10+
- Node.js 18+
- npm 9+
- Webcam access in browser

For GPU acceleration on Windows, install a compatible NVIDIA driver and CUDA runtime stack for your ONNX Runtime build.

## Setup

### 1. Clone and open

```powershell
git clone <your-repo-url>
cd Face_ai
```

### 2. Python environment and backend dependencies

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install fastapi uvicorn python-multipart numpy opencv-python insightface onnxruntime faiss-cpu psycopg[binary]
```

Notes:
- If you plan to run GPU inference, install the ONNX Runtime GPU package variant appropriate for your CUDA setup.
- The app stores FAISS artifacts in the data directory. Ensure data exists.

### 3. Frontend dependencies

```powershell
npm install
```

## Running the app

### Option A: Full web app (recommended)

Run backend:

```powershell
.\venv\Scripts\Activate.ps1
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Run frontend in another terminal:

```powershell
npm run dev
```

Open:
- Frontend: http://localhost:5173
- Backend API docs: http://localhost:8000/docs

### Option B: Backend serves built frontend from dist

Build frontend:

```powershell
npm run build
```

Then run backend:

```powershell
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Open: http://localhost:8000

### Option C: Desktop OpenCV real-time pipeline

```powershell
python app/main.py
```

This starts the OpenCV recognition window and does not run the FastAPI web server.

## API endpoints

### GET /

- Returns dist/index.html if present, otherwise root index.html.

### POST /enroll

- Form fields:
  - name: string
  - files: multiple images
- Behavior:
  - Detects largest face per image.
  - Filters low-quality faces.
  - Normalizes embeddings.
  - Adds all valid embeddings to FAISS.
  - Saves index and labels to data.
  - Optionally logs enrollment session to PostgreSQL.

### POST /recognize

- Form field:
  - file: image
- Returns best match with similarity score.

### POST /recognize/live

- Form field:
  - file: frame image
- Returns list of detections with:
  - normalized bounding box
  - name
  - confidence
  - similarity
  - face pose and keypoints (when available)

## Environment variables

Frontend:
- VITE_API_BASE_URL (optional)
  - If unset, frontend auto-targets port 8000 on the same host.

Backend PostgreSQL metadata logging (optional):
- DATABASE_URL

Or individual variables:
- PGPASSWORD
- PGHOST (default localhost)
- PGPORT (default 5432)
- PGUSER (default postgres)
- PGDATABASE (default face_ai)

You can place these in a local .env file for backend metadata storage.

## How enrollment quality works

The frontend guided flow checks:

- Single-face requirement
- Minimum face size in frame
- Minimum detection confidence
- Pose-specific checks (straight, left, right, up, down)
- Stability hold time before capture acceptance

Captured poses are then posted in one enrollment request.

## Data persistence

- Face vectors are saved in data/faiss.index.
- Label mapping is saved in data/labels.pkl.
- Metadata (if DB configured) is saved in people and enrollment_sessions tables.

## Troubleshooting

### Camera not starting

- Allow browser camera permission.
- Use HTTPS or localhost context supported by your browser.
- Confirm no other app is locking the webcam.

### Enrollment returns No face detected

- Improve lighting.
- Keep one clear face centered.
- Use higher-quality captures and stable pose.

### Low recognition accuracy

- Enroll more varied angles per person.
- Avoid blurry or heavily occluded captures.
- Tune the similarity threshold in app/core/matcher.py and app/main.py.

### Backend starts but frontend cannot connect

- Confirm backend is on port 8000.
- Set VITE_API_BASE_URL if backend is on another host/port.
- Check CORS/network/firewall constraints.

### FAISS files not updating

- Ensure write permission in data.
- Confirm enrollment completed successfully with at least one valid embedding.

## Scripts

Frontend scripts from package.json:

- npm run dev
- npm run build
- npm run preview
- npm run lint

## Current status

- Enrollment UI: implemented
- Live detection UI: implemented
- FastAPI integration: implemented
- FAISS persistence: implemented
- Optional PostgreSQL metadata logging: implemented
- Performance optimization for high FPS: in progress

## License

Add your preferred license here.
