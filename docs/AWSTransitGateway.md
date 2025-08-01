# Transit Gateway

https://catalog.us-east-1.prod.workshops.aws/workshops/e0d1c19d-c80b-4695-a3fc-5c4a25132f47/ja-JP/

## MEMO

- Transit Gateway を共有することでアカウント間ピアリング

## Workshop

- Network: wstgw-lab1
- ServiceProd: wstgw-lab2

- アプライアンスモード有効化
- A-VPC -> Inspection-VPC -> B-VPC
- B-VPC -> Inspection-VPC -> A-VPC

- TODO: 以下の構成の Peering がどういう意味？
- https://catalog.us-east-1.prod.workshops.aws/workshops/e0d1c19d-c80b-4695-a3fc-5c4a25132f47/ja-JP/1-introduction/1-3-design

privatevpc1: tgw-attach-05f056c715d6fde99
privatevpc2: tgw-attach-05d8d52c364f782a4
