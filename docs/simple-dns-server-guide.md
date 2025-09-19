# AWS上で最も簡単にDNSサーバーを構築する方法

## 📌 概要
運用を考慮せず、AWS上で最速・最小限の設定でDNSサーバーを立ち上げる方法です。
**dnsmasq**を使用することで、数分でDNSサーバーが稼働します。

## 🚀 方法1: Pythonスクリプトによる自動構築（推奨）

最も簡単な方法です。EC2インスタンスの作成から設定まで自動化されています。

```bash
# 実行方法
uv run python scripts/deploy_dns_server.py --key-name your-key-pair-name

# オプション指定
uv run python scripts/deploy_dns_server.py \
  --key-name your-key-pair \
  --instance-type t3.micro \
  --vpc-id vpc-xxxxx \
  --subnet-id subnet-xxxxx
```

### 実行後の出力例
```
🚀 DNSサーバーの構築を開始します...
✓ セキュリティグループ作成完了: sg-xxxxxxxxx
✓ AMI ID: ami-xxxxxxxxx
✓ VPC ID: vpc-xxxxxxxxx
✓ Subnet ID: subnet-xxxxxxxxx
✓ インスタンスを起動中: i-xxxxxxxxx
✓ Elastic IP割り当て完了: 52.x.x.x

✅ DNSサーバーの構築が完了しました！

============================================================
📋 サーバー情報:
  インスタンスID: i-xxxxxxxxx
  DNSサーバーIP: 52.x.x.x
  プライベートIP: 172.31.x.x

🔧 接続・テストコマンド:
  SSH接続: ssh -i your-key.pem ec2-user@52.x.x.x
  DNSテスト: dig @52.x.x.x google.com
  ステータス確認: ssh -i your-key.pem ec2-user@52.x.x.x './check_dns.sh'

💡 使用例:
  dig @52.x.x.x example.com
  nslookup google.com 52.x.x.x
============================================================
```

## 🛠️ 方法2: 既存EC2への手動インストール（30秒）

既にEC2インスタンスがある場合の最速セットアップ：

```bash
# ワンライナーでセットアップ
./scripts/quick_dns_setup.sh <EC2_IP> <KEY_FILE>

# または手動でSSH接続して実行
ssh -i your-key.pem ec2-user@your-ec2-ip
sudo yum install -y dnsmasq
sudo systemctl start dnsmasq
```

## 🎯 方法3: 最小限の手動構築

### 1. EC2インスタンス起動（AWS Console経由）
- AMI: Amazon Linux 2023
- インスタンスタイプ: t3.micro
- セキュリティグループ:
  - SSH (22): 0.0.0.0/0
  - DNS (53/UDP): 0.0.0.0/0
  - DNS (53/TCP): 0.0.0.0/0

### 2. SSHでログイン＆セットアップ
```bash
# dnsmasqインストール
sudo yum install -y dnsmasq

# 最小設定（/etc/dnsmasq.conf）
echo "server=8.8.8.8" | sudo tee /etc/dnsmasq.conf

# 起動
sudo systemctl start dnsmasq
```

## ✅ DNSサーバーのテスト

```bash
# ローカルからテスト
dig @<EC2_PUBLIC_IP> google.com

# 応答例
;; ANSWER SECTION:
google.com.     300     IN      A       142.250.x.x

# nslookupでもテスト可能
nslookup example.com <EC2_PUBLIC_IP>
```

## 🔧 カスタムDNSレコードの追加

独自のDNSレコードを追加したい場合：

```bash
# /etc/dnsmasq.conf に追加
address=/myapp.local/192.168.1.100
address=/test.example.com/10.0.0.50

# 設定反映
sudo systemctl restart dnsmasq
```

## 📊 メリット・デメリット

### メリット
- ✅ **最速構築**: 3分以内に稼働
- ✅ **最小設定**: 設定ファイル1行から動作
- ✅ **軽量**: メモリ使用量極小（10MB程度）
- ✅ **キャッシュ機能**: 自動でDNSキャッシュ

### デメリット
- ❌ 冗長性なし（単一障害点）
- ❌ 監視・ログ機能が最小限
- ❌ 大規模環境には不向き
- ❌ セキュリティ設定が基本的

## 💰 コスト目安

- t3.micro: 約$8/月
- Elastic IP: 約$3.6/月（使用時は無料）
- データ転送: 最初の100GBは無料

**月額合計: 約$12（約1,800円）**

## 🗑️ リソースの削除

```bash
# インスタンスの終了
aws ec2 terminate-instances --instance-ids i-xxxxxxxxx

# セキュリティグループの削除
aws ec2 delete-security-group --group-id sg-xxxxxxxxx

# Elastic IPの解放
aws ec2 release-address --allocation-id eipalloc-xxxxxxxxx
```

## 📝 注意事項

1. **本番環境では非推奨** - あくまで検証・開発環境用
2. **セキュリティ** - 必要に応じてアクセス制限を追加
3. **バックアップ** - 設定ファイルのバックアップを推奨
4. **監視** - CloudWatchでの基本監視を推奨

## 🔍 トラブルシューティング

```bash
# dnsmasqの状態確認
sudo systemctl status dnsmasq

# ログ確認
sudo journalctl -u dnsmasq -n 50

# ポート53が開いているか確認
sudo netstat -tuln | grep :53

# DNSクエリのデバッグ
dig @localhost google.com +trace
```

## 🚀 次のステップ

より本格的なDNSサービスが必要な場合：

1. **Route 53 Resolver** - マネージドDNSサービス
2. **BIND9** - フル機能のDNSサーバー
3. **PowerDNS** - 高性能DNSサーバー
4. **CoreDNS** - クラウドネイティブDNS