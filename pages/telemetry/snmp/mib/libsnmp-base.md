# libsnmp-base

[https://packages.debian.org/bookworm/libsnmp-base](https://packages.debian.org/bookworm/libsnmp-base)


## libsnmp-base MIBs

```bash
dpkg -L libsnmp-base | grep mibs
/usr/share/doc/libsnmp-base/README.mibs
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
```

| MIB file | 系統 | 用途 |
| :--- | :--- | :--- |
| [LM-SENSORS-MIB.txt](/usr/share/snmp/mibs/LM-SENSORS-MIB.txt) | lm-sensors | 温度、電圧、ファン回転数などのハードウェアセンサー情報 |
| [NET-SNMP-AGENT-MIB.txt](/usr/share/snmp/mibs/NET-SNMP-AGENT-MIB.txt) | Net-SNMP独自 | Net-SNMP エージェント自身の動作・設定に関する情報 |
| [NET-SNMP-EXAMPLES-MIB.txt](/usr/share/snmp/mibs/NET-SNMP-EXAMPLES-MIB.txt) | Net-SNMP独自 | Net-SNMP のサンプル実装・例示用 |
| [NET-SNMP-EXTEND-MIB.txt](/usr/share/snmp/mibs/NET-SNMP-EXTEND-MIB.txt) | Net-SNMP独自 | `extend` 機能で外部コマンド結果を SNMP ツリーに載せるための MIB |
| [NET-SNMP-MIB.txt](/usr/share/snmp/mibs/NET-SNMP-MIB.txt) | Net-SNMP独自 | Net-SNMP 全体の基礎定義 |
| [NET-SNMP-PASS-MIB.txt](/usr/share/snmp/mibs/NET-SNMP-PASS-MIB.txt) | Net-SNMP独自 | `pass` / `pass_persist` 連携向け |
| [NET-SNMP-TC.txt](/usr/share/snmp/mibs/NET-SNMP-TC.txt) | Net-SNMP独自 | Net-SNMP 独自の Textual Convention |
| [NET-SNMP-VACM-MIB.txt](/usr/share/snmp/mibs/NET-SNMP-VACM-MIB.txt) | Net-SNMP独自 | VACM (View-based Access Control Model) 関連 |
| [UCD-DEMO-MIB.txt](/usr/share/snmp/mibs/UCD-DEMO-MIB.txt) | UCD-SNMP系 | 旧 UCD-SNMP 系のデモ用途 |
| [UCD-DISKIO-MIB.txt](/usr/share/snmp/mibs/UCD-DISKIO-MIB.txt) | UCD-SNMP系 | ディスク I/O 統計 |
| [UCD-DLMOD-MIB.txt](/usr/share/snmp/mibs/UCD-DLMOD-MIB.txt) | UCD-SNMP系 | 動的ロードモジュール関連 |
| [UCD-IPFWACC-MIB.txt](/usr/share/snmp/mibs/UCD-IPFWACC-MIB.txt) | UCD-SNMP系 | IP firewall/accounting 系 |
| [UCD-SNMP-MIB.txt](/usr/share/snmp/mibs/UCD-SNMP-MIB.txt) | UCD-SNMP系 | 旧 UCD-SNMP 由来の基本 MIB |

### `cat /usr/share/doc/libsnmp-base/README.mibs`
```
About the MIBS distributed with Net-SNMP.

This directory contains a very basic set of MIB files, ready for use.
In addition, there are some scripts and table files to help you get a
fuller collection of MIB files.

smistrip - a script that can extract a MIB file from an RFC (or I-D)
mibfetch - a script that will fetch an RFC file from a mirror, and extract
	the hosted MIB from it. It assumes that you have wget installed.
rfclist - a list of RFC numbers and corresponding MIB name(s)
ianalist - a list of files at the IANA server that holds IANA maintained
	MIBs
Makefile.mib - rules for extracting current MIB files from RFC and IANA
	files.
rfcmibs.diff - a set of required patches for MIB files extracted from RFCs

The file Makefile.mib holds rules that fetch and extract MIB files from
their hosting RFCs. Make will use wget to retrieve the RFC files, and,
as I am located in Denmark, use the RFC mirror at NORDUnet. You may change
that at the top of Makefile.mib.

Makefile.mib also holds rules that will collect all the current IETF MIB
definitions, using the lists in rfclist and ianalist. To get them all,
use
	make -f Makefile.mib allmibs

Note, that there are a few fatal syntactic errors in some of the RFC
definitions. To make them all parse successfully with the Net-SNMP parser,
you should apply the patches in the file rfcmibs.diff. These patches are
typical for the problems that are commonly seen with MIB files from various
sources:

- forgetting to import enterprises/mib-2/transmission from SNMPv2-SMI
- thinking that a -- comment ends at end-of-line, not at the next --
- using _ in identifiers. A - may be used in its place
- various misspellings

There is a short-cut rule
	make -f Makefile.mib rfc
that will also apply the patches. Note that Makefile.mib and smistrip has
configurable versions of awk and patch. If you are running Solaris you
must set these to nawk and gpatch respectively.

DISCLAIMER: The patches provided here for the IETF standard MIB files
are not endorsed by anyone, and I don't guarantee that they bring them
accordance with what the authors intended. All I will promise, is that
the MIB files can be parsed.
```

### (参考) `README.mibs` 和訳

```
Net-SNMPに同梱されているMIBについて。

このディレクトリには、すぐに使える非常に基本的な MIB ファイル一式が含まれています。
さらに、より充実した MIB コレクションを得るためのスクリプトや
テーブルファイルもいくつか含まれています。

smistrip - RFC（または I-D）から MIB ファイルを抽出できるスクリプト
mibfetch - ミラーから RFC ファイルを取得し、そこに含まれる MIB を抽出するスクリプト。
            wget がインストールされていることを前提としています。
rfclist  - RFC 番号と対応する MIB 名の一覧
ianalist - IANA サーバ上にある、IANA が管理している MIB を保持するファイルの一覧
Makefile.mib - RFC および IANA のファイルから、現在の MIB ファイルを抽出するためのルール
rfcmibs.diff - RFC から抽出した MIB ファイルに必要なパッチ一式

Makefile.mib には、MIB ファイルを掲載元の RFC から取得して抽出するための
ルールが定義されています。make は RFC ファイルの取得に wget を使用し、
また、私がデンマークにいるため、RFC ミラーとして NORDUnet を使うように
なっています。これは Makefile.mib の先頭で変更できます。

Makefile.mib には、rfclist と ianalist の一覧を使って、現在の IETF MIB
定義をすべて集めるルールも定義されています。全部取得するには、
 make -f Makefile.mib allmibs
を使用してください。

なお、いくつかの RFC の定義には致命的な構文エラーがあります。これらを
すべて Net-SNMP パーサで正しく解析できるようにするには、
rfcmibs.diff ファイル内のパッチを適用する必要があります。これらの
パッチは、さまざまな入手元の MIB ファイルでよく見られる問題の典型例に
対応したものです。

- enterprises/mib-2/transmission を SNMPv2-SMI から import し忘れている
- -- コメントは行末で終わるのではなく、次の -- まで続くことを
  認識していない
- 識別子に _ を使っている。代わりに - を使用できます
- さまざまな綴り間違い

短縮ルールとして、
  make -f Makefile.mib rfc
も用意されており、これを使うとパッチも適用されます。なお、
Makefile.mib と smistrip では、awk と patch の使用コマンドを設定可能です。
Solaris を使っている場合は、それぞれ nawk と gpatch に設定しなければ
なりません。

免責事項: ここで提供される IETF 標準 MIB ファイル向けパッチは、誰からも
承認されたものではなく、また、それらが著者の意図どおりの内容になることを
保証するものでもありません。私が約束できるのは、MIB ファイルが
解析可能になることだけです。
```

(備考)
* NORDUnet とは、北欧諸国の国立研究教育ネットワーク間の国際協力のこと。