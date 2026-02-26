# Lab 4 — Operating Systems & Networking

# Task 1 — Operating System Analysis

## 1.1 Boot Performance Analysis

### systemd-analyze

```bash
$ systemd-analyze
Startup finished in 10.227s (firmware) + 4.408s (loader) + 3.907s (kernel) + 12.596s (userspace) = 31.139s
graphical.target reached after 12.569s in userspace.
````

### systemd-analyze blame (top entries)

```bash
$ systemd-analyze blame
7.395s NetworkManager-wait-online.service
2.677s docker.service
1.070s tor@default.service
909ms input-remapper-daemon.service
901ms logrotate.service
...
```

### Observations

* Total boot time: **31.139 seconds**
* Longest startup service: **NetworkManager-wait-online.service (7.395s)**
* Docker service also contributes noticeably (2.677s)
* Network-related services significantly affect boot time

---

## System Load

```bash
$ uptime
11:15:18 up 7 min,  3 users,  load average: 0.44, 0.82, 0.53
```

```bash
$ w
11:15:34 up 8 min,  3 users,  load average: 0.42, 0.79, 0.53
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
nikkimen tty7     :0               11:08    8:01  25.67s  0.14s /usr/bin/startplasma-x11
nikkimen pts/0    :0               11:08    7:17   0.00s  1.67s /usr/bin/kded5
nikkimen pts/1    :0               11:12    0.00s  0.10s  0.05s w
```

### Observations

* System uptime: 7 minutes
* Load average below 1.0 → system not under heavy load
* 3 active sessions

---

## 1.2 Process Forensics

### Top Memory Consumers

```bash
$ ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head -n 6
PID    PPID CMD                         %MEM %CPU
2958    2232 /opt/vivaldi/vivaldi-bin     2.3  7.2
2281     677 /usr/bin/plasmashell --no-r  1.5  2.1
3045    2975 /opt/vivaldi/vivaldi-bin --  1.2  3.5
3899    2975 /opt/vivaldi/vivaldi-bin --  1.2  3.4
3120    2975 /opt/vivaldi/vivaldi-bin --  1.1  1.8
```

### Top CPU Consumers

```bash
$ ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 6
PID    PPID CMD                         %MEM %CPU
4177    4040 ps -eo pid,ppid,cmd,%mem,%c  0.0  300
570       1 avahi-daemon: running [debi  0.0  7.4
2958    2232 /opt/vivaldi/vivaldi-bin     2.3  7.1
2233     677 /usr/bin/kwin_x11 --replace  0.9  5.9
694     676 /usr/lib/xorg/Xorg -noliste  0.5  5.7
```

### Answer: What is the top memory-consuming process?

**/opt/vivaldi/vivaldi-bin (PID 2958)** using **2.3% memory**

### Observations

* Browser processes consume most memory
* GUI components (plasmashell) are also memory-heavy
* No abnormal CPU spikes observed

---

## 1.3 Service Dependencies

```bash
$ systemctl list-dependencies
default.target
├─accounts-daemon.service
├─docker.service
├─NetworkManager.service
...
```

```bash
$ systemctl list-dependencies multi-user.target
multi-user.target
├─docker.service
├─networking.service
├─tor.service
...
```

### Observations

* `multi-user.target` depends on networking, docker, tor, etc.
* System services organized hierarchically
* Clear dependency chain through `basic.target` and `sysinit.target`

---

## 1.4 User Sessions

```bash
$ who -a
           system boot  2026-02-26 11:07
           run-level 5  2026-02-26 11:07
nikkimen + tty7         2026-02-26 11:08 00:21        1999 (:0)
nikkimen + pts/0        2026-02-26 11:08 00:20        2232 (:0)
nikkimen - pts/1        2026-02-26 11:12   .          4040 (:0)
```

```bash
$ last -n 5
nikkimen pts/1        :0               Thu Feb 26 11:12   still logged in
nikkimen pts/0        :0               Thu Feb 26 11:08   still logged in
nikkimen tty7         :0               Thu Feb 26 11:08   still logged in
reboot   system boot  6.1.0-43-amd64   Thu Feb 26 11:07   still running
nikkimen pts/0        :0               Wed Feb 25 20:03 - 20:04  (00:00)
```

### Observations

* Runlevel 5 (graphical)
* 3 active user sessions
* Recent reboot detected

---

## 1.5 Memory Analysis

```bash
$ free -h
               total        used        free      shared  buff/cache   available
Mem:            19Gi       3.1Gi        13Gi       600Mi       3.6Gi        16Gi
Swap:          976Mi          0B       976Mi
```

```bash
$ cat /proc/meminfo | grep -e MemTotal -e SwapTotal -e MemAvailable
MemTotal:       20383704 kB
MemAvailable:   17104284 kB
SwapTotal:       1000444 kB
```

### Observations

* 19 GB RAM total
* Only ~3.1 GB actively used
* Swap not utilized
* System memory utilization healthy

---

# Resource Utilization Patterns Observed

* Network services slow boot time
* Browser dominates memory usage
* No swap usage → healthy RAM state
* CPU usage distributed normally
* No abnormal spikes

---

# Task 2 — Networking Analysis

## 2.1 Traceroute

```bash
$ traceroute github.com
traceroute to github.com (140.82.121.3), 30 hops max, 60 byte packets
1  _gateway (10.247.1.XXX)  3.656 ms  3.596 ms  3.657 ms
2  10.250.0.XXX (10.250.0.XXX)  1.832 ms  1.897 ms  1.866 ms
...
17  r1-fra3-de.as5405.net (94.103.180.XXX)  59.084 ms  62.372 ms  61.666 ms
...
```

### Insights

* Initial hops are private network addresses
* Traffic exits local network after hop 3
* Route passes through European transit nodes
* Final GitHub IP: 140.82.121.3

---

## DNS Resolution

```bash
$ dig github.com

; <<>> DiG 9.18.44-1~deb12u1-Debian <<>> github.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25487
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;github.com.                    IN      A

;; ANSWER SECTION:
github.com.             12      IN      A       140.82.121.3

;; Query time: 4 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Thu Feb 26 11:30:47 MSK 2026
;; MSG SIZE  rcvd: 55

```

### Observations

* DNS resolved successfully
* Response time: 4 ms
* Local DNS server: 127.0.0.53

---

## 2.2 Packet Capture (Sanitized)

```bash
$ sudo timeout 10 tcpdump -c 5 -i any 'port 443' -nn
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
...
...
11:33:26.985600 enx9c69d343eb64 In  IP 140.82.112.25.443 > 10.247.1.XXX.33782: Flags [P.], seq 1348314370:1348314395, ack 1874601744, win 74, options [nop,nop,TS val 2220590656 ecr 3414793964], length 25
11:33:26.985710 tun2  Out IP 10.247.1.XXX.33782 > 140.82.112.25.443: Flags [.], ack 25, win 501, options [nop,nop,TS val 3414853370 ecr 2220590656], length 0

...
...
5 packets captured
5 packets received by filter
0 packets dropped by kernel
```

### Example Packet Observed

Outbound HTTPS packet:

```
IP 10.247.1.XXX.33782 > 140.82.112.25.443
```

### Analysis

* Encrypted HTTPS traffic detected
* TCP handshake and reset observed
* Communication with GitHub servers confirmed

---

## 2.3 Reverse DNS

### 8.8.4.4

```bash
$ dig -x 8.8.4.4
; <<>> DiG 9.18.44-1~deb12u1-Debian <<>> -x 8.8.4.4
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37987
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;4.4.8.8.in-addr.arpa.          IN      PTR

;; ANSWER SECTION:
4.4.8.8.in-addr.arpa.   6209    IN      PTR     dns.google.

;; Query time: 32 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Thu Feb 26 11:34:11 MSK 2026
;; MSG SIZE  rcvd: 73
```

### 1.1.2.2

```bash
$ dig -x 1.1.2.2
; <<>> DiG 9.18.44-1~deb12u1-Debian <<>> -x 1.1.2.2
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 37876
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;2.2.1.1.in-addr.arpa.          IN      PTR

;; AUTHORITY SECTION:
1.in-addr.arpa.         899     IN      SOA     ns.apnic.net. read-txt-record-of-zone-first-dns-admin.apnic.net. 23597 7200 1800 604800 3600

;; Query time: 472 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Thu Feb 26 11:34:30 MSK 2026
;; MSG SIZE  rcvd: 137
```

### Comparison

* 8.8.4.4 resolves correctly to **dns.google**
* 1.1.2.2 has no PTR record (NXDOMAIN)
* Not all public IPs have reverse DNS entries

---

# Network Observations

* Routing shows multiple transit providers
* DNS resolution fast and stable
* Reverse DNS varies by provider
* HTTPS traffic encrypted
* Internal network uses private IP addressing