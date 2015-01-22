# Mackerelを中心に考える2015年代のサービス運用環境

id:myfinder

___

# 自己紹介
- 久森 達郎
- <span style="color: red">エム・ティー・バーン</span>株式会社
 - サーバサイド / Android

---

# 今日話すこと
- <span style="color: red">"どオンプレ環境"</span>で作ったサービスを最近 AWS に移設しました
- Mackerelを中心に運用環境を<span style="color: red">大きく</span>変えました

___

# 対象のサービス
## http://mtburn.jp/

___

## メディア企業の皆様
# どしどし<span style="color: red">お申込み</span>
# ください

---

# 移行前の環境
オンプレ+S3/CloudFront(とか他のCDN)

___

# 移行前の環境
- <span style="color: red">Single Segment</span>
- データセンター提供のマネージドDNS
- 箱物ロードバランサ / 大手メーカー製サーバ / 一部自作機
- <span style="color: red">CentOS</span> 6.x / cobbler / puppet / RackTables
- <span style="color: red">Hadoop 担当</span>が構築 / 運用
- <span style="color: red">MySQL/Redis 担当</span>が構築 / 運用
- Nagios / CloudForecast / GrowthForecast
- 足元のディスクに吐いたログを<span style="color: red">でっかいサーバに収集</span>して集計

___

# よくある消耗しがちなオンプレ環境でした

---

# 抱えていた懸案
## <span style="color: red">__"調達"__</span>と<span style="color: red">__"実装スピード"__</span>

___

## <span style="color: red">急速</span>に立ち上がっている状況
![high-growth](img/high-growth.png)

___

## 課題
- サーバの性能が箱で規定されるので、複雑なプロビジョニングを必要とする
- 新規リソースの調達リードタイムが大きい
- キャパシティを見切る事が難しい

___

## 解決するには？
- クラウドへ移行すれば<span style="color: red">半分</span>解決できる
- 残りの半分は？

___

## 考え方/仕組みを変える
- マネージドを積極的に活用する
- 運用の考え方を180度変える必要がある

___

## 変えた点
- Mackerelをサーバ管理の中心にする
 - Nagios/CF/GF/Puppet/RackTables -> Mackerel
- マネージドサービスの活用
 - Route53/ELB/EC2 AutoScaling/RDS/ElastiCache
 - BigQuery
- ツール/オペレーションの改善
 - ログ収集方法の変更
 - Puppet廃止、Packer採用
 - デプロイ方法の変更

___

# つまり
# アプリ以外<span style="color: red">全部</span>
# 変えた

---

## Mackerelをサーバ管理の中心に
- マネージドでサーバリストメンテが不要になる
- Roleベースでの管理/グラフの継続
- サービスメトリックが捗る
- 開発者が身近

___

# グラフの<span style="color: red">連続性</span>

___

## roleベースでグラフは<span style="color: red">継続</span>
![appnodes](img/appnodes.png)

___

# CloudWatch連携

___

## 公式の手順で<span style="color: red">カジュアル</span>
![cloudwatch](img/cloudwatch.png)

___

# Norikraの活用

___

## fluent-plugin-(norikra|mackerel)でOK
![norikra](img/norikra.png)

___

## <span style="color: red">分速</span>で売上/支払を見る
![growth](img/agency-media.png)

___

# Mackerel + Slack
![mackerel_notify](img/mackerel_notify.png)

<span style="color: red">携帯がアラートメールでうめつくされる</span>みたいなのもなくなった

---

# マネージドサービスの活用

___

## Route53/RDS/ElastiCache
- 内部DNSもRoute53で扱えるようになったので移行
 - 但しほとんどのサーバには名前をつけていないのであまり必要なくなった
- MySQLやRedisは自前運用しない
 - フェイルオーバーやバックアップを勝手にやってくれたほうがいい

___

## ELB/EC2 AutoScaling
- MackerelとAutoScalingは先述のとおり相性良好

___

## BigQuery
- Hiveと比較してコスト/速度/スケーラビリティすべてで勝っている
- もはや使わない理由がない

---

# ツール/オペレーションの改善

___

## ログ収集方法の変更
- 足元のストレージに吐いたログをscpとかで収集して集計していた
- fluentdでS3/BigQueryへ収集するよう変更
- fluentd自体のログもslackへ投げてトラブル検知

___

## Puppet廃止、Packer採用
- EC2ではインスタンスタイプで必要なリソースを必要な機能単位で割り当てられる
- オンプレのように箱の性能に合わせたプロビジョニングが不要になる
- Packer + ShellScript で AMI を作ってしまえば十分

___

## デプロイ方法の変更
- S3を介したpull型deployへ
 - 種サーバでビルドしたものをS3に同期
 - 各サーバはS3から取得する
 - 操作自体はCapistranoのMackerel連携で十分
 - AutoScalingする際はcloud-initで取得処理を走らせる

---

# 結果
- Mackerelを中心にサービス運用を考えた結果
 - 結果としてInfrastructure as Codeを実践することになった
 - 労働集約的な仕事から開放された
 - 仕事の質をアップさせ、更に新しいことに取り組めるようになった

---

# 質疑など?

---

# おわり
