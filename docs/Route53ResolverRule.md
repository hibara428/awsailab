# ResolverRule+OutboundEndpoint

## 概要

- 以下の構成案で TGW なしで名前解決できるか確認したい
- 構築と動作確認の手順を整理したい

## 構成

- Account-A
  - VPC-A
    - DNS サーバ(dnsmasq)
    - Route 53 Outbound Endpoint
  - Global
    - Route 53 Resolver Rule
      - (Account B へ RAM 共有)
- Account-B
  - VPC-B
    - (Account-A: Route 53 Resolver Rule)

## 構築手順

### 前提

- RAM で組織共有する場合には、管理アカウントで RAM の Organization 連携設定が必要そう

### Account-A

#### VPC-EC2

- VPC 作成
  - AZ2, public + private
  - IGW, NAT-GW
- SecurityGroup 作成
  - DNS サーバ用: 53/tcp,udp VPC-CIDR
  - VPCE: 443/tcp VPC-CIDR
- VPCE
  - ssm,ssmmessages,ec2messages
- EC2 用 IAM ロール作成
  - AmazonSSMManagedInstanceCore
- EC2 作成 -> 10.1.141.115
  - IAM ロール、SecGroup 付与
  - EIP 割り当て
  - SSM 接続確認
  - dnsmasq 設定

#### Resolver

- Resolver 用 SecurityGroup 作成
  - 53/tcp,udp
  - (一旦 0.0.0.0/0 で作成)
- Outbound Endpoint -> 10.1.9.193, 10.1.21.120
  - SecGroup 付与
- Resolver Rule
  - TargetIP アドレスは DNS サーバの IP アドレスを指定
  - RAM 共有

#### 確認

- DNS サーバ自身から VPC 内名前解決確認
  - ✅localhost 指定
  - ❌outbound endpoint 指定 -> そもそも直接 dig はできない
- CloudShell(withVPC) で VPC 内名前解決確認
  - CloudShell with VPC の作成
    - Name, VPC, Subnet, Security Group
    - (public IP が付与できないため、private-subnet x NAT-GW の構成のみ)
  - ✅EC2 の privateIP 指定
    - `pip3 install --user dnspython`
    - `python3 -c "import dns.resolver; r=dns.resolver.Resolver(); r.nameservers=['10.1.141.115']; print(r.resolve('test.local', 'A')[0])"`
  - ❌outbound endpoint 指定 -> そもそも直接 dig はできない

### Account-B

- VPC 作成
  - NAT-GW
  - ResolverRule をDNSに割り当て
- CloudShell から名前解決テスト
  - CloudShell with VPC の作成
    - Name, VPC, Subnet, Security Group
  - ✅ただ名前解決
    - `getent test.local`
