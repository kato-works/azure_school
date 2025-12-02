# Azure クラウド初心者向け 7 回講座

各回 90 分想定。基本の概念説明 → 5〜10 分のデモ → ハンズオン課題の流れで進行します。Azure ポータル操作のスクリーンショットは講師側で補完してください。

## 第1回 データベース編: Storage Account (Table/Blob/Queue)
### ポイント
- ストレージアカウントの作成とアクセスキー/SAS の扱い
- Table: スキーマレスなエンティティ保存とパーティションキー設計
- Blob: 階層構造のオブジェクトストレージ、ライフサイクル管理
- Queue: 簡易キューでの非同期処理

### Python サンプル
```python
from azure.data.tables import TableServiceClient
from azure.storage.blob import BlobServiceClient
from azure.storage.queue import QueueServiceClient
import os

connection_string = os.environ["AZURE_STORAGE_CONNECTION_STRING"]

# Table
table_client = TableServiceClient.from_connection_string(connection_string)
table = table_client.get_table_client("students")
table.create_table_if_not_exists()
table.upsert_entity({"PartitionKey": "classA", "RowKey": "001", "name": "Mika", "score": 92})

# Blob
blob_service = BlobServiceClient.from_connection_string(connection_string)
container = blob_service.get_container_client("materials")
container.create_container()
container.upload_blob(name="intro.txt", data="Hello Azure Storage!", overwrite=True)

# Queue
queue_service = QueueServiceClient.from_connection_string(connection_string)
queue = queue_service.get_queue_client("tasks")
queue.create_queue()
queue.send_message("process-student-score")
```

### 練習問題
1. 同じパーティション内で 5 件の学生データを追加し、スコア 90 以上をクエリで取得する。
2. Blob に画像ファイルをアップロードし、SAS URL を生成して 10 分だけアクセス可能にする。
3. Queue のメッセージをデコードしてコンソールに出力したら削除する小さなスクリプトを書く。

---

## 第2回 関数編: Function App
### ポイント
- トリガーの種類 (HTTP/Timer/Queue など) とバインディング
- コンテナ化デプロイの概要とローカル開発 (func start)
- アプリケーション設定と Key Vault 連携

### Python (HTTP トリガー) サンプル
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

### 練習問題
1. Timer トリガーで 1 分ごとに Storage Queue へメッセージを書く関数を作る。
2. HTTP トリガーで受け取った JSON を Blob に保存し、保存先 URL を返す関数を作る。
3. アプリ設定に保存した API キーを取得し、リクエストヘッダーに echo する関数を作る。

---

## 第3回 サーバ編: App Service
### ポイント
- App Service プラン (無料/Basic/P1v3) とスケールの考え方
- デプロイ方法 (GitHub Actions, ZIP Deploy) とロギング
- マネージド ID を使った Azure リソースアクセス

### Python (Flask) デプロイ用サンプル
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

### 練習問題
1. `/healthz` エンドポイントを追加して 200 OK を返す。
2. Application Insights を有効化し、リクエスト数グラフを確認する。
3. GitHub Actions で main ブランチ push 時に自動デプロイするワークフローを作る。

---

## 第4回 スケジューラ編: Automation Account
### ポイント
- Runbook (PowerShell/Python) とスケジュール
- マネージド ID の利用と資格情報の保管
- ハイブリッド Worker でオンプレ操作も可能

### Python Runbook サンプル
```python
import datetime
from azure.identity import DefaultAzureCredential
from azure.mgmt.resource import ResourceManagementClient

credential = DefaultAzureCredential()
subscription_id = "<subscription-id>"
resource_client = ResourceManagementClient(credential, subscription_id)

for group in resource_client.resource_groups.list():
    print(f"{datetime.datetime.utcnow()}: {group.name}")
```

### 練習問題
1. 1 日 1 回、停止中 VM を一覧取得しメール通知する Runbook を作る。
2. スケジュール実行と手動実行の両方を試し、ログ出力を確認する。
3. ハイブリッド Worker 経由でオンプレの PowerShell コマンドを実行するパターンを調査する。

---

## 第5回 サーバ編: Virtual Machine (PowerShell)
### ポイント
- VM サイズ選定、ネットワーク (NSG, パブリック IP) の基本
- Azure CLI / PowerShell での VM 作成と初期設定
- 拡張機能 (Custom Script Extension) での自動セットアップ

### PowerShell サンプル (Cloud Shell 想定)
```powershell
$rg = "rg-azure-school"
$loc = "japaneast"
$vmName = "demo-vm"

New-AzResourceGroup -Name $rg -Location $loc
New-AzVm \n  -ResourceGroupName $rg \n  -Name $vmName \n  -Location $loc \n  -Image "Win2022Datacenter" \n  -VirtualNetworkName "$vmName-vnet" \n  -SubnetName "default" \n  -SecurityGroupName "$vmName-nsg" \n  -PublicIpAddressName "$vmName-ip" \n  -OpenPorts 80,3389

# IIS をインストールしてテストページを作成
Set-AzVMExtension -ResourceGroupName $rg -VMName $vmName -Name "iis-setup" -Publisher "Microsoft.Compute" -Type "CustomScriptExtension" -TypeHandlerVersion "1.10" -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; Set-Content -Path C:\\inetpub\\wwwroot\\index.html -Value \"Hello from Azure VM!\""}'
```

### 練習問題
1. VM を B 系列と D 系列で作成し、性能と料金を比較するレポートを作る。
2. NSG で 22 番ポートのみ許可し、Just-In-Time アクセスを有効化する手順をまとめる。
3. Custom Script Extension で Python をインストールし、起動時に簡単な HTTP サーバを立ち上げる。

---

## 第6回 イベント処理編: Event Hubs
### ポイント
- パーティションとスループットユニットの考え方
- Producer/Consumer のロールとチェックポイント
- Capture で Blob/ADLS にアーカイブする仕組み

### Python サンプル
Producer:
```python
from azure.eventhub import EventHubProducerClient, EventData
import os

producer = EventHubProducerClient.from_connection_string(
    conn_str=os.environ["EVENTHUB_CONNECTION_STRING"],
    eventhub_name=os.environ["EVENTHUB_NAME"],
)

with producer:
    event_data_batch = producer.create_batch()
    for i in range(10):
        event_data_batch.add(EventData(f"message {i}"))
    producer.send_batch(event_data_batch)
```
Consumer (簡易版):
```python
from azure.eventhub import EventHubConsumerClient
import os

connection_str = os.environ["EVENTHUB_CONNECTION_STRING"]
eventhub_name = os.environ["EVENTHUB_NAME"]
consumer_group = "$Default"

client = EventHubConsumerClient.from_connection_string(connection_str, consumer_group, eventhub_name=eventhub_name)

def on_event(partition_context, event):
    print(f"Partition {partition_context.partition_id}: {event.body_as_str()}")
    partition_context.update_checkpoint(event)

with client:
    client.receive(on_event=on_event, starting_position="-1")
```

### 練習問題
1. Capture を設定し、Blob に保存された Avro ファイルを Python で読み出して表示する。
2. 消費者を 2 つ起動し、パーティション間で負荷分散される挙動を確認する。
3. Azure Monitor メトリックでスループットユニット消費量を追跡し、アラートを設定する。

---

## 第7回 デバイス管理編: IoT Hub
### ポイント
- IoT Hub とデバイス ID の登録、接続文字列/証明書
- Device-to-Cloud (D2C) / Cloud-to-Device (C2D) メッセージ
- Device Twin による設定管理とデバイス更新

### Python サンプル
デバイス送信 (D2C):
```python
from azure.iot.device import IoTHubDeviceClient, Message
import os

conn_str = os.environ["IOTHUB_DEVICE_CONNECTION_STRING"]
device_client = IoTHubDeviceClient.create_from_connection_string(conn_str)

msg = Message('{"temperature": 26.5, "humidity": 60}')
device_client.send_message(msg)
device_client.shutdown()
```
クラウドからデバイスへ (C2D) 送信:
```python
from azure.iot.hub import IoTHubRegistryManager
import os

iothub_conn_str = os.environ["IOTHUB_CONNECTION_STRING"]
device_id = "my-device"

registry_manager = IoTHubRegistryManager(iothub_conn_str)
registry_manager.send_c2d_message(device_id, "update-config")
```

### 練習問題
1. Device Twin に desired property として `samplingInterval` を設定し、デバイス側で適用するコードを書く。
2. IoT Hub のメッセージルーティングで、温度が 30℃ 以上のメッセージを Event Hub に送るルールを作成する。
3. Azure Stream Analytics を利用して IoT Hub のデータを Power BI へリアルタイム表示するパイプライン図を描き、設定手順をまとめる。

---

## 付録: 環境準備メモ
- 受講者は Azure 無料アカウント、Azure CLI、Python 3.10 以降を事前インストール。
- Python クライアントライブラリ例: `pip install azure-storage-blob azure-storage-queue azure-data-tables azure-functions azure-eventhub azure-identity azure-mgmt-resource azure-iot-device azure-iot-hub`
- VS Code の Azure 拡張と Azure Functions 拡張をインストールしておくとデモが容易。
