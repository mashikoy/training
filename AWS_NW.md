## 概要
AWSの以下サービスについて調べる(随時更新)
 - PrivateLink
 - Route Table
 - VPCエンドポイント
 - マネージドプレフィックス
 - エンドポイント
 - エンドポイントサービス

## PrivateLink
#### どんなサービス?


#### どういう時に使う?

#### 似ているサービスはある?ある場合、どう使い分ける?
 - VPCピアリング
   - このサービスもVPC同士の通信に使える。
   - VPCピアリングは通信が一度インターネットを経由するのに対し、PrivateLinkはAWSのネットワーク内で完結する。<br>セキュリティ要件の厳しい場合にPrivateLinkが選ばれるケースがありそう。
 - Direct Connect
   - AWSユーザの施設やデータセンターからAWS VPCに接続することができる。
   - 違いとしては、

#### セットで使うAWSのサービスはある?
 - 

#### 参考資料
 - AWS公式 https://aws.amazon.com/jp/privatelink/
