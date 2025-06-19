# -solana-tracker-bot

import requests
from flask import Flask, request
import threading

app = Flask(__name__)

# Telegram Bot Token
TELEGRAM_TOKEN = "7674489225:AAEdIGUzB-qQWghs3BpLaJXQhqgtD4R3aQY"
TELEGRAM_CHAT_ID = "@OmerPolak"  # אפשר להחליף ב-ID אם צריך

# כתובות ארנקי Solana למעקב
WATCHED_WALLETS = [
    "215nhcAHjQQGgwpQSJQ7zR26etbjjtVdW74NLzwEgQjP",
    "681Cg1mvxyAinUA5R49gzFJKnGKmegSifsjodT71thB8",
    "GGjizkFj1peKww2ndRLo4vGHxgCoMdKLvznf1cS7WKbb",
    "DrD7EwkDy8z6ehw7uYSzFfASoBDSVdnHxMywvAyVTLYV"
]

# כאן תכניס את ה-API Key שלך מ-Helius (חינם בהרשמה)
HELIUS_API_KEY = "YOUR_HELIUS_API_KEY"

def send_telegram_message(message):
    url = f"https://api.telegram.org/bot{TELEGRAM_TOKEN}/sendMessage"
    data = {
        "chat_id": TELEGRAM_CHAT_ID,
        "text": message
    }
    requests.post(url, data=data)

@app.route("/webhook", methods=["POST"])
def helius_webhook():
    payload = request.json
    for tx in payload.get("transactions", []):
        description = tx.get("description", "Unknown transaction")
        signature = tx.get("signature", "")
        message = f"📥 פעולה חדשה בארנק סולאנה:

{description}
🔗 https://solscan.io/tx/{signature}"
        send_telegram_message(message)
    return "", 200

def register_helius_webhook():
    url = f"https://api.helius.xyz/v0/webhooks?api-key={HELIUS_API_KEY}"
    webhook_data = {
        "webhookURL": "https://YOUR_SERVER_URL/webhook",  # להחליף בכתובת השרת שלך
        "transactionTypes": ["ALL"],
        "accountAddresses": WATCHED_WALLETS,
        "webhookType": "enhanced"
    }
    response = requests.post(url, json=webhook_data)
    print("Webhook status:", response.text)

def run_flask():
    app.run(host="0.0.0.0", port=8000)

if __name__ == "__main__":
    threading.Thread(target=register_helius_webhook).start()
    run_flask()
# main.py

from flask import Flask, request
from telegram_bot import send_telegram_message

app = Flask(__name__)

@app.route("/", methods=["GET"])
def home():
    return "Solana Webhook is running!", 200

@app.route("/webhook", methods=["POST"])
def webhook():
    data = request.json

    # דוגמה לפענוח פעולה מה-webhook
    for tx in data.get("transactions", []):
        description = tx.get("description", "No description")
        signature = tx.get("signature", "No signature")
        amount = tx.get("amount", 0)
        sender = tx.get("source", "Unknown")
        receiver = tx.get("destination", "Unknown")

        message = f"""
🔔 פעולה חדשה ב-Solana:

➡️ מ: {sender}
⬅️ אל: {receiver}
💸 סכום: {amount / 1e9:.2f} SOL
🧾 תיאור: {description}
🔗 https://solscan.io/tx/{signature}
        """.strip()

        send_telegram_message(message)

    return {"status": "received"}, 200

if __name__ == "__main__":
    app.run(port=5000)
