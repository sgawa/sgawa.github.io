# libsnmp-base

`libsnmp-base` は Net-SNMP ファミリのMIB/設定/文書の基盤パッケージです。

[https://packages.debian.org/bookworm/libsnmp-base](https://packages.debian.org/bookworm/libsnmp-base)


## Debian

Debian では `libsnmp-base` は stable の `net-snmp` ソースから提供される正式パッケージです。trixie 安定版では `net-snmp` ソースから `libsnmp-base`, `libsnmp`, `snmp`, `snmpd` などが生成されています。`snmp-mibs-downloader` は別ソース・別パッケージで、bookworm では 1.5、trixie では 1.7 です。

Debian で重要なのは次の2点です。

1. **`libsnmp-base` は main にあるが、標準 MIB 一式は別問題**
   MIB のライセンス事情があり、読み込みはデフォルト無効

2. **その穴を埋めるのが `snmp-mibs-downloader`**
   ただしこちらは non-free 系統です。bookworm の package 説明でも `[non-free]` とされています。

したがって Debian では、**フリーなベース部分は `libsnmp-base`、ライセンス上分離された追加 MIB は `snmp-mibs-downloader`** という切り分けです。

## Ubuntu

Ubuntu も基本的には Debian 由来の `net-snmp` パッケージ構成を継承しています。Ubuntu 24.04 noble の `libsnmp-base` は `net-snmp` ヘッドパッケージ配下の `main` にあり、説明も Debian と同様に「SNMP configuration script, MIBs and documentation」です。Ubuntu 22.04 jammy でも同様です。

一方で `snmp-mibs-downloader` は Ubuntu では常に同じ扱いではなく、少なくとも **jammy では `download-mibs(1)` が `snmp-mibs-downloader_1.5_all` 提供**で確認できます。外部のパッケージ集約情報では jammy は multiverse、noble は multiverse で提供されています。これは Debian の non-free に相当する「追加的・制約付きコンテンツ」扱いに近い理解でよいです。([Ubuntu Manpage][10])

要するに Ubuntu では、

* **`libsnmp-base` は Debian 同様に Net-SNMP ベース部分**
* **追加 MIB は `snmp-mibs-downloader` 側**
* **利用時は `/etc/snmp/snmp.conf` の `mibs :` を見直す**

という運用はほぼ同じです。


`libsnmp-base` に **入っていない** ものとして主に以下が挙がる。

* `SNMPv2-MIB`
* `IF-MIB`
* `IP-MIB`
* `TCP-MIB`
* `UDP-MIB`
* `HOST-RESOURCES-MIB`
* `DISMAN-*`

これらは標準 MIB として広く使われるが、Debian/Ubuntu ではそのまま `libsnmp-base` に全部は同梱しない内容です。実際、Debian 系では MIB 読み込み自体がデフォルト無効にされており、その理由として「ライセンス都合で SNMP パッケージに MIB が入っていないため」と説明されています。`snmp-mibs-downloader` 側の README でも、利用には `/etc/snmp/snmp.conf` の `mibs :` 行をコメントアウトするよう案内されています。


## snmp-mibs-downloader

あなたが引用した説明はその通りで、`snmp-mibs-downloader` は **IETF RFC や IANA 由来の MIB を取得・抽出・配置するための補完パッケージ**です。Debian bookworm の package 説明では、RFC に含まれる MIB を抽出して SNMP ライブラリで使える形にし、一部は最新版更新やベンダ追加 MIB の取得にも使えるとされています。Ubuntu jammy の `download-mibs(1)` でも、IETF・IANA などから MIB をダウンロードし、抽出・整列すると説明されています。([Debian Packages][5])

整理すると、役割分担はこうです。

| パッケージ                  | 役割                                                     |
| ---------------------- | ------------------------------------------------------ |
| `libsnmp-base`         | Net-SNMP 系の基本 MIB、設定、ドキュメントを提供                         |
| `snmp-mibs-downloader` | Debian/Ubuntu 本体に同梱されない IETF/IANA/追加 vendor MIB を取得・展開 |
| `snmp`                 | `snmpget`, `snmpwalk`, `snmptranslate` などの実行ツール        |
| `snmpd`                | SNMP エージェント本体                                          |

実運用では、**`snmp` だけ入れると OID が数字のまま出やすい**、**`libsnmp-base` だけでは標準 MIB が不足しやすい**、そこで **`snmp-mibs-downloader` を追加して `mibs :` を無効化解除する**、という流れになります。`snmp-mibs-downloader` の README.Debian でも、`mibs :` をコメントアウトすると「ライブラリが見つけた全 MIB をロードしようとする」と明示されています。

## MIB

実際に覚えておくべきことだけ絞ると、こうです。

### `libsnmp-base` で取得できるMIB

* Net-SNMP 独自 MIB や UCD 系 MIB の名前解決
* `snmp.conf` など設定・MIB 配置の土台
* `snmpd` の拡張機能 (`extend`, `pass`) まわりの MIB 利用

### `libsnmp-base` だけでは足りないMIB

* `IF-MIB::ifDescr` や `SNMPv2-MIB::sysName.0` など、標準 MIB の名前解決が十分でないことがある
* ベンダ MIB の解決
* Wireshark や `snmptranslate` で期待した名前に変換されないことがある

### 追加で必要になるもの

* `snmp-mibs-downloader`
* 必要ならベンダ MIB の追加配置
* `/etc/snmp/snmp.conf` の `mibs :` 無効化解除

## まとめ

* **`libsnmp-base` は Net-SNMP ファミリの「MIB/設定/文書の基盤パッケージ」**です。Debian stable でも Ubuntu 24.04/22.04 でもその位置づけは同じです。([Debian Packages][1])
* あなたの環境で実際に見えている MIB は、**Net-SNMP 独自 MIB + UCD 系 MIB + LM-SENSORS-MIB** が中心です。
* **IETF 標準 MIB 一式は `libsnmp-base` に全部入るわけではない**ため、Debian/Ubuntu では **`snmp-mibs-downloader` が補完役**になります。([Debian Packages][5])
* Debian では `snmp-mibs-downloader` は **non-free**、Ubuntu では少なくとも jammy で `snmp-mibs-downloader_1.5_all` が確認でき、追加リポジトリ側で扱われます。([Debian Packages][7])
* `snmpget`, `snmpwalk`, Wireshark を **「数値 OID ではなく人間可読名で使いたい」**なら、`libsnmp-base` だけでは不十分で、`snmp-mibs-downloader` と MIB 読み込み設定が実質必要です。([Debian Packages][5])

## libsnmp-base で取得できる MIB 一覧

libsnmp-base で取得できるMIBは以下の通り。Debian の net-snmp のソースには Net-SNMP 独自 MIB、UCD 系 MIB、追加の Debian 向け MIB が含まれている。


| MIBファイル                     | 系統         | 用途の要約                                     |
| --------------------------- | ---------- | ----------------------------------------- |
| `LM-SENSORS-MIB.txt`        | lm-sensors | 温度、電圧、ファン回転数などのハードウェアセンサー情報               |
| `NET-SNMP-AGENT-MIB.txt`    | Net-SNMP独自 | Net-SNMP エージェント自身の動作・設定に関する情報             |
| `NET-SNMP-EXAMPLES-MIB.txt` | Net-SNMP独自 | Net-SNMP のサンプル実装・例示用                      |
| `NET-SNMP-EXTEND-MIB.txt`   | Net-SNMP独自 | `extend` 機能で外部コマンド結果を SNMP ツリーに載せるための MIB |
| `NET-SNMP-MIB.txt`          | Net-SNMP独自 | Net-SNMP 全体の基礎定義                          |
| `NET-SNMP-PASS-MIB.txt`     | Net-SNMP独自 | `pass` / `pass_persist` 連携向け              |
| `NET-SNMP-TC.txt`           | Net-SNMP独自 | Net-SNMP 独自の Textual Convention           |
| `NET-SNMP-VACM-MIB.txt`     | Net-SNMP独自 | VACM (View-based Access Control Model) 関連 |
| `UCD-DEMO-MIB.txt`          | UCD-SNMP系  | 旧 UCD-SNMP 系のデモ用途                         |
| `UCD-DISKIO-MIB.txt`        | UCD-SNMP系  | ディスク I/O 統計                               |
| `UCD-DLMOD-MIB.txt`         | UCD-SNMP系  | 動的ロードモジュール関連                              |
| `UCD-IPFWACC-MIB.txt`       | UCD-SNMP系  | IP firewall/accounting 系                  |
| `UCD-SNMP-MIB.txt`          | UCD-SNMP系  | 旧 UCD-SNMP 由来の基本 MIB    



```bash
$ dpkg -L libsnmp-base 
/. 
/etc 
/etc/snmp 
/etc/snmp/snmp.conf 
/usr

/usr/share
/usr/share/doc
/usr/share/doc/libsnmp-base
/usr/share/doc/libsnmp-base/FAQ.gz
/usr/share/doc/libsnmp-base/NEWS.gz
/usr/share/doc/libsnmp-base/README.gz
/usr/share/doc/libsnmp-base/README.mibs
/usr/share/doc/libsnmp-base/README.snmpv3
/usr/share/doc/libsnmp-base/README.thread.gz
/usr/share/doc/libsnmp-base/changelog.Debian.gz
/usr/share/doc/libsnmp-base/copyright
/usr/share/man
/usr/share/man/man5
/usr/share/man/man5/mib2c.conf.5.gz
/usr/share/man/man5/snmp.conf.5.gz
/usr/share/man/man5/snmp_config.5.gz
/usr/share/man/man5/snmpd.examples.5.gz
/usr/share/man/man5/snmpd.internal.5.gz
/usr/share/man/man5/variables.5.gz

/usr/share/snmp
/usr/share/snmp/mib2c-data
/usr/share/snmp/mib2c-data/default-mfd-top.m2c
/usr/share/snmp/mib2c-data/details-enums.m2i
/usr/share/snmp/mib2c-data/details-node.m2i
/usr/share/snmp/mib2c-data/details-table.m2i
/usr/share/snmp/mib2c-data/generic-ctx-copy.m2i
/usr/share/snmp/mib2c-data/generic-ctx-get.m2i
/usr/share/snmp/mib2c-data/generic-ctx-set.m2i
/usr/share/snmp/mib2c-data/generic-data-allocate.m2i
/usr/share/snmp/mib2c-data/generic-data-context.m2i
/usr/share/snmp/mib2c-data/generic-get-U64.m2i
/usr/share/snmp/mib2c-data/generic-get-char.m2i
/usr/share/snmp/mib2c-data/generic-get-decl-bot.m2i
/usr/share/snmp/mib2c-data/generic-get-decl.m2i
/usr/share/snmp/mib2c-data/generic-get-long.m2i
/usr/share/snmp/mib2c-data/generic-get-oid.m2i
/usr/share/snmp/mib2c-data/generic-header-bottom.m2i
/usr/share/snmp/mib2c-data/generic-header-top.m2i
/usr/share/snmp/mib2c-data/generic-source-includes.m2i
/usr/share/snmp/mib2c-data/generic-table-constants.m2c
/usr/share/snmp/mib2c-data/generic-table-enums.m2c
/usr/share/snmp/mib2c-data/generic-table-indexes-from-oid.m2i
/usr/share/snmp/mib2c-data/generic-table-indexes-set.m2i
/usr/share/snmp/mib2c-data/generic-table-indexes-to-oid.m2i
/usr/share/snmp/mib2c-data/generic-table-indexes-varbind-setup.m2i
/usr/share/snmp/mib2c-data/generic-table-indexes.m2i
/usr/share/snmp/mib2c-data/generic-table-oids.m2c
/usr/share/snmp/mib2c-data/generic-value-map-func.m2i
/usr/share/snmp/mib2c-data/generic-value-map-reverse.m2i
/usr/share/snmp/mib2c-data/generic-value-map.m2i
/usr/share/snmp/mib2c-data/m2c-internal-warning.m2i
/usr/share/snmp/mib2c-data/m2c_setup_enum.m2i
/usr/share/snmp/mib2c-data/m2c_setup_node.m2i
/usr/share/snmp/mib2c-data/m2c_setup_table.m2i
/usr/share/snmp/mib2c-data/m2c_table_save_defaults.m2i
/usr/share/snmp/mib2c-data/mfd-access-container-cached-defines.m2i
/usr/share/snmp/mib2c-data/mfd-access-unsorted-external-defines.m2i
/usr/share/snmp/mib2c-data/mfd-data-access.m2c
/usr/share/snmp/mib2c-data/mfd-data-get.m2c
/usr/share/snmp/mib2c-data/mfd-data-set.m2c
/usr/share/snmp/mib2c-data/mfd-doxygen.m2c
/usr/share/snmp/mib2c-data/mfd-interactive-setup.m2c
/usr/share/snmp/mib2c-data/mfd-interface.m2c
/usr/share/snmp/mib2c-data/mfd-makefile.m2m
/usr/share/snmp/mib2c-data/mfd-persistence.m2i
/usr/share/snmp/mib2c-data/mfd-readme.m2c
/usr/share/snmp/mib2c-data/mfd-top.m2c
/usr/share/snmp/mib2c-data/node-get.m2i
/usr/share/snmp/mib2c-data/node-set.m2i
/usr/share/snmp/mib2c-data/node-storage.m2i
/usr/share/snmp/mib2c-data/node-validate.m2i
/usr/share/snmp/mib2c-data/node-varbind-validate.m2i
/usr/share/snmp/mib2c-data/parent-dependencies.m2i
/usr/share/snmp/mib2c-data/parent-set.m2i
/usr/share/snmp/mib2c-data/subagent.m2c
/usr/share/snmp/mib2c-data/syntax-COUNTER64-get.m2i
/usr/share/snmp/mib2c-data/syntax-DateAndTime-get.m2d
/usr/share/snmp/mib2c-data/syntax-DateAndTime-get.m2i
/usr/share/snmp/mib2c-data/syntax-DateAndTime-readme.m2i
/usr/share/snmp/mib2c-data/syntax-InetAddress-get.m2i
/usr/share/snmp/mib2c-data/syntax-InetAddress-set.m2i
/usr/share/snmp/mib2c-data/syntax-InetAddressType-get.m2i
/usr/share/snmp/mib2c-data/syntax-InetAddressType-set.m2i
/usr/share/snmp/mib2c-data/syntax-RowStatus-dependencies.m2i
/usr/share/snmp/mib2c-data/syntax-RowStatus-get.m2i
/usr/share/snmp/mib2c-data/syntax-RowStatus-varbind-validate.m2i
/usr/share/snmp/mib2c-data/syntax-StorageType-dependencies.m2i
/usr/share/snmp/mib2c-data/syntax-TestAndIncr-get.m2i

/usr/share/snmp/mibs
/usr/share/snmp/mibs/LM-SENSORS-MIB.txt
/usr/share/snmp/mibs/NET-SNMP-AGENT-MIB.txt
/usr/share/snmp/mibs/NET-SNMP-EXAMPLES-MIB.txt
/usr/share/snmp/mibs/NET-SNMP-EXTEND-MIB.txt
/usr/share/snmp/mibs/NET-SNMP-MIB.txt
/usr/share/snmp/mibs/NET-SNMP-PASS-MIB.txt
/usr/share/snmp/mibs/NET-SNMP-TC.txt
/usr/share/snmp/mibs/NET-SNMP-VACM-MIB.txt
/usr/share/snmp/mibs/UCD-DEMO-MIB.txt
/usr/share/snmp/mibs/UCD-DISKIO-MIB.txt
/usr/share/snmp/mibs/UCD-DLMOD-MIB.txt
/usr/share/snmp/mibs/UCD-IPFWACC-MIB.txt
/usr/share/snmp/mibs/UCD-SNMP-MIB.txt 

/var 
/var/lib 
/var/lib/snmp
```