# Azure クラウド初心者向け 7 回講座

各回 90 分想定。基本の概念説明 → 5〜10 分のデモ → ハンズオン課題の流れで進行します。Azure ポータル操作のスクリーンショットは講師側で補完してください。

## セッション資料
- [Session01: データベース編 (Storage Account)](./Session01.md)
- [Session02: 関数編 (Function App)](./Session02.md)
- [Session03: サーバ編 (App Service)](./Session03.md)
- [Session04: スケジューラ編 (Automation Account)](./Session04.md)
- [Session05: サーバ編 (Virtual Machine)](./Session05.md)
- [Session06: イベント処理編 (Event Hubs)](./Session06.md)
- [Session07: デバイス管理編 (IoT Hub)](./Session07.md)

## 付録: 環境準備メモ
- 受講者は Azure 無料アカウント、Azure CLI、Python 3.10 以降を事前インストール。
- Python クライアントライブラリ例: `pip install azure-storage-blob azure-storage-queue azure-data-tables azure-functions azure-eventhub azure-identity azure-mgmt-resource azure-iot-device azure-iot-hub`
- VS Code の Azure 拡張と Azure Functions 拡張をインストールしておくとデモが容易。
