※※※VPCエンドポイント、マネージドプレフィックス、エンドポイント、エンドポイントサービスは実機演習をしないとよくわからなかった。
キャプチャやコマンドを貼りたいので現在Qiitaで記事を書いている。後でリンクはる。

## 概要
AWSの以下サービスについて調べる(随時更新)
 - PrivateLink
 - RouteTable
 - VPCエンドポイント
 - マネージドプレフィックス
 - エンドポイント
 - エンドポイントサービス

## PrivateLink
#### どんなサービス?
 - VPCに構築したシステムから外部のVPCやAWSサービスに接続する場合、AWS内のネットワークのみを使用して通信できるサービス

#### どういう時に使う?
 - セキュリティ要件が厳しい場合(PCIDSSとか?)

#### 似ているサービスはある?ある場合、どう使い分ける?
 - VPCピアリング
   - このサービスもVPC同士の通信に使える。
   - VPCピアリングは通信が一度インターネットを経由するのに対し、PrivateLinkはAWSのネットワーク内で完結する。<br>セキュリティ要件の厳しい場合にPrivateLinkが選ばれるケースがありそう。

#### セットで使うAWSのサービスはある?
 - Direct Connect
   - AWSユーザの施設やデータセンターからAWS VPCへ専用線を使って接続することができる。<br>ISP等と契約して専用線を使用するので、維持費用が高い。
   - PrivateLinkはVPC~外のVPCやAWSサービスとの通信をセキュアにする。そのためVPCまでの通信はDirect Connectで安全を保つようなケースもあるらしい。

#### 参考資料
 - AWS公式 https://aws.amazon.com/jp/privatelink/

## RouteTable
#### どんなサービス?
 - ルータの持っているルーティングテーブルと同じようなもの。<br>VPCには暗黙的(?)なルーターがあり、そのルーターはこのルートテーブルを見て経路選択する。<br>経路選択時に適用される経路はルータと同様ロンゲストマッチ
   - 暗黙的: AWSユーザからは意識されない(される必要がない)、のような意味合い?
 - テーブルは以下のような種類がある
   - メインルートテーブル
     - VPC内のサブネットにルートテーブルが1つも割り当てられていない場合に自動で割り当てられるルートテーブル。(デフォルト的なテーブル)
     - VPCを作ると勝手に作られる。
     - サブネットからは明示的にメインルートテーブルを紐づけることもできる。
   - カスタムルートテーブル
     - 例えばインターネットに接続できるpublicなネットワークとlocalなネットワークを作りたい場合、<br>それそれのカスタムルートテーブルを作ってサブネットに紐づける。
     - メインルートテーブルとは違って暗黙的には使用されない。(自動でサブネットに割り当てられることがない。)
     - なお、サブネットにはルートテーブルを1つだけ適用できる。逆にルートテーブルへは複数のサブネットを紐づけられる。
   - ゲートウェイルートテーブル
     - 外から入ってくるトラフィックを制御する

※※カスタムルートテーブルとゲートウェイルートテーブルは後で試す

#### どういう時に使う?
 - VPCを作り内部にサブネットを作った時にルートテーブルも設定して、サブネットをpublicなネットワークにしたりlocalなネットワークにする。

#### 似ているサービスはある?ある場合、どう使い分ける?
 - 特にない。

#### セットで使うAWSのサービスはある?
 - VPC内にEC2等AWSのリソースを配置するような場合は必ず使用することになる。

#### 参考資料
 - AWS公式_RouteTable https://docs.aws.amazon.com/ja_jp/vpc/latest/userguide/VPC_Route_Tables.html
 - AWS公式_VPC https://docs.aws.amazon.com/ja_jp/vpc/latest/userguide/what-is-amazon-vpc.html

## VPCエンドポイント
#### どんなサービス?
 - VPCにエンドポイントを作って、そのエンドポイントを経由して他のAWSサービスに接続することができる。参考資料のブログがイメージしやすい。
 - 内部でPrivateLinkを使用するので、インターネットを経由しない。(≒InternetGWとかNatGWが必要ない。ローバルIPも必要ない)そのためコスト的にもセキュリティ的にもよいらしい。
   - VPCエンドポイントで接続できるサービスは[PrivateLinkの対応しているサービス](https://docs.aws.amazon.com/ja_jp/vpc/latest/privatelink/aws-services-privatelink-support.html)
 
#### どういう時に使う?
 - 通信要件が厳しい場合。インターネットに出たくない場合など。
 - 細かくアクセス制御をしたい場合。例えばNatGWを使ってアウトバウンドのルールを細かく制御したい時、NatGWにはセキュリティグループの紐づけができないのでRoutetableで頑張る必要がある。<br>しかしインターフェース型VPCエンドポイント(?)はセキュリティグループで指定することができるので楽。(例えばEC2のアウトバウンドを特定のVPCエンドポイントだけにしたい場合はEC2に紐づいているSGにそのVPCエンドポイントのみ許可すればよい。)<br>またVPCエンドポイントポリシー(?)もある
   - インターフェース型VPCエンドポイントとは?
     - VPCエンドポイントはゲートウェイ型とインターフェイス型がある。
     - ゲートウェイ型: RouteTableに追加され、RouteTableで制御される。つまり経路選択で選ばれた場合にこのゲートウェイから出ていく。<br>このVPCエンドポイントがグローバルIPを持っているため、例えばEC2からアクセスする場合EC2自体はプライベートIPだけでもよい。<br>但し、VPCエンドポイントが持つグローバルIPで通信を行うことに変わりはないので、RouteTableでプライベートサブネットが通信できる範囲をローカルだけにした場合は通信できなくなるもよう。(後で試す。)<br>ちなみにゲートウェイ型が使えるサービスはS3とDynamoDB だけ(最新の公式ドキュメントを見てもそうだった。)
     - インターフェイス型: VPC内部にエンドポイントが立ち上がりプライベートIPを持つ(※)ため、RouteTableでプライベートサブネットが通信できる範囲をローカルだけにしたとしても通信できるもよう。(後で試す。)
       - ※インターフェイス型はPrivateLinkを使う。PrivateLinkはプライベートIPを使って他のAWSサービスにアクセスする。
   - VPCエンドポイントポリシーとは?

  
※※ゲートウェイ型とインターフェイス型は後で試す

#### 似ているサービスはある?ある場合、どう使い分ける?
 - 💩
#### セットで使うAWSのサービスはある?
 - PrivateLinkを使って他のAWSサービスにアクセスする。
 - セキュリティグループに指定できる
 - ゲートウェイ型のVPCエンドポイントの場合、RouteTable

#### 参考資料
 - わかりやすかったブログ https://jpn.nec.com/clusterpro/blog/20180115.html
 - ゲートウェイ型とインターフェイス型? https://dev.classmethod.jp/articles/vpc-endpoint-gateway-type/
 - エンドポイントポリシーについて https://dev.classmethod.jp/articles/aws-kms-now-supports-vpc-endpoint-policies/

