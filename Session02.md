# Session02: 関数編 - Function App

## Function App の役割
- 複数の Azure Function をまとめてホストするアプリケーション単位で、同じリソースグループ・同じアプリ設定を共有する。
- スケールや認証、アプリ設定 (環境変数) をアプリ単位で集中管理できるため、運用やデプロイの単位として扱う。
- バインディングの接続文字列や Key Vault 参照は Function App のアプリ設定に格納するのが基本。

## トリガーの種類 (代表例)
- HTTP: API や Webhook 入口に利用。認証レベル (anonymous/function/admin) を設定できる。
- Timer: CRON 形式のスケジュール実行。リトライやオフセット管理も可能。
- Queue / Service Bus Queue・Topic: 非同期メッセージ駆動。DLQ や最大デリバリ回数で堅牢化。
- Blob: Blob 追加・更新イベントで実行。イベントグリッド経由で遅延を抑えられる。
- Event Hub / Event Grid: 大量イベント処理や Pub/Sub 連携に利用。

## Azure Functions Core Tools (`func`) のインストール例
- npm 経由: `npm i -g azure-functions-core-tools@4 --unsafe-perm true`
- apt 経由 (Debian/Ubuntu):
  ```bash
  curl -sL https://packages.microsoft.com/keys/microsoft.asc | sudo gpg --dearmor -o /usr/share/keyrings/microsoft.gpg
  echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/azure-functions/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/azure-functions.list
  sudo apt-get update && sudo apt-get install azure-functions-core-tools-4
  ```
- macOS (Homebrew): `brew tap azure/functions && brew install azure-functions-core-tools@4`

## 新規作成の流れ
1) リポジトリ/ディレクトリを準備: `mkdir my-func && cd my-func`
2) 関数アプリの言語とランタイムを初期化: `func init --worker-runtime python --python` (TypeScript なら `--worker-runtime node` 等)
3) 関数を追加: `func new --name HttpTrigger --template "HTTP trigger" --authlevel function`
4) ローカル実行: `func start` (ポート 7071)。必要に応じて `local.settings.json` に接続文字列を設定。

## デプロイの流れ
1) Azure CLI でリソースを作成 (例):
   ```bash
   az group create -n my-rg -l japaneast
   az storage account create -n mystorage12345 -g my-rg -l japaneast --sku Standard_LRS
   az functionapp create -n my-func-app -g my-rg --storage-account mystorage12345 --consumption-plan-location japaneast --runtime python
   ```
2) アプリ設定の同期: `func azure functionapp fetch-app-settings my-func-app`
3) デプロイ: `func azure functionapp publish my-func-app` または CI/CD (GitHub Actions, Azure Pipelines) を構成。

## Python (HTTP トリガー) サンプル
`__init__.py`
```python
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:
    name = req.params.get("name") or "world"
    return func.HttpResponse(f"Hello, {name}!")
```
`function.json`
```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

## v1 / v2 ランタイムの簡易サンプル
- **v1 (Functions 1.x, Windows 専用)**
  - host.json (例):
    ```json
    {
      "functionTimeout": "00:05:00",
      "logger": {"default": "Information"}
    }
    ```
  - C# Script HTTP 例 (`run.csx`):
    ```csharp
    #r "Microsoft.Azure.WebJobs.Extensions.Http"
    using System.Net;
    public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
    {
        var name = req.GetQueryNameValuePairs().FirstOrDefault(q => q.Key == "name").Value ?? "world";
        return req.CreateResponse(HttpStatusCode.OK, $"Hello (v1), {name}!");
    }
    ```
- **v2 以降 (Functions 2.x/3.x/4.x, クロスプラットフォーム)**
  - host.json (例):
    ```json
    {
      "version": "2.0",
      "extensionBundle": {
        "id": "Microsoft.Azure.Functions.ExtensionBundle",
        "version": "[3.*, 4.0.0)"
      }
    }
    ```
  - Python HTTP 例 (`__init__.py`):
    ```python
    import logging
    import azure.functions as func

    def main(req: func.HttpRequest) -> func.HttpResponse:
        logging.info("Processing request on v2+")
        name = req.params.get("name") or "world"
        return func.HttpResponse(f"Hello (v2+), {name}!", status_code=200)
    ```

## 練習問題
1. Timer トリガーで 1 分ごとに Storage Queue へメッセージを書く関数を作る。
2. HTTP トリガーで受け取った JSON を Blob に保存し、保存先 URL を返す関数を作る。
3. アプリ設定に保存した API キーを取得し、リクエストヘッダーに echo する関数を作る。
