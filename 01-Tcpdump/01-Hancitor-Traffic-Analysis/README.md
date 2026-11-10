# Investigating a Suspicious HTTP Executable with tcpdump

This is a write-up of a guided network traffic analysis exercise I completed while studying with TCM Security. I used tcpdump and a few Linux command-line tools to investigate an unusual executable request in a PCAP from September 14, 2021.

The main thing I wanted to find out was whether the machine had contacted a suspicious external server and what the traffic could tell me about the requested file.

## Lab overview

| Item | Details |
| --- | --- |
| Capture | `2021-09-14.pcap` |
| Type | Guided training investigation |
| Tools | tcpdump, grep, less, VirusTotal, DomainTools/WHOIS, CyberChef |
| Internal host investigated | `10.0.0.168` |
| External IP of interest | `103.232.55.148` |
| Other observed destination | `13.107.5.80` |

The research page for the dated dataset is [Malware-Traffic-Analysis.net](https://malware-traffic-analysis.net/2021/09/14/index.html). I have not independently verified that the course copy and the publicly hosted capture are identical. This report therefore focuses on what I actually observed rather than assuming a malware family from the filename or date.

## How I investigated the capture

### Checking the number of packets

I started by checking how much traffic I was dealing with:

```bash
sudo tcpdump -r 2021-09-14.pcap --count
```

The capture contained **3,679 packets**. Knowing the size helped me decide to narrow the traffic rather than read every packet individually.

### Looking at the connections

Next, I read the capture with Unix timestamps:

```bash
tcpdump -tt -r 2021-09-14.pcap
```

One line caught my attention:

```text
1631630134.496167 IP 10.0.0.168.49724 > 103.232.55.148.http: Flags [.], ack 581081, win 32768, length 0
```

This showed traffic from `10.0.0.168` to `103.232.55.148` on the service displayed as HTTP. The packet itself was only a TCP acknowledgement with no application payload, so I couldn't draw a conclusion from that line alone. I needed to examine the HTTP requests.

### Narrowing down HTTP traffic

I filtered for port 80 and counted the matching packets:

```bash
tcpdump -tt -r 2021-09-14.pcap 'port 80' --count
```

There were **2,373 matching packets**. Port 80 is commonly used for HTTP, which made it a useful place to look for requested files and URLs.

I then focused on the internal host and searched for HTTP GET and POST lines:

```bash
tcpdump -tt -r 2021-09-14.pcap 'port 80 and host 10.0.0.168' | grep -E 'GET|POST'
```

The `host` filter kept packets involving `10.0.0.168`. `grep -E` let me look for either `GET` or `POST` in tcpdump's textual output. This worked well for quick triage, although text searches don't replace full TCP stream reconstruction.

### Finding the suspicious file request

Among the requests, I found:

```text
1631630132.552578 IP 10.0.0.168.49724 > 103.232.55.148.http: ... HTTP: GET /service/.audiodg.exe HTTP/1.1
```

The path `/service/.audiodg.exe` stood out to me. Windows has a legitimate process named `audiodg.exe`, but this request used a leading dot in the filename and contacted an external IP address. I treated that as a possible attempt to make the file resemble a trusted Windows component.

I couldn't conclude that the file was malicious just from its name. Also, an HTTP GET proves that the client *requested* the file; it doesn't prove the complete file arrived or was executed.

### Finding related requests

To see where else the filename appeared, I searched the whole capture:

```bash
tcpdump -tt -r 2021-09-14.pcap | grep 'audiodg.exe'
```

I found the direct request to `103.232.55.148` and another request to `13.107.5.80` that included the same file URL as an encoded query parameter.

To read more of the surrounding HTTP text, I ran:

```bash
tcpdump -tt -r 2021-09-14.pcap -A | grep 'audiodg.exe' -A 500 | less
```

Here, `tcpdump -A` prints packet content as ASCII, while `grep -A 500` shows up to 500 **text lines** after a match. Those lines are not necessarily one complete TCP conversation.

The request included these details:

```http
GET /qsml.aspx?query=http%3A%2F%2F103.232.55.148%2Fservice%2F.audiodg.exe&maxwidth=32765&rowheight=20&sectionHeight=160&FORM=IESS02&market=en-US HTTP/1.1
Host: api.bing.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko
```

This told me that a request to a Bing API endpoint *referenced* the suspicious file URL. It did not establish that Bing redirected the victim or that Microsoft's infrastructure was compromised.

### Decoding the URL

I copied the encoded value into CyberChef and used the URL Decode operation:

```text
http%3A%2F%2F103.232.55.148%2Fservice%2F.audiodg.exe
```

It decoded to the following address, shown here in defanged form so it isn't an active link:

```text
hxxp://103[.]232[.]55[.]148/service/.audiodg.exe
```

This helped connect the Bing query to the direct file request. URL encoding itself is normal on the web and isn't evidence of an attack.

### Checking the external IP

I searched `103.232.55.148` on VirusTotal and checked its ownership information through a WHOIS service. At the time of my lab screenshot, **12 of 92 VirusTotal engines** flagged the IP as malicious. The registration information pointed to **VIETSERVER SERVICES TECHNOLOGY COMPANY LIMITED**.

The reputation results made the destination worth investigating further. They don't identify the attacker, and they don't imply that the hosting provider knowingly supported malicious activity.

## What I found

| Observation | Why it mattered |
| --- | --- |
| 3,679 packets in the capture; 2,373 involved port 80 | Helped me narrow the initial investigation |
| `10.0.0.168` requested `/service/.audiodg.exe` from `103.232.55.148` | An unusually named executable on an external server deserved closer inspection |
| A request to `api.bing.com` contained an encoded reference to the same URL | Connected two HTTP requests that were worth examining together |
| The external IP had adverse reputation results | Added supporting context, but was not proof on its own |

The main indicators I recorded were the internal host `10.0.0.168`, external destination `103.232.55.148`, and requested path `/service/.audiodg.exe`. I also recorded `13.107.5.80` as a related Bing destination, **not** as a malicious IP.

## Conclusion

Using tcpdump, I narrowed a capture of several thousand packets to one suspicious executable request and the related HTTP activity. Linux text filtering helped me move quickly from general traffic to a particular host, path, and external IP. I then used CyberChef and public threat intelligence to add context.

What I couldn't confirm from the packet excerpts was whether the executable was fully downloaded, whether it ran on the machine, or which malware family it belonged to. I would need the corresponding HTTP response and, ideally, endpoint evidence to answer those questions.

## What I would investigate next

If I continued the investigation, I would examine the full HTTP response in Wireshark and check whether the object was transferred. If extraction were permitted, I would hash the file safely in an isolated environment without executing it. I would also compare the timestamps with endpoint process and security logs if those were available.

## What I learned

This lab helped me practice reading PCAPs with tcpdump, narrowing traffic with host and port filters, searching HTTP output with `grep`, decoding URLs, and using reputation checks as supporting evidence. More importantly, it reinforced the need to separate what a packet actually shows from what I suspect happened.

## Source and scope

This is my own write-up of a guided training activity, not an independent incident response engagement. Credit goes to TCM Security for the exercise and Malware-Traffic-Analysis.net for the referenced public research page. I have not included the original PCAP, executable, or proprietary course materials.
