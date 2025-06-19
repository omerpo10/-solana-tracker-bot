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
