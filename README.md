# PulmoAI 🫁
### AI-Assisted Pulmonary Nodule Analysis & Clinical Guidance

[![React](https://img.shields.io/badge/Frontend-React%2018-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![Node.js](https://img.shields.io/badge/Backend-Node.js%20%2F%20Express-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![Python](https://img.shields.io/badge/RAG%20Service-Python%203.10%2B-3776AB?logo=python&logoColor=white)](https://python.org/)
[![Gemini](https://img.shields.io/badge/Vision%20AI-Google%20Gemini-4285F4?logo=google&logoColor=white)](https://ai.google.dev/)
[![LangChain](https://img.shields.io/badge/Orchestration-LangChain-1C3C3C?logo=langchain&logoColor=white)](https://langchain.com/)
[![Pinecone](https://img.shields.io/badge/Vector%20DB-Pinecone-000000?logo=pinecone&logoColor=white)](https://www.pinecone.io/)
[![Groq](https://img.shields.io/badge/LLM%20Inference-Groq%20LLaMA%203-F05A28)](https://groq.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Detecting lung nodules early saves lives, but analyzing hundreds of axial CT slices during a busy shift is mentally grueling. Small 2–4 mm lesions can be easily overlooked, and cross-referencing complex clinical guidelines for follow-ups takes valuable time.

**PulmoAI** is an open-source clinical decision support tool designed to serve as a reliable "second pair of eyes." It takes chest CT scan slices, flags potential pulmonary nodules, evaluates their radiological features (size, border margins, attenuation), and maps them directly to **Fleischner Society** follow-up pathways. It also includes an evidence-based medical chatbot that grounds answers in indexed medical literature rather than hallucinating generic advice.

---

## 💡 Why This Project?

1. **Radiologist Workload**: A single chest CT scan routinely contains 300 to 800 slices. Reviewing dozens of scans a day leads to fatigue, increasing the risk of missing subtle, early-stage nodules.
2. **Early Detection Window**: Lung cancer caught at Stage IA has a **5-year survival rate exceeding 80%**, plummeting below **10%** once it metastasizes to Stage IV.
3. **Actionable Recommendations**: Finding a nodule is only half the battle. Clinicians need consistent, guideline-adherent management plans (e.g. low-dose follow-up CT at 3, 6, or 12 months vs. PET-CT or biopsy). PulmoAI automates this mapping instantly.

---

## ⚡ What PulmoAI Does

- **CT Slice Inspection**: Upload thoracic CT slices (PNG, JPG, JPEG) to inspect suspicious pulmonary opacities.
- **Nodule Characterization**: Evaluates core radiological markers:
  - **Estimated Size & Location**: Diameter estimation and anatomical lobe location (e.g. Right Upper Lobe).
  - **Border Morphology**: Distinguishes between benign-leaning smooth contours vs. suspicious lobulated or spiculated (corona radiata) margins.
  - **Attenuation**: Identifies solid, part-solid (subsolid), and pure ground-glass opacities (GGO).
- **Fleischner Society Recommendations**: Translates findings into standardized follow-up intervals based on nodule characteristics and patient risk factors.
- **Model Cascade Resilience**: To avoid being interrupted by Google AI Studio rate limits or quotas, the backend cascades through available Gemini models (`gemini-2.0-flash`, `gemini-1.5-flash`, `gemini-1.5-pro`, etc.) automatically.
- **Literature-Grounded Medical Chatbot**: A dedicated RAG microservice built with LangChain, Pinecone, and Groq LLaMA 3 lets you ask detailed questions about radiological staging, nodule management, or differential diagnoses backed by indexed medical references.
- **One-Click Diagnostic PDF Export**: Clean, client-side PDF generation (`jsPDF`) summarizing the findings, risk tier, and clinical next steps for easy sharing or records.

---

## 🏛️ System Architecture

PulmoAI is split into three decoupled services so each part uses the right tool for the job:

```
                          ┌────────────────────────┐
                          │   PulmoAI Web Client   │
                          │     (React 18 UI)      │
                          │   http://localhost:3000│
                          └───────────┬────────────┘
                                      │
                         REST API     │  Uploads & Chat Queries
                                      ▼
                          ┌────────────────────────┐
                          │   PulmoAI Backend API  │
                          │   (Node.js / Express)  │
                          │   http://localhost:5001│
                          └──────┬──────────┬──────┘
                                 │          │
           Vision Analysis       │          │  Delegated RAG Queries
           (Model Cascade)       │          │
                                 ▼          ▼
             ┌──────────────────────┐   ┌───────────────────────────┐
             │  Google Gemini AI    │   │  Medical RAG Microservice │
             │  Vision Engine       │   │  (Python / Flask)         │
             │  (Flash / Pro)       │   │  http://localhost:5000    │
             └──────────────────────┘   └─────────────┬─────────────┘
                                                      │
                                                      ▼
                                        ┌───────────────────────────┐
                                        │  Pinecone Vector DB       │
                                        │  + Groq LLaMA 3 (LangChain)│
                                        │  (Medical Literature RAG) │
                                        └───────────────────────────┘
```

- **Frontend (`frontend/`)**: React 18 single-page application with responsive dark styling, drag-and-drop file upload, real-time image preview, interactive findings cards, and client-side PDF export.
- **Backend Gateway (`backend/`)**: Node.js & Express server handling image uploads, validating file constraints, orchestrating Gemini Vision calls with automatic failover, and routing chat queries.
- **RAG Microservice (`medical-chatbot/`)**: Python Flask service that queries medical reference embeddings in Pinecone and generates grounded answers via Groq LLaMA 3 LPUs.

---

## 📂 Project Organization

```
PulmoAI/
├── frontend/                 # React 18 user interface
│   ├── src/
│   │   ├── components/       # Upload, Results, Chatbot, and Navbar components
│   │   ├── services/         # Axios API clients
│   │   ├── styles/           # Modern dark-mode stylesheets
│   │   └── App.js            # Main dashboard state & orchestration
│   ├── package.json
│   └── .env.example
│
├── backend/                  # Node.js API gateway
│   ├── routes/
│   │   ├── prediction.js     # Upload handling & Gemini Vision call
│   │   └── chatbot.js        # Chat endpoint proxying to the Python RAG service
│   ├── utils/
│   │   └── geminiAI.js       # Gemini Vision prompt engineering & model cascade
│   ├── test-dicoms/          # Sample benign and nodule cases
│   ├── uploads/              # Local upload buffer (git-ignored)
│   ├── server.js             # Express app setup & CORS config
│   ├── package.json
│   └── .env.example
│
├── medical-chatbot/          # Python Flask RAG microservice
│   ├── data/                 # Medical texts and references for vector store
│   ├── src/
│   │   └── helper.py         # FastEmbed, Pinecone setup, and LangChain pipeline
│   ├── app.py                # Flask server exposing /ask
│   ├── store_index.py        # One-time script to chunk & embed docs into Pinecone
│   ├── requirements.txt
│   └── .env.example
│
└── README.md
```

---

## 🛠️ Getting Started

### Prerequisites

Make sure you have installed:
- **Node.js** (v18 or higher) — [Download](https://nodejs.org/)
- **Python** (3.10 or higher) — [Download](https://python.org/)
- **Git**

You will also need free API keys from:
- [Google AI Studio](https://aistudio.google.com/app/apikey) (for Gemini Vision)
- [Groq Console](https://console.groq.com/) (for ultra-fast LLaMA 3 inference)
- [Pinecone](https://www.pinecone.io/) (for the serverless medical vector index)

---

### Step 1: Clone the Repository

```bash
git clone https://github.com/Sourav-nitj/PulmoAI.git
cd PulmoAI
```

---

### Step 2: Environment Configuration

Each service has its own `.env` file. You can create them quickly from the provided examples:

#### 1. Backend (`backend/.env`)
```bash
cp backend/.env.example backend/.env
```
Open `backend/.env` and supply your Gemini API key:
```env
PORT=5001
FRONTEND_URL=http://localhost:3000
GEMINI_API_KEY=your_gemini_api_key_here
RAG_SERVICE_URL=http://localhost:5000/ask
```

#### 2. Frontend (`frontend/.env`)
```bash
cp frontend/.env.example frontend/.env
```
Default values usually work right out of the box:
```env
PORT=3000
BROWSER=none
REACT_APP_API_URL=http://localhost:5001/api
```

#### 3. Medical Chatbot (`medical-chatbot/.env`)
```bash
cp medical-chatbot/.env.example medical-chatbot/.env
```
Open `medical-chatbot/.env` and add your Pinecone and Groq credentials:
```env
PINECONE_API_KEY=your_pinecone_api_key_here
GROQ_API_KEY=your_groq_api_key_here
PORT=5000
SECRET_KEY=your_random_secret_key
```

*(Optional)* If populating the Pinecone index from scratch using `data/Medical_book.pdf`:
```bash
cd medical-chatbot
python store_index.py
cd ..
```

---

### Step 3: Start the Services

Run each service in its own terminal tab:

#### Terminal 1: Python RAG Service
```bash
cd medical-chatbot
python -m venv venv

# Activate virtual environment:
# On macOS/Linux:
source venv/bin/activate
# On Windows:
# .\venv\Scripts\activate

pip install -r requirements.txt
python app.py
```
> Running at **`http://localhost:5000`**

#### Terminal 2: Node.js Backend API
```bash
cd backend
npm install
npm start
```
> Running at **`http://localhost:5001`**

#### Terminal 3: React Frontend UI
```bash
cd frontend
npm install
npm start
```
> Running at **`http://localhost:3000`**

Open **[http://localhost:3000](http://localhost:3000)** in your browser to start using PulmoAI.

---

## 🧪 Testing with Sample Scans

If you don't have CT scan slices handy:
1. Sample test cases are located in `backend/test-dicoms/`:
   - `backend/test-dicoms/benign/`: Cases with non-malignant or clear lung fields.
   - `backend/test-dicoms/nodules/`: Cases featuring identifiable pulmonary nodules.
2. You can export or capture individual axial CT slices as PNG or JPG images to test the model's morphology analysis and Fleischner classification.

---

## 📡 API Endpoints

### Backend API (`http://localhost:5001`)

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/api/health` | Service health check |
| `POST` | `/api/predict` | Multipart upload (`image`) analyzing CT slice with Gemini Vision |
| `POST` | `/api/chatbot` | Conversation endpoint forwarding query and history to RAG service |
| `GET` | `/api/chatbot/info` | Status and details of active AI engines and disclaimers |

### RAG Microservice (`http://localhost:5000`)

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/` | Standalone interactive chat interface |
| `POST` | `/ask` | JSON payload `{ "query": "...", "history": [...] }` querying Pinecone + Groq |

---

## ⚠️ Clinical Disclaimer

**PulmoAI is an educational, research, and technical demonstration project.**  
It is **not** an FDA-cleared or CE-certified medical diagnostic device, and it must never replace professional clinical evaluation. All model inferences, risk scores, and guideline suggestions should always be reviewed and confirmed by a certified radiologist or licensed medical professional prior to any patient care decision.

---

## 🤝 Contributing & Feedback

Contributions, feedback, and issue reports are very welcome! If you'd like to improve nodule characterization prompts, enhance the RAG pipeline, or optimize UI performance:

1. Fork the repo.
2. Create a feature branch (`git checkout -b feature/improved-guidelines`).
3. Commit your changes (`git commit -m 'Add improved Fleischner pathway handling'`).
4. Push to your branch and open a Pull Request.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
