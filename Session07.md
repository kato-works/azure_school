# Session07: デバイス管理編 - IoT Hub

## ポイント
- IoT Hub とデバイス ID の登録、接続文字列/証明書
- Device-to-Cloud (D2C) / Cloud-to-Device (C2D) メッセージ
- Device Twin による設定管理とデバイス更新

## Python サンプル
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

## 練習問題
1. Device Twin に desired property として `samplingInterval` を設定し、デバイス側で適用するコードを書く。
2. IoT Hub のメッセージルーティングで、温度が 30℃ 以上のメッセージを Event Hub に送るルールを作成する。
3. Azure Stream Analytics を利用して IoT Hub のデータを Power BI へリアルタイム表示するパイプライン図を描き、設定手順をまとめる。
