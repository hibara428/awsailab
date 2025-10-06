# Web App Template Design

## 設計

- 3 層アーキテクチャの Web+App 部
  - VPCやDBは別のテンプレートで作成
  - DBアクセス用のセキュリティグループはそちらで作成したものを指定して利用
- AZ は 2 つ
- サブネットは3種類
  - public-subnet: ALB
  - protected-subnet: Web,App
  - private-subnet: DB

## コンポーネント

- Web
  - EC2 with LaunchTemplate
  - LaunchTemplate with AMI
  - AutoScaling Group
  - IAM インスタンスプロファイル
  - Target Group
  - Security Group
- App
  - EC2 with LaunchTemplate
  - LaunchTemplate with AMI
  - AutoScaling Group
  - IAM インスタンスプロファイル
  - Target Group
  - Security Group
- ALB

### 命名規則プレフィックス

`{System}-{Environment}-`

- System: システム名
- Environment: prod,stg,dev のいずれか

### タグ

- Name: リソースごとの名前
- Environment: dev,stg,prod
- System: システム名
- Project プロジェクト名
- Owner: 部署名
- CostCenter: 4 桁数字
