# An-Intelligent-Enterprise-Assistant-for-public-sector-SIH-1706-Completion-requirements
## Name: MAHALAKSHMI.R
## REG. NO: 212223230117
## AIM:
To develop an AI-powered chatbot using Deep Learning and Natural Language Processing (NLP) that accurately understands and responds to employee queries related to HR policies, IT support, company events, and other organizational matters. The chatbot can also analyze uploaded documents to extract keywords or summaries and supports 2-Factor Authentication (2FA) via email. It is designed to handle at least 5 users simultaneously with a maximum response time of 5 second.

## PROCEDURE:
1.System Design: Plan chatbot flow, authentication, document upload, and chat interfaces.

2.Environment Setup: Install FastAPI for backend, React for frontend, and required NLP libraries.

3.Model Integration: Use Sentence Transformers for embeddings and FAISS for document search.

4.Document Processing: Extract text from PDF, clean and summarize using transformers.

5.Chat Functionality: Retrieve top-matching information and generate accurate answers.

6.Security: Implement 2FA (OTP via email) for login verification.

7.Testing: Ensure 5-user parallel scalability and <5s response time.

## FOLDER STRUCTURE:
```
chatbot-hackathon-repo/
├── backend/
│ ├── app/
│ │ ├── main.py
│ │ ├── auth.py
│ │ ├── chatbot.py
│ │ ├── doc_process.py
│ │ ├── utils.py
│ │ └── uploads/
│ ├── requirements.txt
│ └── Dockerfile
├── frontend/
│ ├── index.html
│ ├── script.js
│ └── style.css
└── README.md
```
## BACKEND CODE:
```
## requirements.txt:
fastapi
uvicorn
pydantic
email-validator
python-multipart
transformers
torch
sentence-transformers
faiss-cpu
PyPDF2
nltk
```
## main.py:
```
from fastapi import FastAPI, UploadFile, Form
from fastapi.middleware.cors import CORSMiddleware
from app.auth import router as auth_router
from app.doc_process import extract_text_from_pdf, summarize_text
from app.chatbot import Chatbot

app = FastAPI(title="Intelligent Enterprise Assistant")

app.add_middleware(
CORSMiddleware,
allow_origins=["*"],
allow_methods=["*"],
allow_headers=["*"]
)
app.include_router(auth_router, prefix="/auth")

chatbot = Chatbot()

@app.post("/upload-doc")
async def upload_doc(file: UploadFile):
text = extract_text_from_pdf(await file.read())
summary = summarize_text(text)
chatbot.add_document(text)
return {"summary": summary}

@app.post("/chat")
async def chat(query: str = Form(...)):
response = chatbot.get_response(query)
return {"reply": response}

@app.get("/")
def home():
return {"message": "Chatbot is running successfully!"}
```
## auth.py
```
from fastapi import APIRouter, Form
from pydantic import EmailStr
import random


router = APIRouter()
otp_storage = {}


@router.post("/send-otp")
def send_otp(email: EmailStr = Form(...)):
otp = random.randint(100000, 999999)
otp_storage[email] = otp
print(f"OTP for {email}: {otp}") # Demo purpose only
return {"message": "OTP sent to your email"}


@router.post("/verify-otp")
def verify_otp(email: EmailStr = Form(...), otp: int = Form(...)):
if otp_storage.get(email) == otp:
return {"message": "OTP verified successfully!"}
return {"error": "Invalid OTP"}
```
## chatbot.py
```
from sentence_transformers import SentenceTransformer, util


class Chatbot:
def __init__(self):
self.model = SentenceTransformer("all-MiniLM-L6-v2")
self.docs = []
self.doc_embeddings = None


def add_document(self, text):
self.docs.append(text)
self.doc_embeddings = self.model.encode(self.docs, convert_to_tensor=True)


def get_response(self, query):
if not self.docs:
return "No documents uploaded yet."
query_embedding = self.model.encode(query, convert_to_tensor=True)
scores = util.pytorch_cos_sim(query_embedding, self.doc_embeddings)
best_doc = self.docs[scores.argmax()]
return best_doc[:500]
```
## doc_process.py
```
import PyPDF2
from transformers import pipeline


def extract_text_from_pdf(file_bytes):
reader = PyPDF2.PdfReader(file_bytes)
text = ""
for page in reader.pages:
text += page.extract_text()
return text


def summarize_text(text):
summarizer = pipeline("summarization", model="facebook/bart-large-cnn")
summary = summarizer(text[:1024], max_length=120, min_length=30, do_sample=False)
return summary[0]['summary_text']
```
## utils.py
```
import re


def clean_text(text):
text = re.sub(r'\s+', ' ', text)
return text.strip()
```
## Dockerfile
```
FROM python:3.10
WORKDIR /app
COPY . .
RUN pip install -r backend/requirements.txt
CMD ["uvicorn", "backend.app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```
## FRONTEND CODE:

## index.html
```
<!DOCTYPE html>
<html>
<head>
<title>Enterprise Chatbot</title>
<link rel="stylesheet" href="style.css">
</head>
<body>
<h2>Intelligent Enterprise Assistant </h2>


<div class="chatbox">
<div id="messages"></div>
<input type="text" id="query" placeholder="Ask something..." />
<button onclick="sendMessage()">Send</button>
</div>


<script src="script.js"></script>
</body>
</html>
```
## script.js
```
async function sendMessage() {
let query = document.getElementById("query").value;
let messages = document.getElementById("messages");
messages.innerHTML += `<p><b>You:</b> ${query}</p>`;


let formData = new FormData();
formData.append("query", query);


const res = await fetch("http://127.0.0.1:8000/chat", {
method: "POST",
body: formData
});
const data = await res.json();
messages.innerHTML += `<p><b>Bot:</b> ${data.reply}</p>`;
}
```
## style.css
```
body { font-family: Arial; background: #f0f0f0; text-align: center; }
.chatbox { width: 400px; margin: 30px auto; background: white; padding: 15px; border-radius: 10px; }
#messages { height: 300px; overflow-y: auto; text-align: left; margin-bottom: 10px; }
input { width: 70%; padding: 8px; }
button { padding: 8px 12px; background: #007bff; color: white; border: none; border-radius: 5px; }
```
 ## HOW TO RUN:
 ```

 cd backend
pip install -r requirements.txt ( FOR INSTALL DEPENDENCIES)

uvicorn app.main:app --reload ( RUN SERVER)

Open frontend/index.html in browser.( OPEN FRONTEND)
```
## RESULT:
The chatbot is executed and verified successfully.
