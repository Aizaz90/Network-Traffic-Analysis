# tcpdump SOC Investigation Methodology

A reusable workflow for **authorized** network investigations. All file names and documentation-only IP examples below are generic; they are **not findings from a specific incident**.

## 1. Establish scope

Before filtering traffic, identify where the capture came from, what it covers, and which questions you are trying to answer.

```bash
tcpdump -nn -tttt -r example.pcap -c 10
tcpdump -nn -r example.pcap --count
```

- `-r`: read an existing PCAP; it does not modify the source file.
- `-nn`: display numeric IP addresses and port numbers.
- `-tttt`: display human-readable date and time (check the workstation timezone).
- `-c 10`: limit the output to ten matching packets.
- `--count`: report matching packet counts on supported tcpdump versions.

Record packet count, capture time range (where available), protocols, and possible collection limitations.

## 2. Find prominent sources and destinations

For **IPv4 TCP packets with the typical tcpdump text format**, a simple learning-oriented pipeline is:

```bash
tcpdump -nn -tt -r example.pcap 'ip and tcp' |
  cut -d ' ' -f 3 | cut -d '.' -f 1-4 |
  sort | uniq -c | sort -nr
```

- `cut -d ' ' -f 3`: source endpoint in this output format.
- `cut -d '.' -f 1-4`: retain the IPv4 address, excluding the port.
- `sort`: group identical addresses.
- `uniq -c`: count occurrences.
- `sort -nr`: display largest packet counts first.

Change `-f 3` to `-f 5` to analyze destination endpoints (the fifth field ends with a colon; the IPv4 extraction strips the port and colon).

**Limitations:** This pipeline is tied to a specific text format and IPv4; it is not a robust IPv6 parser. It counts printed packet lines, not bytes, distinct connections, or confirmed malicious events. A high count is a lead, not a verdict.

## 3. Pivot to a specific conversation

The addresses below are documentation examples only (RFC 5737 ranges):

```bash
tcpdump -nn -tt -r example.pcap \
  'host 192.0.2.10 and host 203.0.113.50'
```

Filter one direction when needed:

```bash
tcpdump -nn -tt -r example.pcap \
  'tcp and src host 192.0.2.10 and dst host 203.0.113.50'
```

Compare both directions. Identify ports and TCP flags, then ask whether the connection appears established and what applications are involved.

## 4. Inspect HTTP requests and headers

```bash
tcpdump -nn -tt -r example.pcap 'tcp port 80' | grep -E 'GET |POST '
```

For readable plaintext in a relevant connection:

```bash
tcpdump -nn -A -r example.pcap 'tcp port 80 and host 192.0.2.10' | less
```

Look at HTTP methods, paths, `Host`, `User-Agent`, response status, `Content-Type`, and `Content-Disposition`. Explain **why** a value is unusual, rather than labeling every executable or unfamiliar IP as malware.

**Limitations:** `grep` is a text search, not a protocol parser. TCP segments may split requests, and retransmissions may repeat packet content. A GET request does not prove a successful download; a POST request does not prove exfiltration.

## 5. Search other relevant protocols

DNS query traffic:

```bash
tcpdump -nn -r example.pcap 'udp port 53 or tcp port 53'
```

ICMP:

```bash
tcpdump -nn -r example.pcap icmp
```

FTP control-channel messages (review locally; may expose passwords):

```bash
tcpdump -nn -A -r example.pcap 'tcp port 21' |
  grep -iE 'USER |PASS |230 |530 |RETR |STOR '
```

HTTPS packet metadata can be inspected, but encrypted HTTP contents generally cannot be viewed without authorized TLS session secrets. A public server certificate alone is not sufficient.

## 6. Investigate timing and possible beaconing

Use timestamps to look for repeated requests or connections at regular intervals. Account for TCP retransmissions and distinguish protocol packets from unique application requests. Periodicity is a hypothesis to correlate, not proof of malware.

```bash
tcpdump -nn -ttt -r example.pcap 'host 203.0.113.50'
```

`-ttt` displays elapsed time between consecutively printed packets, which can help reveal gaps and bursts; compare equivalent events (e.g., request starts), not arbitrary packets.

## 7. Enrich with context

If allowed, research relevant IPs, domains, file hashes, and unusual User-Agent strings. Record the source and the **date checked**. Hosting ownership and reputation detections provide context; they do not, alone, prove malicious activity.

For possible file downloads, inspect the matching server response and transferred bytes before claiming a file was delivered. For complex TCP reassembly or HTTP object export, use Wireshark or another authorized analyzer.

## 8. Document conclusions

Use a simple evidence matrix:

| Evidence ID | Observation | Why it matters | Confidence | Follow-up |
| --- | --- | --- | --- | --- |
| E-01 | [Describe a directly observed event] | [Reason] | [High/Medium/Low] | [Next check] |

Keep these categories distinct:

- **Observed:** Directly supported by the capture.
- **Corroborated:** Supported by additional reliable evidence.
- **Hypothesis:** Plausible explanation that still needs testing.
- **Unknown:** Information the dataset cannot establish.

A final report should include scope, a compact timeline, evidence, indicators when appropriate, findings, limitations, and recommended next steps.

## Publication safety

Only use captures and lab materials that you are authorized to publish. Remove real usernames, passwords, tokens, session cookies, private addresses when needed, internal hostnames, and unnecessary personal data. Do not upload executable payloads or restricted training solutions.
