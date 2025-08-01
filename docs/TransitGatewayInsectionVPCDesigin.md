# Transit Gateway Inspection VPC 設計書（Network Firewall 版）

> NOTE: Claude Desktop で聞いたこと

## 1. アーキテクチャ概要

### 1.1 設計思想

- **集約型セキュリティアーキテクチャ**: すべてのトラフィックが Inspection VPC を経由
- **AWS Network Firewall 中心**: マネージドサービスによる運用負荷軽減
- **マルチアカウント対応**: AWS RAM による Transit Gateway 共有
- **高可用性**: 複数 AZ にまたがる Network Firewall 配置

### 1.2 トラフィックフロー

- **East-West**: VPC 間トラフィック
- **North-South**: インターネット ↔ AWS、オンプレミス ↔ AWS

## 2. ネットワーク構成

### 2.1 アカウント構成

#### Network Account（ネットワーク管理アカウント）

- Transit Gateway
- Inspection VPC
- AWS Network Firewall

#### Workload Accounts（ワークロードアカウント）

- Spoke VPC 1, 2, 3...
- アプリケーションサーバー

#### On-premises Connection Account

- Direct Connect Gateway
- VPN 接続

### 2.2 Inspection VPC 詳細設計

```
Inspection VPC (CIDR: 100.64.0.0/16)
├── AZ-a (100.64.1.0/24)
│   ├── TGW Subnet (100.64.1.0/28)
│   ├── Network Firewall Subnet (100.64.1.16/28)
│   └── NAT Gateway Subnet (100.64.1.32/28)
├── AZ-b (100.64.2.0/24)
│   ├── TGW Subnet (100.64.2.0/28)
│   ├── Network Firewall Subnet (100.64.2.16/28)
│   └── NAT Gateway Subnet (100.64.2.32/28)
└── AZ-c (100.64.3.0/24)
    ├── TGW Subnet (100.64.3.0/28)
    ├── Network Firewall Subnet (100.64.3.16/28)
    └── NAT Gateway Subnet (100.64.3.32/28)
```

### 2.3 AWS Network Firewall 構成

#### 配置方式

- **各 AZ に 1 つのファイアウォールエンドポイント**を配置
- **ステートフル検査**によるセッション追跡
- **Suricata IPS**ルールによる高度な検査

#### 主な機能

- Layer 3-7 ファイアウォール
- IPS/IDS（侵入検知・防御）
- URL/ドメインフィルタリング
- 地理的ブロッキング
- 脅威インテリジェンス連携

## 3. Transit Gateway ルートテーブル設計

### 3.1 ルートテーブル構成

#### RT-Spoke（Spoke VPC 用）

```
Destination         Target                     説明
0.0.0.0/0          Inspection-VPC-Attachment  全トラフィックを検査VPCへ
10.0.0.0/8         Inspection-VPC-Attachment  プライベート帯域
172.16.0.0/12      Inspection-VPC-Attachment  プライベート帯域
192.168.0.0/16     Inspection-VPC-Attachment  プライベート帯域
オンプレミスCIDR    Inspection-VPC-Attachment  オンプレミス宛
```

#### RT-Inspection（Inspection VPC 用）

```
Destination            Target                     説明
10.1.0.0/16           Spoke-VPC-1-Attachment     Spoke VPC 1宛
10.2.0.0/16           Spoke-VPC-2-Attachment     Spoke VPC 2宛
10.3.0.0/16           Spoke-VPC-3-Attachment     Spoke VPC 3宛
オンプレミスCIDR       DX-Gateway-Attachment      オンプレミス宛
0.0.0.0/0            <propagated>               インターネット宛
```

#### RT-OnPrem（オンプレミス接続用）

```
Destination         Target                     説明
10.0.0.0/8         Inspection-VPC-Attachment  AWS宛トラフィック
172.16.0.0/12      Inspection-VPC-Attachment  AWS宛トラフィック
192.168.0.0/16     Inspection-VPC-Attachment  AWS宛トラフィック
```

### 3.2 Appliance Mode 設定

**重要**: Inspection VPC の Transit Gateway Attachment で Appliance Mode を有効化

```bash
aws ec2 modify-transit-gateway-vpc-attachment \
  --transit-gateway-attachment-id tgw-attach-xxxxxxxxx \
  --options ApplianceModeSupport=enable
```

## 4. VPC ルートテーブル設計

### 4.1 Inspection VPC 内ルーティング

#### TGW Subnet Route Table

```
Destination                    Target
100.64.1.16/28 (NFW Subnet)   Network-Firewall-Endpoint-AZ-a
100.64.1.32/28 (NAT Subnet)   Network-Firewall-Endpoint-AZ-a
0.0.0.0/0                     Transit-Gateway
```

#### Network Firewall Subnet Route Table

```
Destination         Target
100.64.1.32/28     local (NAT Gateway Subnet)
0.0.0.0/0          Transit-Gateway
```

#### NAT Gateway Subnet Route Table

```
Destination         Target
100.64.0.0/16      local
0.0.0.0/0          Internet-Gateway
```

### 4.2 Spoke VPC ルーティング

#### Application Subnet Route Table

```
Destination         Target
10.x.0.0/16        local
0.0.0.0/0          Transit-Gateway
```

## 5. AWS Network Firewall 設定

### 5.1 ファイアウォールポリシー構成

#### Stateless Rule Groups（高優先度・高速処理）

```yaml
Priority: 1000
Rules:
  - Source: Any
    Destination: Any
    Protocol: TCP
    Ports: 80, 443
    Action: PASS

Priority: 2000
Rules:
  - Source: Any
    Destination: Any
    Protocol: ICMP
    Action: PASS

Priority: 3000
Rules:
  - Source: Any
    Destination: Any
    Protocol: Any
    Action: FORWARD_TO_STATEFUL
```

#### Stateful Rule Groups（詳細検査）

##### Domain Filtering

```yaml
Domain-List:
  Allow:
    - company.com
    - *.amazonaws.com
    - *.amazon.com
  Block:
    - *.malware-domain.com
    - *.phishing-site.com
```

##### Suricata IPS Rules

```yaml
Rules:
  # マルウェア検知
  - 'alert http any any -> any any (msg:"Malware C2 Communication"; content:"malware"; sid:1000001;)'

  # SQLインジェクション検知
  - 'alert http any any -> any any (msg:"SQL Injection Attempt"; content:"union select"; nocase; sid:1000002;)'

  # 異常なアップロード検知
  - 'alert http any any -> any any (msg:"Large File Upload"; content:"Content-Length"; fast_pattern; pcre:"/Content-Length:\s*[0-9]{8,}/i"; sid:1000003;)'

  # 地理的ブロッキング
  - 'drop ip [!$HOME_NET] any -> $HOME_NET any (msg:"Block traffic from restricted countries"; geoip:src,CN,RU,KP; sid:1000004;)'
```

##### Custom Rules

```yaml
Rules:
  # 業務時間外アクセス制御
  - Source: Any
    Destination: 10.1.0.0/16
    Protocol: TCP
    Ports: 22, 3389
    Time: "weekdays 09:00-18:00"
    Action: ALLOW

  # 開発環境への本番アクセス制限
  - Source: 10.1.0.0/16 # Production VPC
    Destination: 10.3.0.0/16 # Development VPC
    Protocol: Any
    Action: DROP
```

### 5.2 ログ設定

#### CloudWatch Logs

```yaml
Log-Destination-Type: CloudWatch
Log-Group: /aws/networkfirewall/flow
Log-Type: FLOW
Log-Format: JSON
```

#### S3 バケット

```yaml
Log-Destination-Type: S3
S3-Bucket: security-logs-bucket
Log-Type: ALERT
Prefix: network-firewall/alerts/
```

## 6. 冗長性・可用性設計

### 6.1 Multi-AZ 配置

- **Network Firewall エンドポイント**を 3 つの AZ に配置
- AZ 障害時の自動フェイルオーバー
- **Appliance Mode**による対称ルーティング保証

### 6.2 スケーリング

- **自動スケーリング**: トラフィック負荷に応じて自動拡張
- **パフォーマンス**: AZ あたり最大 100Gbps のスループット
- **容量計画**: 数万のファイアウォールルールをサポート

## 7. 運用監視設計

### 7.1 CloudWatch メトリクス

```yaml
主要メトリクス:
  - NetworkFirewall.PacketsDropped
  - NetworkFirewall.PacketsPassed
  - NetworkFirewall.RuleEvaluations
  - TransitGateway.PacketDropCount
  - TransitGateway.BytesIn/BytesOut
```

### 7.2 アラート設定

```yaml
Critical-Alerts:
  - ファイアウォール障害
  - 大量のパケットドロップ
  - 異常トラフィック検知
  - セキュリティルール違反

Warning-Alerts:
  - 高いCPU使用率
  - 帯域幅使用量の増加
  - 新しい脅威パターン検知
```

### 7.3 ダッシュボード

- **リアルタイム監視**: トラフィックフロー可視化
- **セキュリティイベント**: 脅威検知状況
- **パフォーマンス**: レイテンシ・スループット
- **コスト**: データ処理量・コスト推移

## 8. 実装手順

### 8.1 Phase 1: 基盤構築（週 1-2）

1. Network Account 内に Transit Gateway 作成
2. Inspection VPC 作成・サブネット配置
3. TGW Attachment で Appliance Mode 有効化
4. 基本ルーティング設定

### 8.2 Phase 2: Network Firewall 展開（週 3-4）

1. ファイアウォールポリシー作成
2. 各 AZ に Firewall エンドポイント配置
3. ルートテーブル更新
4. 基本テスト実施

### 8.3 Phase 3: Spoke VPC 接続（週 5-8）

1. 各 Workload Account で Spoke VPC 作成
2. RAM 経由で Transit Gateway 共有・アクセプト
3. 段階的にトラフィック移行
4. セキュリティポリシー調整

### 8.4 Phase 4: オンプレミス接続（週 9-10）

1. Direct Connect Gateway 接続
2. VPN Backup 接続
3. ルーティング最終調整
4. 運用監視体制確立

## 9. セキュリティポリシー例

### 9.1 ゾーンベースセキュリティ

```yaml
Security-Zones:
  Production:
    VPCs: [10.1.0.0/16]
    Rules: restrictive

  Development:
    VPCs: [10.3.0.0/16]
    Rules: permissive

  DMZ:
    VPCs: [10.2.0.0/16]
    Rules: moderate

Inter-Zone-Rules:
  Production-to-Development: DENY
  Development-to-Production: DENY
  DMZ-to-Production: CONDITIONAL_ALLOW
```

### 9.2 アプリケーション別制御

```yaml
Web-Tier:
  Allowed-Ports: [80, 443]
  Inspection: DEEP_PACKET_INSPECTION

App-Tier:
  Allowed-Ports: [8080, 8443]
  Source-Restriction: WEB_TIER_ONLY

DB-Tier:
  Allowed-Ports: [3306, 5432]
  Source-Restriction: APP_TIER_ONLY
  Encryption: REQUIRED
```

## 10. コスト最適化

### 10.1 コスト要素

- **Transit Gateway**: $36/月 + データ処理 $0.02/GB
- **Network Firewall**: エンドポイント $0.395/時間 + データ処理 $0.065/GB
- **NAT Gateway**: $32.85/月 + データ処理 $0.045/GB
- **CloudWatch Logs**: $0.50/GB（保存）+ $0.03/GB（取り込み）

### 10.2 最適化手法

- **選択的検査**: 信頼できるトラフィックの除外
- **ログローテーション**: 古いログの自動削除
- **Reserved Capacity**: 予測可能な使用量での割引適用
- **効率的ルーティング**: 不要な検査パスの排除

この設計により、AWS Network Firewall を中心としたシンプルで効果的な集約型セキュリティアーキテクチャを実現できます。マネージドサービスの利点を活かしながら、包括的なネットワークセキュリティを提供します。
