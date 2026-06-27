# 🛡️ Lesson 2: IDS/IPS - Suricata

## Suricata Workflow

### 1. Packet Acquisition
Paketlərin toplanması üsulları:

| Method | Təsvir |
|--------|--------|
| AF-Packet | Linux-da yüksək performanslı paket toplama |
| NFQueue | iptables ilə inteqrasiya (IPS rejimi üçün) |
| PCAP file | Diskdəki pcap fayllarını oxumaq |
| WinPcap / npcap | Windows üçün |

### 2. Packet Decode
Paketlərin decode edilməsi (Ethernet, IP, TCP, UDP və s.)

### 3. Application Layer Protocol Detection (ALPR)
Tətbiq qatı protokollarının aşkarlanması:
- HTTP
- DNS
- TLS
- SMTP
- FTP
- SSH

### 4. Protocol Parsers
Hər bir protokol üçün xüsusi parser-lər

### 5. Detection Engine
Rule-ların yoxlanması və uyğunluğun müəyyən edilməsi

### 6. Alerting & Logging
- eve.json (strukturlaşdırılmış log)
- fast.log (sadə format)
- custom output

---

## Installation

```bash
sudo apt-get update
sudo apt-get install suricata -y
Config faylları
/etc/suricata/suricata.yaml - Əsas konfiqurasiya

/etc/suricata/rules/ - Rule faylları

/var/log/suricata/ - Log qovluğu

Initial Configuration
1. Şəbəkə interfeysini təyin edin
bash
sudo nano /etc/suricata/suricata.yaml
yaml
af-packet:
  - interface: eth0
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
    use-mmap: yes
2. Rule faylını təyin edin
yaml
default-rule-path: /etc/suricata/rules
rule-files:
  - suricata.rules
Rule Syntax
text
<ACTION> <PROTOCOL> <SRC_IP> <SRC_PORT> -> <DST_IP> <DST_PORT> (<MSG>)
Nümunə:
text
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"SQL Injection Attempt"; content:"SELECT"; http_uri; sid:1000001; rev:1;)
Rule hissələri:
Hissə	Təsvir
ACTION	alert, drop, reject, pass
PROTOCOL	tcp, udp, icmp, http, dns
SRC_IP	Mənbə IP (any, $HOME_NET)
SRC_PORT	Mənbə port
DIRECTION	-> (tək istiqamət), <> (iki istiqamət)
DST_IP	Hədəf IP
DST_PORT	Hədəf port
OPTIONS	msg, content, sid, rev
Alerting
Alert formatları:
eve.json (JSON formatı)

bash
tail -f /var/log/suricata/eve.json | jq '.'
fast.log (sadə format)

bash
tail -f /var/log/suricata/fast.log
Custom Rules
Nümunə custom rule-lar:
SSH brute force detection
text
alert tcp $HOME_NET any -> $EXTERNAL_NET 22 (msg:"SSH Brute Force Attempt"; flow:to_server,established; threshold:type both, track by_src, count 5, seconds 60; sid:1000002; rev:1;)
SQL Injection detection
text
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"SQL Injection - SELECT"; content:"SELECT"; http_uri; nocase; sid:1000003; rev:1;)
XSS detection
text
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"XSS Attempt - script"; content:"<script"; http_uri; nocase; sid:1000004; rev:1;)
URLhaus Integration
URLhaus - malicious URL-lərin verilənlər bazası.

URLhaus rule-larını yükləmək:
bash
sudo suricata-update --enable-source urlhaus
sudo systemctl restart suricata
Manual update:
bash
sudo curl -o /etc/suricata/rules/urlhaus.rules https://urlhaus.abuse.ch/downloads/suricata/
Suricata IPS Mode
IPS rejimi üçün NFQueue:
bash
sudo iptables -I INPUT -j NFQUEUE --queue-num 0
sudo iptables -I OUTPUT -j NFQUEUE --queue-num 0
suricata.yaml konfiqurasiyası:
yaml
nfq:
  mode: accept
  repeat-mark: 1
  repeat-mask: 1
  queue-num: 0
IPS rejimində işə salmaq:
bash
sudo systemctl stop suricata
sudo suricata -c /etc/suricata/suricata.yaml -q 0
Tasks
✅ Suricata quraşdırılması
✅ Custom rule yazılması
✅ URLhaus inteqrasiyası
✅ IPS rejiminin test edilməsi
✅ Alert-lərin monitorinqi

Faydalı komandalar
bash
# Suricata-nı yoxlamaq
sudo suricata -T -c /etc/suricata/suricata.yaml

# Suricata-nı başlatmaq
sudo systemctl start suricata

# Statusu yoxlamaq
sudo systemctl status suricata

# Log-ları izləmək
sudo tail -f /var/log/suricata/fast.log

# Eve.json-u izləmək
sudo tail -f /var/log/suricata/eve.json | jq '.'
