# NET-SNMP-MIB.txt を読む

2025/10/28 WED

##  lmSensorsMIB

このファイルで確認できる内容は、`lmSensorsMIB` という MIB モジュールの定義です。Linux 系のセンサー情報を SNMP で参照するための構造が、温度・ファン・電圧・その他、という単位で整理されています。  

### 1. 冒頭コメントと import

```text
LM-SENSORS-MIB DEFINITIONS ::= BEGIN

--
-- Derived from the original VEST-INTERNETT-MIB. Open issues:
--
-- (a) where to register this MIB?
-- (b) use not-accessible for diskIOIndex?
--


IMPORTS
    MODULE-IDENTITY, OBJECT-TYPE, Integer32, Gauge32
        FROM SNMPv2-SMI
    DisplayString
        FROM SNMPv2-TC
    ucdExperimental
        FROM UCD-SNMP-MIB;
```



ここでは、MIB モジュールの開始宣言と、必要な型・識別子の import が書かれています。
コメントには、この MIB が `VEST-INTERNETT-MIB` を元にしていること、また登録先などに未解決事項が残っていることが記されています。完成済みの厳密な標準 MIB というより、既存実装を引き継いだ実務寄りの MIB であることが読み取れます。
`Gauge32`、`Integer32`、`DisplayString` を使うために `SNMPv2-SMI` と `SNMPv2-TC` を参照し、OID の親として `UCD-SNMP-MIB` の `ucdExperimental` を使っています。 

### 2. モジュール識別情報

```text
lmSensorsMIB MODULE-IDENTITY
    LAST-UPDATED "200011050000Z"
    ORGANIZATION "AdamsNames Ltd"
    CONTACT-INFO    
        "Primary Contact: M J Oldfield
         email:     m@mail.tc"
    DESCRIPTION
        "This MIB module defines objects for lm_sensor derived data."
    REVISION     "200011050000Z"
    DESCRIPTION
        "Derived from DISKIO-MIB ex UCD."
    ::= { lmSensors 1 }

lmSensors      OBJECT IDENTIFIER ::= { ucdExperimental 16 }
```



この部分では MIB モジュールのメタ情報を定義しています。
更新日時、組織名、連絡先、説明、改訂履歴が書かれており、この MIB が `lm_sensor` 由来のデータを扱うこと、さらに UCD 系の `DISKIO-MIB` から派生していることが示されています。
また `lmSensors` は `ucdExperimental 16` として定義されているため、この MIB 全体は UCD-SNMP の experimental ツリー配下に置かれています。標準ツリーではなく拡張領域に属することがわかります。 

`ORGANIZATION "AdamsNames Ltd"`はMIB の作成主体、あるいは公開主体として AdamsNames Ltd を書いている。
`CONTACT-INFO "Primary Contact: M J Oldfield ..."`は、技術的な問い合わせ窓口が M J Oldfield であることを示す。
`email: m@mail.tc`は連絡先メール。.tc は Turks and Caicos Islands の ccTLD です。AdamsNames はこの .tc を含む複数の海外 ccTLD を英国企業として運営していたことが確認できます。

これは net-snmp の中核開発組織名ではなく、その MIB を持ち込んだ／管理していた側の表示です。LM-SENSORS-MIB 自体も UCD-SNMP-MIB 配下の ucdExperimental ツリーにぶら下がっており、標準 MIB というより、拡張的・実装依存の MIB です。ファイル中にも “Open issues: where to register this MIB?” というメモがあります。

次に、AdamsNames Ltd とは何か。これは英国 Companies House に登記されていた会社で、正式名は ADAMSNAMES LIMITED、会社番号は 03714632、現在は解散済みの法人のようです。
Companies House では Martin John Oldfield が同社の director だったことも確認できます。
したがって、MIB にある M J Oldfield はほぼこの人物を指すと見てよいです。

この英国企業 AdamsNames についてWEBで検索すると、Turks and Caicos (.tc)、Grenada (.gd)、British Virgin Islands (.vg) などの ccTLD を英国から実質運営し、2013 年に大きな運用・支配権トラブルを起こした、という記事を確認できます。

特に 2013 年には、AdamsNames を巡って .tc .gd .vg のレジストリ運営が混乱し、DomainIncite は「英国ベースの AdamsNames が nominally managed していた」と報じています。さらに ICANN SSAC の 2015 年文書でも、“Adamsnames incident” は 2013 年の業務紛争に起因し、元従業員が AdamsNames.net を掌握し、レジストリが一時的に UK から Turkey に移されたとされています。ここは単なる噂ではなく、後年の ICANN 文書にも事故例として残っています。

一方で、2013 年 3 月時点の業界報道では、当時の AdamsNames 側は「なりすまし」「不正な移行」だと主張しており、利害関係者の言い分は一致していないようです。したがって、何が“攻撃”で何が“社内・契約上の支配権争い”だったのかは、公開情報だけでは完全には断定できません。

### 3. 温度センサーテーブル

```text
lmTempSensorsTable OBJECT-TYPE
    SYNTAX      SEQUENCE OF LMTempSensorsEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION
        "Table of temperature sensors and their values."
    ::= { lmSensors 2 }

lmTempSensorsEntry OBJECT-TYPE
    SYNTAX      LMTempSensorsEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION
        "An entry containing a device and its statistics."
    INDEX       { lmTempSensorsIndex }
    ::= { lmTempSensorsTable 1 }

LMTempSensorsEntry ::= SEQUENCE {
    lmTempSensorsIndex    Integer32,
    lmTempSensorsDevice   DisplayString,
    lmTempSensorsValue    Gauge32
}
```



ここでは温度センサー用のテーブルを定義しています。
`lmTempSensorsTable` は温度センサーの一覧テーブル、`lmTempSensorsEntry` はその 1 行分の構造です。
エントリは 3 項目で構成されます。

* `lmTempSensorsIndex`
* `lmTempSensorsDevice`
* `lmTempSensorsValue`

`Table` と `Entry` 自体は `not-accessible` になっており、SNMP では表そのものではなく列オブジェクトを読む、という一般的なテーブル設計に従っています。 

### 4. 温度センサー各列

```text
lmTempSensorsIndex OBJECT-TYPE
    SYNTAX      Integer32 (0..65535)
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "Reference index for each observed device."
    ::= { lmTempSensorsEntry 1 }

lmTempSensorsDevice OBJECT-TYPE
    SYNTAX      DisplayString
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "The name of the temperature sensor we are reading."
    ::= { lmTempSensorsEntry 2 }

lmTempSensorsValue OBJECT-TYPE
    SYNTAX      Gauge32
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "The temperature of this sensor in mC."
    ::= { lmTempSensorsEntry 3 }
```



ここでは温度テーブルの各列の意味が定義されています。
`lmTempSensorsIndex` は行識別用の番号、`lmTempSensorsDevice` はセンサー名、`lmTempSensorsValue` はセンサー値です。
値の説明に `"in mC"` とあるので、単位はミリ摂氏です。小数ではなく整数で温度を表す設計になっています。
すべて `read-only` であり、この MIB は監視用であって設定変更用ではありません。 

### 5. ファンセンサーテーブル

```text
lmFanSensorsTable OBJECT-TYPE
    SYNTAX      SEQUENCE OF LMFanSensorsEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION
        "Table of fan sensors and their values."
    ::= { lmSensors 3 }

lmFanSensorsEntry OBJECT-TYPE
    SYNTAX      LMFanSensorsEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION
        "An entry containing a device and its statistics."
    INDEX       { lmFanSensorsIndex }
    ::= { lmFanSensorsTable 1 }

LMFanSensorsEntry ::= SEQUENCE {
    lmFanSensorsIndex    Integer32,
    lmFanSensorsDevice   DisplayString,
    lmFanSensorsValue    Gauge32
}
```



この部分はファンセンサー用のテーブルです。
構造は温度センサーのテーブルと同じで、インデックス、デバイス名、値の 3 列からなります。
テーブルの OID は `lmSensors 3` で、温度テーブルの次に配置されています。全体として、センサー種別ごとに同じ形のテーブルを並べる設計です。 

### 6. ファンセンサー各列

```text
lmFanSensorsIndex OBJECT-TYPE
    SYNTAX      Integer32 (0..65535)
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "Reference index for each observed device."
    ::= { lmFanSensorsEntry 1 }

lmFanSensorsDevice OBJECT-TYPE
    SYNTAX      DisplayString
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "The name of the fan sensor we are reading."
    ::= { lmFanSensorsEntry 2 }

lmFanSensorsValue OBJECT-TYPE
    SYNTAX      Gauge32
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "The rotation speed of the fan in RPM."
    ::= { lmFanSensorsEntry 3 }
```



ここではファンテーブルの列定義が行われています。
`lmFanSensorsDevice` はファン名、`lmFanSensorsValue` はファンの回転数で、説明文にある通り単位は RPM です。
温度と同じく値の型は `Gauge32` で、増減しうる瞬時値を表しています。累積カウンタではありません。 

### 7. 電圧センサーテーブル

```text
lmVoltSensorsTable OBJECT-TYPE
    SYNTAX      SEQUENCE OF LMVoltSensorsEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION
        "Table of voltage sensors and their values."
    ::= { lmSensors 4 }

lmVoltSensorsEntry OBJECT-TYPE
    SYNTAX      LMVoltSensorsEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION
        "An entry containing a device and its statistics."
    INDEX       { lmVoltSensorsIndex }
    ::= { lmVoltSensorsTable 1 }

LMVoltSensorsEntry ::= SEQUENCE {
    lmVoltSensorsIndex    Integer32,
    lmVoltSensorsDevice   DisplayString,
    lmVoltSensorsValue    Gauge32
}
```



この部分は電圧センサー用のテーブルです。
やはり設計は同型で、センサー種別だけが変わっています。
OID は `lmSensors 4` です。温度、ファン、電圧という監視でよく使う値が、独立したテーブルとして並べられています。 

### 8. 電圧センサー各列

```text
lmVoltSensorsIndex OBJECT-TYPE
    SYNTAX      Integer32 (0..65535)
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "Reference index for each observed device."
    ::= { lmVoltSensorsEntry 1 }

lmVoltSensorsDevice OBJECT-TYPE
    SYNTAX      DisplayString
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "The name of the device we are reading."
    ::= { lmVoltSensorsEntry 2 }

lmVoltSensorsValue OBJECT-TYPE
    SYNTAX      Gauge32
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "The voltage in mV."
    ::= { lmVoltSensorsEntry 3 }
```



ここでは電圧テーブルの各列を定義しています。
`lmVoltSensorsValue` の説明は `"The voltage in mV."` なので、単位はミリボルトです。
温度では mC、電圧では mV を使っており、整数値で物理量を表す設計が統一されています。管理ソフト側が必要に応じて表示用に換算する前提です。 

### 9. その他センサーテーブル

```text
lmMiscSensorsTable OBJECT-TYPE
    SYNTAX      SEQUENCE OF LMMiscSensorsEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION
        "Table of miscellaneous sensor devices and their values."
    ::= { lmSensors 5 }

lmMiscSensorsEntry OBJECT-TYPE
    SYNTAX      LMMiscSensorsEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION
        "An entry containing a device and its statistics."
    INDEX       { lmMiscSensorsIndex }
    ::= { lmMiscSensorsTable 1 }

LMMiscSensorsEntry ::= SEQUENCE {
    lmMiscSensorsIndex    Integer32,
    lmMiscSensorsDevice   DisplayString,
    lmMiscSensorsValue    Gauge32
}
```



ここでは「その他」のセンサー用テーブルを定義しています。
温度、ファン、電圧のいずれにも当てはまらないセンサー値を収めるための汎用的な受け皿です。
構造自体は他と同じで、統一的な設計を崩さないまま拡張余地を持たせています。 

### 10. その他センサー各列

```text
lmMiscSensorsIndex OBJECT-TYPE
    SYNTAX      Integer32 (0..65535)
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "Reference index for each observed device."
    ::= { lmMiscSensorsEntry 1 }

lmMiscSensorsDevice OBJECT-TYPE
    SYNTAX      DisplayString
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "The name of the device we are reading."
    ::= { lmMiscSensorsEntry 2 }

lmMiscSensorsValue OBJECT-TYPE
    SYNTAX      Gauge32
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
        "The value of this sensor."
    ::= { lmMiscSensorsEntry 3 }


END
```

ここではその他センサーテーブルの列定義と、モジュール終端が書かれています。
`lmMiscSensorsValue` の説明は `"The value of this sensor."` とだけあり、単位は固定されていません。
この点が、温度・ファン・電圧の専用テーブルとの違いです。つまりこのテーブルは、値の型だけを共通化し、意味や単位は個々のセンサー実装に委ねる形になっています。
最後の `END` で MIB 定義が終了します。 
