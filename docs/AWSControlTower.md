# Control Tower 学習

- https://dev.classmethod.jp/articles/aws-control-tower-guide/

## 注意

- 既存の Control Tower 化
  - 管理が重複するため AWS Config の無効化が必要
    - Cloud Trail は何も言われなかった
  - 「ランディングゾーンの初期セットアップ時を除き、既存のアカウントを監査アカウントまたはログアーカイブアカウントとして登録することはできません。」
  - https://docs.aws.amazon.com/ja_jp/controltower/latest/userguide//enroll-account.html
    - つまり、初期セットアップなら登録できる？でもどうせ S3 のログ構造は変わってしまうみたいなので何かしら対応は必要そう
- AccountFactory
  - デフォルトで VPC 作る機能はあるが細かいことはできないのでここでは無効化が妥当->リージョンの指定を 0 にすれば良い

## MEMO

- 基本的な機能
  - マルチアカウントにおけるベストプラクティスの作成と、SCP/SecurityHub などの統合ルールセット
- 拡張機能
  - CFCT: オリジナル SCP/StackSets などを追加でき、AccountFactory にも反映
    - AccountFactory 後に起動し、テンプレート展開やポリシー適用などを実行
    - 既存アカウントにも反映可能
    - サンプルを見ると、CFCT でアカウントの細かい設定制御をするイメージではなさそうな感じはする。あくまで追加ポリシー(SCP)、ルール(Config)、必須リソース(IAMrole)といったかなり統制に纏わる部分に限ったほうが良さそう。そもそも他と依存が少ないものでないと結局は無理だよね。
  - AFC: Account Factory Customization
    - Service Catalog を使って Account Factory カスタマイズすることみたい
  - AFT: Account Factory for Terraform
    - Account Factory を Terraform でできる、のでより細かく制御できる面がある

## TODO

ワークショップ：https://catalog.workshops.aws/control-tower/ja-JP

- [x] 準備
  - [x] AWS Config の設定削除 -> StackSets で有効化してたので粛々と削除
  - [x] OU/アカウント追加
- [x] 確認
  - [x] ログ集約
  - [x] 通知
    - [x] lambda を経由して最後に Audit アカウントの SNS から転送
- [x] Control/Guardrail
  - [x] デフォルトの Control
  - [x] コンプライアンスダッシュボード
  - [x] CloudTrail と Config 統合
- [x] Account Factory
- [x] そのほか日本語ない文書を一応さらっと読む
  - [x] Service Catalog 統合(AFC)
    - サービスカタログのポートフォリオを指定する -> サービスカタログを勉強しなきゃ
    - ポートフォリオに AWS Control Tower Account Factory 以外の製品を追加すれば良い
