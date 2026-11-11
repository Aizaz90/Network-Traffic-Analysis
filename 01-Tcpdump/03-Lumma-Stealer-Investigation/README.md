# Lumma Stealer — Network Traffic Investigation

## Investigation Overview

This report documents my analysis of suspicious network traffic using tcpdump.

I completed this investigation as part of a TCM Security SOC 101 practical challenge, applying the packet analysis techniques I learned during training.

## Tools Used

tcpdump, Linux command-line utilities, and OSINT resources.

## Investigation Methodology

I started by examining the packet capture to understand the
volume of network traffic and identify the protocols involved.

I first checked the total packet count and then filtered for
ICMP traffic to investigate unusual ping activity.

Next, I examined HTTP requests, focusing on GET and POST
methods. I used tcpdump with grep to search for interesting
headers, URLs, and plaintext information.

I also investigated FTP traffic on port 21. By examining
authentication commands and server responses, I was able
to distinguish successful login attempts from failed ones
and identify file-transfer activity.

Finally, I examined HTTP User-Agent strings and used
public threat intelligence to investigate potentially
malicious software associated with the observed traffic.

## Evidence and Findings

The capture contained 1,344 packets, including 132 ICMP packets.

During the investigation, I examined ICMP traffic to identify external communication and used public IP intelligence to investigate the destination network.

I then inspected HTTP traffic and identified a POST request. Further inspection of readable HTTP payloads revealed authentication-related information, demonstrating the risks of transmitting sensitive information over unencrypted protocols.

I also investigated FTP traffic on TCP port 21. By examining FTP commands and server responses, I identified authentication activity and a file retrieval request.

Finally, I examined HTTP User-Agent strings. One unusual User-Agent was associated with Lumma Stealer, an information-stealing malware family.

These observations provided several indicators worth further investigation, although the User-Agent alone was not sufficient to prove malware execution.

## Conclusion

During this challenge, I analyzed a packet capture containing suspicious network activity using tcpdump and Linux command-line tools.

I investigated ICMP requests, HTTP communication, exposed authentication information, and FTP activity. I also examined HTTP User-Agent strings and identified an indicator associated with Lumma Stealer.

The investigation demonstrated how different network protocols can reveal important security information. In particular, the HTTP and FTP traffic showed why transmitting sensitive information without encryption creates security risks.

Although the suspicious User-Agent was associated with Lumma Stealer, confirming an actual infection would require additional evidence, such as endpoint logs or malware analysis.

## Lessons Learned

This challenge gave me an opportunity to apply the tcpdump techniques I had learned without following the instructor's analysis step by step.

I became more comfortable filtering traffic by protocol and port, searching packet contents using grep, and identifying important information within HTTP and FTP communication.

One of the most useful things I learned was how to investigate FTP authentication. By examining commands such as USER and PASS alongside server response codes, I could identify successful and unsuccessful login attempts.

I also learned how useful HTTP User-Agent strings can be during malware investigations. However, I understand that a suspicious User-Agent should be treated as an investigative lead rather than definitive proof of infection.

Overall, this exercise improved my confidence in using the Linux terminal for network security investigations.

## Source

## Source and Attribution

This investigation was completed as part of the TCM Security SOC 101 training course.

Training provider: [TCM Security](https://academy.tcm-sec.com/)

Exercise: tcpdump Challenge 1

Analysis environment: Ubuntu Linux

I completed the challenge by applying the tcpdump and Linux command-line techniques I learned during training. The investigation involved examining network traffic, identifying suspicious communication, analyzing HTTP and FTP activity, and researching potential malware indicators.

This report documents my own investigation methodology and learning outcomes. It is a training exercise, not a real-world incident response engagement.

The original PCAP, course instructions, challenge answers, and proprietary training materials are not included in this repository.
