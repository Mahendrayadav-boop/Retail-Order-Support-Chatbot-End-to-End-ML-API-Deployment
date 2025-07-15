# Retail-Order-Support-Chatbot-End-to-End-ML-API-Deployment
This is a simple Retail Order Support Chatbot that handles common customer queries such as: - Order Status Lookup - Return Policy Information - Store Locator - General FAQs
retail-chatbot/
├── app/
│   ├── main.py
│   ├── nlp_engine.py
│   ├── responses.py
│   ├── orders_db.py
├── tests/
│   └── test_chat.py
├── requirements.txt
├── Dockerfile
└── README.md
from fastapi import FastAPI
from pydantic import BaseModel
from app.nlp_engine import detect_intent
from app.responses import get_response
from app.orders_db import get_order_status

app = FastAPI()

class Query(BaseModel):
    message: str

@app.post("/chat")
def chat(query: Query):
    intent = detect_intent(query.message)
    response = get_response(intent, query.message)
    return {"reply": response}


 nlp_engine.py (Intent Detection)
python
Copy
Edit
import re

def detect_intent(text):
    text = text.lower()

    if "order" in text and re.search(r"\d{5}", text):
        return "order_status"
    elif "return" in text:
        return "return_policy"
    elif "store" in text or "location" in text:
        return "store_info"
    else:
        return "fallback"

        orders_db.py (Mock Order Data)
python
Copy
Edit
mock_orders = {
    "12345": "Your order #12345 is out for delivery and will arrive today.",
    "67890": "Your order #67890 has been delivered.",
    "54321": "Your order #54321 is being packed and will ship tomorrow."
}

def get_order_status(order_id):
    return mock_orders.get(order_id, "Sorry, we couldn't find that order number.")


    Dockerfile 
Dockerfile
Copy
Edit
FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]



tests/test_chat.py (Basic API Test)
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_chat_order():
    response = client.post("/chat", json={"message": "Where is my order 12345?"})
    assert response.status_code == 200
    assert "12345" in response.json()["reply"]

