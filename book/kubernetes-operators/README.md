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

### Standard Scaling: The ReplicaSet Resource 

### Custom Resources 

#### CR or ConfigMap? 

### Custom Controllers 

### Operator Scopes 

#### Namespace Scope 

#### Cluster-Scoped Operators 

### Authorization 

#### Service Accounts 

#### Roles 

#### RoleBindings 

#### ClusterRoles and ClusterRoleBindings 

### Summary 

## 4. The Operator Framework

### Operator Framework Origins 

### Operator Maturity Model 

### Operator SDK 

#### Installing the Operator SDK Tool 

### Operator Lifecycle Manager 

### Operator Metering 

### Summary 

## 5. Sample Application: Visitors Site

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

### SRE for Every Application 

### Toil Not, Neither Spin 

#### Automatable: Work Your Computer Would Like 

#### Running in Place: Work of No Enduring Value 

#### Growing Pains: Work That Expands with the System 

### Operators: Kubernetes Application Reliability Engineering 

#### Managing Application State 

#### Golden Signals Sent to Software 

### Seven Habits of Highly Successful Operators 

### Summary 

## 10. Getting Involved

### Feature Requests and Reporting Bugs 

### Contributing 

### Sharing Operators 

### Summary 

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