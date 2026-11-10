# Network Traffic Analysis | SOC & Blue Team Labs

A practical cybersecurity portfolio focused on packet analysis, network investigations, and detection engineering using **tcpdump**, **Wireshark**, and **Snort**.

## Focus

My goal is to document how I approach an investigation: define a question, filter noisy traffic, examine evidence, correlate observations, and explain what is **confirmed** versus **suspected**.

## Explore the work

| Area | What it demonstrates | Current content |
| --- | --- | --- |
| [tcpdump](01-Tcpdump/README.md) | Packet capture, BPF filtering, Linux pipelines, HTTP/FTP analysis, network triage | [Investigation methodology](01-Tcpdump/methodology.md) and [command reference](01-Tcpdump/command-reference.md) |
| Wireshark | TCP stream analysis, protocol investigation, evidence review | To be added after review |
| Snort IDS/IPS | Detection rules, traffic inspection, alert validation | To be added after review |

## Skills demonstrated

- **Network triage:** packet counts, timestamps, source/destination IPs, ports, and protocol filtering
- **Linux command-line analysis:** `grep`, `cut`, `sort`, `uniq -c`, and pipelines
- **Application-layer review:** HTTP methods and headers; FTP control-channel analysis
- **Investigative reasoning:** distinguish observed facts, hypotheses, and gaps in evidence
- **Responsible reporting:** avoid exposing raw captures, private data, or unauthorized training materials

## Investigation approach

1. Define scope and check capture metadata.
2. Identify prominent hosts and communicating pairs.
3. Narrow by protocol, direction, and port.
4. Examine requests, headers, timing, and other observable behavior.
5. Correlate indicators with trustworthy supporting evidence.
6. Document confirmed findings, uncertainties, and next steps.

See the [full tcpdump methodology](01-Tcpdump/methodology.md) for reproducible, generalized examples.

## Training and attribution

I developed these skills through hands-on exercises, including **TCM Security** training. This repository contains my own generalized notes and command examples, **not** official course materials or challenge solutions. Guided practice and independently performed investigations will be labeled separately.

PCAPs, course-provided datasets, extracted credentials, proprietary screenshots, and executable payloads are not distributed here without appropriate rights and safeguards.

## About this portfolio

**Interests:** SOC operations, blue-team investigation, network traffic analysis, detection engineering, and threat intelligence.

The repository will grow as I complete and validate additional publishable investigations. Findings will be labeled with their evidence and limitations rather than overstated.
