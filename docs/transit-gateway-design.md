# AWS Transit Gateway ハブ＆スポーク型ネットワーク設計

## 概要
AWS Transit Gateway、Direct Connect、およびInspection VPCを使用したエンタープライズグレードのハブ＆スポーク型ネットワークアーキテクチャの設計です。この設計はAWS Well-Architectedフレームワークのベストプラクティスに準拠し、すべてのトラフィックをNetwork Firewallで検査する高セキュリティ構成を採用しています。

## アーキテクチャ図
![Transit Gateway with Direct Connect and Inspection VPC](generated-diagrams/transit-gateway-dx-inspection.png.png)

## 設計の特徴

### 1. ネットワーク構成
- **Transit Gateway（ハブ）**: すべてのVPCとオンプレミス接続の中央ハブ
- **Inspection VPC**: すべてのトラフィックを検査するNetwork Firewallを集約配置
- **Egress VPC**: インターネット向けトラフィックの出口を集約
- **ワークロード別VPC**: Production、Development、Test環境を分離
- **Security VPC**: セキュリティ監視ツールを集約

### 2. IPアドレス設計
```
Inspection VPC:       10.0.0.0/16
Egress VPC:          10.10.0.0/16
Production VPC:       10.1.0.0/16
Development VPC:      10.2.0.0/16
Test VPC:            10.3.0.0/16
Security VPC:        10.4.0.0/16
```

### 3. セキュリティ設計
- **全トラフィック検査**: Inspection VPCのNetwork Firewallですべての通信を検査
- **ネットワークセグメンテーション**: 環境ごとにVPCを分離
- **マルチAZ冗長性**: Network FirewallエンドポイントをAZ毎に配置
- **監視とコンプライアンス**: Security VPCで集中監視

## ベストプラクティスの実装

### 1. Transit Gateway設計
- **専用サブネット**: 各VPCにTransit Gateway接続用の小さなサブネット（/28）を作成
- **ルートテーブル管理**: 必要な通信のみを許可するルート設計
- **高可用性**: Transit Gatewayは設計上高可用性を提供

### 2. ネットワークACL
- Transit Gateway サブネット用に1つのネットワークACLを作成
- インバウンド/アウトバウンド両方向でオープンに設定
- ワークロードサブネットで適切なACLを適用

### 3. ルーティング設計
- **ルート伝播**: Direct ConnectゲートウェイとBGP Site-to-Site VPN接続で有効化
- **分離ポリシー**: 相互依存のないワークロード間のルートは省略
- **集中管理**: Network Servicesアカウントでハブを管理

### 4. 接続性
- **オンプレミス接続**: Site-to-Site VPN（本番環境ではDirect Connectを推奨）
- **インターネット接続**: Network Services VPC経由で集中管理
- **VPC間通信**: Transit Gateway経由で必要な通信のみを許可

## 実装手順

### 1. 準備
- [ ] ネットワーク設計の確認（CIDR範囲の重複がないことを確認）
- [ ] Network Servicesアカウントの作成（存在しない場合）

### 2. Transit Gatewayの作成
```bash
# Transit Gatewayの作成
aws ec2 create-transit-gateway \
    --description "Central Hub Transit Gateway" \
    --options=AmazonSideAsn=64512,DefaultRouteTableAssociation=disable,DefaultRouteTablePropagation=disable
```

### 3. VPCの作成と接続
```bash
# 各VPCでTransit Gateway接続用サブネットを作成
aws ec2 create-subnet \
    --vpc-id vpc-xxxxx \
    --cidr-block 10.x.255.0/28 \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=TGW-Subnet}]'

# Transit Gateway Attachmentの作成
aws ec2 create-transit-gateway-vpc-attachment \
    --transit-gateway-id tgw-xxxxx \
    --vpc-id vpc-xxxxx \
    --subnet-ids subnet-xxxxx
```

### 4. ルートテーブルの設定
```bash
# Transit Gatewayルートテーブルの作成
aws ec2 create-transit-gateway-route-table \
    --transit-gateway-id tgw-xxxxx \
    --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=Main-Route-Table}]'

# ルートの追加
aws ec2 create-transit-gateway-route \
    --destination-cidr-block 10.1.0.0/16 \
    --transit-gateway-route-table-id tgw-rtb-xxxxx \
    --transit-gateway-attachment-id tgw-attach-xxxxx
```

### 5. セキュリティグループの設定
- Transit Gateway ENI用のセキュリティグループを作成
- 必要なトラフィックのみを許可

### 6. 監視の設定
```bash
# CloudWatchメトリクスの有効化
aws ec2 modify-transit-gateway \
    --transit-gateway-id tgw-xxxxx \
    --options AmazonSideAsn=64512,EnableDnsSupport=enable,EnableMulticast=disable

# VPC Flow Logsの有効化
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-xxxxx \
    --traffic-type ALL \
    --log-destination-type cloud-watch-logs
```

## 運用上の考慮事項

### 1. スケーラビリティ
- Transit Gatewayは最大5,000個のVPC接続をサポート
- 帯域幅は最大50 Gbps（バースト時100 Gbps）

### 2. コスト最適化
- アタッチメントごとに時間単位で課金
- データ処理料金（GB単位）
- 不要なアタッチメントは削除

### 3. 災害復旧
- 各リージョンに1つのTransit Gatewayを配置
- リージョン間ピアリングで冗長性を確保

### 4. トラブルシューティング
- Transit Gateway Network Managerで可視化
- VPC Reachability Analyzerで接続性を検証
- CloudWatch Logsでトラフィックを分析

## セキュリティのベストプラクティス

### 1. 最小権限の原則
- IAMポリシーで必要最小限の権限のみを付与
- ネットワークレベルでもトラフィックを制限

### 2. 暗号化
- VPN接続はIPsecで暗号化
- Direct Connectは専用線（必要に応じてVPN over Direct Connect）

### 3. 監査とコンプライアンス
- AWS CloudTrailですべての変更を記録
- AWS Configで設定の準拠性を監視
- Security Hubで統合的なセキュリティ監視

## まとめ
この設計により、スケーラブルで安全なハブ＆スポーク型ネットワークを実現できます。Transit Gatewayを中心としたアーキテクチャにより、ネットワーク管理の簡素化と運用効率の向上が期待できます。