# Session05: サーバ編 - Virtual Machine

## Virtual Machine とは
- ハイパーバイザー上で仮想的に分割されたコンピューティング環境。
- 物理ホストとは独立した OS カーネルとファイルシステムを持ち、リソースを柔軟に増減可能。
- 代表的な利用シーン: 既存アプリのクラウド移行、検証環境の即席立ち上げ、オンプレに近い権限が必要なワークロード。

## VM の作成方法の例
- Azure ポータル: GUI でネットワークやサイズを対話的に設定。
- Azure CLI: `az vm create` で再現性の高いデプロイ。例: `az vm create -g rg-azure-school -n demo-vm -l japaneast --image Ubuntu2204 --size Standard_B2s --public-ip-sku Standard --generate-ssh-keys`
- ARM/Bicep: IaC としてテンプレート化し、複数環境へ繰り返しデプロイ。
- Docker Desktop / Multipass などローカル仮想基盤: 開発用の軽量 VM をローカル PC 上に作成し、クラウドに近い挙動を再現可能。

## 常駐型 Python アプリをデプロイする例
- 要件: systemd で自動起動する Python サービスを VM に配置。
- ディレクトリ例: `/opt/myapp` にコードと仮想環境を配置し、`/etc/systemd/system/myapp.service` を作成。

### サービス定義例
```ini
[Unit]
Description=My Python Daemon
After=network.target

[Service]
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/venv/bin/python /opt/myapp/app.py
Restart=always
User=www-data
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

### 初期セットアップ手順 (Ubuntu 想定)
1. パッケージ更新: `sudo apt update && sudo apt install -y python3-venv git`
2. アプリ配置: `sudo git clone <repo> /opt/myapp`
3. 仮想環境作成: `python3 -m venv /opt/myapp/venv && source /opt/myapp/venv/bin/activate`
4. 依存インストール: `pip install -r /opt/myapp/requirements.txt`
5. systemd 登録: `/etc/systemd/system/myapp.service` を作成し、`sudo systemctl daemon-reload && sudo systemctl enable --now myapp`

## Dockerfile で VM 配置を再現する
- VM 上でのセットアップをコンテナビルド手順に落とし込み、再現性と移植性を高める。
- 例: `docker build` したイメージを VM 上で `docker run --restart=always` すれば、デーモン相当の振る舞いを実現。

```Dockerfile
FROM python:3.11-slim
WORKDIR /opt/myapp
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENV PYTHONUNBUFFERED=1
CMD ["python", "app.py"]
```

- VM での起動例: `docker run -d --name myapp -p 8080:8080 --restart=always myorg/myapp:latest`
- これにより systemd に頼らずコンテナの再起動ポリシーで常駐化できる。

## ネットワークとセキュリティのポイント
- NSG で必要最小限のポートのみ開放。SSH は JIT アクセスや IP 制限を併用。
- パブリック IP は Standard SKU を推奨し、DDoS Protection や Azure Bastion も検討。
- OS パッチは自動化し、拡張機能やクラウドイニットで初期設定をコード化。

## 練習問題
1. VM を B 系列と D 系列で作成し、性能と料金を比較するレポートを作る。
2. NSG で 22 番ポートのみ許可し、Just-In-Time アクセスを有効化する手順をまとめる。
3. Custom Script Extension で Python をインストールし、起動時に簡単な HTTP サーバを立ち上げる。
