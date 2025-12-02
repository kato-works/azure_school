# Session03: サーバ編 - App Service

## ポイント
- App Service プラン (無料/Basic/P1v3) とスケールの考え方
- デプロイ方法 (GitHub Actions, ZIP Deploy) とロギング
- マネージド ID を使った Azure リソースアクセス

## App Service の役割
- **フルマネージドな Web ホスティング**: OS/ランタイムのパッチやスケールを Azure が管理し、開発者はアプリコードに集中できる。
- **スケールと SLA**: プランに応じて手軽に垂直/水平スケールが可能で、SLA が明確に定義されている。
- **統合サービス**: マネージド ID による安全なリソースアクセス、デプロイ スロット、診断ログ、App Insights などの運用機能を備える。

## az コマンドによる作成とデプロイの流れ
以下は Linux 上に Python アプリをデプロイする例。
1. リソースグループと App Service プラン、Web App を作成
   ```bash
   az group create -n my-rg -l japaneast
   az appservice plan create -n my-plan -g my-rg --sku B1 --is-linux
   az webapp create -n my-webapp -g my-rg --plan my-plan --runtime "PYTHON|3.10"
   ```
2. デプロイ (ZIP Deploy の例)
   ```bash
   # アプリ一式を zip 化してアップロード
   zip -r app.zip .
   az webapp deploy --resource-group my-rg --name my-webapp --src-path app.zip --type zip
   ```
3. ログの確認とストリーミング
   ```bash
   az webapp log config --name my-webapp --resource-group my-rg --application-logging filesystem
   az webapp log tail --name my-webapp --resource-group my-rg
   ```

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
