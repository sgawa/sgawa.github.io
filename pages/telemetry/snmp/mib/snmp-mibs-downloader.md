# snmp-mibs-downloader

[https://packages.debian.org/ja/sid/snmp-mibs-downloader](https://packages.debian.org/ja/sid/snmp-mibs-downloader)

install and manage Management Information Base (MIB) files
This package ships the IETF RFCs containing MIB files and a script which extracts them to be used by Simple Network Management libraries. The script can be used to update some MIBs to the latest version or to download extra vendor MIBs.
These MIBs can be useful for programs like wireshark or snmpget to enable them to translating the received information into human readable text.

### MIBs

| MIB名                                    | ベンダ名 OR RFC                |  行数  | 種別                 |
| :-------------------------------------- | :------------------------- | :--: | :----------------- |
| (empty)                                 | 不明                         |      | 不明                 |
| ACCOUNTING-CONTROL-MIB                  | IETF                       |  768 | その他                |
| ADSL-LINE-EXT-MIB                       | IETF DSL Forum / DSL       | 1169 | DSL                |
| ADSL-LINE-MIB                           | IETF DSL Forum / DSL       | 4328 | DSL                |
| ADSL-TC-MIB                             | IETF DSL Forum / DSL       |  113 | DSL                |
| ADSL2-LINE-MIB                          | IETF DSL Forum / DSL       | 5476 | DSL                |
| ADSL2-LINE-TC-MIB                       | IETF DSL Forum / DSL       |  729 | DSL                |
| AGENTX-MIB                              | IETF SNMP                  |  527 | SNMP               |
| AGGREGATE-MIB                           | IETF                       |  477 | その他                |
| ALARM-MIB                               | IETF                       | 1127 | 管理/監視              |
| APM-MIB                                 | IETF                       | 2127 | その他                |
| APPC-MIB                                | IBM / APPN                 | 5104 | IBM/SNA            |
| APPLETALK-MIB                           | IETF                       | 3398 | その他                |
| APPLICATION-MIB                         | IETF                       | 2995 | ホスト/アプリ/印刷         |
| APPN-DLUR-MIB                           | IBM / APPN                 |  632 | IBM/SNA            |
| APPN-MIB                                | IBM / APPN                 | 5611 | IBM/SNA            |
| APPN-TRAP-MIB                           | IBM / APPN                 |  477 | IBM/SNA            |
| APS-MIB                                 | IETF                       | 1659 | その他                |
| ARC-MIB                                 | IETF                       |  396 | その他                |
| ATM-ACCOUNTING-INFORMATION-MIB          | IETF WAN/Carrier           |  402 | WAN/伝送             |
| ATM-MIB                                 | IETF WAN/Carrier           | 2995 | WAN/伝送             |
| ATM-TC-MIB                              | IETF WAN/Carrier           |  713 | WAN/伝送             |
| ATM2-MIB                                | IETF WAN/Carrier           | 3220 | WAN/伝送             |
| BGP4-MIB                                | IETF BGP                   | 1232 | ルーティング             |
| BRIDGE-MIB                              | IETF Bridge                | 1472 | ブリッジ/STP           |
| CAPWAP-BASE-MIB                         | IETF CAPWAP                | 2618 | その他                |
| CAPWAP-DOT11-MIB                        | IETF CAPWAP                |  369 | その他                |
| CHARACTER-MIB                           | IETF                       |  646 | その他                |
| CIRCUIT-IF-MIB                          | IETF                       |  369 | その他                |
| CLNS-MIB                                | IETF                       | 1294 | その他                |
| COPS-CLIENT-MIB                         | IETF                       |  844 | その他                |
| DECNET-PHIV-MIB                         | DEC                        | 3030 | その他                |
| DIAL-CONTROL-MIB                        | IETF                       | 1270 | その他                |
| DIFFSERV-CONFIG-MIB                     | IETF                       |  243 | その他                |
| DIFFSERV-DSCP-TC                        | IETF / IANA                |  64  | テキスト規約             |
| DIFFSERV-MIB                            | IETF                       | 3514 | その他                |
| DIRECTORY-SERVER-MIB                    | IETF                       |  772 | その他                |
| DISMAN-EVENT-MIB                        | IETF DISMAN                | 1882 | 管理/監視              |
| DISMAN-EXPRESSION-MIB                   | IETF DISMAN                | 1182 | 管理/監視              |
| DISMAN-NSLOOKUP-MIB                     | IETF DISMAN                |  509 | DNS                |
| DISMAN-PING-MIB                         | IETF DISMAN                | 1561 | 管理/監視              |
| DISMAN-SCHEDULE-MIB                     | IETF DISMAN                |  699 | 管理/監視              |
| DISMAN-SCRIPT-MIB                       | IETF DISMAN                | 1764 | 管理/監視              |
| DISMAN-TRACEROUTE-MIB                   | IETF DISMAN                | 1850 | 管理/監視              |
| DLSW-MIB                                | IETF                       | 3560 | その他                |
| DNS-RESOLVER-MIB                        | IETF DNS                   | 1196 | DNS                |
| DNS-SERVER-MIB                          | IETF DNS                   | 1078 | DNS                |
| DOCS-BPI-MIB                            | CableLabs / DOCSIS         | 1569 | DOCSIS/PacketCable |
| DOCS-CABLE-DEVICE-MIB                   | CableLabs / DOCSIS         | 3141 | DOCSIS/PacketCable |
| DOCS-IETF-BPI2-MIB                      | CableLabs / DOCSIS         | 3451 | DOCSIS/PacketCable |
| DOCS-IETF-CABLE-DEVICE-NOTIFICATION-MIB | CableLabs / DOCSIS         | 1453 | DOCSIS/PacketCable |
| DOCS-IETF-QOS-MIB                       | CableLabs / DOCSIS         | 3060 | DOCSIS/PacketCable |
| DOCS-IETF-SUBMGT-MIB                    | CableLabs / DOCSIS         |  672 | DOCSIS/PacketCable |
| DOCS-IF-MIB                             | CableLabs / DOCSIS         | 5291 | DOCSIS/PacketCable |
| DOT12-IF-MIB                            | IEEE / IETF                |  772 | L2/インタフェース         |
| DOT12-RPTR-MIB                          | IEEE / IETF                | 1978 | L2/インタフェース         |
| DOT3-EPON-MIB                           | IEEE / IETF                | 2532 | L2/インタフェース         |
| DOT3-OAM-MIB                            | IEEE / IETF                | 2115 | L2/インタフェース         |
| DPI20-MIB                               | IETF                       |  47  | その他                |
| DS0-MIB                                 | IETF WAN/Carrier           |  305 | WAN/伝送             |
| DS0BUNDLE-MIB                           | IETF WAN/Carrier           |  311 | WAN/伝送             |
| DS1-MIB                                 | IETF WAN/Carrier           | 3015 | WAN/伝送             |
| DS3-MIB                                 | IETF WAN/Carrier           | 1786 | WAN/伝送             |
| DSA-MIB                                 | IETF                       |  642 | その他                |
| DSMON-MIB                               | IETF RMON                  | 4449 | 計測/フロー監視           |
| DVB-RCS-MIB                             | DVB Project                | 3329 | その他                |
| EBN-MIB                                 | IBM / APPN                 |  702 | IBM/SNA            |
| EFM-CU-MIB                              | IETF                       | 2995 | L2/インタフェース         |
| ENTITY-MIB                              | IETF Entity MIB            | 1411 | エンティティ/センサー        |
| ENTITY-SENSOR-MIB                       | IETF Entity MIB            |  440 | エンティティ/センサー        |
| ENTITY-STATE-MIB                        | IETF Entity MIB            |  332 | エンティティ/センサー        |
| ENTITY-STATE-TC-MIB                     | IETF Entity MIB            |  169 | エンティティ/センサー        |
| ETHER-CHIPSET-MIB                       | IETF                       |  532 | その他                |
| EtherLike-MIB                           | IETF Ethernet              | 1862 | L2/インタフェース         |
| FC-MGMT-MIB                             | INCITS T11 / Fibre Channel | 2205 | ストレージSAN           |
| FCIP-MGMT-MIB                           | INCITS T11 / Fibre Channel | 1037 | ストレージSAN           |
| FDDI-SMT73-MIB                          | IETF                       | 2126 | その他                |
| FIBRE-CHANNEL-FE-MIB                    | INCITS T11 / Fibre Channel | 1781 | ストレージSAN           |
| Finisher-MIB                            | IETF Printer               |  869 | ホスト/アプリ/印刷         |
| FLOW-METER-MIB                          | IETF                       | 1901 | 計測/フロー監視           |
| FORCES-MIB                              | IETF                       |  391 | その他                |
| FR-ATM-PVC-SERVICE-IWF-MIB              | IETF WAN/Carrier           | 1066 | WAN/伝送             |
| FR-MFR-MIB                              | IETF WAN/Carrier           |  888 | WAN/伝送             |
| FRAME-RELAY-DTE-MIB                     | IETF WAN/Carrier           |  992 | WAN/伝送             |
| FRNETSERV-MIB                           | IETF WAN/Carrier           | 2479 | WAN/伝送             |
| FRSLD-MIB                               | IETF WAN/Carrier           | 1768 | WAN/伝送             |
| FIZBIN-MIB                              | IETF                       |      | その他                |
| GMPLS-LABEL-STD-MIB                     | IETF GMPLS                 |  689 | MPLS/PWE3          |
| GMPLS-LSR-STD-MIB                       | IETF GMPLS                 |  503 | MPLS/PWE3          |
| GMPLS-TC-STD-MIB                        | IETF GMPLS                 |  124 | MPLS/PWE3          |
| GMPLS-TE-STD-MIB                        | IETF GMPLS                 | 1749 | MPLS/PWE3          |
| GSMP-MIB                                | IETF                       | 1582 | その他                |
| HC-ALARM-MIB                            | IETF                       |  707 | 管理/監視              |
| HC-PerfHist-TC-MIB                      | IETF                       |  222 | テキスト規約             |
| HC-RMON-MIB                             | IETF RMON                  | 3149 | 計測/フロー監視           |
| HCNUM-TC                                | IETF                       |  118 | テキスト規約             |
| HDSL2-SHDSL-LINE-MIB                    | IETF DSL Forum / DSL       | 2503 | DSL                |
| HOST-RESOURCES-MIB                      | IETF Host Resources        | 1540 | ホスト/アプリ/印刷         |
| HOST-RESOURCES-TYPES                    | IETF Host Resources        |  389 | ホスト/アプリ/印刷         |
| HPR-IP-MIB                              | IBM / APPN                 |  487 | IBM/SNA            |
| HPR-MIB                                 | IBM / APPN                 | 1270 | IBM/SNA            |
| IANA-ADDRESS-FAMILY-NUMBERS-MIB         | IANA                       |  170 | IANAテキスト規約         |
| IANA-CHARSET-MIB                        | IANA                       |  361 | IANAテキスト規約         |
| IANA-FINISHER-MIB                       | IANA                       |  286 | IANAテキスト規約         |
| IANA-GMPLS-TC-MIB                       | IANA                       |  359 | IANAテキスト規約         |
| IANA-IPPM-METRICS-REGISTRY-MIB          | IANA                       |  818 | IANAテキスト規約         |
| IANA-ITU-ALARM-TC-MIB                   | IANA                       |  335 | IANAテキスト規約         |
| IANA-LANGUAGE-MIB                       | IANA                       |  126 | IANAテキスト規約         |
| IANA-MALLOC-MIB                         | IANA                       |  69  | IANAテキスト規約         |
| IANA-MAU-MIB                            | IANA                       |  984 | IANAテキスト規約         |
| IANA-PRINTER-MIB                        | IANA                       | 2111 | IANAテキスト規約         |
| IANA-PWE3-MIB                           | IANA                       |  137 | IANAテキスト規約         |
| IANA-RTPROTO-MIB                        | IANA                       |  102 | IANAテキスト規約         |
| IANAifType-MIB                          | IANA                       |  685 | IANAテキスト規約         |
| IANATn3270eTC-MIB                       | IANA                       |  303 | IANAテキスト規約         |
| IF-CAP-STACK-MIB                        | IETF                       |  284 | L2/インタフェース         |
| IF-INVERTED-STACK-MIB                   | IETF Interfaces            |  149 | L2/インタフェース         |
| IF-MIB                                  | IETF Interfaces            | 1814 | L2/インタフェース         |
| IFCP-MGMT-MIB                           | INCITS T11 / Fibre Channel | 1015 | ストレージSAN           |
| IGMP-STD-MIB                            | IETF IGMP                  |  516 | ルーティング             |
| INET-ADDRESS-MIB                        | IETF / IANA                |  402 | IP/トランスポート         |
| INTEGRATED-SERVICES-GUARANTEED-MIB      | IETF                       |  218 | その他                |
| INTEGRATED-SERVICES-MIB                 | IETF                       |  750 | その他                |
| INTERFACETOPN-MIB                       | IETF                       | 1023 | その他                |
| IPOA-MIB                                | IETF WAN/Carrier           | 1654 | WAN/伝送             |
| IPATM-IPMC-MIB                          | IETF WAN/Carrier           | 3244 | WAN/伝送             |
| IPFIX-MIB                               | IETF IPFIX                 | 1677 | その他                |
| IPFIX-SELECTOR-MIB                      | IETF IPFIX                 |  173 | その他                |
| IPMCAST-MIB                             | IETF IP Multicast          | 2391 | IP/トランスポート         |
| IPMROUTE-STD-MIB                        | IETF IP Multicast          |  869 | IP/トランスポート         |
| IPSEC-SPD-MIB                           | IETF IPsec                 | 2682 | IP/トランスポート         |
| IPS-AUTH-MIB                            | IETF                       | 1156 | その他                |
| ISCSI-MIB                               | IETF iSCSI                 | 3097 | ストレージSAN           |
| ISDN-MIB                                | IETF                       | 1260 | その他                |
| ISIS-MIB                                | IETF IS-IS                 | 4317 | ルーティング             |
| ISNS-MIB                                | IETF iSNS                  | 3243 | ストレージSAN           |
| ITU-ALARM-MIB                           | IETF                       |  486 | 管理/監視              |
| ITU-ALARM-TC-MIB                        | IETF                       |  86  | テキスト規約             |
| Job-Monitoring-MIB                      | IETF                       | 1652 | ホスト/アプリ/印刷         |
| L2TP-MIB                                | IETF                       | 2664 | その他                |
| LANGTAG-TC-MIB                          | IETF / IANA                |  56  | テキスト規約             |
| LMP-MIB                                 | IETF                       | 3185 | その他                |
| MAU-MIB                                 | IETF Ethernet              | 1740 | L2/インタフェース         |
| MALLOC-MIB                              | IETF                       | 1364 | その他                |
| MGMD-STD-MIB                            | IETF                       | 1524 | その他                |
| MIDCOM-MIB                              | IETF                       | 2260 | その他                |
| MIOX25-MIB                              | IETF                       |  708 | その他                |
| MIP-MIB                                 | IETF                       | 2127 | その他                |
| MOBILEIPV6-MIB                          | IETF IPv6                  | 3984 | IPv6               |
| Modem-MIB                               | IETF                       | 1340 | その他                |
| MPLS-FTN-STD-MIB                        | IETF MPLS                  | 1030 | MPLS/PWE3          |
| MPLS-L3VPN-STD-MIB                      | IETF MPLS                  | 1588 | MPLS/PWE3          |
| MPLS-LC-ATM-STD-MIB                     | IETF MPLS                  |  336 | MPLS/PWE3          |
| MPLS-LC-FR-STD-MIB                      | IETF MPLS                  |  263 | MPLS/PWE3          |
| MPLS-LDP-ATM-STD-MIB                    | IETF MPLS                  |  757 | MPLS/PWE3          |
| MPLS-LDP-FRAME-RELAY-STD-MIB            | IETF MPLS                  |  641 | MPLS/PWE3          |
| MPLS-LDP-GENERIC-STD-MIB                | IETF MPLS                  |  321 | MPLS/PWE3          |
| MPLS-LDP-STD-MIB                        | IETF MPLS                  | 2408 | MPLS/PWE3          |
| MPLS-LSR-STD-MIB                        | IETF MPLS                  | 2106 | MPLS/PWE3          |
| MPLS-TC-STD-MIB                         | IETF MPLS                  |  635 | MPLS/PWE3          |
| MPLS-TE-STD-MIB                         | IETF MPLS                  | 2483 | MPLS/PWE3          |
| MTA-MIB                                 | IETF                       | 1226 | その他                |
| MSDP-MIB                                | IETF MSDP                  | 1182 | ルーティング             |
| NAT-MIB                                 | IETF                       | 2391 | IP/トランスポート         |
| NEMO-MIB                                | IETF                       | 1739 | IPv6               |
| NETWORK-SERVICES-MIB                    | IETF                       |  626 | その他                |
| NHRP-MIB                                | IETF                       | 2596 | その他                |
| NOTIFICATION-LOG-MIB                    | IETF                       |  753 | 管理/監視              |
| OPT-IF-MIB                              | IETF                       | 6616 | その他                |
| OSPF-MIB                                | IETF OSPF                  | 4164 | ルーティング             |
| OSPF-TRAP-MIB                           | IETF OSPF                  |  584 | ルーティング             |
| OSPFV3-MIB                              | IETF OSPF                  | 3951 | ルーティング             |
| P-BRIDGE-MIB                            | IETF Bridge                | 1157 | ブリッジ/STP           |
| PARALLEL-MIB                            | IETF                       |  286 | その他                |
| PIM-BSR-MIB                             | IETF PIM                   |  699 | ルーティング             |
| PIM-MIB                                 | IETF PIM                   |  889 | ルーティング             |
| PIM-STD-MIB                             | IETF PIM                   | 3746 | ルーティング             |
| PINT-MIB                                | IETF                       |  573 | その他                |
| PKTC-IETF-EVENT-MIB                     | PacketCable                | 1163 | DOCSIS/PacketCable |
| PKTC-IETF-MTA-MIB                       | PacketCable                | 2081 | DOCSIS/PacketCable |
| PKTC-IETF-SIG-MIB                       | PacketCable                | 3021 | DOCSIS/PacketCable |
| POLICY-BASED-MANAGEMENT-MIB             | IETF                       | 2060 | その他                |
| POWER-ETHERNET-MIB                      | IETF Ethernet              |  621 | L2/インタフェース         |
| PerfHist-TC-MIB                         | IETF                       |  178 | テキスト規約             |
| PTOPO-MIB                               | IETF                       |  804 | その他                |
| PW-ATM-MIB                              | IETF PWE3                  | 1205 | MPLS/PWE3          |
| PW-ENET-STD-MIB                         | IETF PWE3                  |  491 | MPLS/PWE3          |
| PW-MPLS-STD-MIB                         | IETF PWE3                  |  914 | MPLS/PWE3          |
| PW-STD-MIB                              | IETF PWE3                  | 2438 | MPLS/PWE3          |
| PW-TC-STD-MIB                           | IETF PWE3                  |  288 | MPLS/PWE3          |
| PW-TDM-MIB                              | IETF PWE3                  | 1336 | MPLS/PWE3          |
| Printer-MIB                             | IETF Printer               | 4389 | ホスト/アプリ/印刷         |
| Q-BRIDGE-MIB                            | IETF Bridge                | 2367 | ブリッジ/STP           |
| RAQMON-MIB                              | IETF                       | 1417 | その他                |
| RAQMON-RDS-MIB                          | IETF                       |  672 | その他                |
| RADIUS-ACC-CLIENT-MIB                   | IETF RADIUS                |  638 | RADIUS             |
| RADIUS-ACC-SERVER-MIB                   | IETF RADIUS                |  727 | RADIUS             |
| RADIUS-AUTH-CLIENT-MIB                  | IETF RADIUS                |  710 | RADIUS             |
| RADIUS-AUTH-SERVER-MIB                  | IETF RADIUS                |  774 | RADIUS             |
| RADIUS-DYNAUTH-CLIENT-MIB               | IETF RADIUS                |  767 | RADIUS             |
| RADIUS-DYNAUTH-SERVER-MIB               | IETF RADIUS                |  699 | RADIUS             |
| RDBMS-MIB                               | IETF                       | 1377 | その他                |
| RFC1155-SMI                             | RFC 1155                   |  119 | RFC                |
| RFC1213-MIB                             | RFC 1213                   | 2613 | RFC                |
| RFC1381-MIB                             | RFC 1381                   | 1007 | RFC                |
| RFC1382-MIB                             | RFC 1382                   | 2627 | RFC                |
| RFC1414-MIB                             | RFC 1414                   |  131 | RFC                |
| RIPv2-MIB                               | IETF RIP                   |  532 | ルーティング             |
| RMON-MIB                                | IETF RMON                  | 3980 | 計測/フロー監視           |
| RMON2-MIB                               | IETF RMON                  | 5711 | 計測/フロー監視           |
| ROHC-MIB                                | IETF                       | 1133 | その他                |
| ROHC-RTP-MIB                            | IETF                       |  636 | その他                |
| ROHC-UNCOMPRESSED-MIB                   | IETF                       |  197 | その他                |
| RS-232-MIB                              | IETF                       |  788 | その他                |
| RSERPOOL-MIB                            | IETF                       | 1439 | その他                |
| RSVP-MIB                                | IETF                       | 2660 | その他                |
| RSTP-MIB                                | IETF Bridge                |  306 | ブリッジ/STP           |
| RTP-MIB                                 | IETF                       |  981 | IP/トランスポート         |
| SCSI-MIB                                | IETF SCSI                  | 2758 | ストレージSAN           |
| SFLOW-MIB                               | IETF                       |  389 | 計測/フロー監視           |
| SIP-COMMON-MIB                          | IETF SIP                   | 1913 | SIP                |
| SIP-MIB                                 | IETF SIP                   | 1099 | SIP                |
| SIP-SERVER-MIB                          | IETF SIP                   |  869 | SIP                |
| SIP-TC-MIB                              | IETF SIP                   |  177 | SIP                |
| SIP-UA-MIB                              | IETF SIP                   |  200 | SIP                |
| SLAPM-MIB                               | IETF                       | 2842 | その他                |
| SMON-MIB                                | IETF                       | 1254 | 計測/フロー監視           |
| SMUX-MIB                                | IETF SMUX                  |  158 | SNMP               |
| SNA-NAU-MIB                             | IETF                       | 2765 | その他                |
| SNA-SDLC-MIB                            | IETF                       | 2761 | その他                |
| SNMP-COMMUNITY-MIB                      | IETF SNMP                  |  505 | SNMP               |
| SNMP-FRAMEWORK-MIB                      | IETF SNMP                  |  526 | SNMP               |
| SNMP-IEEE802-TM-MIB                     | IETF SNMP                  |  40  | SNMP               |
| SNMP-MPD-MIB                            | IETF SNMP                  |  145 | SNMP               |
| SNMP-NOTIFICATION-MIB                   | IETF SNMP                  |  589 | SNMP               |
| SNMP-PROXY-MIB                          | IETF SNMP                  |  294 | SNMP               |
| SNMP-REPEATER-MIB                       | IETF SNMP                  | 3265 | SNMP               |
| SNMP-SSH-TM-MIB                         | IETF SNMP                  |  329 | SNMP               |
| SNMP-TARGET-MIB                         | IETF SNMP                  |  660 | SNMP               |
| SNMP-TSM-MIB                            | IETF SNMP                  |  234 | SNMP               |
| SNMP-USER-BASED-SM-MIB                  | IETF SNMP                  |  912 | SNMP               |
| SNMP-USM-AES-MIB                        | IETF SNMP                  |  62  | SNMP               |
| SNMP-USM-DH-OBJECTS-MIB                 | IETF SNMP                  |  532 | SNMP               |
| SNMP-VIEW-BASED-ACM-MIB                 | IETF SNMP                  |  830 | SNMP               |
| SNMPv2-CONF                             | IETF SNMPv2                |  322 | SNMP               |
| SNMPv2-M2M-MIB                          | IETF SNMPv2                |  807 | SNMP               |
| SNMPv2-MIB                              | IETF SNMPv2                |  854 | SNMP               |
| SNMPv2-PARTY-MIB                        | IETF SNMPv2                | 1410 | SNMP               |
| SNMPv2-PDU                              | IETF SNMPv2                |  133 | SNMP               |
| SNMPv2-SMI                              | IETF SNMPv2                |  344 | SNMP               |
| SNMPv2-TC                               | IETF SNMPv2                |  772 | SNMP               |
| SNMPv2-TM                               | IETF SNMPv2                |  176 | SNMP               |
| SNMPv2-USEC-MIB                         | IETF SNMPv2                |  238 | SNMP               |
| SONET-MIB                               | IETF WAN/Carrier           | 2360 | WAN/伝送             |
| SOURCE-ROUTING-MIB                      | IETF                       |  450 | その他                |
| SSPM-MIB                                | IETF                       | 1029 | その他                |
| SYSAPPL-MIB                             | IBM / APPN                 | 1539 | IBM/SNA            |
| SYSLOG-MSG-MIB                          | IETF                       |  598 | その他                |
| SYSLOG-TC-MIB                           | IETF                       |  202 | テキスト規約             |
| SCTP-MIB                                | IETF                       | 1342 | IP/トランスポート         |
| SMUX-MIB                                | IETF SMUX                  |  158 | SNMP               |
| T11-FC-FABRIC-ADDR-MGR-MIB              | INCITS T11                 | 1241 | ストレージSAN           |
| T11-FC-FABRIC-CONFIG-SERVER-MIB         | INCITS T11                 | 1717 | ストレージSAN           |
| T11-FC-FABRIC-LOCK-MIB                  | INCITS T11                 |  490 | ストレージSAN           |
| T11-FC-FSPF-MIB                         | INCITS T11                 | 1170 | ストレージSAN           |
| T11-FC-NAME-SERVER-MIB                  | INCITS T11                 | 1136 | ストレージSAN           |
| T11-FC-ROUTE-MIB                        | INCITS T11                 |  448 | ストレージSAN           |
| T11-FC-RSCN-MIB                         | INCITS T11                 |  751 | ストレージSAN           |
| T11-FC-SP-AUTHENTICATION-MIB            | INCITS T11                 |  920 | ストレージSAN           |
| T11-FC-SP-POLICY-MIB                    | INCITS T11                 | 4274 | ストレージSAN           |
| T11-FC-SP-SA-MIB                        | INCITS T11                 | 2483 | ストレージSAN           |
| T11-FC-SP-TC-MIB                        | INCITS T11                 |  637 | ストレージSAN           |
| T11-FC-SP-ZONING-MIB                    | INCITS T11                 |  550 | ストレージSAN           |
| T11-FC-VIRTUAL-FABRIC-MIB               | INCITS T11                 |  523 | ストレージSAN           |
| T11-FC-ZONE-SERVER-MIB                  | INCITS T11                 | 2638 | ストレージSAN           |
| T11-TC-MIB                              | INCITS T11                 |  67  | ストレージSAN           |
| TCP-ESTATS-MIB                          | IETF TCP                   | 2941 | IP/トランスポート         |
| TCP-MIB                                 | IETF TCP                   |  785 | IP/トランスポート         |
| TCPIPX-MIB                              | IETF                       |  331 | その他                |
| TE-LINK-STD-MIB                         | IETF                       | 1745 | その他                |
| TE-MIB                                  | IETF                       | 1679 | その他                |
| TIME-AGGREGATE-MIB                      | IETF                       |  375 | その他                |
| TN3270E-MIB                             | IBM / APPN                 | 1953 | IBM/SNA            |
| TN3270E-RT-MIB                          | IBM / APPN                 |  896 | IBM/SNA            |
| TOKEN-RING-RMON-MIB                     | IEEE / IETF                | 2302 | L2/インタフェース         |
| TOKENRING-MIB                           | IEEE / IETF                |  836 | L2/インタフェース         |
| TOKENRING-STATION-SR-MIB                | IEEE / IETF                |  175 | L2/インタフェース         |
| TPM-MIB                                 | IETF                       | 1916 | その他                |
| TRANSPORT-ADDRESS-MIB                   | IETF / IANA                |  421 | その他                |
| TRIP-MIB                                | IETF                       | 2007 | その他                |
| TRIP-TC-MIB                             | IETF / IANA                |  132 | テキスト規約             |
| TUNNEL-MIB                              | IETF                       |  738 | WAN/伝送             |
| UDP-MIB                                 | IETF UDP                   |  549 | IP/トランスポート         |
| UDPLITE-MIB                             | IETF UDP-Lite              |  513 | IP/トランスポート         |
| UPS-MIB                                 | IETF UPS                   | 1899 | UPS                |
| URI-TC-MIB                              | IETF / IANA                |  133 | テキスト規約             |
| VDSL-LINE-EXT-MCM-MIB                   | IETF DSL Forum / DSL       |  662 | DSL                |
| VDSL-LINE-EXT-SCM-MIB                   | IETF DSL Forum / DSL       |  420 | DSL                |
| VDSL-LINE-MIB                           | IETF DSL Forum / DSL       | 2844 | DSL                |
| VDSL2-LINE-MIB                          | IETF DSL Forum / DSL       | 7189 | DSL                |
| VDSL2-LINE-TC-MIB                       | IETF DSL Forum / DSL       | 1479 | DSL                |
| VPN-TC-STD-MIB                          | IETF                       |  72  | その他                |
| VRRP-MIB                                | IETF VRRP                  |  789 | その他                |
| WWW-MIB                                 | IETF                       | 1272 | その他                |
