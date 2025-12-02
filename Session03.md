# Session03: サーバ編 - App Service

## ポイント
- App Service プラン (無料/Basic/P1v3) とスケールの考え方
- デプロイ方法 (GitHub Actions, ZIP Deploy) とロギング
- マネージド ID を使った Azure リソースアクセス

## Python (Flask) デプロイ用サンプル
`app.py`
```python
from flask import Flask
import os

app = Flask(__name__)

@app.route("/")
def index():
    return "Hello from Azure App Service!"

@app.route("/config")
def show_config():
    return {"WEBSITE_SITE_NAME": os.getenv("WEBSITE_SITE_NAME", "local")}

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

## 練習問題
1. `/healthz` エンドポイントを追加して 200 OK を返す。
2. Application Insights を有効化し、リクエスト数グラフを確認する。
3. GitHub Actions で main ブランチ push 時に自動デプロイするワークフローを作る。
