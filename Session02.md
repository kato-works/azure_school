# Session02: 関数編 - Function App

## ポイント
- トリガーの種類 (HTTP/Timer/Queue など) とバインディング
- コンテナ化デプロイの概要とローカル開発 (func start)
- アプリケーション設定と Key Vault 連携

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

## 練習問題
1. Timer トリガーで 1 分ごとに Storage Queue へメッセージを書く関数を作る。
2. HTTP トリガーで受け取った JSON を Blob に保存し、保存先 URL を返す関数を作る。
3. アプリ設定に保存した API キーを取得し、リクエストヘッダーに echo する関数を作る。
