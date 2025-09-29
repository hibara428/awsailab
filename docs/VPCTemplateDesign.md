# VPC Template Design

## コンポーネント

- VPC /24
  - IPAM Pool 指定
  - （別で作成された）Route 53 Resolver Rule のアタッチメント
- Subnets
  - public subnet /27 x 2AZ
  - protected subnet /27 x 2AZ
  - private subnet /27 x 2AZ
  - TGW subnet /28 x 2AZ
- Route tables
  - public route table
    - VPC 内通信
    - 0.0.0.0/0: IGW へ
  - protected route table
    - VPC 内通信
    - 0.0.0.0/0：TGW subnet へ
  - private route table
    - VPC 内通信
    - 0.0.0.0/0：TGW subnet へ
  - TGW route table
    - VPC 内通信
    - 0.0.0.0/0：TGW へ
- IGW

## 設計指針

- AZ は二つ
- 入口は IGW、出口は TGW 側に転送

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
- CostCenter: 4桁数字

## 参考

https://github.com/aws-cloudformation/aws-cloudformation-templates/blob/main/VPC/VPC_With_Managed_NAT_And_Private_Subnet.yaml
