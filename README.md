# 🌾 CropGuard — AI-Powered Crop Disease Detection

CropGuard is a full-stack mobile application that helps farmers detect crop diseases by photographing plant leaves. It combines a fine-tuned computer-vision model (served via a Python/Flask API) with a Node.js + Express backend, a React Native / Expo mobile client, and a Google Gemini AI fallback — all with multi-language support for Indian farmers.

---

## Table of Contents

- [Features](#features)
- [Architecture Overview](#architecture-overview)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [1. Python ML API](#1-python-ml-api)
  - [2. Node.js Backend](#2-nodejs-backend)
  - [3. React Native Frontend](#3-react-native-frontend)
- [Usage](#usage)
  - [CLI Diagnosis Tool](#cli-diagnosis-tool)
  - [REST API](#rest-api)
- [Supported Crops & Diseases](#supported-crops--diseases)
- [Multi-Language Support](#multi-language-support)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- 📸 **Camera-based diagnosis** — capture or upload a leaf photo to get an instant diagnosis.
- 🤖 **Dual AI pipeline** — primary local vision model (Hugging Face) with automatic fallback to Google Gemini 1.5 Flash.
- 💊 **Treatment recommendations** — detailed organic, chemical, and prevention advice per disease.
- 🌍 **Multi-language UI** — supports English, Hindi (हिंदी), and Marathi (मराठी).
- 📜 **Scan history** — every diagnosis is stored in MongoDB and accessible from the app.
- 🔒 **Authentication** — JWT-based sign-up / login flow.
- ☁️ **Cloud image storage** — images are uploaded to ImageKit CDN.
- 📵 **Offline-friendly** — authentication tokens and scan history are cached locally.

---

## Architecture Overview

```
Mobile App (Expo / React Native)
        │
        ▼
Node.js Express Backend  ───────────────┐
        │                               │
        ▼ (primary)                     ▼ (fallback)
Python Flask ML API             Google Gemini AI
(Hugging Face model)            (Gemini 1.5 Flash)
        │
        ▼
  MongoDB  +  ImageKit CDN
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Mobile | React Native, Expo |
| Backend | Node.js, Express, MongoDB / Mongoose |
| ML API | Python, Flask, PyTorch, Hugging Face Transformers |
| Fallback AI | Google Gemini 1.5 Flash (`@google/genai`) |
| Image Storage | ImageKit |
| Auth | JWT, bcrypt |
| i18n | React Context API + AsyncStorage |

---

## Repository Structure

```
Crop-detection-project/
├── api_server.py              # Flask inference server (port 5000)
├── run_diagnosis.py           # CLI tool for single-image diagnosis
├── disease_data.py            # Python disease-solution database
├── disease_utils.py           # Helper functions (label parsing, lookups)
├── disease_solutions.js       # Comprehensive JS disease-solution database
├── react_native_integration.js # Example integration snippet
├── requirements.txt           # Core Python dependencies
├── requirements_api.txt       # Flask API dependencies
├── API_SETUP_GUIDE.md         # Step-by-step API setup guide
│
├── backend/                   # Node.js + Express + MongoDB
│   └── src/
│       ├── controllers/       # scan, auth, post
│       ├── models/            # Mongoose schemas
│       ├── routes/            # Express routes
│       ├── services/          # diseaseDetection, ai, storage
│       └── middlewares/       # JWT auth
│
├── frontend/                  # React Native (Expo) app
│   └── src/
│       ├── locales/           # en.json, hi.json, mr.json
│       ├── services/          # API client, upload, auth
│       └── core/              # theme, config
│
└── crop-disease-app/          # Self-contained hackathon demo app
    ├── backend/               # Express + MongoDB + Google OAuth
    └── frontend/              # React Native with mock ML
```

---

## Getting Started

### Prerequisites

- **Python** ≥ 3.9
- **Node.js** ≥ 18 and **npm**
- **MongoDB** instance (local or Atlas)
- **Expo CLI** (`npm install -g expo-cli`)
- (Optional) **Google Gemini API key**
- (Optional) **ImageKit** account credentials

---

### 1. Python ML API

```bash
# Install dependencies
pip install -r requirements_api.txt

# Start the Flask inference server (default port 5000)
python api_server.py
```

The server loads the `Diginsa/Plant-Disease-Detection-Project` model from Hugging Face on startup (first run will download the model weights).

**Endpoint**: `POST http://localhost:5000/predict`  
**Body**: `multipart/form-data` with field `image` (JPEG/PNG).

---

### 2. Node.js Backend

```bash
cd backend
npm install

# Copy and fill in environment variables
cp .env.example .env
```

**.env variables:**

```env
PORT=3000
MONGODB_URI=mongodb://localhost:27017/cropguard
JWT_SECRET=your_jwt_secret
IMAGEKIT_PUBLIC_KEY=...
IMAGEKIT_PRIVATE_KEY=...
IMAGEKIT_URL_ENDPOINT=...
GEMINI_API_KEY=...          # Optional fallback AI
PYTHON_API_URL=http://localhost:5000/predict
```

```bash
npm start
```

---

### 3. React Native Frontend

```bash
cd frontend
npm install

# Update API base URL in src/core/config/
# Then start the Expo dev server
npx expo start
```

Scan the QR code with the **Expo Go** app on your device, or run on an Android/iOS emulator.

---

## Usage

### CLI Diagnosis Tool

Run a single-image diagnosis directly from the command line:

```bash
# Human-readable output
python run_diagnosis.py path/to/leaf.jpg

# JSON output (useful for scripting)
python run_diagnosis.py path/to/leaf.jpg --json
```

Example output:

```
================================================================================
📷 Analyzing: tomato_leaf.jpg
================================================================================

🌱 Crop:       Tomato
🦠 Disease:    Early Blight
📊 Confidence: 95.42% (Very High)

⚠️  Severity: Medium

🌿 Organic/Natural Treatment:
   • Neem oil spray (3 ml/L water)
   • Remove and destroy infected leaves immediately

💊 Chemical Treatment:
   • Mancozeb 75% WP (2 g/L water)

🛡️  Prevention Measures:
   • Avoid overhead irrigation
================================================================================
```

### REST API

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/auth/signup` | Register a new user |
| `POST` | `/auth/login` | Login and receive JWT |
| `POST` | `/api/scan` | Upload image and get diagnosis |
| `GET` | `/api/scans` | Retrieve user's scan history |
| `GET` | `/api/scan/:id` | Get a specific scan result |
| `GET` | `/api/profile/:userId` | Fetch user profile |
| `PUT` | `/api/profile/:userId` | Update user profile |

**Example scan request:**

```bash
curl -X POST http://localhost:3000/api/scan \
  -H "Authorization: Bearer <token>" \
  -F "image=@leaf.jpg"
```

**Example response:**

```json
{
  "crop": "Tomato",
  "disease": "Late Blight",
  "confidence": 92.35,
  "confidence_level": "High",
  "severity": "High",
  "organic": ["Copper oxychloride spray ..."],
  "chemical": ["Mancozeb 75% WP ..."],
  "prevention": ["Avoid overhead irrigation ..."]
}
```

---

## Supported Crops & Diseases

| Crop | Diseases |
|------|---------|
| Tomato | Early Blight, Late Blight, Bacterial Spot |
| Potato | Early Blight, Late Blight |
| Apple | Black Rot, Scab |
| Pepper (Bell) | Bacterial Spot |
| Grape | Black Rot |
| Corn | Common Rust |
| Strawberry | Leaf Scorch |
| Peach | Bacterial Spot |

Healthy plants are also recognized with maintenance tips. New crops and diseases can be added by extending `disease_data.py` and `disease_solutions.js`.

---

## Multi-Language Support

The mobile app ships with translations for:

| Language | Code |
|----------|------|
| English | `en` |
| Hindi | `hi` |
| Marathi | `mr` |

Language preference is persisted via AsyncStorage and can be changed at any time from the Settings screen. Translation files are located in `frontend/src/locales/`.

---

## Contributing

1. Fork the repository and create a feature branch.
2. Make your changes, following the existing code style.
3. Test your changes locally (Python API, backend, and mobile app).
4. Open a pull request with a clear description of what you changed and why.

---

## License

This project is open-source. See [LICENSE](LICENSE) for details.
