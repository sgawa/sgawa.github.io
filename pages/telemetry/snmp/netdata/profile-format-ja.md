# [和訳] SNMP Profile Format


ソース：
[https://github.com/netdata/netdata/blob/master/src/go/plugin/go.d/collector/snmp/profile-format.md](https://github.com/netdata/netdata/blob/master/src/go/plugin/go.d/collector/snmp/profile-format.md)

## 概要

**SNMP プロファイル** は、_特定クラスのデバイスを SNMP 経由でどのように監視するか_ を定義します。

:::info

SNMP プロファイルは再利用可能で宣言的です。新しいデバイスをサポートするために、collector のソースコードを変更する必要はありません。

:::

これは Netdata の SNMP collector に対して、次の内容を伝えます。

- どの **OID** を問い合わせるか
- 返された値をどのように **解釈** するか
- それらを **metrics**、**dimensions**、**tags**、**metadata** にどのように **変換** するか

プロファイルを使うと、デバイスごとに Go にロジックをハードコードしたり、各デバイス向けにメトリクスを手作業で定義したりせずに、_デバイスファミリー全体_（スイッチ、ルーター、UPS、ファイアウォール、プリンターなど）を宣言的に記述できます。

各プロファイルは、再利用・拡張・組み合わせが可能な単一の YAML ファイルです。

### プロファイルの動作

Netdata が SNMP デバイスに接続すると、collector は次の処理を行います。

1. デバイスの **sysObjectID** と **sysDescr** を読み取る。
2. 利用可能なすべてのプロファイルを評価する。
3. [selectors](#1-selector) に一致するすべてのプロファイルを適用する（複数のプロファイルが同時に適用される場合があります）。
4. 結合後の設定を使って、以下を行う。
   - [スカラーメトリクス](#scalar-metrics-single-values) を収集する（稼働時間や温度のような単一値）。
   - [テーブルメトリクス](#table-metrics-multiple-rows) を収集する（インターフェイスごとのトラフィックのような複数行の値）。
   - [仮想メトリクス](#virtual-metrics) を構築する（派生合計、フォールバック、集計）。
   - ラベリングやグルーピングのために [metadata](#3-metadata) と [tags](#5-metric_tags) を収集する。

**プロファイルのライフサイクル**

```text
┌──────────────────────┐
│ SNMP Device          │ → sysObjectID/sysDescr を提供
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ selector             │ → デバイスプロファイルに一致
├──────────────────────┤
│ extends              │ → ベースプロファイルを継承
├──────────────────────┤
│ metadata             │ → デバイス情報（ベンダー、モデルなど）
├──────────────────────┤
│ metrics              │ → 収集する OID
├──────────────────────┤
│ metric_tags          │ → 全メトリクス用の動的タグ
├──────────────────────┤
│ static_tags          │ → 全メトリクス用の固定タグ
├──────────────────────┤
│ virtual_metrics      │ → 計算または集約されたメトリクス
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Netdata charts & UI  │ → ダッシュボードに可視化
└──────────────────────┘
```

### 例: 完全な SNMP プロファイル

```yaml
# example-device.yaml

# このプロファイルを適用するデバイスを選択する。
selector:
  - sysobjectid:
      include: ["1.3.6.1.4.1.9.*"]     # Cisco デバイス
    sysdescr:
      include: ["IOS"]

# 共通のベースメトリクスを読み込む
extends:
  - _system-base.yaml
  - _std-if-mib.yaml

# デバイスレベルのラベル（仮想ノード）を定義する
metadata:
  device:
    fields:
      vendor:
        value: "Cisco"
      model:
        symbol:
          OID: 1.3.6.1.2.1.47.1.1.1.1.2.1
          name: entPhysicalModelName

# 収集する OID を指定する
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.2.2
      name: ifTable
    symbols:
      - OID: 1.3.6.1.2.1.2.2.1.10
        name: ifInOctets
        chart_meta:
          description: インターフェイス受信トラフィック
          family: 'Network/Interface/Traffic/In'
          unit: "bit/s"
        scale_factor: 8
    metric_tags:
      - tag: interface
        symbol:
          OID: 1.3.6.1.2.1.31.1.1.1.1
          name: ifName

# 全メトリクスに動的タグを追加
metric_tags:
  - tag: fs_sys_version
    symbol:
      OID: 1.3.6.1.4.1.9.2.1.73.0
      name: fsSysVersion

# 全メトリクスに固定タグを追加
static_tags:
  - tag: region
    value: "us-east-1"
  - tag: environment
    value: "production"

# 結合メトリクスを計算する
virtual_metrics:
  - name: ifTotalTraffic
    sources:
      - { metric: _ifHCInOctets,  table: ifXTable, as: in }
      - { metric: _ifHCOutOctets, table: ifXTable, as: out }
    chart_meta:
      description: 全インターフェイスの合計トラフィック
      family: 'Network/Total/Traffic'
      unit: "bit/s"
```

## プロファイル構造

各 SNMP プロファイルは、**Netdata がデバイスから SNMP メトリクスをどのように収集・解釈・ラベル付けするか** を定義する YAML ファイルです。

プロファイルはモジュール化されています。ほかのプロファイルを拡張したり、metadata を定義したり、何を収集するかを指定したりできます。

```yaml
selector: <デバイス一致パターン>
extends: <含めるベースプロファイル>
metadata: <デバイス情報>
metrics: <収集する内容>
metric_tags: <グローバルタグ>
static_tags: <固定タグ>
virtual_metrics: <計算メトリクス>
```

| セクション | 目的 |
|---|---|
| [**selector**](#1-selector) | このプロファイルを適用するデバイスを定義します。 |
| [**extends**](#2-extends) | ほかのベースプロファイルを継承・マージします。 |
| [**metadata**](#3-metadata) | デバイスレベルの情報（ホストラベル）を収集します。 |
| [**metrics**](#4-metrics) | 収集する OID と、そのチャート化方法を定義します。 |
| [**metric_tags**](#5-metric_tags) | デバイスごとに一度収集され、すべてのメトリクスに付与されるグローバル動的タグを定義します。 |
| [**static_tags**](#6-static_tags) | すべてのメトリクスに適用される固定タグを定義します。 |
| [**virtual_metrics**](#7-virtual_metrics) | 他のメトリクスを基にした計算メトリクスまたは集約メトリクスを定義します。 |

### 1. selector

selector は次の用途に使います。

- 特定の **デバイスファミリー**（例: Cisco、Juniper、HP）を対象にする
- 特定の **MIB** をサポートするデバイスに一致させる
- 不要なデバイスを除外する

Netdata はディスカバリ時にすべてのプロファイルを評価し、selector に一致したプロファイルを **適用** します。

```yaml
selector:
  - sysobjectid:
      include: ["1.3.6.1.4.1.9.*"]      # 正規表現: Cisco enterprise OID サブツリー
      exclude: ["1.3.6.1.4.1.9.9.666"]  # 任意の除外条件
    sysdescr:
      include: ["IOS"]                  # 部分文字列（大文字小文字を区別しない）
      exclude: ["emulator", "lab"]    # 任意の除外条件
```

**動作**:

- 各 selector ルールは、`sysobjectid`、`sysdescr` などの条件セットです。
- ルールが一致するには、**そのルール内のすべての条件** が通る必要があります。
- `selector` リスト内の **少なくとも 1 つのルール** が一致すれば、そのプロファイルが適用されます。
- 同じルール内で `sysobjectid` と `sysdescr` の両方を定義した場合、**両方とも成功** しなければなりません。

**サポートされる条件**:

| キー | 判定対象 | 一致条件（成功） | 失敗条件 |
|---|---|---|---|
| `sysobjectid.include` | デバイスの `sysObjectID` | リスト内の **少なくとも 1 つ** のパターンに一致する | 何も一致しない |
| `sysobjectid.exclude` | デバイスの `sysObjectID` | リスト内のどのパターンにも **一致しない** | いずれかに一致する |
| `sysdescr.include` | デバイスの `sysDescr`（大文字小文字を区別しない） | リスト内の **少なくとも 1 つ** の部分文字列を含む | 指定部分文字列が見つからない |
| `sysdescr.exclude` | デバイスの `sysDescr`（大文字小文字を区別しない） | リスト内のどの部分文字列も **含まない** | いずれかを含む |

### 2. extends

`extends` は、共通メトリクスを重複定義する代わりに、別のプロファイルから metrics、tags、metadata を継承するために使います。ベース MIB のベンダー固有差分を表現するのに適しています。

実際のプロファイルの多くは、いくつかの共有ビルディングブロックを拡張し、その上にデバイス固有の定義を追加します。

```yaml
extends:
  - _system-base.yaml        # システム基本情報（uptime、contact、location）
  - _std-if-mib.yaml         # ネットワークインターフェイス（IF-MIB）
  - _std-ip-mib.yaml         # IP 統計
```

最終的なプロファイルは、継承したすべてのプロファイルと現在のファイル内容を **マージした結果** です。

**継承の仕組み**:

1. **順序が重要** — プロファイルは記述順に読み込まれます。
2. **Metrics はマージ** される — 参照されたすべてのプロファイルの metrics が含まれます。
3. **後勝ち** — 同じフィールドが複数回定義された場合、最後の定義が優先されます。

**よく使うベースプロファイル**

| プロファイル | 提供内容 | 典型的な用途 |
|---|---|---|
| `_system-base.yaml` | 基本システム情報（uptime、name、contact） | すべてのデバイス |
| `_std-if-mib.yaml` | インターフェイス統計（IF-MIB） | ネットワーク機器 |
| `_std-ip-mib.yaml` | IP レベル統計（IP-MIB） | ルーター、スイッチ |
| `_std-tcp-mib.yaml` | TCP 統計 | サーバー、ファイアウォール |
| `_std-udp-mib.yaml` | UDP 統計 | サーバー、ファイアウォール |
| `_std-ups-mib.yaml` | 電源および UPS メトリクス | UPS デバイス |

### 3. metadata

`metadata` セクションは、**デバイスレベルの情報**（metric tags ではない）を定義します。

これは **デバイスごとに 1 回だけ** 収集され、Netdata 上のデバイス **host labels**（デバイスページに表示される「仮想ノード」ラベル）を構成します。

構造は常に `metadata → device → fields` で、各 field が 1 つのラベルを定義します。

各 field は次のいずれかです。

- **Static** — `value:` が固定文字列
- **Dynamic** — 値を 1 つ以上の SNMP OID から読み取る。方法は以下のいずれか。
  - `symbol` — 1 つの OID から読み取る
  - `symbols` — 試行する OID の順序付きリスト。**最初に空でない値** が採用される

```yaml
metadata:
  device:
    fields:
      vendor:
        value: "Cisco"   # 静的ラベル

      model:
        symbols: # フォールバック付きの動的ラベル
          - OID: 1.3.6.1.4.1.9.3.6.3.0
            name: ciscoModelA
          - OID: 1.3.6.1.2.1.47.1.1.1.1.13.1
            name: entPhysicalModelName
```

**動作**:

- `vendor` は静的に `"Cisco"` に設定されます。
- `model` は動的に収集されます。collector は列挙された OID を **順番に** 試し、**最初に空でない** 値を使います。
- これらの値は Netdata UI に **デバイス（仮想ノード）の host labels** として表示されます。
- これらは **メトリクス単位のタグではなく**、個々のチャートではなくデバイス自体に適用されます。

:::tip

サポートされる変換と構文例は [**Tag Transformation**](#tag-transformation) を参照してください。

:::

### 4. metrics

`metrics` セクションは、デバイスから **何を収集するか** を定義します。つまり、どの OID を問い合わせ、返値をどう解釈し、Netdata 上でどう表示するかです。

**Metrics は次のいずれかです**:

- **Scalars** — デバイス全体に適用される単一値（例: uptime）
- **Tables** — 関連する値の繰り返し行（例: インターフェイス、ディスク、センサー）

:::note

メトリクスは **スカラー**（単一値）または **テーブルベース**（複数行）のどちらかです。
同一の metric エントリ内で両者を混在させてはいけません。

:::

collector は、スカラーには **SNMP GET** を、テーブルには **SNMP BULKWALK** を自動的に使用します。

```yaml
metrics:
  - MIB: HOST-RESOURCES-MIB
    symbol:
      OID: 1.3.6.1.2.1.1.3.0
      name: systemUptime
      scale_factor: 0.01  # 値は 100 分の 1 秒単位
      chart_meta:
        description: システムが最後に再起動または電源投入されてからの時間
        family: 'System/Uptime'
        unit: "s"

  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.31.1.1
      name: ifXTable
    symbols:
      - OID: 1.3.6.1.2.1.31.1.1.1.6
        name: ifHCInOctets
        chart_meta:
          description: トラフィック
          family: 'Network/Interface/Traffic/In'
          unit: "bit/s"
        scale_factor: 8  # Octets → bits
    metric_tags:
      - tag: interface
        symbol:
          OID: 1.3.6.1.2.1.31.1.1.1.1
          name: ifName
```

**動作**:

- 各エントリは、SNMP で収集する metric または metrics のテーブルを定義します。
- スカラーは単一の `symbol` を使い、テーブルは `table` と 1 個以上の `symbols` を定義します。
- metrics には `extract_value`、`scale_factor` などの変換や chart metadata を含められます。
- テーブルメトリクスには、インターフェイスやディスクなどの行を識別する `metric_tags` を含められます。

:::tip

関連項目:

- [Collecting Metrics](#collecting-metrics) — SNMP OID、スカラー、テーブルの説明
- [Scalar Metrics](#scalar-metrics-single-values) — 単一値メトリクスの詳細構文
- [Table Metrics](#table-metrics-multiple-rows) — テーブルと行インデックスの仕組み
- [Adding Tags to Metrics](#adding-tags-to-metrics) — タグの種類とテーブル行へのラベル付け
- [Tag Transformations](#tag-transformation) — タグ値の抽出・一致・マッピング
- [Value Transformations](#value-transformation) — 収集値の加工とスケーリング
- [Virtual Metrics](#virtual-metrics) — 既存メトリクスから新しいメトリクスを構築

:::

#### アンダースコア接頭辞のメトリクス

アンダースコアで始まる metric 名（例: `_ifHCInOctets`）は **private** です。収集はされますが、SNMP collector の出力には **伝播されません**。これらは内部用のビルディングブロックとして使用します（通常は [virtual_metrics](#7-virtual_metrics) の入力）。これにより、最終的な metric セットを整理された状態に保てます。仮想メトリクスの計算後、collector はアンダースコア付き metrics をエクスポート対象から除外します。

```yaml
# IF-MIB::ifXTable
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.31.1.1
      name: ifXTable
    symbols:
      - { OID: 1.3.6.1.2.1.31.1.1.1.6,  name: _ifHCInOctets,  scale_factor: 8 }
      - { OID: 1.3.6.1.2.1.31.1.1.1.10, name: _ifHCOutOctets, scale_factor: 8 }

virtual_metrics:
  - name: ifTraffic
    per_row: true
    group_by: ["interface"]
    sources:
      - { metric: _ifHCInOctets,  table: ifXTable, as: in }
      - { metric: _ifHCOutOctets, table: ifXTable, as: out }
```

#### 複数 symbol のフォールバック

「この OID を試し、なければ別の OID を試す」を表現するには、**同じ** `symbol.name` を持つ複数の metrics を宣言し、それぞれ異なる OID を指すようにします。実行時に collector は定義されたスカラー OID をすべて **GET** し、欠損を記録したうえで、データを返した OID から metric を **出力** します。存在しない OID は安全にスキップされます。

```yaml
metrics:
  - MIB: HOST-RESOURCES-MIB
    symbol:
      OID: 1.3.6.1.2.1.25.1.1.0
      name: systemUptime
      scale_factor: 0.01
      chart_meta:
        description: システムが最後に再起動または電源投入されてからの時間。
        family: 'System/Uptime'
        unit: "s"

  - MIB: HOST-RESOURCES-MIB
    symbol:
      OID: 1.3.6.1.2.1.1.3.0
      name: systemUptime
      scale_factor: 0.01
      chart_meta:
        description: システムが最後に再起動または電源投入されてからの時間。
        family: 'System/Uptime'
        unit: "s"
```

### 5. metric_tags

`metric_tags` セクションは、**グローバル動的タグ** を定義します。これはデバイスから一度だけ収集され、プロファイル内の **すべての metric** に適用されます。

これらは他の SNMP symbol と同様に収集中に評価され、そのデバイスのすべての metrics で共通の値になります。

**典型的な用途**:

- シリアル番号、ファームウェアバージョン、モデルなど、デバイス全体に関わるメタデータを付与する
- ハードウェア、OS、ベンダー属性によるグルーピングやフィルタリングを可能にする
- テーブル内の metric 単位タグ（例: インターフェイスごと、センサーごと）を補完する

```yaml
metric_tags:
  - tag: fs_sys_serial
    symbol:
      OID: 1.3.6.1.4.1.12356.106.1.1.1.0
      name: fsSysSerial
  - tag: fs_sys_version
    symbol:
      OID: 1.3.6.1.4.1.12356.106.4.1.1.0
      name: fsSysVersion
```

**動作**:

- 各タグは metric ごとでもテーブル行ごとでもなく、デバイスごとに一度だけ収集されます。
- 生成されたタグ値は、このプロファイルで収集される **すべての metrics** に付与されます。
- タグは、metric 単位タグと同じルールで変換（整形やマッピングなど）できます。

:::tip

サポートされる変換と構文例は [**Tag Transformation**](#tag-transformation) を参照してください。

:::

#### アンダースコア接頭辞のタグ

アンダースコアで始まるタグ名（例: `_if_type`）は **label としては出力されます** が、これらの metrics を消費する SNMP collector では **chart ID の構成から無視** されます。たとえば `interface` のような別タグが一意性を保証している場合、アンダースコアタグを使うと chart ID を短く保てます（値そのものは chart label として残ります）。

```yaml
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.31.1.1
      name: ifXTable
    symbols:
      - OID: 1.3.6.1.2.1.31.1.1.1.6
        name: ifHCInOctets
        chart_meta:
          description: Traffic
          family: 'Network/Interface/Traffic/In'
          unit: "bit/s"
        scale_factor: 8
    metric_tags:
      - tag: interface
        symbol: { OID: 1.3.6.1.2.1.31.1.1.1.1, name: ifName }
      - tag: _if_type
        table: ifTable
        symbol: { OID: 1.3.6.1.2.1.2.2.1.3, name: ifType }
        mapping:
          1: "other"
          6: "ethernet"
          24: "loopback"
          131: "tunnel"
          161: "lag"
```

#### タグのフォールバック（最初の非空値が採用される）

同じタグ名を複数回宣言すると、タグは **定義順** に評価され、**最初に空でない** 値が保持されます。これにより、優先列から代替列へのフォールバックを表現できます（内部的には、タグ追加処理は未設定または空のときだけタグを設定します）。

```yaml
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.31.1.1
      name: ifXTable
    symbols:
      - OID: 1.3.6.1.2.1.31.1.1.1.6
        name: ifHCInOctets
        chart_meta:
          description: Traffic
          family: 'Network/Interface/Traffic/In'
          unit: "bit/s"
        scale_factor: 8  # Octets → bits
    metric_tags:
      - tag: interface
        symbol: { OID: 1.3.6.1.2.1.31.1.1.1.1, name: ifName } # 優先
      - tag: interface
        table: ifTable
        symbol: { OID: 1.3.6.1.2.1.2.2.1.2, name: ifDescr } # フォールバック
```

### 6. static_tags

`static_tags` セクションは、このプロファイルで収集されるすべての metrics に付与される **固定のキー–値ペア** を定義します。

これらは SNMP データに依存せず、このプロファイルを使うすべてのデバイスで一定です。

**典型的な用途**:

- `environment`、`region`、`service` などの環境・配備識別子を追加する
- 複数デバイスからの metrics に対するフィルタリング、グルーピング、アラートを簡素化する
- データセンターやチーム所有など、一貫した文脈情報を与える

```yaml
static_tags:
  - tag: environment
    value: production
  - tag: region
    value: us-east-1
  - tag: service
    value: network
```

**動作**:

- 各タグは、このプロファイルで収集される **すべての metrics** に追加されます。
- static tags は `metric_tags` で定義した動的タグとマージされます。
- 競合した場合は、デバイス固有または動的タグが優先されます。

### 7. virtual_metrics

`virtual_metrics` セクションは、すでに収集済みの他の metrics から構築される **計算メトリクス** を定義します。

これらは SNMP を直接問い合わせません。代わりに既存の metric 値を再利用して、合計、和、フォールバックを生成します。

:::tip

完全なリファレンス、設定オプション、高度な例は [**Virtual Metrics**](#virtual-metrics) を参照してください。

:::

**典型的な用途**:

- 関連するカウンタを結合する（例: `in` + `out` トラフィック、エラー）
- フォールバックを作る（64-bit カウンタを優先し、なければ 32-bit にフォールバック）
- タグごとに metrics を集約またはグループ化する（例: インターフェイスごと、タイプごとの合計）

```yaml
  - name: ifTotalTraffic
    sources:
      - { metric: ifHCInOctets,  table: ifXTable, as: in }
      - { metric: ifHCOutOctets, table: ifXTable, as: out }
    chart_meta:
      description: 全インターフェイスの合計トラフィック
      family: 'Network/Total/Traffic'
      unit: "bit/s"
```

**動作**:

- `ifTotalTraffic` という新しい仮想 metric を定義します。
- 既存の metrics（`ifHCInOctets`、`ifHCOutOctets`）をソースとして使います。
- `as` フィールドは結果の dimension 名（`in`、`out`）を指定します。
- 生成された chart は通常の metric と同様に振る舞います。ダッシュボードに表示され、アラート対象になり、エクスポートにも含まれます。

## メトリクスの収集

このセクションでは、SNMP データがどのように構造化され、それが Netdata プロファイル内の metrics にどう対応するかを説明します。

### SNMP データの理解

SNMP データは、**OID**（**Object Identifier**）と呼ばれる数値識別子の **階層ツリー** として構成されます。

各 OID はデバイス上の値を一意に識別します。ファイルシステム上のパスのようなものです。

```text
1.3.6.1.2.1.1.3.0
│ │ │ │ │ │ │ └── インスタンス（0 = scalar）
│ │ │ │ │ │ └──── オブジェクト（3 = sysUpTime）
│ │ │ │ │ └────── ブランチ: system (MIB-2)
│ │ │ └────────── MIB-2 ルート
└─ SNMP グローバルプレフィックス
```

- **MIB**（Management Information Base）は、関連する OID の名前付きコレクションです。

  例: `IF-MIB`（インターフェイス）、`IP-MIB`（IP 統計）、`HOST-RESOURCES-MIB`（システム情報）
- 各 OID は、`Counter64`、`Gauge32`、`Integer`、`TimeTicks` などの **型付き値** に対応します。
- OID の中には **単一値**（scalar）を表すものと、関連する値の **テーブル**（行）を表すものがあります。

### Scalar Metrics（単一値）

Scalar metrics は、**デバイス全体に対する単一の値** を表します。

その OID は常に `.0` で終わります。これはスカラーオブジェクトの **インスタンス番号** を表します。

```yaml
metrics:
  - MIB: HOST-RESOURCES-MIB
    symbol:
      OID: 1.3.6.1.2.1.1.3.0
      name: systemUptime
      scale_factor: 0.01  # 値は 100 分の 1 秒単位
      chart_meta:
        description: システムが最後に再起動または電源投入されてからの時間。
        family: 'System/Uptime'
        unit: "s"
```

**これで何が起きるか**:

- `sysUpTime` の値をデバイスごとに一度だけ収集します。
- 末尾の `.0` は、この値のインスタンスが **1 つだけ** であることを示します。
- 代表的な scalar metrics は、デバイス uptime、総メモリ、全体温度などです。

### Table Metrics（複数行）

Table metrics は、ネットワークインターフェイス、ディスク、CPU のような **関連する値の一覧** を表します。

テーブル内の各行は、ベース OID に付加される **インデックス** で識別されます。たとえば次のようになります。

```text
ifHCInOctets.1 = 1024
ifHCInOctets.2 = 2048
```

- `.1`、`.2` などは `row indexes` で、インスタンス（例: interface #1、interface #2）を識別します。
- テーブル内の各列（symbol）は独自の OID パターンを持ちますが、同じ行インデックスを共有します。

> Table metrics は、各行を識別するために少なくとも 1 つのタグ（`metric_tags`）を **必ず定義** しなければなりません。
> タグがない場合は、1 行しか出力できません。

```yaml
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.31.1.1
      name: ifXTable
    symbols:
      - OID: 1.3.6.1.2.1.31.1.1.1.6
        name: ifHCInOctets
        chart_meta:
          description: Traffic
          family: 'Network/Interface/Traffic/In'
          unit: "bit/s"
        scale_factor: 8  # Octets → bits
    metric_tags:
      - tag: interface
        symbol:
          OID: 1.3.6.1.2.1.31.1.1.1.1
          name: ifName
```

**Table Metrics が行に展開される仕組み**

```text
SNMP Table: ifTable
───────────────────────────────────────────────
Index | ifName | ifHCInOctets
───────────────────────────────────────────────
1     | eth0   | 1024
2     | eth1   | 2048
───────────────────────────────────────────────

metric_tags:
  - tag: interface
    symbol:
      OID: 1.3.6.1.2.1.31.1.1.1.1   # ifName

Resulting metrics:
───────────────────────────────────────────────
ifHCInOctets{interface="eth0"} = 1024
ifHCInOctets{interface="eth1"} = 2048
───────────────────────────────────────────────
```

**動作**:

1. collector は `ifHCInOctets` と `ifName` の両列を同じテーブルから読み取ります。
2. 共有の SNMP インデックス（`1`、`2`、…）を使って行を対応付けます。
3. 各 metric は、同じ行の対応するタグ付きで出力されます。

**これで何が起きるか**:

- 各インターフェイスのトラフィック（`ifHCInOctets`）を収集します。
- 各行に、同じインデックスの `ifName` をタグとして付与します。
- 次のような metrics を生成します。
  ```text
  ifHCInOctets{interface="eth0"} = 1024
  ifHCInOctets{interface="eth1"} = 2048
  ```

### Metric Types

各 SNMP 値には、その値を **Netdata がどう解釈し表示するか** を決めるデータ型があります。

collector は適切な **metric type**（例: `gauge`、`rate`）を自動判定しますが、手動で上書きすることもできます。

**自動型判定**

| SNMP 型 | デフォルトの Netdata 型 | 代表的な用途 |
|---|---|---|
| `Counter32`, `Counter64` | `rate` | ネットワークトラフィック、パケットカウンタ |
| `Gauge32`, `Integer` | `gauge` | 温度、使用率、状態 |
| `TimeTicks` | `gauge` | uptime、時間ベースの値 |

**Metric Type の上書き**

`metric_type` フィールドを symbol 定義内に指定することで、metric の型を明示的に設定できます。

```yaml
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.2.2
      name: ifTable
    symbols:
      - OID: 1.3.6.1.2.1.2.2.1.10
        name: ifInOctets
        metric_type: gauge  # デフォルトの 'rate' を上書き
```

**これで何が起きるか**:

- `ifInOctets` を `rate` ではなく **gauge**（瞬時値）として扱います。
- 通常、`Counter` 型は自動的に秒あたりレートへ変換されます。

### Chart Metadata

各 metric または virtual metric には、Netdata のチャート上での表示方法を定義する任意の `chart_meta` ブロックを含められます。

この metadata は **データ収集には影響しません**。Netdata ダッシュボード上でチャートをどう **命名** し、どう **グループ化** するかだけを制御します。

```yaml
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.2.2
      name: ifTable
    symbols:
      - OID: 1.3.6.1.2.1.2.2.1.10
        name: ifInOctets
        chart_meta:
          description: 受信ネットワークトラフィック
          family: 'Network/Interface/Traffic/In'
          unit: "bit/s"
```

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `description` | string | no | ダッシュボードやアラートで表示される人間向け説明 |
| `family` | string | no | チャートのグルーピングパス（スラッシュ `/` で階層を定義） |
| `unit` | string | no | 表示単位。例: `"bit/s"`、`"%"`、`"{status}"`、`"Cel"` |
| `type` | string | no | 任意のチャートスタイル上書き。`line`、`area`、`stacked` |

## メトリクスへのタグ追加

タグは SNMP metrics に **文脈** と **識別性** を与えます。

これにより、どのインターフェイス・ディスク・IP なのかといったインスタンスを区別でき、Netdata UI でのフィルタリングやグルーピングが可能になります。

**collector は**:

- 各 metric にラベルとしてタグを付与する
- チャート構築時に行を区別するためタグを使う
- すべての **table metric** に少なくとも 1 つのタグを要求する（各行を識別するため）
- デバイス単位の単一値を表す **scalar metrics** ではタグを無視する

**重要な概念**:

| 概念 | 説明 |
|---|---|
| **Table metrics must have tags** | 各テーブル行は少なくとも 1 つのタグ（例: interface 名や index）で一意に識別される必要があります。タグがないと 1 行しか出力されません。 |
| **Scalar metrics don’t need tags** | Scalar はデバイス全体の 1 値を表し、インスタンス単位データではありません。 |
| **Static tags** | 変化しない固定値（例: datacenter、environment）。 |
| **Dynamic tags** | SNMP データから抽出されるタグ。テーブル列、関連テーブル、行インデックスから取得できます。 |
| **Global tags** | プロファイル上位の `metric_tags` セクションで定義され、すべての metrics に適用されるタグ。 |

**タグの種類と使用可能な変換**:

| タグ種別 | 説明 | 使用可能な変換 |
|---|---|---|
| **Static** | 定数値を持つ固定タグ | なし |
| **Same-Table** | metric と同じテーブルの列から得る値 | `mapping`、`extract_value`、`match_pattern` + `match_value`、`match` + `tags` |
| **Cross-Table** | 別テーブルから得る値 | `mapping`、`extract_value`、`match_pattern` + `match_value`、`match` + `tags` |
| **Index-Based** | 各行の OID index から導出する値 | `mapping`（任意） |
| **Index Transform** | 複数要素 index を調整し、cross-table tags を正しく整列させる | — |

**要約**:

- 各 **table metric** は、行を区別するため少なくとも 1 つの **tag source**（`metric_tags`）を定義する必要があります。
- タグは **同じテーブル**、**別テーブル**、または **行 index 自体** から取得できます。
- タグ変換（`mapping`、`extract_value`、`match_pattern`、`match + tags`）は、生の値を加工・抽出できます。
- **Static tags** はグローバルに適用され、変換対象にはなりません。
- **Index transformations** は、複数要素 index を持つテーブル間で整列を取るための特別な仕組みです。

**collector が値とタグを対応付ける仕組み**:

```text
SNMP Table (ifTable)
───────────────────────────────────────────────
Index | ifDescr        | ifInOctets
───────────────────────────────────────────────
1     | eth0           | 1024
2     | eth1           | 2048
───────────────────────────────────────────────

metric_tags:
  - tag: interface
    symbol:
      OID: 1.3.6.1.2.1.2.2.1.2   # ifDescr

Resulting metrics:
───────────────────────────────────────────────
ifInOctets{interface="eth0"} = 1024
ifInOctets{interface="eth1"} = 2048
───────────────────────────────────────────────
```

動作:

1. collector は `ifInOctets`（値）と `ifDescr`（タグソース）の両列を収集します。
2. 共有の SNMP index（`1`、`2`、…）を使って対応付けます。
3. 各 metric 行に、同じ index の対応する列値をタグとして付与します。

**Cross-Table の例**:

```text
SNMP Tables: ifTable + ifXTable
───────────────────────────────────────────────
ifTable.ifInOctets.1  = 1024
ifTable.ifInOctets.2  = 2048

ifXTable.ifName.1     = "eth0"
ifXTable.ifName.2     = "eth1"
───────────────────────────────────────────────

metric_tags:
  - tag: interface
    table: ifXTable
    symbol:
      OID: 1.3.6.1.2.1.31.1.1.1.1   # ifName

Result:
───────────────────────────────────────────────
ifInOctets{interface="eth0"} = 1024
ifInOctets{interface="eth1"} = 2048
───────────────────────────────────────────────
```

動作:

- collector は `ifTable` から metrics を収集し、`ifXTable` からタグ値を取得します。
- 両テーブルの行は共有 index（`.1`、`.2`、…）で対応付けられます。
- `interface` タグは各一致行の `ifXTable.ifName` から設定されます。

### Static

Static tags は、SNMP から収集せずに metrics に付与される **固定のキー–値ペア** を定義します。

これは **environment** や **location** など、収集データ全体に適用される文脈情報の識別に有用です。

#### プロファイルレベルの Static Tags

プロファイルレベルの static tags は、そのプロファイルで定義された **すべての metrics** に適用されます。

```yaml
# グローバル static tags（すべての metrics に適用）
static_tags:
  - tag: datacenter
    value: "DC1"
  - tag: environment
    value: "production"
```

**これで何が起きるか**:

- これらのタグが、このプロファイルで報告されるすべての metrics に注入されます。

**典型的な用途**:

- デバイスが属するデータセンター、リージョン、クラスターを識別する
- staging / production 環境からの metrics を区別する
- SNMP データに依存しない組織共通の文脈情報を追加する

#### メトリクスレベルの Static Tags

メトリクスレベルの static tags は、**特定の metrics のみに** 適用されます。

```yaml
# metric 固有の static tags
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.2.2
      name: ifTable
    symbols:
      - OID: 1.3.6.1.2.1.2.2.1.10
        name: ifInOctets
    static_tags:
      - tag: "source"
        value: "snmp"
      - tag: "interface_type"
        value: "physical"
```

**これで何が起きるか**:

- `source=snmp` と `interface_type=physical` を `ifInOctets` metric にのみ追加します。
- 同じプロファイル内の他の metrics には影響しません。

> メトリクスレベルの static tags は技術的にはサポートされていますが、必要になることはまれです。
> 多くの場合、一貫性と単純さのためにプロファイルレベルの `static_tags` を優先してください。

### Same-Table

Same-table tags は、metric と **同じ SNMP テーブル内の列** から値を取得します。

これは、インターフェイス名や index のような識別子で行単位 metrics をラベル付けする最も一般的な方法です。

**collector は**:

- 同じテーブル行から metric 値とタグ列の両方を取得する
- 共有 index によって行を自動的に整列させる
- その行から収集された各 metric にタグを追加する

```yaml
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.2.2
      name: ifTable
    symbols:
      - OID: 1.3.6.1.2.1.2.2.1.10
        name: ifInOctets
    metric_tags:
      - tag: interface
        symbol:
          OID: 1.3.6.1.2.1.2.2.1.2
          name: ifDescr
```

**これで何が起きるか**:

- `ifTable` の各行について `ifInOctets`（受信バイト数）を収集します。
- 同じテーブルの `ifDescr` 列を読み、各行のラベルに使います。
- 次のような metrics を生成します。
  ```text
  ifInOctets{interface="eth0"} = 1000
  ifInOctets{interface="eth1"} = 2000
  ```

### Cross-Table

Cross-table tags は、**別の SNMP テーブルのデータ** をタグソースとして使えるようにします。

**collector は**:

- 現在のテーブルではなく、指定された `table:` からタグ値を読み取る
- **index** によってテーブル間の行を対応付ける
- index 構造が異なる場合は、任意の `index_transform` で現在テーブルの index を加工し、対象テーブルに揃える

#### 同じ index の場合

2 つのテーブルが **同じ index** を持つとは、ベース OID の後ろに続く行識別子（OID 接尾辞）が一致しており、同じ実体を記述していることを意味します。

実際には、一方のテーブルの行番号（index）が他方のテーブルの同じ行に直接対応するということです。

たとえば:

```text
ifTable.ifInOctets.2  = 123456
ifXTable.ifName.2     = "xe-0/0/1"
```

両方の OID は `.2` で終わっているため、同じインターフェイスを指しています。

これにより、`ifTable` から収集した metrics のタグとして、`ifXTable` の `ifName` を利用できます。

```yaml
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.2.2
      name: ifTable
    symbols:
      - OID: 1.3.6.1.2.1.2.2.1.10
        name: ifInOctets
    metric_tags:
      - tag: interface
        table: ifXTable
        symbol:
          OID: 1.3.6.1.2.1.31.1.1.1.1
          name: ifName
```

**これで何が起きるか**:

- `ifTable` から `ifInOctets` を収集します。
- `ifXTable` 内で同じ index を持つ行（例: `.2`）を見つけます。
- `ifName` を `interface` タグとして使用します。
- 次のような metrics を生成します。
  ```text
  ifInOctets{interface="xe-0/0/1"} = 123456
  ```

#### Index 変換を使う場合

関連するデータを表すテーブルでも、OID 接尾辞の構造が異なる **別の index 構造** を持つことがあります。

たとえば `ipIfStatsTable` では、index が **2 つの要素** を持ちます。

```text
ipIfStatsTable.ipIfStatsHCInOctets.2.1 = 38560
ipIfStatsTable.ipIfStatsHCInOctets.2.2 = 44408
```

ここで:

- 最初の要素 (`2`) は **IP バージョン**（例: 2 = IPv4、3 = IPv6）です。
- 2 番目の要素（`1`、`2`、`3`、…）は **インターフェイス index** です。
- 一方 `ifXTable` はインターフェイス index（`1`、`2`、`3`、…）だけを使います。

index が異なるため、そのままでは対応付けできません。

この問題を解決するには `index_transform` を使い、index の **必要な部分だけを選択** して、対象テーブルの形式に一致させます。

```yaml
metrics:
  - MIB: IP-MIB
    table:
      OID: 1.3.6.1.2.1.4.31.3
      name: ipIfStatsTable
    symbols:
      - OID: 1.3.6.1.2.1.4.31.3.1.6
        name: ipIfStatsHCInOctets
        chart_meta:
          description: IP 受信オクテット合計（エラーを含む）
          family: 'Network/Interface/IP/Traffic/Total/In'
          unit: "bit/s"
        scale_factor: 8
    metric_tags:
      - tag: _interface
        table: ifXTable
        symbol:
          OID: 1.3.6.1.2.1.31.1.1.1.1
          name: ifName
        index_transform:
          - start: 1
            end: 1
```

**これで何が起きるか**:

- `ipIfStatsTable` から IP トラフィック metrics を収集します。
- `2.1` のうち **2 番目の index 要素**（`start: 1`、`end: 1`）だけを残し、`1` にします。
- そのインターフェイス index を `ifXTable` で引き、対応する `ifName` を見つけます。
- 次の結果を生成します。
  ```yaml
  ipIfStatsHCInOctets{_interface="xe-0/0/1"} = 38560
  ```

##### `index_transform` の仕組み

`index_transform` は、テーブル間で行を対応付ける際に、現在のテーブルの index のどの部分を保持するかを collector に指示します。

| 概念 | 例 |
|---|---|
| **Original index** | `2.1`（`ipIfStatsTable` 由来）→ `[ipVersion, ifIndex]` |
| **Target index** | `1`（`ifXTable` 由来） |
| **Transform** | `index_transform: [ { start: 1, end: 1 } ]` |
| **Result** | collector は **2 番目の要素**（`ifIndex = 1`）だけを保持し、`ifXTable` に一致させる |

**要点**:

- `start` と `end` は **0 始まり**（0 = 最初の要素）です。
- 各 range は、保持する index 部分を定義します。
- 複数 range を並べれば、離れた部分を組み合わせることもできます。
- 目的は、現在テーブルの index を対象テーブルの index に **一致させる** ことです。

### Index-Based

Index-based tags は、SNMP テーブルの列ではなく、**OID index そのもの** から値を抽出します。

これは、テーブルが識別子（メソッド、コード、ポート番号など）を別列ではなく OID 自体に埋め込んでいる場合に有用です。

**collector は**:

- テーブル行の index を番号付き要素（1 始まり）に分割する
- 各 `index:` ルールについて、指定位置の値をタグとして割り当てる
- 数値 index 要素を自動的に文字列へ変換する
- 得られたタグをその行から収集された metric に付与する

```yaml
metrics:
  - MIB: SIP-COMMON-MIB
    table:
      name: sipCommonStatusCodeTable
      OID: 1.3.6.1.2.1.149.1.5.1
    symbols:
      - OID: 1.3.6.1.2.1.149.1.5.1.1.3
        name: sipCommonStatusCodeIns
        chart_meta:
          family: 'Network/VoIP/SIP/Response/StatusCode/In'
          description: 指定されたステータスコードで受信した応答メッセージ総数
          unit: "{response}/s"
    metric_tags:
      - index: 1
        tag: applIndex
      - index: 2
        tag: sipCommonStatusCodeMethod
      - index: 3
        tag: sipCommonStatusCodeValue
```

**これで何が起きるか**:

- 各行の OID index の先頭 3 要素を抽出してタグに使います。
- たとえば完全な OID が次の場合:
  ```text
  1.3.6.1.2.1.149.1.5.1.1.3.1.6.200
  ```
  collector は次のように解釈します。
  ```ini
  applIndex=1
  sipCommonStatusCodeMethod=6
  sipCommonStatusCodeValue=200
  ```
- 次のような metrics を生成します。
  ```text
  sipCommonStatusCodeIns{applIndex="1", sipCommonStatusCodeMethod="6", sipCommonStatusCodeValue="200"} = 42
  ```

## タグ変換

タグ変換を使うと、SNMP 値の一部を加工または抽出して、明確で人間が読みやすいタグを生成できます。

これは次の **両方** で同じように機能します。

- `metadata`（例: デバイスモデル、OS 名）
- `metric_tags`（例: 行ごとのインターフェイスラベル）

**利用できるタグ変換**:

| 変換 | 目的 | 例 |
|---|---|---|
| `mapping` | 数値／文字列コードを名前に置き換える | `1 → "ethernet"`、`161 → "lag"` |
| `extract_value` | 正規表現で部分文字列を抽出する（最初のキャプチャグループ） | `"RouterOS CCR2004-16G-2S+" → "CCR2004-16G-2S+"` |
| `match_pattern` + `match_value` | 正規表現グループまたは固定値で値を置き換える | `"Palo Alto Networks VM-Series firewall" → "VM-Series firewall"` |
| `match` + `tags` | 1 つの値から **複数のタグ** を作る | `"xe-0/0/1" → if_family=xe, fpc=0, pic=0, port=1` |

**組み合わせと挙動**:

| ルール | 説明 |
|---|---|
| **Where** | `metadata.*.fields.*.symbols[]` と `metric_tags[]` の中で使えます。 |
| **Order of application** | 1️⃣ `match_pattern` + `match_value` **または** `extract_value` → 2️⃣ `mapping` → 3️⃣ `match` + `tags` |
| **No match behavior** | `extract_value` は元の値を保持。`match_pattern` は値をスキップ。`match` + `tags` はタグを生成しない。 |
| **Multiple symbols** | 同じタグに対して複数の `symbols` が定義されている場合、**最初の非空結果** が使われます。 |
| **Mapping key consistency** | `mapping` のキー型はすべて統一する必要があります（すべて数値またはすべて文字列）。 |
| **Safety** | 正規表現は可能な限り単純にし、必要なら `^pattern$` のようにアンカーしてください。 |

**簡易構文**:

- `mapping`
  ```yaml
  mapping:
    6: "ethernet"
    161: "lag"
  ```
- `extract_value`
  ```yaml
  extract_value: 'RouterOS ([A-Za-z0-9-+]+)'   # 最初のキャプチャグループを使用
  ```
- `match_pattern` + `match_value`
  ```yaml
  match_pattern: 'Palo Alto Networks\s+(PA-\d+ series firewall|VM-Series firewall)'
  match_value: '$1'
  ```
- `match` + `tags`
  ```yaml
  match: '^([A-Za-z]+)[-_]?(\d+)\/(\d+)\/(\d+)$'
  tags:
    if_family: $1
    fpc: $2
    pic: $3
    port: $4
  ```

### Mapping

`mapping` は、生のタグ値を **人間が読めるテキストラベル** に置き換えるために使います。

**collector は**:

- 生の値を mapping テーブルで引く
- 対応する文字列で置き換える
- 対応がなければ **元の値を保持** する
- キーは数値でも文字列でもよいが、型は統一する必要がある
- `metadata` と `metric_tags` の両方に適用できる

```yaml
metrics:
  - MIB: IF-MIB
    table:
      OID: 1.3.6.1.2.1.2.2
      name: ifTable
    symbols:
      - OID: 1.3.6.1.2.1.2.2.1.10
        name: ifInOctets
    metric_tags:
      - tag: if_type
        symbol:
          OID: 1.3.6.1.2.1.2.2.1.3
          name: ifType
        mapping:
          1: "other"
          6: "ethernet"
          24: "loopback"
          131: "tunnel"
          161: "lag"
```

**これで何が起きるか**:

- 数値のインターフェイスタイプコード（1、6、24、131、161）を、`other`、`ethernet`、`loopback`、`tunnel`、`lag` に置き換えます。
- 未知の型をデバイスが返した場合は、元の数値をそのまま使います。
- `metadata` と `metric_tags` の両方で同じように機能します。

### Extract Value

`extract_value` は、**正規表現** を使って文字列の一部を取り出すために使います。

**collector は**:

- 生の値に対してパターンを適用する
- **最初のキャプチャグループ** `( … )` を値として使用する
- 一致しなければ **元の値を保持** する
- パターンを `^` や `$` で囲まない限り、文字列内のどこでも検索する
- 複数の `symbols` がある場合は、最初の非空結果を使う

```yaml
metadata:
  device:
    fields:
      model:
        symbols:
          # 例: 'RouterOS CCR2004-16G-2S+' → 'CCR2004-16G-2S+'
          - OID: 1.3.6.1.2.1.1.1.0
            name: sysDescr
            extract_value: 'RouterOS ([A-Za-z0-9-+]+)'
          # 例: 'CSS326-24G-2S+ SwOS v2.13' → 'CSS326-24G-2S+'
          - OID: 1.3.6.1.2.1.1.1.0
            name: sysDescr
            extract_value: '([A-Za-z0-9-+]+) SwOS'
```

### Match Pattern

`match_pattern` と `match_value` を組み合わせると、複数の **正規表現キャプチャグループ** を使ってタグ値を構築できます。

**collector は**:

- `match_pattern` の正規表現で値をテストする
- **一致した場合**、`match_value` で値を置き換える
- `match_value` 内では `$1`、`$2`、`$3` のようにキャプチャグループを参照できる
- **一致しない場合** はその値をスキップする
- 抽出テキストの再整形にも、一致時の固定値設定にも使える

**例 1 — キャプチャグループを使って再整形**:

```yaml
metadata:
  device:
    fields:
      product_name:
        symbol:
          OID: 1.3.6.1.2.1.1.1.0
          name: sysDescr
          match_pattern: 'Palo Alto Networks\s+(PA-\d+ series firewall|WildFire Appliance|VM-Series firewall)'
          match_value: "$1"
          # 例:
          #  - Palo Alto Networks VM-Series firewall  →  VM-Series firewall
          #  - Palo Alto Networks PA-3200 series firewall  →  PA-3200 series firewall
          #  - Palo Alto Networks WildFire Appliance  →  WildFire Appliance
```

**例 2 — 一致時に固定値を設定**:

```yaml
metadata:
  device:
    fields:
      type:
        symbols:
          - OID: 1.3.6.1.2.1.1.1.0
            name: sysDescr
            # RouterOS デバイス
            match_pattern: 'RouterOS (CCR.*)'
            match_value: 'Router'
```

### Match（複数タグ）

`match` と `tags` を使うと、**正規表現** とキャプチャグループに基づいて、1 つの SNMP 値から **複数のタグ** を生成できます。

**collector は**:

- 生の値に対して `match` の正規表現を適用する
- **一致した場合**、`tags` 配下に列挙したすべてのタグを作成し、`$1`、`$2`、`$3` などを置換する
- **一致しない場合**、それらのタグは追加されない
- 空のキャプチャに解決されるタグは追加されない

**例 1 — sysDescr から OS 名とモデルを分割する（metadata）**:

```yaml
metadata:
  device:
    fields:
      type:
        symbols:
          - OID: 1.3.6.1.2.1.1.1.0
            name: sysDescr
            match: '^(\S+)\s+(.*)$'
            tags:
              os_name: $1    # 例: 'RouterOS'
              model: $2      # 例: 'CCR2004-16G-2S+'
```

> `RouterOS CCR2004-16G-2S+` のような入力は、`os_name=RouterOS`、`model=CCR2004-16G-2S+` になります。

**例 2 — インターフェイス名から複数ラベルを導出する（metric_tags）**:

```yaml
metric_tags:
  - symbol:
      OID: 1.3.6.1.2.1.2.2.1.2
      name: ifDescr
    match: '^([A-Za-z]+)[-_]?(\d+)\/(\d+)\/(\d+)$'
    tags:
      if_family: $1   # 例: 'xe'、'ge'、'GigabitEthernet'
      fpc: $2         # '0'
      pic: $3         # '0'
      port: $4        # '1'
```

- `xe-0/0/1`、`ge-0/0/0`、`GigabitEthernet1/0/24` のような一般的なパターンに対応します。
- 出力タグは `if_family=xe`、`fpc=0`、`pic=0`、`port=1` のようになります。

## 値変換

値変換を使うと、格納やチャート化の前に、生の SNMP metric 値を **加工または正規化** できます。

これらは SNMP データ収集中に **symbol 単位（OID 単位）** で適用されます。変更対象は **metric 値のみ** で、tags や metadata は対象外です。また **virtual metrics には適用されません**。

典型的な用途:

- 混在文字列から数値部分だけを抽出する
- 単位をスケール変換する（bytes → bits、megabits → bits）
- 離散状態（1 = up、2 = down など）を名前付き dimension にマッピングする

**利用できる値変換**:

| 変換 | 目的 | 例 |
|---|---|---|
| `mapping` | 数値または文字列コードを状態 dimension に変換する | `1 → up`、`2 → down`、`3 → testing` |
| `extract_value` | 正規表現で数値部分を抽出する | `"23.8 °C" → "23"` |
| `scale_factor` | 定数倍して単位を調整する | `"1.5" (MBps) × 8 → 12 (Mbps)` |
| `match_pattern` + `match_value` | *metric 値には適用しない*（代わりに `extract_value` を使う） | — |

**組み合わせと挙動**:

| ルール | 説明 |
|---|---|
| **Where** | 値変換は `metrics[*].symbol` または `metrics[*].symbols[]` 内で使用します。 |
| **Order of application** | 1️⃣ `extract_value` → 2️⃣ `mapping` → 3️⃣ `scale_factor` |
| **Scale factor position** | `scale_factor` は常に最後に適用されます。 |
| **Data type handling** | `mapping` により multi-value metric へ変換されない限り、数値型（int/float）は保持されます。 |
| **Error handling** | 変換が失敗しても（正規表現不一致など）、collector は元の値を保持します。 |
| **Applicability** | 値変換は metric 値にのみ影響し、metadata や tags には影響しません。 |
| **Mapping behavior** | 各 mapping エントリが dimension になり、一致するものが `1`、それ以外は `0` を返す multi-value metric を生成します。 |

**簡易構文**:

- `mapping`
  ```yaml
  mapping:
    1: up
    2: down
    3: testing
  ```

- `extract_value`
  ```yaml
  extract_value: '(\d+)'   # 最初のキャプチャグループを使用
  ```

- `scale_factor`
  ```yaml
  scale_factor: 8   # Octets → bits
  ```

### Mapping

`mapping` は、生の metric 値を **状態 dimension** に変換するために使います。

各 mapping エントリは、**dimension 名** と、それを有効にする数値または文字列値を定義します。

**collector は**:

- 値を mapping テーブルと照合する
- 各 mapping エントリについて、対応する名前の **dimension** を作る
- 現在値がそのキーに一致すれば `1`、そうでなければ `0` を設定する
- どのキーにも一致しなければ、すべての dimension は `0`
- metric 値にのみ適用され、tags や metadata には適用されない

```yaml
metrics:
  - OID: 1.3.6.1.2.1.2.2.1.7
    name: ifAdminStatus
    chart_meta:
      description: インターフェイスの現在の管理状態
      family: 'Network/Interface/Status/Admin'
      unit: "{status}"
    mapping:
      1: up
      2: down
      3: testing
```

**これで何が起きるか**:

- SNMP の整数値（1、2、3）を、`up`、`down`、`testing` の dimension を持つ **multi-value metric** に変換します。
- 現在値に対応する dimension は `1`、その他は `0` を返します。

### Extract Value

`extract_value` は、**正規表現** を使って生の SNMP 値から **数値または文字列の一部** を抽出します。

これは、metric が数値を含む文字列（例: `"23.8 °C"`）としてエンコードされている場合によく使われます。

**collector は**:

- 生の SNMP 値に正規表現を適用する
- **最初のキャプチャグループ** `( … )` を新しい metric 値として使う
- パターンに一致しなければ元の値を保持する
- 文字列型・数値型を問わず使える
- 複数の `symbols` が定義されている場合、最初の非空結果を使う

```yaml
metrics:
  - MIB: CORIANT-GROOVE-MIB
    table:
      OID: 1.3.6.1.4.1.42229.1.2.3.1.1
      name: shelfTable
    symbols:
      - OID: 1.3.6.1.4.1.42229.1.2.3.1.1.1.3
        name: coriant.groove.shelfInletTemperature
        # 例: "23.8 °C" → "23"
        extract_value: '(\d+)'
        chart_meta:
          description: Shelf inlet temperature
          family: 'Hardware/Shelf/Temperature/Inlet'
          unit: "Cel"
```

**これで何が起きるか**:

- 文字列 `"23.8 °C"` に対して正規表現 `(
\d+)` を適用します。
- 数値部分 `"23"` だけを取り出し、metric 値として使います。
- 一致しなければ元の文字列を保持します。
- 数値、単位、ラベルを含む文字列 metric に適しています。

### Scale Factor

`scale_factor` は、収集した metric 値を定数倍するために使います。

これは一般に、単位変換（例: bytes → bits）に使われます。

**collector は**:

- 生の SNMP 値に指定された係数を掛ける
- 整数値にも浮動小数点値にも適用できる
- 結果の数値型（int または float）を維持する
- **metric 値にのみ** 適用される（tags や metadata には適用されない）
- 同じ metric 上の他の変換（`extract_value` など）の **後** に適用される

```yaml
metrics:
  - MIB: IP-MIB
    table:
      OID: 1.3.6.1.2.1.4.31.1
      name: ipSystemStatsTable
    symbols:
      - OID: 1.3.6.1.2.1.4.31.1.1.6
        name: ipSystemStatsHCInOctets
        chart_meta:
          description: 受信 IP データグラム内のオクテット数
          family: 'Network/IP/Traffic/Total/In'
          unit: "bit/s"
        scale_factor: 8   # Octets → bits

  - MIB: IF-MIB
    symbol:
      OID: 1.3.6.1.2.1.31.1.1.1.15
      name: ifHighSpeed
      chart_meta:
        description: インターフェイスの現在帯域幅の推定値
        family: 'Network/Interface/Speed'
        unit: "bit/s"
      scale_factor: 1000000   # Megabits → bits
```

**これで何が起きるか**:

- オクテットカウンタに `8` を掛け、バイトではなく **ビット/秒** でトラフィックを報告します。
- `ifHighSpeed` を **メガビット** から **ビット** に変換します。
- スケーリングは、値抽出や正規表現処理などの他の変換 **後** に実行されます。

## Virtual Metrics

- Virtual metrics は、プロファイル内（または継承元）の他 metrics から作られる **計算メトリクス** です。
- SNMP を直接問い合わせず、既存の metric 値を **再利用** して、合計、フォールバック、行単位集約を作ります。
- 計算後は通常の metrics と同様に扱われます。チャート化され、タグ付けされ、アラート対象にもなります。

よくある用途:

- **Fallbacks**: 64-bit カウンタを優先し、なければ 32-bit にフォールバック
- **Sums/Combines**: 関連 metrics を加算する（例: in + out traffic）
- **Per-row totals**: 複数列をインターフェイス単位の 1 つの metric に集約する

### 構造

```yaml
virtual_metrics:
  - name: <string>

    # Option 1 — 直接ソース（フォールバックなし）
    sources:
      - { metric: <metricName>, table: <tableName>, as: <dimensionName> }

    # Option 2 — Alternatives（フォールバックセットあり）
    alternatives:
      - sources: # 最初に試す（優先）
          - { metric: <metricNameA>, table: <tableName>, as: <dimensionName> }
          - { metric: <metricNameB>, table: <tableName>, as: <dimensionName> }
      - sources: # 最初のセットが欠けている場合のフォールバック
          - { metric: <fallbackMetricA>, table: <tableName>, as: <dimensionName> }
          - { metric: <fallbackMetricB>, table: <tableName>, as: <dimensionName> }

    per_row: <true|false>
    group_by: <label | [labels]>
    chart_meta:
      description: ...
      family: ...
      unit: ...
```

**Sources と Alternatives の違い**:

- `sources:` は **主たる入力セット** を定義します。metric の計算方法が 1 通りしかない場合に使います。
- `alternatives:` は **順序付きフォールバックセット** を定義します。各要素はそれぞれ独自の `sources:` ブロックを持ちます。

collector は alternatives を **順番に** 評価し、データを正常に生成できた **最初のセット** を使用します。

### 設定リファレンス

| 項目 | フィールド | 型 | 必須 | デフォルト | 適用先 | 説明 |
|---|---|---|---|---|---|---|
| **Virtual Metric** | `name` | string | yes | — | all | プロファイル内で一意。metric/chart のベース名として使う。 |
|  | `sources` | array<Source> | no* | — | totals, per_row, grouped | 直接ソースセット。`alternatives` がある場合は無視される。 |
|  | `alternatives` | array<Alternative> | no* | — | totals, per_row, grouped | 順序付きフォールバックセット。最初にデータを生成できた alternative が使われる。 |
|  | `per_row` | bool | no | false | per-row/grouped | `true` の場合、入力行ごとに 1 つの出力を生成する。sources は dimension になる。 |
|  | `group_by` | string / array | no | — | per-row/grouped | 行キーのヒントとして使う label 群。空や欠損の場合は全タグから安定キーを生成する。`per_row:false` では PromQL の `sum by (...)` に相当。 |
|  | `chart_meta` | object | no | — | all | 表示用 metadata（`description`、`family`、`unit`、`type`） |
| **Source** | `metric` | string | yes | — | — | 既存 metric 名（scalar または table column metric） |
|  | `table` | string | yes | — | — | 元 metric の table 名。per-row/grouped では metric 側の table と一致する必要がある。 |
|  | `as` | string | yes | — | — | 合成結果内の dimension 名（例: `in`、`out`） |
| **Alternative** | `sources` | array<Source> | yes | — | — | 1 つの alternative 内のすべての sources は一緒に評価される。どれもデータを生成できなければ次の alternative を試す。 |

> `sources` または `alternatives` の少なくとも 1 つを **必ず定義** しなければなりません。

#### ルールと制約

| ルール | 説明 |
|---|---|
| **Precedence** | `sources` と `alternatives` が両方ある場合、`alternatives` が優先されます。 |
| **Same-table requirement** | `per_row` または `group_by` を使う場合、すべての sources は同じ table 由来でなければなりません。 |
| **per_row: true** | 入力行ごとに 1 出力を生成する。複数 sources は chart dimensions（`as`）になる。行タグは自動的に付与される。 |
| **group_by (with per_row:true)** | 行キーのヒントとして機能する。欠損や空値のときは全タグ複合キーにフォールバック。 |
| **group_by (with per_row:false)** | 指定 label ごとに行を集約する。PromQL の `sum by (...)` に近い。 |
| **Alternative evaluation** | alternatives は順番に評価され、最初にデータを生成できたものが採用される。 |
| **Parent metadata** | 仮想 metric は、データ元が alternative であっても、自身の `name` と `chart_meta` を使って chart を出力する。 |
| **Dimensions** | 各 `as` 値が結果チャート内の dimension を定義する。 |
| **Totals vs per-row** | `per_row` と `group_by` の両方を省略すると、全行を跨ぐ単一の合計チャートになる。 |

### 例

#### 1 つのテーブルからの行単位集約（in + out traffic）

```yaml
virtual_metrics:
  - name: ifTotalTraffic
    sources:
      - { metric: _ifHCInOctets,  table: ifXTable, as: in }
      - { metric: _ifHCOutOctets, table: ifXTable, as: out }
    per_row: true
    group_by: ["interface"]
    chart_meta:
      description: インターフェイスごとのトラフィック
      family: 'Network/Interface/Traffic'
      unit: "bit/s"
```

**これで何が起きるか**:

- `ifXTable` の **入力行ごとに 1 つの出力** を生成します。
- 各チャートは 1 つのインターフェイスを表し、`in` と `out` の 2 dimension を持ちます。
- `group_by: ["interface"]` は、インターフェイスごとのチャートを安定させるためのキー情報を提供します。
- ヒントが欠けているか空の場合は、代わりに全タグ複合キーを使います。
- **制約**: `per_row` または `group_by` を使う場合、すべての sources は同じ table 由来でなければなりません。

#### 合計集約（全インターフェイスを合算）

```yaml
virtual_metrics:
  - name: ifTotalTraffic
    sources:
      - { metric: _ifHCInOctets,  table: ifXTable, as: in }
      - { metric: _ifHCOutOctets, table: ifXTable, as: out }
    chart_meta:
      description: 全インターフェイスの合計トラフィック
      family: 'Network/Total/Traffic'
      unit: "bit/s"
```

**これで何が起きるか**:

- `ifXTable` の **全行** のデータを、**1 つのチャート** に集約します。
- デバイス全体のインターフェイストラフィック合計を、`in` と `out` の 2 dimension で表します。
- `per_row` と `group_by` がないため、単一の合計チャートになります。

#### グループ集約（ラベルごとに sum）

```yaml
virtual_metrics:
  - name: ifTypeTraffic
    sources:
      - { metric: _ifHCInOctets,  table: ifXTable, as: in }
      - { metric: _ifHCOutOctets, table: ifXTable, as: out }
    per_row: false
    group_by: ["ifType"]
    chart_meta:
      description: インターフェイスタイプ別に集約したトラフィック
      family: 'Network/InterfaceType/Traffic'
      unit: "bit/s"
```

**これで何が起きるか**:

- **PromQL 風の `sum by (ifType)` 集約** を行います。
- 同じ `ifType` ラベルを持つ全行をまとめて合計します。
- 結果は、インターフェイスタイプごとに集約された `in` と `out` の dimension を持つチャートになります。
- **制約**: すべての sources は同じ table から来ている必要があります。

#### Alternatives（合計; 64-bit を優先し、32-bit にフォールバック）

```yaml
virtual_metrics:
  - name: ifTotalPacketsUcast
    alternatives:
      - sources:
          - { metric: _ifHCInUcastPkts,  table: ifXTable, as: in }
          - { metric: _ifHCOutUcastPkts, table: ifXTable, as: out }
      - sources:
          - { metric: _ifInUcastPkts,  table: ifTable, as: in }
          - { metric: _ifOutUcastPkts, table: ifTable, as: out }
    chart_meta:
      description: 全インターフェイスのユニキャストパケット合計（in/out）
      family: 'Network/Total/Packet/Unicast'
      unit: "{packet}/s"
```

**これで何が起きるか**:

- 2 つの **alternative** を定義します。それぞれが source のリストです。
- 実行時、collector は **順番に** alternatives を評価し、**すべての source metrics が存在しデータを持つ** 最初のものを使います（最初に HC を試す）。
- 勝者が決まったら、**後続の alternatives は無視** されます。
- 親 metric は、自身の `name` と `chart_meta` を使って metrics を出力し、値だけを選択された子 source から取得します。
- `sources` と `alternatives` の両方がある場合、`alternatives` が優先されます。

#### Composite（複数ソースの合計: unicast / multicast / broadcast）

```yaml
virtual_metrics:
  - name: ifTotalPacketsByKind
    sources:
      - { metric: _ifHCInUcastPkts,      table: ifXTable, as: in_ucast }
      - { metric: _ifHCOutUcastPkts,     table: ifXTable, as: out_ucast }
      - { metric: _ifHCInMulticastPkts,  table: ifXTable, as: in_mcast }
      - { metric: _ifHCOutMulticastPkts, table: ifXTable, as: out_mcast }
      - { metric: _ifHCInBroadcastPkts,  table: ifXTable, as: in_bcast }
      - { metric: _ifHCOutBroadcastPkts, table: ifXTable, as: out_bcast }
    chart_meta:
      description: 種別ごとの全インターフェイス合計パケット数（in/out）
      family: 'Network/Total/Packet/ByKind'
      unit: "{packet}/s"
```

**これで何が起きるか**:

- 複数の関連パケットカウンタを組み合わせた **単一の合計チャート** を構築します。
- 各 `as` が `in_ucast`、`out_ucast`、`in_mcast` などの **dimension** になります。
- `per_row` と `group_by` を指定していないため、全インターフェイスを跨いだ合計になります。
