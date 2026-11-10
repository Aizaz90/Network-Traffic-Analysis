# tcpdump Command Reference

Examples use a generic `example.pcap` file and documentation-only IP addresses. Replace them with your authorized lab values.

| Goal | Command |
| --- | --- |
| List capture interfaces | `tcpdump -D` |
| Read a PCAP | `tcpdump -nn -r example.pcap` |
| Read first 10 matching packets | `tcpdump -nn -r example.pcap -c 10` |
| Count packets (supported versions) | `tcpdump -nn -r example.pcap --count` |
| Full timestamps | `tcpdump -nn -tttt -r example.pcap` |
| Epoch timestamps | `tcpdump -nn -tt -r example.pcap` |
| Relative time between printed packets | `tcpdump -nn -ttt -r example.pcap` |
| Inspect one IP in either direction | `tcpdump -nn -r example.pcap 'host 192.0.2.10'` |
| Inspect source IP only | `tcpdump -nn -r example.pcap 'src host 192.0.2.10'` |
| Inspect destination IP only | `tcpdump -nn -r example.pcap 'dst host 203.0.113.50'` |
| Filter HTTP's common port | `tcpdump -nn -r example.pcap 'tcp port 80'` |
| Filter TCP port 443 | `tcpdump -nn -r example.pcap 'tcp port 443'` |
| Filter DNS | `tcpdump -nn -r example.pcap 'udp port 53 or tcp port 53'` |
| Filter FTP control | `tcpdump -nn -r example.pcap 'tcp port 21'` |
| Filter ICMP | `tcpdump -nn -r example.pcap icmp` |
| Inspect ASCII | `tcpdump -nn -A -r example.pcap 'tcp port 80'` |
| Filter port 80 or 443 | `tcpdump -nn -r example.pcap 'tcp port 80 or tcp port 443'` |

## Linux text-processing commands

```bash
# Identify top IPv4 TCP source IPs in this tcpdump output format
tcpdump -nn -tt -r example.pcap 'ip and tcp' |
  cut -d ' ' -f 3 | cut -d '.' -f 1-4 |
  sort | uniq -c | sort -nr

# Find visible HTTP GET/POST lines (text search, not full HTTP parsing)
tcpdump -nn -tt -r example.pcap 'tcp port 80' |
  grep -E 'GET |POST '

# Inspect plaintext FTP control commands (never publish extracted credentials)
tcpdump -nn -A -r example.pcap 'tcp port 21' |
  grep -iE 'USER |PASS |230 |530 |RETR |STOR '
```

**Remember:** `tcpdump -A` prints ASCII payload, whereas `grep -A 5` prints five *lines of text* after a matching line. They are unrelated options belonging to different commands.

**Notes:** Filters on an existing PCAP do not alter the source file. `-nn` suppresses hostname and service-name resolution. The simplified `cut` pipeline assumes IPv4 and tcpdump's displayed field layout; for reliable production parsing use a structured packet-analysis tool.
