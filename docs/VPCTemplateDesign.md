# VPC Template Design

## コンポーネント

- VPC
  - IPAM Pool 指定
  - （別で作成された）Route 53 Resolver Rule のアタッチメント
- Subnets
  - public subnet x 2
  - protected subnet x 2
  - private subnet x 2
  - TGW subnet x 2
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

## TODO

タグ
CIDR 分割

## 参考

https://github.com/aws-cloudformation/aws-cloudformation-templates/blob/main/VPC/VPC_With_Managed_NAT_And_Private_Subnet.yaml
