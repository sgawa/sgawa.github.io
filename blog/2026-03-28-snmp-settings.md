# Furukawa F70 の SNMP WALK / MIB 導入手順まとめ

## 1. 前提

対象機器:
- `192.168.0.70`

SNMP バージョン:
- SNMP v2c

コミュニティ:
- `public`

目的:
- `snmpwalk` を使って Furukawa F70 の SNMP 情報を取得する
- Furukawa 独自 MIB を読めるようにする
- `snmpwalk -v2c -c public 192.168.0.70 furukawa` のように名前付きで WALK できるようにする

---

## 2. `snmpwalk` のインストール

Ubuntu / Debian 系では `snmpwalk` は `snmp` パッケージに含まれる。

```bash
sudo apt update
sudo apt install snmp
````

確認:

```bash
snmpwalk --version
```

---

## 3. 基本の `snmpwalk` 実行方法

### 3.1 標準的な実行

```bash
snmpwalk -v2c -c public 192.168.0.70
```

これは標準 MIB 領域から WALK を開始するため、主に次のような情報が見える。

* `sysDescr`
* `sysObjectID`
* `sysUpTime`
* `ifNumber`
* `ifTable`

### 3.2 Furukawa 企業 OID 配下だけを WALK

```bash
snmpwalk -v2c -c public 192.168.0.70 .1.3.6.1.4.1.246
```

または、MIB 解決できる状態なら:

```bash
snmpwalk -v2c -c public 192.168.0.70 furukawa
```

`furukawa` は Furukawa の enterprise OID に対応する。

---

## 4. Furukawa MIB ZIP の取得と展開

今回は以下の ZIP を利用した。

* `f70_f71_f310asn1.zip`

展開例:

```bash
mkdir -p ~/mibs/furukawa
cd ~/mibs/furukawa
unzip f70_f71_f310asn1.zip
```

展開後の主な構成:

```text
f70_f71_f310asn1/
├── furukawa/
│   ├── furukawa-smi-v1.my
│   ├── furukawa.my
│   ├── infbase.my
│   ├── infonetProductID.my
│   └── ...
└── standards/
    ├── SNMPv2-SMI-rfc2578.my
    ├── SNMPv2-TC-rfc2579.my
    ├── IF-MIB-rfc2863.my
    └── ...
```

---

## 5. MIB ファイルの配置

### 5.1 配置先

一般的な配置先:

```bash
/usr/share/snmp/mibs/
```

今回は Furukawa 用にサブディレクトリを作る。

```bash
sudo mkdir -p /usr/share/snmp/mibs/furukawa
```

### 5.2 Furukawa MIB をコピー

```bash
sudo cp ~/mibs/furukawa/f70_f71_f310asn1/furukawa/* /usr/share/snmp/mibs/furukawa/
sudo cp ~/mibs/furukawa/f70_f71_f310asn1/standards/* /usr/share/snmp/mibs/furukawa/
```

---

## 6. `snmp.conf` の設定

## 6.1 どこに書くか

ユーザー単位:

```bash
~/.snmp/snmp.conf
```

システム全体:

```bash
/etc/snmp/snmp.conf
```

通常はまずユーザー単位が安全。

```bash
mkdir -p ~/.snmp
nano ~/.snmp/snmp.conf
```

---

## 6.2 `snmp.conf` の正しい書き方

`snmp.conf` ではシェルのように `MIBDIRS=` `MIBS=` とは書かない。

### 誤り

```conf
MIBDIRS=/usr/share/snmp/mibs/furukawa:/usr/share/snmp/mibs
MIBS=INFONET-PRODUCTID-MIB:FURUKAWA-MIB
```

### 正しい書き方

```conf
mibdirs /usr/share/snmp/mibs/furukawa:/usr/share/snmp/mibs
mibs INFONET-PRODUCTID-MIB:FURUKAWA-MIB:FURUKAWA-INFONETBASE-MIB:SNMPv2-MIB:IF-MIB
```

---

## 6.3 注意: `mibs :` の存在

Debian / Ubuntu 系では `snmp.conf` に次の行があることがある。

```conf
mibs :
```

これは MIB 自動読込を抑止する設定。

必要に応じてコメントアウトするか、必要な MIB を明示指定する。

例:

```conf
# mibs :
mibdirs /usr/share/snmp/mibs/furukawa:/usr/share/snmp/mibs
mibs INFONET-PRODUCTID-MIB:FURUKAWA-MIB:FURUKAWA-INFONETBASE-MIB:SNMPv2-MIB:IF-MIB
```

---

## 7. 一時的に MIB を指定して実行する方法

切り分け時は環境変数でも指定できる。

```bash
MIBDIRS=/usr/share/snmp/mibs/furukawa:/usr/share/snmp/mibs \
MIBS=INFONET-PRODUCTID-MIB:FURUKAWA-MIB:FURUKAWA-INFONETBASE-MIB:SNMPv2-MIB:IF-MIB \
snmpwalk -v2c -c public 192.168.0.70 .1.3.6.1.4.1.246
```

この方法は一時的な確認向け。
恒久運用は `snmp.conf` に書く。

---

## 8. MIB モジュール名とオブジェクト名の違い

`MIBS=` に書くのは **MIB モジュール名** である。
オブジェクト名やディレクトリ名は書けない。

### 誤り

```bash
MIBS=+infonetProductID:+furukawa
```

* `infonetProductID` はオブジェクト名
* `furukawa` は枝名 / ラベル名

### 正しい例

```bash
MIBS=INFONET-PRODUCTID-MIB:FURUKAWA-MIB:FURUKAWA-INFONETBASE-MIB:SNMPv2-MIB:IF-MIB
```

---

## 9. `snmptranslate` による確認

### 9.1 オブジェクト名を OID に変換

```bash
snmptranslate -IR -On infonetProductID
```

### 9.2 `FURUKAWA-SMI-v1` の定義一覧を見る

```bash
snmptranslate -m FURUKAWA-SMI-v1 -M /usr/share/snmp/mibs/furukawa:/usr/share/snmp/mibs -Tz
```

出力例:

```text
"furukawa"      "1.3.6.1.4.1.246"
"products"      "1.3.6.1.4.1.246.1"
"infonet"       "1.3.6.1.4.1.246.1.1"
"infonetBase"   "1.3.6.1.4.1.246.1.1.1"
```

これは `FURUKAWA-SMI-v1` が「枝の定義」を持つ土台 MIB であることを示す。

### 9.3 ツリー表示

```bash
snmptranslate -m FURUKAWA-SMI-v1 -M /usr/share/snmp/mibs/furukawa:/usr/share/snmp/mibs -Tp
```

---

## 10. `FURUKAWA-SMI-v1` とは何か

`FURUKAWA-SMI-v1` は主に以下を定義する。

* `furukawa`
* `products`
* `infonet`
* `infonetBase`
* `infonetMgt`

これは **親ノード / OID ツリーの土台** であり、通常は実データを持たない。

そのため、普通の `snmpwalk` では `FURUKAWA-SMI-v1::...` のような値が大量に見えるわけではない。
実際に値を持つのは、その下位にぶら下がる MIB である。

例:

* `FURUKAWA-INFONETBASE-MIB`
* `INFONET-PRODUCTID-MIB`

---

## 11. `snmpwalk host furukawa` と `snmpwalk host` の違い

### 11.1 Furukawa 配下だけ

```bash
snmpwalk -v2c -c public 192.168.0.70 furukawa
```

これは実質:

```bash
snmpwalk -v2c -c public 192.168.0.70 .1.3.6.1.4.1.246
```

Furukawa 独自 OID 配下だけを WALK する。

### 11.2 起点省略

```bash
snmpwalk -v2c -c public 192.168.0.70
```

これは標準 MIB 側から歩くため、先頭には主に以下が出る。

* `sysDescr.0`
* `sysObjectID.0`
* `sysUpTime.0`
* `ifNumber.0`
* `ifIndex.*`

Furukawa 独自情報が取れていないのではなく、**起点が標準 MIB 領域である** という違い。

---

## 12. `sysObjectID.0` の `.0` とは何か

`.0` は **scalar object の唯一のインスタンス識別子**。

例:

* `sysDescr.0`
* `sysObjectID.0`
* `sysUpTime.0`
* `ifNumber.0`

これらは 1 台につき 1 個しかない scalar であり、実際に参照するときは `.0` を付ける。

対してテーブルはインデックスが付く。

例:

* `ifDescr.100000000`
* `ifOperStatus.300000001`

---

## 13. 実際に確認できた Furukawa F70 の情報

標準 WALK から確認できた内容:

```text
sysDescr.0 = "F70 Version ..."
sysObjectID.0 = INFONET-PRODUCTID-MIB::infProdF70
sysUpTime.0 = ...
ifNumber.0 = 14
```

確認できた事実:

* 機種は F70
* `sysObjectID.0` は Furukawa 製品識別 OID を返している
* インターフェース数は 14
* 標準 MIB も Furukawa MIB も利用可能な状態

---

## 14. Furukawa 独自イベント WALK の例

```bash
snmpwalk -v2c -c public 192.168.0.70 furukawa
```

取得例:

```text
FURUKAWA-INFONETBASE-MIB::infonetEvent.1.1.0 = INTEGER: 6
FURUKAWA-INFONETBASE-MIB::infonetEvent.1.2.0 = INTEGER: 1
FURUKAWA-INFONETBASE-MIB::infonetEvent.1.3.1.1.1 = INTEGER: 1
...
```

これにより、Furukawa 独自イベントテーブルが参照できていることが確認できた。

---

## 15. 監視に使える主な項目

### 15.1 標準 MIB

| 項目        | オブジェクト          |
| --------- | --------------- |
| 機器説明      | `sysDescr.0`    |
| 製品識別      | `sysObjectID.0` |
| 稼働時間      | `sysUpTime.0`   |
| インターフェース数 | `ifNumber.0`    |
| IF 名      | `ifDescr`       |
| IF 状態     | `ifOperStatus`  |
| IF 速度     | `ifSpeed`       |
| 受信バイト     | `ifInOctets`    |
| 送信バイト     | `ifOutOctets`   |

### 15.2 Furukawa 独自

| 項目     | オブジェクト               |
| ------ | -------------------- |
| イベント情報 | `infonetEvent...`    |
| 製品 ID  | `infonetProductID` 系 |

---

## 16. よく使う確認コマンド

### 機器基本情報

```bash
snmpwalk -v2c -c public 192.168.0.70 sysDescr
snmpwalk -v2c -c public 192.168.0.70 sysObjectID
snmpwalk -v2c -c public 192.168.0.70 sysUpTime
```

### インターフェース確認

```bash
snmpwalk -v2c -c public 192.168.0.70 ifDescr
snmpwalk -v2c -c public 192.168.0.70 ifOperStatus
snmpwalk -v2c -c public 192.168.0.70 ifAlias
```

### Furukawa 独自領域

```bash
snmpwalk -v2c -c public 192.168.0.70 furukawa
snmpwalk -v2c -c public 192.168.0.70 .1.3.6.1.4.1.246
```

### 名前解決確認

```bash
snmptranslate -IR -On infonetProductID
snmptranslate -m FURUKAWA-SMI-v1 -M /usr/share/snmp/mibs/furukawa:/usr/share/snmp/mibs -Tz
```

---

## 17. 完成形の `~/.snmp/snmp.conf` 例

```conf
# Furukawa F70 用 MIB 設定
mibdirs /usr/share/snmp/mibs/furukawa:/usr/share/snmp/mibs
mibs INFONET-PRODUCTID-MIB:FURUKAWA-MIB:FURUKAWA-INFONETBASE-MIB:SNMPv2-MIB:IF-MIB
```

この設定を入れた後は、次のように簡単に実行できる。

```bash
snmpwalk -v2c -c public 192.168.0.70
snmpwalk -v2c -c public 192.168.0.70 furukawa
snmpwalk -v2c -c public 192.168.0.70 ifDescr
```

---

## 18. 運用上のポイント

* `MIBS=+ALL` は不要な MIB まで読み込み、警告が増えやすい
* Furukawa では必要な MIB だけ明示した方が安定する
* `FURUKAWA-SMI-v1` は土台 MIB であり、通常は値そのものの主役ではない
* `furukawa` という枝名を使って WALK できるようになれば、Furukawa MIB の読込は概ね成功している
* `sysObjectID.0 = INFONET-PRODUCTID-MIB::infProdF70` が見えていれば、製品識別系 MIB も機能している

---

## 19. 最終的な理解

* `snmpwalk -v2c -c public 192.168.0.70`

  * 標準 MIB 中心に表示される

* `snmpwalk -v2c -c public 192.168.0.70 furukawa`

  * Furukawa 独自 OID 配下を WALK する

* `FURUKAWA-SMI-v1`

  * Furukawa OID ツリーの土台を定義する SMI モジュール
  * 実データを持つ主役ではない

* `FURUKAWA-INFONETBASE-MIB`

  * Furukawa 独自イベント等の実オブジェクトを提供する MIB

* `INFONET-PRODUCTID-MIB`

  * F70 などの製品識別に関与する MIB
