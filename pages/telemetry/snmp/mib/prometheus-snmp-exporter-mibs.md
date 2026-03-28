# prometheus SNMP Exporter MIBs

Prometheus SNMP Exporter
- [https://github.com/prometheus/snmp_exporter/](https://github.com/prometheus/snmp_exporter/)

Prometheus SNMP Exporter generator
- [https://github.com/prometheus/snmp_exporter/tree/main/generator](https://github.com/prometheus/snmp_exporter/tree/main/generator)

Prometheus SNMP Exporter generator Makefile (The source of the following URL list)
- [https://github.com/prometheus/snmp_exporter/blob/main/generator/Makefile](https://github.com/prometheus/snmp_exporter/blob/main/generator/Makefile)

## Vendor MIBs URL List

| Vendor | URL |
| :--- | :--- |
| APC | https://raw.githubusercontent.com/prometheus-community/snmp/refs/heads/main/apc/PowerNet-MIB |
| ARISTA | https://www.arista.com/assets/data/docs/MIBS |
| CISCO | https://raw.githubusercontent.com/cisco/cisco-mibs/f55dc443daff58dfc86a764047ded2248bb94e12/v2 |
| DELL_OM | https://dl.dell.com/FOLDER11196144M/1/Dell-OM-MIBS-11010_A00.zip |
| DLINK | https://ftp.dlink.de/des/des-3200-18/driver_software/DES-3200-18_mib_revC_4-04_all_en_20130603.zip |
| ELTEX_MES | https://api.prod.eltex-co.ru/storage/upload_center/files/42/mibs_10.4.3.4.zip |
| DELL_NETWORK | https://supportkb.dell.com/attachment/ka02R000000I7TFQA0/Current_MIBs_pkb_en_US_1.zip |
| HPE | https://downloads.hpe.com/pub/softlib2/software1/pubsw-linux/p1580676047/v229101/upd11.85mib.tar.gz |
| IANA_CHARSET | https://www.iana.org/assignments/ianacharset-mib/ianacharset-mib |
| IANA_IFTYPE | https://www.iana.org/assignments/ianaiftype-mib/ianaiftype-mib |
| IANA_PRINTER | https://www.iana.org/assignments/ianaprinter-mib/ianaprinter-mib |
| FIELDSERVER | https://media.msanet.com/NA/USA/SMC/SoftwareDownloads/FS-8704-26%20SNMP%20Standard%20MIB%20File.zip |
| KEEPALIVED | https://raw.githubusercontent.com/acassen/keepalived/v2.3.4/doc/KEEPALIVED-MIB.txt |
| VRRP | https://raw.githubusercontent.com/acassen/keepalived/v2.3.4/doc/VRRP-MIB.txt |
| VRRPV3 | https://raw.githubusercontent.com/acassen/keepalived/v2.3.4/doc/VRRPv3-MIB.txt |
| KEMP_LM | https://kemptechnologies.com/files/packages/current/LM_mibs.zip |
| MIKROTIK | https://download.mikrotik.com/routeros/7.18.2/mikrotik.mib |
| NEC | https://jpn.nec.com/univerge/ix/Manual/MIB |
| NET_SNMP | https://raw.githubusercontent.com/net-snmp/net-snmp/v5.9/mibs |
| PALOALTO | https://docs.paloaltonetworks.com/content/dam/techdocs/en_US/zip/snmp-mib/pan-10-0-snmp-mib-modules.zip |
| PRINTER | https://ftp.pwg.org/pub/pwg/pmp/mibs/rfc3805b.mib |
| SERVERTECH | https://cdn10.servertech.com/assets/documents/documents/817/original/Sentry3.mib |
| SERVERTECH4 | https://cdn10.servertech.com/assets/documents/documents/815/original/Sentry4.mib |
| SYNOLOGY | https://global.download.synology.com/download/Document/Software/DeveloperGuide/Firmware/DSM/All/enu/Synology_MIB_File.zip |
| UBNT_AIROS | https://dl.ubnt.com/firmwares/airos-ubnt-mib/ubnt-mib.zip |
| UBNT_AIROS_OLD | https://raw.githubusercontent.com/pgmillon/observium/refs/heads/master/mibs/FROGFOOT-RESOURCES-MIB |
| UBNT_AIRFIBER | https://dl.ubnt.com/firmwares/airfiber5X/v4.1.0/UBNT-MIB.txt |
| UBNT_DL | http://dl.ubnt-ut.com/snmp |
| UPS_MIB | https://raw.githubusercontent.com/pgmillon/observium/refs/heads/master/mibs/UPS-MIB |
| RARITAN | https://cdn.raritan.com/download/PX/v1.5.20/PDU-MIB.txt |
| RARITAN2 | https://cdn1.raritan.com/download/pdu-g2/4.3.10/PDU2_MIB_4.3.10_51837.txt |
| RARITAN_AM | https://cdn1.raritan.com/download/pdu-g2/4.3.10/AssetManagement_MIB_4.3.10_51837.txt |
| INFRAPOWER | https://www.austin-hughes.com/wp-content/uploads/2021/05/IPD-03-S-MIB.zip |
| LIEBERT | https://www.vertiv.com/globalassets/documents/software/monitoring/lgpmib-win_rev16_299461_0.zip |
| READYNAS | https://www.downloads.netgear.com/files/ReadyNAS/READYNAS-MIB.txt |
| READYDATAOS | https://www.downloads.netgear.com/files/GDC/RD5200/READYDATA_MIB.zip |
| SOPHOS_XG | https://docs.sophos.com/nsg/sophos-firewall/MIB/SOPHOS-XG-MIB.zip |
| POWERCOM | https://raw.githubusercontent.com/prometheus-community/snmp/refs/heads/main/powercom/XPPC-MIB |
| TPLINK_DDM | https://static.tp-link.com/upload/software/2024/202402/20240229/L2-tplinkMibs.zip |
| CISCO_CUCS | https://raw.githubusercontent.com/cisco/cisco-mibs/refs/heads/main/ucs-mibs |
| CISCO_CUCS_v2 | https://raw.githubusercontent.com/cisco/cisco-mibs/refs/heads/main/v2 |
| YAMAHA | https://www.rtpro.yamaha.co.jp/RT/docs/mib/ |

### (quote) Where to get MIBs
Some of these are quite sluggish, so use wget to download.

Put the extracted mibs in a location NetSNMP can read them from. $HOME/.snmp/mibs is one option.

Cisco: https://cfnng.cisco.com/mibs or https://github.com/cisco/cisco-mibs
APC: https://download.schneider-electric.com/files?p_File_Name=powernet432.mib
Servertech: ftp://ftp.servertech.com/Pub/SNMP/sentry3/Sentry3.mib
Palo Alto PanOS 7.0 enterprise MIBs: https://www.paloaltonetworks.com/content/dam/pan/en_US/assets/zip/technical-documentation/snmp-mib-modules/PAN-MIB-MODULES-7.0.zip
Arista Networks: https://www.arista.com/assets/data/docs/MIBS/ARISTA-ENTITY-SENSOR-MIB.txt 
https://www.arista.com/assets/data/docs/MIBS/ARISTA-SW-IP-FORWARDING-MIB.txt 
https://www.arista.com/assets/data/docs/MIBS/ARISTA-SMI-MIB.txt
Synology: https://global.download.synology.com/download/Document/Software/DeveloperGuide/Firmware/DSM/All/enu/Synology_MIB_File.zip
MikroTik: http://download2.mikrotik.com/Mikrotik.mib
UCD-SNMP-MIB (Net-SNMP): http://www.net-snmp.org/docs/mibs/UCD-SNMP-MIB.txt
Ubiquiti Networks: 
http://dl.ubnt-ut.com/snmp/UBNT-MIB 
http://dl.ubnt-ut.com/snmp/UBNT-UniFi-MIB 
https://dl.ubnt.com/firmwares/airos-ubnt-mib/ubnt-mib.zip
https://github.com/librenms/librenms/tree/master/mibs 

can also be a good source of MIBs.

http://oidref.com is recommended for browsing MIBs.