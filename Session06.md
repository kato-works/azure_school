# Session06: イベント処理編 - Event Hubs

## ポイント
- パーティションとスループットユニットの考え方
- Producer/Consumer のロールとチェックポイント
- Capture で Blob/ADLS にアーカイブする仕組み

## Python サンプル
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

## 練習問題
1. Capture を設定し、Blob に保存された Avro ファイルを Python で読み出して表示する。
2. 消費者を 2 つ起動し、パーティション間で負荷分散される挙動を確認する。
3. Azure Monitor メトリックでスループットユニット消費量を追跡し、アラートを設定する。
