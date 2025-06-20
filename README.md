import time
from solana.rpc.api import Client
from telegram import Bot

# פרטי טלגרם וארנקים
TELEGRAM_TOKEN = "7877874608:AAEAK4rHLhUd-HIG0MvLpTQT_aWKS2tvgEA"
CHAT_ID = "7801163656"

WALLETS = [
    "9xQeWvG816bUbDgc5F1ozcMwVKhj8tTgfTChZcDNr3xA",
    "681Cg1mvxyAinUA5R49gzFJKnGKmegSifsjodT71thB8",
    "215nhcAHjQQGgwpQSJQ7zR26etbjjtVdW74NLzwEgQjP",
    "GFHMc9BegxJXLdHJrABxNVoPRdnmVxXiNeoUCEpgXVHw",
    "973vghafz4fQYB3MquWdLZd8dBMzJWcsTyBxH2GAMqcY"
]

# התחברות ל-Solana
sol = Client("https://api.mainnet-beta.solana.com")
bot = Bot(token=TELEGRAM_TOKEN)
last_signatures = {}

def fetch_signatures(wallet):
    resp = sol.get_signatures_for_address(wallet, limit=5)
    return [sig["signature"] for sig in resp["result"]]

def check_wallet(wallet):
    global last_signatures
    sigs = fetch_signatures(wallet)
    prev = last_signatures.get(wallet, [])
    new = [s for s in sigs if s not in prev]
    if new:
        for sig in reversed(new):
            bot.send_message(
                chat_id=CHAT_ID,
                text=f"🤖 ברנאר מדווח:\n"
                     f"📍 פעילות חדשה בארנק:\n{wallet}\n"
                     f"🔗 https://solscan.io/tx/{sig}"
            )
    last_signatures[wallet] = sigs

def main():
    for w in WALLETS:
        last_signatures[w] = fetch_signatures(w)
    while True:
        for w in WALLETS:
            check_wallet(w)
        time.sleep(30)

if __name__ == "__main__":
    main()