# Session05: サーバ編 - Virtual Machine (PowerShell)

## ポイント
- VM サイズ選定、ネットワーク (NSG, パブリック IP) の基本
- Azure CLI / PowerShell での VM 作成と初期設定
- 拡張機能 (Custom Script Extension) での自動セットアップ

## PowerShell サンプル (Cloud Shell 想定)
```powershell
$rg = "rg-azure-school"
$loc = "japaneast"
$vmName = "demo-vm"

New-AzResourceGroup -Name $rg -Location $loc
New-AzVm `
  -ResourceGroupName $rg `
  -Name $vmName `
  -Location $loc `
  -Image "Win2022Datacenter" `
  -VirtualNetworkName "$vmName-vnet" `
  -SubnetName "default" `
  -SecurityGroupName "$vmName-nsg" `
  -PublicIpAddressName "$vmName-ip" `
  -OpenPorts 80,3389

# IIS をインストールしてテストページを作成
Set-AzVMExtension -ResourceGroupName $rg -VMName $vmName -Name "iis-setup" -Publisher "Microsoft.Compute" -Type "CustomScriptExtension" -TypeHandlerVersion "1.10" -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; Set-Content -Path C:\\inetpub\\wwwroot\\index.html -Value \"Hello from Azure VM!\""}'
```

## 練習問題
1. VM を B 系列と D 系列で作成し、性能と料金を比較するレポートを作る。
2. NSG で 22 番ポートのみ許可し、Just-In-Time アクセスを有効化する手順をまとめる。
3. Custom Script Extension で Python をインストールし、起動時に簡単な HTTP サーバを立ち上げる。
