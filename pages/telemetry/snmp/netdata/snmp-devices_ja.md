<!--startmeta
custom_edit_url: "https://github.com/netdata/netdata/edit/master/src/go/plugin/go.d/collector/snmp/README.md"
meta_yaml: "https://github.com/netdata/netdata/edit/master/src/go/plugin/go.d/collector/snmp/metadata.yaml"
sidebar_label: "SNMP デバイス"
learn_status: "Published"
learn_rel_path: "Collecting Metrics/Networking"
keywords: ['snmp', 'mib', 'oid', 'network', 'router', 'switch', 'firewall', 'ap', 'access point', 'wireless controller', 'wlc', 'wifi', 'vpn', 'pdu', 'ups', 'nas', 'san', 'printer', 'bgp', 'ospf', 'ucd', '3com', 'a10', 'alcatel', 'lucent', 'nokia', 'anue', 'apc', 'netbotz', 'arista', 'aruba', 'audiocodes', 'avaya', 'avocent', 'avtech', 'roomalert', 'barracuda', 'bluecat', 'brocade', 'brother', 'chatsworth', 'checkpoint', 'chrysalis', 'cisco', 'cisco asa', 'cisco asr', 'cisco catalyst', 'cisco nexus', 'cisco ironport', 'cisco ics', 'cisco wlc', 'cisco ucs', 'meraki', 'citrix', 'netscaler', 'cradlepoint', 'cyberpower', 'dell', 'dell emc', 'poweredge', 'sonicwall', 'dialogic', 'dlink', 'd-link', 'eaton', 'exagrid', 'extreme', 'f5', 'big-ip', 'fireeye', 'fortinet', 'fortigate', 'fortiswitch', 'gigamon', 'hp', 'hewlett packard', 'hp ilo', 'ilo', 'ilo4', 'hp h3c', 'hp icf', 'hpe', 'proliant', 'huawei', '3com huawei', 'ibm', 'datapower', 'lenovo', 'idrac', 'dell idrac', 'infinera', 'coriant', 'infoblox', 'isilon', 'ixsystems', 'truenas', 'juniper', 'junos', 'kyocera', 'linksys', 'mcafee', 'mikrotik', 'nasuni', 'nec', 'net-snmp', 'netsnmp', 'netapp', 'netgear', 'readynas', 'omron', 'opengear', 'palo alto', 'cloudgenix', 'peplink', 'raritan', 'riverbed', 'ruckus', 'serveriron', 'server-iron', 'servertech', 'silverpeak', 'silver peak', 'edgeconnect', 'sinetica', 'sophos', 'synology', 'diskstation', 'tp-link', 'tplink', 'tripplite', 'tripp lite', 'ubiquiti', 'unifi', 'velocloud', 'vertiv', 'liebert', 'watchguard', 'western digital', 'wd', 'mycloud', 'zebra', 'zyxel']
message: "このファイルを直接編集しないでください。これはコレクターの metadata.yaml ファイルから生成されています。"
endmeta-->

# SNMP デバイス


<img src="https://netdata.cloud/img/SNMP.png" width="150"/>


Plugin: go.d.plugin
Module: snmp

<img src="https://img.shields.io/badge/maintained%20by-Netdata-%2300ab44" />

## 概要

このコレクターは、SNMP 対応のあらゆるネットワークデバイスを検出し、監視します。

- **組み込みベンダープロファイル**: Netdata には主要ベンダー向けの[大規模なプロファイルライブラリ](https://github.com/netdata/netdata/tree/master/src/go/plugin/go.d/config/go.d/snmp.profiles/default)が同梱されており、一般的なハードウェアについては **OID を手動設定せずに** そのまま自動監視できます。
- **カスタムプロファイルをサポート**: ユーザーは標準プロファイルを拡張または上書きして、新しいデバイスの追加、チャートの変更、追加 OID の収集を行えます。
- **ベンダー / モデルの自動検出**: `sysObjectID` や `sysDescr` などのセレクターを使って、デバイスを適切なプロファイルにマッチさせます。
- **ICMP ping**: SNMP とあわせて往復遅延の監視を任意で有効化でき、`ping_only` モードも利用できます。
- **SNMP v1 / v2c / v3 をサポート**: [gosnmp](https://github.com/gosnmp/gosnmp) ライブラリにより完全実装されています。


**主要ベンダー向け組み込みプロファイル:**

| カテゴリ | ベンダー |
|----------|---------|
| スイッチ / ルーター | Cisco (Catalyst, Nexus, ASR, ISR), Arista, Juniper, HP/HPE, Dell, Extreme |
| ファイアウォール | Palo Alto, Fortinet FortiGate, Cisco ASA, Checkpoint, SonicWall |
| ワイヤレス | Aruba, Cisco WLC, Ubiquiti, Alcatel-Lucent |
| ロードバランサー | F5 BIG-IP, Citrix NetScaler, A10 Thunder |
| インフラ | APC UPS/PDU, Dell サーバー、および標準 MIB (BGP, OSPF, TCP/UDP) |

> この表は一般的なベンダーを抜粋したものです。**実際のライブラリにはさらに多く** のベンダーが含まれます。


:::info

独自プロファイルの作成や標準プロファイルの拡張方法については [SNMP Profile Format](https://github.com/netdata/netdata/blob/master/src/go/plugin/go.d/collector/snmp/profile-format.md) を参照してください。

:::

**プロファイルの配置場所**

| 種別 | 既定パス | 備考 |
|------|--------------|-------|
| **標準プロファイル** | `/usr/lib/netdata/conf.d/go.d/snmp.profiles/default/` | Netdata に同梱 |
| **ユーザープロファイル** | `/etc/netdata/go.d/snmp.profiles/` | カスタムまたは変更したプロファイルを配置 |

> インストール形態によっては、パスの先頭に `/opt/netdata` が付く場合があります。

**プロファイル** では以下を定義します。

- 自動マッチング用のデバイスセレクター（例: `sysObjectID`, `sysDescr`）
- 収集する正確な OID（スカラーおよびテーブル）
- テーブル行へのラベル付け方法（metric tags）
- チャート / メトリクスメタデータ（単位、ファミリー、タイプ）および任意の **virtual metrics**

**実行時にコレクターが行うこと:**

1. 標準システム OID（例: `sysObjectID`, `sysDescr`）を読み取り、デバイスを識別する
2. 最適なベンダー / モデルプロファイルを選択する
3. それらのプロファイルで定義されたメトリクスだけを収集する


このコレクターはすべてのプラットフォームでサポートされています。

このコレクターは、リモートインスタンスを含む、このインテグレーションの複数インスタンスからのメトリクス収集をサポートします。


### 既定の動作

#### 自動検出

SNMP サービスディスカバリーは、設定済みネットワークを自動スキャンし、検出したデバイスを SNMP コレクターに渡すことができます。

- 既定では無効です。明示的に有効化して設定してください。
- 単一 IP、IP 範囲、CIDR ブロックをサポートします（サブネットあたり最大 512 IP）。
- 指定した SNMP 認証情報（v1/v2c/v3）を使用してデバイスをプローブします。
- ネットワーク負荷を減らすため、検出結果をキャッシュします（設定可能）。
- 収集中、各デバイスは `sysObjectID`、`sysDescr`、およびプロファイルのセレクタールールに基づいて適切な[プロファイル](https://github.com/netdata/netdata/blob/master/src/go/plugin/go.d/collector/snmp/profile-format.md)にマッチされます。

設定ファイル名は [go.d/sd/snmp.conf](https://github.com/netdata/netdata/blob/master/src/go/plugin/go.d/config/go.d/sd/snmp.conf) です。

設定ファイルは、Netdata の[設定ディレクトリ](https://learn.netdata.cloud/docs/netdata-agent/configuration#the-netdata-config-directory)にある edit-config スクリプトで編集できます。

```bash
cd /etc/netdata 2>/dev/null || cd /opt/netdata/etc/netdata
sudo ./edit-config go.d/sd/snmp.conf
```


#### 制限事項

このインテグレーションの既定設定では、データ収集に制限は課されません。

#### パフォーマンスへの影響

**デバイス側の制約**: 多くの SNMP デバイス（例: アクセススイッチ）は、管理処理に使える CPU / ASIC 時間が限られています。タイムアウトや欠損が見られる場合は、`update_every` や `max_repetitions` を下げるか、複数デバイスへのポーリング時刻を分散してください。

**並列ポーリング**: 複数ツールからの並列アクセスにより、一部デバイスでカウンター取得漏れが起きることがあります。リクエスト圧力を減らすため、収集間隔（`update_every`）を長くしてください。


## セットアップ


**snmp** コレクターは次の 2 通りで設定できます。

| 方法 | 適している用途 | 手順 |
|-----------------------|------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| [**UI**](#via-ui)     | ファイル編集なしで素早く設定したい場合 | **Nodes → Configure this node → Collectors → Jobs** に移動し、**snmp** を検索して **+** をクリックし、ジョブを追加します。 |
| [**File**](#via-file) | ファイルで設定したい場合、または自動化（例: Ansible）が必要な場合 | `go.d/snmp.conf` を編集してジョブを追加します。 |

:::important

UI での設定には有料の Netdata Cloud プランが必要です。

:::


### 前提条件

#### SNMP デバイスの準備

コレクターを設定する前に:
- 対象デバイスで SNMP サービスを有効にします（管理インターフェース経由）。
- Netdata ノードから対象デバイスの UDP/161 に到達できることを確認します。
- 接続情報を用意します。IP / DNS、SNMP バージョン、v1/v2c の場合は community、v3 の場合は認証情報（user、auth/priv）が必要です。



### 設定

#### オプション

以下のオプションはグローバルに定義できます: `update_every`, `autodetection_retry`。


<details open><summary>設定オプション</summary>



| グループ | オプション | 説明 | デフォルト | 必須 |
|:------|:-----|:------------|:--------|:---------:|
| **Collection** | update_every | データ収集頻度。 | 10 | no |
|  | autodetection_retry | 再確認までの秒数。0 の場合は再確認をスケジュールしません。 | 0 | no |
| **Target** | hostname | 対象ホスト（IP または DNS 名、IPv4/IPv6）。 |  | yes |
| **SNMPv1/2** | community | SNMPv1/2 の community 文字列。 | public | no |
| **SNMPv3** | user.name | SNMPv3 のユーザー名。 |  | no |
|  | [user.level](#option-snmpv3-user-level) | SNMPv3 メッセージのセキュリティレベル。 |  | no |
|  | [user.auth_proto](#option-snmpv3-user-auth-proto) | SNMPv3 メッセージの認証プロトコル。 |  | no |
|  | user.auth_key | SNMPv3 メッセージの認証パスフレーズ。 |  | no |
|  | [user.priv_proto](#option-snmpv3-user-priv-proto) | SNMPv3 メッセージのプライバシープロトコル。 |  | no |
|  | user.priv_key | SNMPv3 メッセージのプライバシーパスフレーズ。 |  | no |
| **SNMP transport** | options.version | SNMP バージョン。利用可能: 1, 2, 3。 | 2 | no |
|  | options.port | 対象ポート。 | 161 | no |
|  | options.retries | 再試行回数。 | 1 | no |
|  | options.timeout | SNMP リクエスト / レスポンスのタイムアウト。 | 5 | no |
|  | options.max_repetitions | 1 回の GETBULK リクエストで取得する SNMP 変数数を制御します。 | 25 | no |
|  | options.max_request_size | 1 回の GET リクエストで許可される OID の最大数。 | 60 | no |
| **Ping** | ping_only | ICMP の往復メトリクスのみ収集し、定期的な SNMP ポーリングをスキップします。セットアップ時には名前・ラベル・メタデータ取得のため、最小限の SNMP sysInfo プローブは実行されます。 | no | no |
|  | ping.enabled | ICMP 往復計測を有効にします（SNMP と並行実行）。無効にすると ping メトリクスは収集されません。 | yes | no |
|  | ping.privileged | raw ICMP（特権モード）を使用します。false の場合は非特権モードを使用します。 | yes | no |
|  | ping.packets | 各イテレーションで送信する ping パケット数。 | 3 | no |
|  | ping.interval | ping パケット送信間隔。 | 100ms | no |
| **Profiles** | manual_profiles | 自動検出が使えない場合に強制適用するプロファイル一覧。 | [] | no |
| **Virtual node** | create_vnode | 有効にすると、この SNMP デバイス用の Netdata Virtual Node を作成し、Netdata 上で別ノードとして表示します。 | true | no |
|  | vnode_device_down_threshold | デバイスを down と判定するまでの連続収集失敗回数。 | 3 | no |
|  | vnode.guid | Virtual Node の一意識別子。未設定時はデバイス IP から自動生成されます。 |  | no |
|  | vnode.hostname | Virtual Node に使用するホスト名。未設定時はデバイスのホスト名を使用します。 |  | no |
|  | vnode.labels | Virtual Node に関連付ける追加のキー / 値ペア。 |  | no |

<a id="option-snmpv3-user-level"></a>
##### user.level

RFC 3414 に基づく SNMPv3 メッセージのセキュリティ（`user.level`）:

| 文字列値 | 整数値 | 説明 |
|:------------:|:---------:|------------------------------------------|
|     none     |     1     | メッセージ認証も暗号化も行わない |
|  authNoPriv  |     2     | メッセージ認証あり、暗号化なし |
|   authPriv   |     3     | メッセージ認証あり、暗号化あり |


<a id="option-snmpv3-user-auth-proto"></a>
##### user.auth_proto

認証が必要な SNMPv3 メッセージに使うダイジェストアルゴリズム（`user.auth_proto`）:

| 文字列値 | 整数値 | 説明 |
|:------------:|:---------:|-------------------------------------------|
|     none     |     1     | メッセージ認証なし |
|     md5      |     2     | MD5 メッセージ認証（HMAC-MD5-96） |
|     sha      |     3     | SHA メッセージ認証（HMAC-SHA-96） |
|    sha224    |     4     | SHA メッセージ認証（HMAC-SHA-224） |
|    sha256    |     5     | SHA メッセージ認証（HMAC-SHA-256） |
|    sha384    |     6     | SHA メッセージ認証（HMAC-SHA-384） |
|    sha512    |     7     | SHA メッセージ認証（HMAC-SHA-512） |


<a id="option-snmpv3-user-priv-proto"></a>
##### user.priv_proto

プライバシーが必要な SNMPv3 メッセージに使う暗号化アルゴリズム（`user.priv_proto`）:

| 文字列値 | 整数値 | 説明 |
|:------------:|:---------:|-------------------------------------------------------------------------|
|     none     |     1     | メッセージ暗号化なし |
|     des      |     2     | DES 暗号化（CBC-DES） |
|     aes      |     3     | 128 ビット AES 暗号化（CFB-AES-128） |
|    aes192    |     4     | 192 ビット AES 暗号化（CFB-AES-192）、「Blumenthal」鍵ローカライゼーション |
|    aes256    |     5     | 256 ビット AES 暗号化（CFB-AES-256）、「Blumenthal」鍵ローカライゼーション |
|   aes192c    |     6     | 192 ビット AES 暗号化（CFB-AES-192）、「Reeder」鍵ローカライゼーション |
|   aes256c    |     7     | 256 ビット AES 暗号化（CFB-AES-256）、「Reeder」鍵ローカライゼーション |



</details>


#### via UI

Netdata の Web インターフェースから **snmp** コレクターを設定します。

1. **Nodes** に移動します。
2. **snmp データ収集ジョブを実行したいノード** を選択し、:gear:（**Configure this node**）をクリックします。そのノードがデータ収集を実行します。
3. 既定で **Collectors → Jobs** ビューが開きます。
4. 検索ボックスで _snmp_ と入力するか一覧をスクロールし、**snmp** コレクターを探します。
5. **snmp** コレクターの横にある **+** をクリックし、新しいジョブを追加します。
6. ジョブ項目を入力し、**Test** で設定を検証し、**Submit** で保存します。
    - **Test** は入力した設定でジョブを実行し、データ収集可能かどうかを表示します。
    - 失敗した場合は、接続拒否、タイムアウト、コマンド実行エラーなどの詳細付きエラーメッセージが表示されるため、調整して再試行できます。


#### via File

このインテグレーションの設定ファイル名は `go.d/snmp.conf` です。

ファイル形式は YAML です。一般的な構造は次のとおりです。

```yaml
update_every: 1
autodetection_retry: 0
jobs:
  - name: some_name1
  - name: some_name2
```
設定ファイルは、Netdata の[設定ディレクトリ](https://github.com/netdata/netdata/blob/master/docs/netdata-agent/configuration/README.md#locate-your-config-directory)にある [`edit-config`](https://github.com/netdata/netdata/blob/master/docs/netdata-agent/configuration/README.md#edit-configuration-files) スクリプトで編集できます。

```bash
cd /etc/netdata 2>/dev/null || cd /opt/netdata/etc/netdata
sudo ./edit-config go.d/snmp.conf
```

##### 例

###### SNMPv1/2

この例では:

- SNMP デバイスは `192.0.2.1` です。
- SNMP バージョンは `2` です。
- SNMP community は `public` です。
- 10 秒ごとに値を更新します。

プロファイルは実行時に自動選択されます。


<details open><summary>設定</summary>

```yaml
jobs:
  - name: switch
    update_every: 10
    hostname: 192.0.2.1
    community: public
    options:
      version: 2

```
</details>

###### SNMPv3

SNMPv3 を使うには:

- `community` の代わりに `user` を使います。
- `options.version` を 3 に設定します。


<details open><summary>設定</summary>

```yaml
jobs:
  - name: switch
    update_every: 10
    hostname: 192.0.2.1
    options:
      version: 3
    user:
      name: username
      level: authPriv
      auth_proto: sha256
      auth_key: auth_protocol_passphrase
      priv_proto: aes256
      priv_key: priv_protocol_passphrase

```
</details>

###### 複数デバイスでの SNMPv3

この例では、同じ SNMPv3 認証情報を共有する複数の SNMP デバイスを監視します。

[YAML anchors](https://yaml.org/spec/1.2.2/#3222-anchors-and-aliases) を使って、完全なジョブ定義を一度だけ（`&snmp_v3_job`）記述し、その後 `<<: *snmp_v3_job` で再利用し、追加デバイスごとに `name` と `hostname` だけを上書きしています。


<details open><summary>設定</summary>

```yaml
jobs:
  - &snmp_v3_job
    name: switch1
    update_every: 10
    hostname: 192.0.2.1
    options:
      version: 3
    user:
      name: username
      level: authPriv
      auth_proto: sha256
      auth_key: auth_protocol_passphrase
      priv_proto: aes256
      priv_key: priv_protocol_passphrase

  - <<: *snmp_v3_job
    name: switch2
    hostname: 192.0.2.2

  - <<: *snmp_v3_job
    name: switch3
    hostname: 192.0.2.3

```
</details>



## アラート

このインテグレーションには、既定ではアラートは設定されていません。


## メトリクス

メトリクスとチャートは、実行時に **マッチした SNMP プロファイル** によって定義されます。内容はベンダー / モデル / OS によって異なり、たとえばインターフェースカウンター、光学情報、CPU / メモリ、温度、VLAN などが含まれる場合があります。そのデバイスで実際に何が収集されるかは、ダッシュボードの **Metrics** タブで確認してください。

:::tip

これらのプロファイルの構造（metrics、tags、virtual metrics など）を理解するには、**[SNMP Profile Format](https://github.com/netdata/netdata/blob/master/src/go/plugin/go.d/collector/snmp/profile-format.md)** を参照してください。

:::

`ping.enabled` が true の場合、ICMP 遅延 / パケットロスのチャートも提供されます（`ping_only: true` の場合はそれのみが提供されます）。



## ライブデータ

このコレクターは、Live タブで対話的なトラブルシュートを行うためのリアルタイム機能を提供します。


### ネットワークインターフェース

SNMP 対応デバイスから、ネットワークインターフェースのトラフィックおよび状態メトリクスの詳細を提供します。

この機能は、通常のポーリングサイクル中に収集されたキャッシュ済み SNMP インターフェースデータを問い合わせ、ソートおよびフィルタリング可能なテーブルとして表示します。各行は監視対象 SNMP デバイス上の 1 つのネットワークインターフェースを表し、トラフィック分析、エラー監視、運用状態の追跡に必要な包括的メトリクスを含みます。

利用例:
- ルーター、スイッチ、アクセスポイントで帯域消費の大きいインターフェースを特定する
- ネットワーク健全性のためにインターフェースの運用状態および管理状態を監視する
- パケットエラー、破棄、異常なトラフィックパターンを調査する

データは IF-MIB（RFC 2863）のインターフェースカウンターを元にしており、最後に成功した SNMP 収集結果からキャッシュされています。この機能呼び出し時に追加の SNMP リクエストは発生しません。


| 項目 | 説明 |
|:-------|:------------|
| Name | `Snmp:interfaces` |
| Require Cloud | no |
| Performance | キャッシュ済み SNMP データのみを使用し、追加の SNMP リクエストは発生しません。<br/>• 応答はメモリキャッシュから即時返されます<br/>• 多数のインターフェースを持つ大規模デバイスでは多くの行が返る場合があります |
| Security | 公開されるのはインターフェース名、運用状態、トラフィックカウンターのみです。<br/>• パケットペイロードや認証情報は公開されません<br/>• デバイス設定の詳細は公開されません |
| Availability | 次の場合に利用可能です。<br/>• コレクターが少なくとも 1 回データ収集を完了している<br/>• 最後に成功した SNMP 収集結果からインターフェースデータがキャッシュされている<br/>• キャッシュ未準備の場合は HTTP 503 を返します |

#### 前提条件

追加設定は不要です。

#### パラメータ

| Parameter | Type | Description | Required | Default | Options |
|:---------|:-----|:------------|:--------:|:--------|:--------|
| Type Group | select | 種別分類グループでインターフェースをフィルタします。カスタムマッピングにより IANA インターフェースタイプを実用的なグループへ分類し、絞り込みやすくしています。 | yes | ethernet | Ethernet (default), Aggregation, Virtual, Other |

#### 返り値

キャッシュ済み SNMP データから得たネットワークインターフェースメトリクスを返します。トラフィックレート、パケット統計、運用状態、エラーカウンターが含まれます。各行は 1 つの物理または仮想インターフェースを表します。

| Column | Type | Unit | Visibility | Description |
|:-------|:-----|:-----|:-----------|:------------|
| Interface | string |  |  | ネットワークインターフェース名または識別子（例: eth0, GigabitEthernet1/0/1, Vlan100） |
| Type | string |  |  | IF-MIB で定義された IANA 割り当て済みインターフェースタイプ（例: ethernetCsmacd, ieee80211, softwareLoopback） |
| Type Group | string |  |  | IANA インターフェースタイプを実用的なグループに分類するカスタムカテゴリ。Ethernet（物理 Ethernet インターフェース）、Aggregation（LAG / port-channels、bond）、Virtual（VLAN、loopback）、Other（その他すべて） |
| Admin Status | string |  |  | インターフェースに設定された管理状態。up（使用可能）、down（管理上無効）、testing（テストモード）。運用状態とは異なります。 |
| Oper Status | string |  |  | 現在の運用状態。up（動作中でトラフィック転送可能）、down（非動作）、testing（テストモード）、unknown（状態判定不可）、dormant（外部アクション待ち）、notPresent（インターフェースは削除済みだが設定は残存）、lowerLayerDown（下位レイヤー障害によりダウン） |
| Traffic In | float | bit/s |  | 受信ネットワークトラフィックレート（bit/s）。値が高い場合、容量計画が必要なほど受信データ量が多いことを示します。 |
| Traffic Out | float | bit/s |  | 送信ネットワークトラフィックレート（bit/s）。高い値は送信データ量が多いことを示します。Traffic In と比較することで非対称な利用傾向を把握できます。 |
| Unicast In | float | packets/s | hidden | 1 つの宛先向けユニキャストパケットの受信レート（packets/s）。ポイントツーポイント通信で一般的なトラフィックパターンです。 |
| Unicast Out | float | packets/s | hidden | 1 つの宛先に送信されたユニキャストパケットの送信レート。 |
| Broadcast In | float | packets/s | hidden | ブロードキャストパケットの受信レート（packets/s）。高い値はネットワークストーム、ARP フラッディング、設定不備を示す場合があります。 |
| Broadcast Out | float | packets/s | hidden | ブロードキャストパケットの送信レート。継続的に高い場合、ネットワーク性能を低下させる可能性があります。 |
| Packets In | float | packets/s |  | 総受信パケットレート（ユニキャスト、ブロードキャスト、マルチキャストの合計）。インターフェース全体の負荷評価に有用です。 |
| Packets Out | float | packets/s |  | 総送信パケットレート（ユニキャスト、ブロードキャスト、マルチキャストの合計）。 |
| Errors In | float | packets/s | hidden | 配信できなかった受信エラーパケットのレート。0 以外の場合、物理層障害（ケーブル不良、信号品質）やバッファオーバーランを示す可能性があります。 |
| Errors Out | float | packets/s | hidden | 送信エラーパケットのレート。0 以外の場合、インターフェースハードウェア障害、ケーブル不良、duplex mismatch などを示す可能性があります。 |
| Discards In | float | packets/s |  | デバイスが意図的に破棄した受信パケットのレート。多くはリソース制約、セキュリティポリシー、未認識フレームなどが原因です。エラーとは異なり、インターフェース自体は正常でもパケットを破棄した可能性があります。 |
| Discards Out | float | packets/s |  | 意図的に破棄した送信パケットのレート。出力キューあふれ、ACL によるドロップ、セキュリティポリシー拒否を示す場合があります。 |
| Multicast In | float | packets/s | hidden | グループ宛マルチキャストパケットの受信レート。動画配信、マルチキャストアプリケーション、ルーティングプロトコルで一般的です。 |
| Multicast Out | float | packets/s | hidden | マルチキャストパケットの送信レート。 |



## トラブルシューティング

### デバッグモード

**重要**: Dyncfg 機能を使って UI から作成したデータ収集ジョブでは、デバッグモードはサポートされません。

`snmp` コレクターの問題を調査するには、デバッグオプション付きで `go.d.plugin` を実行します。出力には、コレクターが動作しない理由を判断する手がかりが含まれます。

- `plugins.d` ディレクトリに移動します。通常は `/usr/libexec/netdata/plugins.d/` です。システムによって異なる場合は、`netdata.conf` を開いて `[directories]` セクションの `plugins` 設定を確認してください。

  ```bash
  cd /usr/libexec/netdata/plugins.d/
  ```

- `netdata` ユーザーに切り替えます。

  ```bash
  sudo -u netdata -s
  ```

- `go.d.plugin` を実行してコレクターをデバッグします。

  ```bash
  ./go.d.plugin -d -m snmp
  ```

  特定のジョブをデバッグする場合:

  ```bash
  ./go.d.plugin -d -m snmp -j jobName
  ```

### ログの取得

`snmp` コレクターで問題が発生している場合は、以下の手順でログを取得し、潜在的な問題を特定してください。

- システムに応じたコマンド（systemd、非 systemd、Docker コンテナ）を実行します。
- 出力を確認し、警告またはエラーメッセージがないか調べます。これらのメッセージは根本原因の手がかりになります。

#### systemd を使用するシステム

Netdata サービスの前回再起動以降に生成されたログを表示するには、次のコマンドを使用します。

```bash
journalctl _SYSTEMD_INVOCATION_ID="$(systemctl show --value --property=InvocationID netdata)" --namespace=netdata --grep snmp
```

#### systemd を使用しないシステム

通常は `/var/log/netdata/collector.log` にあるコレクターログファイルを特定し、`grep` でコレクター名を抽出します。

```bash
grep snmp /var/log/netdata/collector.log
```

**注**: この方法では、すべての再起動分のログが表示されます。現在の問題を調査する際は、**最新のエントリ** に注目してください。

#### Docker コンテナ

Netdata が `netdata` という名前の Docker コンテナで動作している場合（異なる場合は置き換えてください）、次のコマンドを使用します。

```bash
docker logs netdata 2>&1 | grep snmp
```

### チャートの欠損をデバッグする

SNMP チャートに欠損がある場合、それは次の予定実行までにコレクターがメトリクス収集を完了できなかったことを意味します。通常、SNMP テーブルの収集に `update_every` で設定した間隔より長い時間がかかる場合に起こります。

この欠損は、デバイスが SNMP メトリクスの出力を停止したことを意味するわけではありません。単にコレクターが一部サイクルをスキップしただけです。

**Step 1: ログを確認する**

次のようなメッセージを[ログから探してください](#getting-logs)。

```text
level=warn msg="skipping data collection: previous run is still in progress for 4s (skipped 4 times in a row, interval 1s)" collector=snmp job=your_device
level=info msg="data collection resumed after 4.36s (skipped 4 times)" collector=snmp job=your_device
```

“resumed after” メッセージには、前回の収集に実際どれだけ時間がかかったかが表示されます。  
たとえば 1 回の実行に約 4.4 秒かかり、`update_every` が 1 秒なら、4 サイクルがスキップされます。


**Step 2: 収集時間を確認する**

ダッシュボードの **SNMP → Internal → Stats** を開きます。  
**SNMP profile collection timings** チャートには、SNMP ポーリングの各部分にどれだけ時間がかかっているかが表示されます。  
通常、最も遅いのはテーブルメトリクスであり、全体の収集時間を決める要因になることが多いです。

**Step 3: データ収集間隔を長くする**

ネットワークのばらつきに備えた余裕も見込んで、最も遅い収集時間より **大きい値** に [`update_every`](#setup) を設定します。

| 一般的な収集時間 | 推奨 `update_every` |
|-------------------------|-----------------------------|
| < 2 秒             | 2 秒                  |
| 2–5 秒             | 5 秒                  |
| 5–10 秒            | 10 秒                 |
| > 10 秒            | collection_time × 2        |

:::info

- **経験則:** `update_every` は、最も遅いテーブル収集時間の少なくとも 2 倍にしてください。  
- 既定の `update_every: 10` は、ほとんどの環境で適切に動作します。  
- デバイスが十分速く応答することが継続的に確認できる場合にのみ、この値を下げてください。

:::

**簡易チェックリスト**
1. ログに “skipping data collection” が出ているか。  
2. *Internal → Stats* で収集時間が `update_every` を超えているか。  
3. スキップがなくなるまで `update_every` を増やす。
