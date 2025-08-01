# AWS Organizations

## OU 設計

### MEMO

先に Foundation から作る。

- Admin
- [Foundation]
  - [Infrastructure]
    - [Prod]
      - Network(Inspection)
      - Shared Infra Service Prod
    - [SLDC]
      - XXXX Dev
  - [Security]
    - [Prod]
      - Audit/Security Tooling(SecurityHub や GuardDuty など)
      - Log Archive(RO ログ)
      - Security ReadOnly Access(踏み台アカウント:RO)
      - Security Breakglass(踏み台アカウント:RW)
    - [SLDC]
      - XXXX Dev
- [Sandbox]
  - (実験などの sandbox 用途)
- [Workloads]
  - [Prod]
    - Service Prod
  - [SDLC]
    - XXXX Dev
- [PolicyStaging]
  - (OU へ反映するポリシーテスト用)
- [Suspended]
  - (完全に削除されるまで 90 日間待機するアカウント)
- [Transitional]
  - 移行や再構築などの一時的なプロセス用
- [IndivisualBusinessUsers]
  - (ビジネスユーザアクセス用とのことだけど・・・OU にまでする？)
- [Exceptions]
  - (権限管理上で例外的なプロジェクトなど)
- [Deployments]
  - (なかなか大変そうだが・・・たぶん SOX 法や ISO 関連で分離したいんだろうなー。あと複数アカウントに紐づくようなデプロイメントを想定)
- [BusinessContinuity]
  - (ビジネス継続性用途の特殊なリソース群)

## ポリシー設計

- SCP / RCP
  - 「これは絶対にやってはいけない」を定義、例外なしの強制ルール
  - IAM ポリシーみたいなフォーマットで書く
  - SCP がアクセス権限全般
  - RCP が各リソース作成時の属性や設定などのルール
- Tag policy
  - タグの利用ポリシーをレポート or 強制できる
  - アカウントや OU にポリシーを付与する形
  - リソースに対して何をどんなタグを強制するか指定
- Backup policy
  - Organization 単位の Backup policy 設定が各アカウントの AWS Backup に反映される形
- EC2 declative policy
  - 設置値を強制させることができる（SCP とかだとエラーになるだけ）
  - 今のところ EC2,VPC,EBS。増強予定。
  - (2025/7 時点にある内容を見ると、まあ別に SCP/RCP でも良いようなものが多いかなあ)
