# Session01: データベース編 - Storage Account (Table/Blob/Queue)

## ポイント
- ストレージアカウントの作成とアクセスキー/SAS の扱い
- Table: スキーマレスなエンティティ保存とパーティションキー設計
- Blob: 階層構造のオブジェクトストレージ、ライフサイクル管理
- Queue: 簡易キューでの非同期処理

## Python サンプル
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

## 練習問題
1. 同じパーティション内で 5 件の学生データを追加し、スコア 90 以上をクエリで取得する。
2. Blob に画像ファイルをアップロードし、SAS URL を生成して 10 分だけアクセス可能にする。
3. Queue のメッセージをデコードしてコンソールに出力したら削除する小さなスクリプトを書く。
