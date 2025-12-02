# Session04: スケジューラ編 - Automation Account

## ポイント
- Runbook (PowerShell/Python) とスケジュール
- マネージド ID の利用と資格情報の保管
- ハイブリッド Worker でオンプレ操作も可能

## Automation Account の基本操作
### 役割の付与（Managed Identity + RBAC）
1. Automation Account 作成時にシステム割り当てマネージド ID を有効化する。
2. リソース グループや対象リソースに対して「ロールの割り当て」を追加し、`Virtual Machine Contributor` など必要最小限のロールを付与する。
3. Runbook 内では資格情報をコードに埋め込まず、`DefaultAzureCredential` や `Connect-AzAccount -Identity` でマネージド ID を利用する。

### Runbook 作成とスケジュール紐づけ
1. Automation Account > Runbook で新規作成（PowerShell 7.2 / Python 3.10 を選択）。
2. 発行後に「スケジュール」を追加し、実行頻度（例: 毎日 8:00 JST）やタイムゾーンを設定。
3. 複数 Runbook を同じスケジュールに紐づけることも可能。必要に応じてパラメーターを渡す。
4. 「リンクされたロールの割り当て」や「資格情報資産」を確認し、実行権限を担保する。

## PowerShell Runbook サンプル（VM の起動確認）
```powershell
param(
    [string]$SubscriptionId,
    [string]$ResourceGroupName
)

# マネージド ID でログイン
Connect-AzAccount -Identity
Select-AzSubscription -SubscriptionId $SubscriptionId

$vmList = Get-AzVM -ResourceGroupName $ResourceGroupName
foreach ($vm in $vmList) {
    Write-Output "$(Get-Date -Format o): $($vm.Name) status = $((Get-AzVM -Name $vm.Name -ResourceGroupName $ResourceGroupName -Status).Statuses[1].DisplayStatus)"
}
```

## Python Runbook サンプル（リソース グループ列挙）
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

### 利用可能なライブラリと追加方法（Python）
- 既定で `azure-identity`, `azure-mgmt-resource`, `azure-mgmt-compute` など主要な Azure SDK と、標準ライブラリが利用可能。
- 追加が必要な場合は Automation Account > **Python 3 パッケージ** で ZIP/Wheel をアップロードするか、`requirements.txt` を指定して管理パッケージとしてインストールする。
- 外部依存はサイズを抑え、動作確認済みのバージョンを明記する。

## 練習問題
1. 1 日 1 回、停止中 VM を一覧取得しメール通知する Runbook を作る。
2. スケジュール実行と手動実行の両方を試し、ログ出力を確認する。
3. ハイブリッド Worker 経由でオンプレの PowerShell コマンドを実行するパターンを調査する。
