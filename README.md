solana-watcher/
├── solana_watcher.py       ← הקוד הראשי
├── requirements.txt        ← ספריות דרושות
├── render.yaml             ← הגדרות ל־Render
requests
services:
  - type: web
    name: solana-watcher
    env: python
    plan: free
    buildCommand: ""
    startCommand: python solana_watcher.py
    envVars:
      - key: TELEGRAM_BOT_TOKEN
        value: 7877874608:AAEAK4rHLhUd-HIG0MvLpTQT_aWKS2tvgEA
      - key: TELEGRAM_USER_ID
        value: "@OmerPolak"
import requests
import time
from collections import defaultdict

TELEGRAM_BOT_TOKEN = "7877874608:AAEAK4rHLhUd-HIG0MvLpTQT_aWKS2tvgEA"
TELEGRAM_USER_ID = "@OmerPolak"

WATCHED_ADDRESSES = [
    "215nhcAHjQQGgwpQSJQ7zR26etbjjtVdW74NLzwEgQjP",
    "681Cg1mvxyAinUA5R49gzFJKnGKmegSifsjodT71thB8",
    "GGjizkFj1peKww2ndRLo4vGHxgCoMdKLvznf1cS7WKbb",
    "DrD7EwkDy8z6ehw7uYSzFfASoBDSVdnHxMywvAyVTLYV"
]

SOLANA_API = "https://api.mainnet-beta.solana.com"
last_seen_tx = defaultdict(str)

def get_latest_tx(address):
    payload = {
        "jsonrpc": "2.0",
        "id": 1,
        "method": "getSignaturesForAddress",
        "params": [address, {"limit": 1}]
    }
    response = requests.post(SOLANA_API, json=payload)
    result = response.json()
    if 'result' in result and result['result']:
        return result['result'][0]['signature']
    return None

def send_telegram_message(message):
    url = f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage"
    payload = {
        "chat_id": TELEGRAM_USER_ID,
        "text": message,
        "parse_mode": "HTML"
    }
    try:
        requests.post(url, data=payload)
    except Exception as e:
        print(f"â Failed to send Telegram message: {e}")

def check_for_updates():
    for address in WATCHED_ADDRESSES:
        latest_tx = get_latest_tx(address)
        if latest_tx and latest_tx != last_seen_tx[address]:
            print(f"â New transaction for {address}: {latest_tx}")
            last_seen_tx[address] = latest_tx
            tx_url = f"https://solscan.io/tx/{latest_tx}"
            message = f"ð¥ <b>New transaction detected</b>\nð <a href='{tx_url}'>View on Solscan</a>\nð¦ Wallet: <code>{address}</code>"
            send_telegram_message(message)

if __name__ == "__main__":
    print("ð Solana wallet watcher with Telegram alerts is running...")
    while True:
        try:
            check_for_updates()
        except Exception as e:
            print(f"â ï¸ Error: {e}")
        time.sleep(10)