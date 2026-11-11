# tcpdump | Network Traffic Investigations

## Purpose

This section demonstrates how I use tcpdump and Linux terminal tools to triage traffic, investigate suspicious connections, and document evidence in a SOC-style workflow.

## What I practiced

- Reading packet captures with `-r`, counting packets, and comparing timestamp formats
- Filtering by host, source/destination, port, and protocol with BPF syntax
- Inspecting plaintext application data using `-A`
- Identifying high-frequency source and destination IPs with `cut | sort | uniq -c | sort -nr`
- Reviewing HTTP requests and headers, FTP control activity, ICMP, DNS, and TCP flags
- Applying OSINT carefully and distinguishing suspicious signals from proof of compromise

## Documents

- [Investigation methodology](methodology.md) — a step-by-step, reusable workflow with reasoning
- [Command reference](command-reference.md) — essential commands and explanations

## Case studies

I have completed guided traffic-analysis exercises and a tcpdump challenge in my training environment. Full course datasets, answer keys, and restricted walkthroughs are not hosted here.

I will add **independently documented case studies based on shareable datasets or appropriately authorized exercises**, each with its own scope, evidence, findings, limitations, and attribution.

## What makes a strong case study?

A good report answers: **What did I investigate? What evidence did I inspect? Why did I run each command? What can I actually conclude? What would I investigate next?**

## Data handling

Do not publish original PCAPs, captured credentials, cookies, raw malware, or third-party course content unless authorization and safety have been checked. Mask sensitive values in screenshots and reports.

## My Investigations

### 1. Hancitor Traffic Analysis
[View Investigation](01-Hancitor-Traffic-Analysis/README.md)

Analyzed suspicious HTTP communication, identified an unusual executable request, and investigated external IP addresses using threat intelligence.

### 2. SSLoad Malware Investigation
[View Investigation](02-SSLoad-Malware-Investigation/README.md)

Identified high-activity hosts, investigated HTTP communication patterns, analyzed a suspicious User-Agent, and examined potential additional payloads.

### 3. Lumma Stealer Investigation
[View Investigation](03-Lumma-Stealer-Investigation/README.md)

Completed a SOC investigation challenge involving ICMP, HTTP, FTP authentication, plaintext credential exposure, and malware-related indicators.
