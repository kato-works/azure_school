# Session01: データベース編 - Storage Account (Table/Blob/Queue)

## ポイント
- ストレージアカウントの役割と 4 種のストレージ (Blob/Table/Queue/Files) の特徴
- アクセスキー・SAS の使い分け、データ レジデンシーと冗長化オプションの確認
- Table: スキーマレスなエンティティ保存と PK/RK 設計のベストプラクティス
- Blob: フォルダーに見えるパス構造でのオブジェクト配置、ライフサイクル管理
- Queue: 簡易キューでの非同期処理

## ストレージアカウント概要
- 単一の名前空間で Blob/Table/Queue/Files をホストする論理コンテナー。
- パフォーマンスはアクセスパターン (シーケンシャル/ランダム) と冗長化 (LRS/ZRS/GRS) に影響。
- 安全な公開には共有アクセス署名 (SAS) を用い、アプリ側からは接続文字列を Key Vault 経由で取得する運用が推奨。

## ストレージごとの用途・特徴
- **Blob Storage**: 大容量バイナリの格納。ホット/クール/アーカイブの階層化、静的サイトホスティング、イベントグリッド連携に適する。
- **Table Storage**: スキーマレスなキー・バリュー (NoSQL) で、巨大なフラットデータを水平分割しやすい。PK/RK で高速ポイント/レンジクエリが可能。
- **Queue Storage**: 軽量なメッセージング。可視性タイムアウトでワーカーの冪等性を担保し、最大 7 日保持。
- **Azure Files**: SMB/NFS 共有として扱える。リフト&シフトやオンプレ/クラウド間のファイル共有用途。

## Table の PK・RK 設計
- **PartitionKey (PK)**: スケール単位 (パーティション) を決め、同じ PK 内はトランザクション (バッチ) を共有。高頻度アクセスのホットスポットを避けるよう、時間ベースやハッシュで分散させる。
- **RowKey (RK)**: パーティション内で一意なソートキー。レンジクエリに使うフィールド (例: `timestamp` や `category#id`) を含めると効率的。
- **設計ポイント**:
  - アクセスパターンから PK/RK を逆算する (頻出クエリを O(1) またはレンジで完結させる)。
  - 1 パーティションのスループット上限を意識し、均等分散するキーを採用する。
  - 変更が多いフィールドは RK に含めない (更新頻度とバッチ粒度を意識)。

## Blob のパス設計
- **パス構造**: 仮想フォルダーとして `/container/prefix1/prefix2/file` の形で付与。前方一致でリストされるため、同一プレフィックスに近いオブジェクトが集まる。
- **設計ポイント**:
  - アクセス頻度が高いオブジェクトが 1 プレフィックスに集中しないよう、日時やハッシュで階層を分散する (例: `images/2024/05/31/<hash>.jpg`)。
  - 主要なフィルター条件をパスに含め、Prefix Search で高速にリスト取得できるようにする。
  - 拡張子やコンテンツタイプを明示し、ライフサイクル ポリシー (クール/アーカイブ) が適用しやすい配置を選ぶ。

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
