# 再現手順（提案・未実行）

以下は今後の再実行用です。過去と同一の OS、構成、コマンドを保証しません。実リソース値は非公開メモで管理してください。

## 1. 準備

1. 対象リージョン、予算、AMI / OS、インスタンスタイプ、EBS 容量を選び、料金と無料枠・クレジットの適用条件を確認する。無料とは仮定しない。
2. 学習専用 EC2 を作成する。外部 HTTP 試験にはパブリック IPv4 と Internet Gateway への適切な経路が必要。追加サービスの作成は必要性を確認する。
3. HTTP/TCP 80 の送信元を自分の現在のパブリック IPv4 `/32` に限定する。これは今回の提案値であり、過去の設定値ではない。
4. 接続方法を選ぶ。SSH なら TCP 22 を自分の IP に限定し、秘密鍵を Git 管理外に保管する。Session Manager なら対応する IAM・エージェント・通信経路の前提を確認する。

## 2. Nginx の起動

OS を確認して、該当するブロックだけを実行する。

```sh
cat /etc/os-release
```

Amazon Linux 2023 の例：

```sh
sudo dnf install -y nginx
sudo systemctl enable --now nginx
```

Ubuntu の例：

```sh
sudo apt-get update
sudo apt-get install -y nginx
sudo systemctl enable --now nginx
```

上記以外の OS では、その OS に対応した公式手順を確認する。

```sh
systemctl is-active nginx
sudo nginx -t
curl -sS -o /dev/null -w '%{http_code}\n' --max-time 10 http://127.0.0.1/
```

期待値は `active`、設定検証成功、HTTP `200`。期待値と実測値を区別して記録する。

## 3. 外部接続と遮断試験

1. Mac のブラウザで `http://<PUBLIC_IPV4>/` を開き、表示結果を非公開メモに記録する。HTTPS ではなく HTTP を指定する。
2. 変更前の HTTP 80 ルールのプロトコル・ポート・送信元を非公開で控える。
3. 学習用の HTTP 80 許可だけを一時的に外す。管理接続ルールには触れない。他の関連 Security Group が HTTP を許可していないことも確認する。
4. キャッシュ済み画面で判断せず、新規リクエストで確認する。必要なら Mac で下記を実行し、内部 curl と比較する。

```sh
curl -I --connect-timeout 5 --max-time 10 'http://<PUBLIC_IPV4>/'
```

`<PUBLIC_IPV4>` は実値へ置き換える。公開ファイルには実値を書かない。遮断時の期待値は外部 timeout・内部 200 だが、実際の結果をそのまま記録する。

5. 控えた HTTP 80 ルールを復旧し、新規の外部リクエストで再確認する。広い送信元範囲が残る場合は、試験後に必要最小限へ狭める。
6. 実験終了後は [削除手順](cleanup.md) に進む。
