# O'REILLY - Kubernetes Operators


## 1. Operators Teach Kubernetes New Tricks
Operatorとは？
→Kubernetesアプリケーションをパッケージ化、実行、そして保守するための方法。

### How Kubernetes Works 
Kubernetesの基本的な説明。
コントロールプレーンとアプリケーションプレーン。

### Example: Stateless Web Server 
静的ページを返すコンテナ（ステートレスなコンテナ）はどのコンテナも同じ状態になるため、レプリカ増減時などでKubernetesは特別な対応をする必要がない。

### Stateful Is Hard 
多くのアプリケーションは状態を持っている。起動順序であったり各コンポーネントの依存関係もいろいろとある。また、「クラスタ」と言っても各々の製品で独自の概念がある。
こういったアプリケーションを統一された仕組みで管理できると理想的。
Kubernetesは上記のようなアプリケーションの管理を抽象化することができる。

### Operators Are Software SREs 
アプリケーションの管理の自動化はSREに似ている。
OperatorはSoftware Reliability Engineering。

### How Operators Work 
OperatorはKubernetesのControl PlaneとAPIを拡張することで機能する。
単純なOperatorでは、Custom Resourceと呼ばれるエンドポイントをKubernetes APIに追加し、新しいCRを監視及び維持するControl Planeコンポーネントを追加する。
このOperatorはResourceの状態に基づいてアクションを実行する。

#### Kubernetes CRs 
Custom Resource DefinitionsでCustom Resourceを定義できる。
公式APIとはことなり、CRDが適用されたKubernetesクラスタでのみCRは利用できる。
CRはエンドポイントを提供し、クラスタのユーザは通常のAPI Resourceと同じくkubectlなどでCRの情報を取得できる。

### How Operators Are Made 
OperatorはCRを監視し、アプリケーション固有のアクションを実行してCRと現在の状態を一致させるためのCustom Controllerである。
Operatorを作るということは、CRDを作成し、CRを監視するプログラムを提供することを意味する。
Operatorが実行するアクションは、複雑なアプリのスケーリングや、アプリケーションバージョンのアップグレードなど多岐にわたる。

### Example: The etcd Operator 
etcdを管理するには下記のような知識が必要
- 新規etcdノードの追加の仕方（エンドポイントの構成だったり、データの永続化だったり、他のetcdノードに認識させたり...）
- etcdのバックアップ
- etcdクラスタのバージョンアップ

etcd Operatorは上記のようなタスクをアプリケーションの状態を常に監視しながら行う。（ように作る必要がある）

#### The Case of the Missing Member 
etcd Operatorで管理された3台構成のetcd Clusterを考える。
etcdのPodの1つが削除されると、etcd Operatorは正しい状態にするために処理を開始する。


### Who Are Operators For? 
Operatorを使うと、クラスタ管理者は管理コストを抑えつつKubernetes上で提供可能であり、開発者は簡単に利用できるようになる。
あるDB製品にOperatorがあるばあい、そのDBの専門家になる必要はなくそのDB製品をデプロイすることができる。

#### Operator Adoption 
いろいろな開発者や企業がOperator開発しているよという話。

### Let’s Get Going! 
2章からはetcd Operatorを使っていろいろやるよという話。

## 2. Running Operators

はじめに手を動かすための準備を行い、その後は実際に動かしてみる。



### Setting Up an Operator Lab 

1,11.0以降のクラスタを用意する。

かつCluster-adminのアカウントを用意する。



#### Cluster Version Requirements 

1.11～1.16でテストした。



#### Authorization Requirements 

Operatorをデプロイするにはクラスタ管理者レベル（通常はcluster-admin）なアカウントが必要。

運用する際はRBACを適切に管理する必要がある。



#### Standard Tools and Techniques

Operatorは複雑なアプリケーションをKubernetes APIのfirst-class citizensにする。その意味は次の章の例で示す。



#### Suggested Cluster Configurations 

環境の準備



#### Checking Your Cluster Version 

クラスタバージョン確認

`kubectl version`で確認できる。



### Running a Simple Operator 

簡単なOperatorを動かしてみる。



#### A Common Starting Point 

etcdは、CoreOSにルーツを持つ分散KVS。現在はCNCFの支援を受けている。

K8sのコアとなるデータストアであり、メンバの間でコンセンサスをとるためにRaftを使い信頼できるストレージを提供する。



etcd OperatorはOperator Patternの紹介のための「Hello world」としてよく利用されるので、それに倣って説明に利用する。

etcdの基本的な使用法の説明であれば難しくはないが、etcdクラスタのセットアップなどはetcd固有のノウハウが必要。

etcd Operatorにはその方法が入っている。



#### Fetching the etcd Operator Manifests 

etcd Operatorの取得



#### CRs: Custom API Endpoints

CRDの適用。



#### Who Am I: Defining an Operator Service Account 

Operator用のServiceAccountの設定



#### Deploying the etcd Operator 

etcd OperatorのDeploymentの作成と適用

#### Declaring an etcd Cluster 

CRを作成して、etcd Clusterを作る。



#### Exercising etcd 

OperatorによってServiceとかも自動で作られる。

動いていることがわかる。



#### Scaling the etcd Cluster 

CRを編集してetcd Clusterをスケールアップする。



#### Failure and Automated Recovery 

etcdの特定のメンバが障害になったとき、etcdのオペレーター（人）にはメンバの障害を確認し、あたらしいメンバを追加し、

etcdクラスタの残りのメンバと協業できるようにする作業が必要。

etcd Operatorも同じく、内部状態を常に監視し、回復作業を行う。

Eventなどにログを出力することができる。



#### Upgrading etcd Clusters 

3.1.10のetcdから更新してみる。



Upgrading the hard wayだと下記のような作業が必要になる。

- 各メンバのhealth checkとversionの確認
- クラスタのスナップショットを作成する
- メンバを1つだけ停止し、新しいバージョン（3.2.13）に変更し起動する。
- 他のメンバにも同じ作業をする。

easy wayであれば、etcd OperatorのImageを変更するだけであり、あとはOperatorが実施してくれる。



#### Cleaning Up 

お掃除。



### Summary 

CRはアプリのバージョンや構成などの望ましい状態を規定し、Custom ControllerはResourceを監視し、クラスタの中で

望ましい状態が維持される。

Operator FrameworkとSDK、Operatorの構築に使用するツールキットを紹介する前に、

まずはOperatorを構成するためのKubernetes APIの紹介をする。



## 3. Operators at the Kubernetes Interface

Operatorは2つのキーコンセプト（Resources, Controllers）を拡張する。
Kubernetes APIには、新しいリソースを定義するためのメカニズムであるCustom Resource Definitionsがある。

### Standard Scaling: The ReplicaSet Resource 
ReplicaSet ControllerはReplicaSetを作成し、それらを継続的に監視する。
ReplicaSet Controllerが実行する操作は、アプリケーションに依存しない。

### Custom Resources 
ユーザはCustomResourceDefinitionを定義することで、実行中のクラスタでCustomResourceを利用できるようになる。
CustomResourceDefinitionはCustomResourceのスキーマのようなもので、CustomResourceのフィールドやそのフィールドに含まれる値の型を定義することができる。

#### CR or ConfigMap? 
ConfigMapはクラスタ状のPod内のコンテナで実行されているプログラムの設定を構成することに適している。
通常アプリケーションは、Kubernetes APIからではなく、ファイルや環境変数の値としてコンテナから情報を取得しようとする。
CustomResourceはkubectlなどのクライアントから作成およびアクセスすることができ、`.spec`や`.status`などのKubernetesの規則に従う。

### Custom Controllers 
CustomResourceを作成することで、Kubernetes APIからアクセスすることができるようになるが、
CustomResourceはデータを表現しているだけであるため、実際に処理をおこなうコンポーネントが必要となる。
その役割を負うのがCustomControllerである。

### Operator Scopes 
Operatorの適用範囲は、特定のNamespaceに制限することもクラスタ全体に適用することもできる。

#### Namespace Scope 
通常は単一のNamespaceに制限した方が良い。
複数のチームが使用するクラスタなどではより顕著。

#### Cluster-Scoped Operators 
ServiceMeshを提供するIstioやTLS証明書を管理するcert-managerのようなOperatorはクラスタ全体の状態を監視及び操作するため、
クラスタ全体に適用することが多い。
NamespacedなOperatorはRole/RoleBindingにより操作の権限を割り当て、Cluster ScopeなOperatorはClusterRole/ClusterRoleBindingによって
権限を付与する。

### Authorization 
KubernetesではいくつかのAuthorizationの仕組みがあるが、RBACが推奨されている。

#### Service Accounts 
Kubernetesでは開発者などの人を表すアカウントはクラスタが管理しておらず、それらを表すAPIリソースはない。
一方でServiceAccountはKubernetesによって管理され、APIを介して作成および操作ができる。

#### Roles 

Roleはなにができるかを記載したもの。

#### RoleBindings 

RoleBindingは誰にRoleをアタッチするか記載したもの。

NamespacedなOperatorを適用するときは、適切なRoleをOperatorのServiceAccountにRoleBindingする必要がある。



#### ClusterRoles and ClusterRoleBindings 

Namespace内に制限されるRole/RoleBindingに対して、クラスタ全体に適用されるのがClusterRoleとClusterRoleBindingsである。

クラスタ全体に適用する必要なあるOperatorには、適切なClusterRole/ClusterRoleBindingをアタッチする必要がある。



### Summary 

OperatorはKubernetesのコアコンセプトにのっとり作成されるため、アプリケーションを「Kubernetes Native」なものにすることができる。



## 4. The Operator Framework

Operatorの開発や配布、および運用管理には避けられない複雑さがある

そのサポートを行うのがOperator Frameworkである。

Operator SDKによりOperatorを開発し、

Operator Lifecycle ManagerによりOperatorの管理を行い、

Operator MeteringによってOperatorのメトリクスを監視することができる。



### Operator Framework Origins 

Operator SDKはKubernetesのcontroller-runtimeをベースにしている。

Operator Frameworkの一部としてOperator SDKによって作成されたOperatorはOLMによって管理され、

OMによってメトリックスを管理される。

Operator Frameworkはオープンソースであり、ベンダー中立であるCNCFに寄贈されている。



### Operator Maturity Model 

Operatorには成熟度モデルがある。

- Phase1 Basic Install

  - アプリケーションのインストールや設定を行う

- Phase2 Seamless Upgrades

  - アプリケーションのパッチあてやバージョンアップを行うことができる

- Phase3 Full Lifecycle

  - ストレージのバックアップ、障害復旧などを自動で行うことができる

- Phase4 Deep Insights

  - MetricsやAlert、分析の機能をもっている

- Phase5 Auto Pilot

  - 水平/垂直分散や自動設定、異常検知などの機能をもっている

  

### Operator SDK 

Operator SDKはOperatorのコーディングをするためのツールである。

SDKには現状Goによるサポートが含まれており、他の言語による開発も計画されている。

また、SDKはHelmチャートやAnsible Playbook用のAdapterも提供する。



#### Installing the Operator SDK Tool 

Operator SDKによって、プロジェクトのひな形やGoのスケルトンコードを作成することができる。

YAMLのマニフェストも生成することができる。

バイナリをダウンロードし、PATHに通すかソースをビルドする。



### Operator Lifecycle Manager 

Operatorがアプリケーションを管理するが、そのOperatorを管理する仕組みがOperator Lifecycle Managerである。

OLMはOperatorとその依存関係を管理するためにClusterServiceVersionと呼ばれるmetadataを定義する。

CSVありのOperatorはOLMで使用できるカタログのエントリとしてリストされる。

そのあとにユーザはCatalogからOperatorをSubscribheして利用する。

OperatorのCSVをもとにOLMはOperatorを管理する。



### Operator Metering 

Operator MeteringはKubernetesで実行されるOperatorのリソース使用量を分析するためのシステムである。

OMはKubernetesのCPU/Memoryその他のリソースMetricsを分析してコストを計算する。



### Summary 

Operator Frameworkは以下の3つの要素から構成される。

- Operatorを開発するためのOperator SDK
- Operatorをクラスタ上で管理するためのOperator Lifecycle Manager
- Operatorのパフォーマンスやリソース消費を測定するためのOperator Metering



## 5. Sample Application: Visitors Site

（サンプルアプリケーションの説明のため割愛）

### Application Overview 

### Installation with Manifests 

#### Deploying MySQL 

#### Backend 

#### Frontend 

### Deploying the Manifests 

### Accessing the Visitors Site 

### Cleaning Up 

### Summary 



## 6. Adapter Operators

### Helm Operator 

#### Building the Operator 

#### Fleshing Out the CRD 

#### Reviewing Operator Permissions 

#### Running the Helm Operator 

### Ansible Operator 

#### Building the Operator 

#### Fleshing Out the CRD 

#### Reviewing Operator Permissions 

#### Running the Ansible Operator 

### Testing an Operator 

### Summary 

### Resources 

## 7. Operators in Go with the Operator SDK

### Initializing the Operator 

### Operator Scope 

### Custom Resource Definitions 

#### Defining the Go Types 

#### The CRD Manifest 

### Operator Permissions 

### Controller 

#### The Reconcile Function 

### Operator Writing Tips 

#### Retrieving the Resource 

#### Child Resource Creation 

#### Child Resource Deletion 

#### Child Resource Naming 

#### Idempotency 

#### Operator Impact 

### Running an Operator Locally 

### Visitors Site Example 

### Summary 

### Resources 

## 8. Operator Lifecycle Manager

### OLM Custom Resources 

#### ClusterServiceVersion 

#### CatalogSource 

#### Subscription 

#### InstallPlan 

#### OperatorGroup 

### Installing OLM 

### Using OLM 

#### Exploring the Operator 

#### Deleting the Operator 

### OLM Bundle Metadata Files 

#### Custom Resource Definitions 

#### Cluster Service Version File 

#### Package Manifest File 

### Writing a Cluster Service Version File 

#### Generating a File Skeleton 

#### Metadata 

#### Owned CRDs 

#### Required CRDs 

#### Install Modes 

#### Versioning and Updating 

### Writing a Package Manifest File 

### Running Locally 

#### Prerequisites 

#### Building the OLM Bundle 

#### Installing the Operator Through OLM 

#### Testing the Running Operator 

### Visitors Site Operator Example 

### Summary 

### Resources 

## 9. Operator Philosophy

Operatorの考え方はSREからきている。

### SRE for Every Application 

SREは展開、運用、および保守タスクを自動化するためのコードを各。SREは他のシステムを実行するためのソフトウェアを作成し、

それを実行し、長期にわたって管理する。SREは、自動化を中心とする幅広いエンジリアにンぐ手法である。

Operator成熟度モデルではAuto Pilotに該当する。



### Toil Not, Neither Spin 

SREはシステム運用のために必要な作業を自動化することにより、労力を減らす。

Toilは「自動化可能で、戦術的で、永続的な価値がなく、サービスの成長に比例して拡大していくもの」と定義される。



#### Automatable: Work Your Computer Would Like 

機械にできることであれば自動化できる。人間の判断が必要なものはできない。



#### Running in Place: Work of No Enduring Value 

ある仕事には価値がない、と考えるのは不快なことかもしれないが、その仕事をしてもServiceが変わらない場合は「永続的な価値がない仕事」とされる。

たとえばDBのバックアップをとってもDBが高速化したり、信頼性を高めるような効果はない。

しかし、永続的な価値はないがバックアップは明らかに必要なタスクになる。

こういったタスクは多くの場合Operatorを適用する範囲になる。



#### Growing Pains: Work That Expands with the System 

水平スケールするシステムを設計することがあるかもしれない。その時にエンジニアの手によってインスタンスの追加をする場合は自動ではない。

このようなシステムの場合、サービスが10%増えるごとに管理作業も10%増える可能性がある。



ステートレスなWebサーバを追加する場合、VMを立ち上げ、一意なIPアドレスを割り当て、LBにひもづけるなどすればシステムが拡張できることは確かである。

しかし、この効果はシステムが大きくなるにつれて悪化する。例えば、すでに1000インスタンスが立ち上がっている状態で1台VMを立ち上げても性能向上は微々たるものである。



代わりにKubernetesでWebサーバをデプロイする場合は、kubectlコマンドを利用するだけでスケールアップおよびスケールダウンを行うことができる。

スケールアップするときにWebサーバ用インスタンスを構成する必要はなく、スケールダウンするときに意図的にIPを買い覇王する必要もない。

Kubernetesがそのような作業を肩代わりして実施してくれる。



### Operators: Kubernetes Application Reliability Engineering 
OperatorはKubernetesを拡張して、自動化の原理を各アプリケーションに適用することができる。
独自のクラスタリングの考え方を持ったアプリケーションを考えてみると、
etcd Operatorは障害時のメンバ交換の時に、エンドポイントと認証をつかって新しいメンバを
既存クラスタに追加する。

KubernetesのReconsilation LoopはResourceを監視し、望ましい状態にする。
Operatorによって、このLoopのカスタマイズができる。


#### Managing Application State 
多くの場合、アプリケーションは内部状態を持っており、レプリカ間で同期などを行う必要がある。

#### Golden Signals Sent to Software
- Latency
  - ある処理にどれくらいの時間がかかっているか
- Traffic
  - 時間あたりにどれくらいのHTTP Requestが来ているか
- Errors
  - どれくらいのエラーが発生しているが
- Saturation
  - リソースがどれくらい使用されているか（ひっ迫しているか）

### Seven Habits of Highly Successful Operators 
1. Operatorは1つのDeploymentで管理されること

2. CRDを定義すること

3. 可能な限りKubernetesの標準機能で作ること

4. Operatorの障害が管理対象に影響しないこと

5. 後方互換性を保つこと

6. アップグレードが行える仕組みをつくること

7. Chaos Testingも含めた十分なテストを行うこと

### Summary 
Operatorにアプリケーションの配布、デプロイ、および管理を行わせると、アプリケーションがKubernetesの機能を活用できるようになる。
7つの習慣に従うOperatorで作るべき。

## 10. Getting Involved

Operator Frameworkは発展途上。バグレポートを出すだけの簡単なところからコントリビュートする開発者になるまで、さまざまな方法で貢献することができる。

GoogleのSIGがあるので入ると良い。



### Feature Requests and Reporting Bugs 

バグを提出する際は下記を意識するとよい。

- 具体的にすること
  - 利用したバージョン
  - 環境
  - 再現手順
- 1つずつバグを報告すること
  - 複数のバグの場合だとトリアージが大変
- 該当プロジェクトを選択すること
  - 特定のプロジェクトに問題が発生する場合は、そのリポジトリに報告すること
- 既存のIssueがあればそれを利用すること
  - もしバグが継続的に発生していた場合は、そのissueを再オープンして利用する



### Contributing 

もしコードが書けるなら、コントリビュートももちろん歓迎する。

開発者ガイドも用意しているので、そこを参照する。



### Sharing Operators 

OperatorHub.ioはコミュニティ作成のOperatorのホスティングサイト。

OperatorHub.ioはCSVも確認できるので、適切なメタフィールドが適用されているかの確認もできる。

### Summary 

メーリングリストへの参加やバグ報告、新機能のためのコード提供までさまざまなコミュニティの活動によって

Operator Frameworkは成長し続けている。

OperatorHub.ioへの貢献は、Operator促進にも役立つ。



## A Running an Operator as a Deployment Inside a Cluster

クラスタの外部でOperatorを稼働させるのはテストやデバッグでは便利だが、プロダクション環境のOperatorであればKubernetesのクラスタ内で実行した方が良い。



## B Custom Resource Validation

新たなAPIを作成すると、Operator SKはCustom Resourceのスケルトンを作成する。このスケルトンはそのまま利用できる。

スケルトンのCustom Resource DefinitionsはspecとstatusをどちらもObject Typeと定義しているため柔軟に利用できる。

ただし、この場合はKubernetesでパラメータのValidationができない。

必要に応じてspecを詳細化し、Validationが行えるようにする。



## C Role-Based Access Control (RBAC)

Go-based/Helm/Ansible OperatorのどれでもOperator SDKはOperatorの実行にかかわるRBAC Manifestを作成する。

- deploy/service_account.yaml
  - 人を表すUserAccountにたいして、Kubernetes APIを利用するプログラムやシステムを表すのがServiceAccount。
- deploy/role.yaml
  - 「どのような操作ができるか」という権限を構成するもの。
- deploy/role_binding.yaml
  - Roleとあわせて「誰に権限を渡すか」を構成するもの。

開発するOperatorによって必要となる権限はことなるため、それぞれで適切なRole/RoleBindingを設定する必要がある。