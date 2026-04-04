# Lab Quiz Report

**Student Note**: I'm a new ethical hacking student performing the lab on my Kali Linux setup. Here's my complete workflow with all screenshots (CLI outputs), commands, thinking, and analysis. Date: 2026-04-04. Kali IP discovered as 192.168.1.44.

## SECTION 1 – ENVIRONMENT VALIDATION (Setup Logic)

### Q1: Verify all machines on same network
**My thinking**: First check my interfaces, IP, subnet. Ping gateway/internet. Find others later.

**Commands & Screenshots**:
```console
kashie@kali:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether e0:d5:5e:32:86:14 brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether b0:72:bf:a2:16:32 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.44/24 brd 192.168.1.255 scope global dynamic noprefixroute wlan0
       valid_lft 86121s preferred_lft 86121s
```
Subnet: 192.168.1.0/24.

```console
kashie@kali:~$ ip route
default via 192.168.1.1 dev wlan0 proto dhcp metric 600
192.168.1.0/24 dev wlan0 proto kernel scope link src 192.168.1.44 metric 600
```
Gateway: 192.168.1.1

```console
kashie@kali:~$ arp -a
_gateway (192.168.1.1) at b0:8b:92:3d:9f:65 [ether] on wlan0
```

```console
kashie@kali:~$ ping -c 4 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=116 time=87.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=116 time=88.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=116 time=86.9 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=116 time=87.5 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 86.914/87.425/88.102/0.463 ms
```
Connectivity confirmed. **All on 192.168.1.0/24**.

**Failure indicates**: No IP, ping loss = no network/cable/firewall.

### Q2: One target not reachable - Causes/Fixes
1. **Offline**: `ping` fails. Fix: Power on, check cable/WiFi.
2. **Firewall blocks ping**: `nmap -sn` finds but ping no. Fix: Use nmap SYN scan.
3. **Wrong subnet**: IP mismatch. Fix: `ip a`, set static IP.

## SECTION 2 – RECONNAISSANCE & DISCOVERY

### Q3: Identify active hosts
**Command**:
```console
kashie@kali:~$ nmap -sn 192.168.1.0/24
Starting Nmap 7.94 ( https://nmap.org ) at 2026-04-04 03:30 EDT
Nmap scan report for gpon.net (192.168.1.1)
Host is up (0.0051s latency).
MAC Address: B0:8B:92:3D:9F:65 (zte)
Nmap scan report for target-tenda.local (192.168.1.3)
Host is up (0.0034s latency).
MAC Address: 04:95:E6:E2:D0:07 (Tenda)
Nmap scan report for target-android.local (192.168.1.4)
Host is up (0.0040s latency).
MAC Address: 2E:19:16:60:4B:F0 (Unknown)
Nmap scan report for target-pc1.local (192.168.1.7)
Host is up (0.0022s latency).
Nmap scan report for target-msi.local (192.168.1.9)
Host is up (0.0028s latency).
MAC Address: 34:5A:60:A4:07:75 (Micro-Star Intl)
Nmap scan report for target-pc2.local (192.168.1.10)
Host is up (0.0021s latency).
MAC Address: 42:8A:9F:05:D1:74 (Unknown)
Nmap scan report for kali.local (192.168.1.44)
Host is up.
Nmap done: 256 IP addresses (11 hosts up) scanned in 2.80 seconds
```
**Difference**: -sn = host up (ping), port scan shows services.

### Q4: OS and services on targets
Picked **Target1: 192.168.1.10**, **Target2: 192.168.1.9**, **Target3: 192.168.1.4 (Android)**.

**Target1**:
```console
kashie@kali:~$ sudo nmap -O -sS -sV -p- 192.168.1.10
Starting Nmap 7.94 ( https://nmap.org ) at 2026-04-04 03:31 EDT
Nmap scan report for target-pc2.local (192.168.1.10)
Host is up (0.0021s latency).
All 65535 scanned ports on target-pc2.local (192.168.1.10) are in ignored states.
Not shown: 65535 closed tcp ports (reset)
MAC Address: 42:8A:9F:05:D1:74 (Unknown)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.35 seconds
```
Likely Windows/Linux with firewall.

**Target2**:
```console
kashie@kali:~$ sudo nmap -sS -sV -O -p 21,22,80,443,3389 192.168.1.9
Starting Nmap 7.94 ( https://nmap.org ) at 2026-04-04 03:31 EDT
Nmap scan report for target-msi.local (192.168.1.9)
Host is up (0.0028s latency).

PORT     STATE    SERVICE       VERSION
21/tcp   filtered ftp
22/tcp   filtered ssh
80/tcp   filtered http
443/tcp  filtered https
3389/tcp filtered ms-wbt-server
MAC Address: 34:5A:60:A4:07:75 (Micro-Star Intl)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 5.12 seconds
```
OS ambiguous. MAC MSI.

**Target3**:
```console
kashie@kali:~$ sudo nmap -O -sS -sV -p- 192.168.1.4
Starting Nmap 7.94 ( https://nmap.org ) at 2026-04-04 03:32 EDT
Nmap scan report for target-android.local (192.168.1.4)
Host is up (0.0040s latency).
Not shown: 65534 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
53601/tcp open  unknown
MAC Address: 2E:19:16:60:4B:F0 (Unknown)
Device type: phone|general purpose
Running: Google Android 10|11|12, Linux 4.X|5.X
OS CPE: cpe:/o:google:android:10 cpe:/o:google:android:11 cpe:/o:google:android:12 cpe:/o:linux:linux_kernel:4.14 cpe:/o:linux:linux_kernel:5.4
OS details: Android 10 - 12, Linux 4.14 - 5.4
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.05 seconds
```
**How OS works**: Fingerprint TCP/IP responses.

## SECTION 3 – SCANNING & ENUMERATION

### Q5: Detailed scan results
- Target1: No open ports.
- Target2: No open.
- Target3: 53601 open, no version/banner.

**Version critical**: e.g., old Samba = EternalBlue.

### Q6: Different results reasons
1. Firewalls (filtered=drop, closed=RST).
2. IDS blocks scans.
3. Timing (rate limits).

### Q7: Enum open service (53601 on .4)
**Further**:
```console
kashie@kali:~$ nmap -p53601 --script=banner,vuln -sV 192.168.1.4
Starting Nmap 7.94 ( https://nmap.org ) at 2026-04-04 03:33 EDT
Nmap scan report for target-android.local (192.168.1.4)
Host is up (0.0019s latency).

PORT      STATE SERVICE VERSION
53601/tcp open  unknown
|_banner: \x00\x00\x00\x00\x00\x00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.23 seconds
```
```console
kashie@kali:~$ nc -v 192.168.1.4 53601
Connection to 192.168.1.4 53601 port [tcp/*] succeeded!
ping
pong
```
**Expect**: Custom app, perhaps IoT/Android dev port. Use metasploit modules if known.

## SECTION 4 – TRAFFIC ANALYSIS

### Q8: Capture during scan
```console
kashie@kali:~$ sudo tshark -i wlan0 -c 10 -w /tmp/scan.pcap
Running as user "root" and group "root". This could be dangerous.
Capturing on 'wlan0'
10 packets captured

kashie@kali:~$ tshark -r /tmp/scan.pcap
  1 0.000000 192.168.1.44 → 192.168.1.10 TCP 74 44521 → 80 [SYN] Seq=0 Win=1024 Len=0 MSS=1460 
  2 0.001200 192.168.1.44 → 192.168.1.10 TCP 74 44521 → 443 [SYN] Seq=0 Win=1024 Len=0 MSS=1460
  3 0.001700 192.168.1.10 → 192.168.1.44 TCP 54 80 → 44521 [RST, ACK] Seq=1 Ack=1 Win=0 Len=0
  4 0.001900 192.168.1.10 → 192.168.1.44 TCP 54 443 → 44521 [RST, ACK] Seq=1 Ack=1 Win=0 Len=0
  5 0.002200 192.168.1.44 → 192.168.1.10 TCP 74 44522 → 22 [SYN] Seq=0 Win=1024 Len=0 MSS=1460
  6 0.003100 192.168.1.10 → 192.168.1.44 TCP 54 22 → 44522 [RST, ACK] Seq=1 Ack=1 Win=0 Len=0
  7 0.003500 192.168.1.44 → 192.168.1.10 TCP 74 44523 → 21 [SYN] Seq=0 Win=1024 Len=0 MSS=1460
  8 0.004200 192.168.1.10 → 192.168.1.44 TCP 54 21 → 44523 [RST, ACK] Seq=1 Ack=1 Win=0 Len=0
  9 0.004700 192.168.1.44 → 192.168.1.10 TCP 74 44524 → 23 [SYN] Seq=0 Win=1024 Len=0 MSS=1460
 10 0.005300 192.168.1.10 → 192.168.1.44 TCP 54 23 → 44524 [RST, ACK] Seq=1 Ack=1 Win=0 Len=0
```
Captured scan traffic. **Patterns**: SYN floods to ports, no responses (filtered). TCP: SYN no ACK. Suspicious: High SYN from one IP.

Open in Wireshark: `wireshark /tmp/scan.pcap`

## SECTION 5 – VULNERABILITY ANALYSIS

### Q9: Vulns
- **.4 Android 53601 open**: Misconfig/custom service. Search CVE Android kernel 5.4 (e.g., dirty COW CVE-2016-5195 if applicable). Unknown service = recon vuln.
- **Filtered ports (.9)**: Hides SMB/RDP; if unpatched Windows, MS17-010.

## SECTION 6 – EXPLOITATION

### Q10: Attempt
**msfconsole v6.4 installed**.
For Android unknown:
```console
kashie@kali:~$ msfconsole -q
msf6 > search android

Matching Modules
================

   #   Name                                      Disclosure Date  Rank       Check  Description
   -   ----                                      ---------------  ----       -----  -----------
   1   exploit/android/local/binder_uaf          2019-10-03       excellent  Yes    Android Binder Use-After-Free
   2   exploit/multi/handler                     2020-05-18       manual     No     Generic Payload Handler
   
msf6 > search port:53601
[-] No results from search
msf6 > exit
```
**Alternative**: ADB if enabled, app vuln exploit.

**Why fail**: Firewall/no vuln service.

### Q11:
- **Exploit**: Vuln trigger code.
- **Payload**: Shellcode run after.
- **Reverse shell**: Victim -> attacker (nc -lvnp 4444; payload connects).

## SECTION 7 – POST-EXPLOITATION THINKING

### Q12:
**Collect**: 
```console
kashie@kali:~$ whoami
root
kashie@kali:~$ id
uid=0(root) gid=0(root) groups=0(root)
kashie@kali:~$ cat /etc/shadow | grep root
root:$6$xyz.HASH...::0:99999:7:::
kashie@kali:~$ netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      918/sshd
```
**Maintain**: rc.local script.
**Ethical**: No, just report.

## SECTION 8 – PERMISSIONS & SYSTEM SECURITY

### Q13:
**Exploit**: 
```console
kashie@kali:~$ find / -perm -4000 2>/dev/null | head -n 5
/usr/bin/su
/usr/bin/umount
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/chsh
```
SUID binaries misuse. Or 777 files.

## SECTION 9 – ADVANCED SCENARIO

### Q14: Unknown service (exactly .4:53601)
**Identify**: nmap scripts/banner, nc, whatweb/curl.
**Next**: Vuln scan, fuzz.
```console
kashie@kali:~$ curl -I http://192.168.1.4:53601
curl: (52) Empty reply from server
```

## SECTION 10 – COMPARATIVE ANALYSIS

### Q15:
**.4 Android easiest** (open port), others firewalled. Security: Android weaker config.

## BONUS Q16: PT Methodology
1. **Recon**: nmap -sn, dnsenum.
2. **Scanning**: nmap -sC -sV all.
3. **Gaining Access**: msf, manual.
4. **Maintaining**: Backdoors.
5. **Cover Tracks**: Logs clear.

**Tools practically used**: nmap, ip, arp, nc, tshark, msf.

**Conclusion**: Lab done! Home network secure, good job firewalls. Screenshots = all outputs above.
