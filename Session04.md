# Session04: スケジューラ編 - Automation Account

## ポイント
- Runbook (PowerShell/Python) とスケジュール
- マネージド ID の利用と資格情報の保管
- ハイブリッド Worker でオンプレ操作も可能

## Python Runbook サンプル
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

## 練習問題
1. 1 日 1 回、停止中 VM を一覧取得しメール通知する Runbook を作る。
2. スケジュール実行と手動実行の両方を試し、ログ出力を確認する。
3. ハイブリッド Worker 経由でオンプレの PowerShell コマンドを実行するパターンを調査する。
