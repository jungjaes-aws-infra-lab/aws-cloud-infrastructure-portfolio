# EC2・EBS・パブリック IPv4・費用の整理

以下は実施用チェックリストです。削除・請求確認は未実施・未確認であり、チェック済みとして公開しません。対象リージョン、リソースの所有用途、必要データを確認してから操作します。

## 削除前

- [ ] 学習対象と他用途のリソースを区別する。識別子は非公開メモに保存する。
- [ ] 必要なデータを退避する。終了時削除対象の EBS データは復元できない。
- [ ] 各 EBS の「終了時に削除 / DeleteOnTermination」を確認する。
- [ ] Elastic IP の割り当て有無、スナップショット、作成した AMI を確認する。

## EC2

- [ ] 再開不要なら EC2 コンソールで対象を選び、「インスタンスの状態」→「終了（削除）」を実行する。終了保護がある場合は対象を再確認して解除する。
- [ ] 状態が `terminated` になったことを確認する。
- [ ] 一時休止なら `stopped` を確認する。ただし停止だけでは EBS 等の費用はなくならない。

## EBS・スナップショット

- [ ] 終了後、対象 EBS が削除されたか個別に確認する。自動削除を前提にしない。
- [ ] 残った不要な学習用ボリュームだけを削除する。`available` という状態だけで不要と判断しない。
- [ ] 不要な学習用スナップショットを確認する。自作 AMI が参照する場合は、AMI の登録解除と関連スナップショットの削除をそれぞれ確認する。

## パブリック IPv4

- [ ] 自動割り当てのパブリック IPv4 が停止・終了後に解放されたか確認する。
- [ ] 学習用 Elastic IP があれば、関連付け解除だけで終わらず、不要な割り当てを「解放」する。他用途で使用中のものは解放しない。
- [ ] 使用中・未使用のパブリック IPv4 はともに料金確認の対象とする。無料枠やクレジットの適用はアカウントで確認する。

## 最終確認

- [ ] 実習で触れたすべてのリージョンで EC2、EBS、Elastic IP、スナップショットの残存を確認する。
- [ ] 追加作成していた場合のみ、ロードバランサー、NAT Gateway、VPC エンドポイント、CloudWatch ログ等も確認する。これらの作成実績は未確認。
- [ ] 不要な実習専用 Security Group・キーペアを確認する。共有リソースやデフォルト VPC を一括削除しない。
- [ ] Billing の請求書 / Cost Explorer で期間・リージョン・サービス・使用タイプを確認し、EC2 計算、EBS、スナップショット、パブリック IPv4、データ転送を確認する。
- [ ] 請求データの反映後に再確認する。削除前の使用料は残るため、累計額が 0 になることを完了条件にしない。
- [ ] 非公開メモに確認日時・残存資源・最終確認状況を記録する。予算通知は確認の補助であり、自動的な支出停止を保証しない。

停止で計算料金が止まっても EBS は課金対象になり得ます。終了後の EBS は DeleteOnTermination に依存し、Elastic IP は関連付け解除後も残ります。

公式資料（2026-11-13 確認）：

- [EC2 のライフサイクル](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)
- [終了時のリソース動作](https://docs.aws.amazon.com/us_en/AWSEC2/latest/UserGuide/how-ec2-instance-termination-works.html)
- [EC2 の終了手順](https://docs.aws.amazon.com/us_en/AWSEC2/latest/UserGuide/terminating-instances.html)
- [VPC・パブリック IPv4 料金](https://aws.amazon.com/vpc/pricing/)
- [残存リソースと費用の確認](https://repost.aws/knowledge-center/delete-terminate-ec2)
