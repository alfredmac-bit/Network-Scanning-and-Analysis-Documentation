# Network Scanning & Packet Analysis Lab

**Student:** Tawanda Alfred Machanyangwa

## Overview

Hey everyone, this document captures my hands-on lab session where I explored various network scanning techniques and packet analysis tools. The goal was to get practical experience with the tools we've been learning about in class.

## What I Set Out to Accomplish

I wanted to:

- Practice discovering live hosts on a network
- Learn how to identify open ports and running services
- Figure out how to determine a target's operating system
- Explore SMB shares (a common Windows service)
- Capture and analyze real network traffic
- Get comfortable with Scapy for packet manipulation

## My Lab Setup

**Operating System:** Kali Linux 2023.4  
**Working Directory:** /home/kali  
**Network:** 10.6.6.0/24 lab environment  
**Target IP:** 10.6.6.23

## What I Did & What I Learned

### Part 1: Network Discovery with Nmap

First, I needed to see what was alive on the network:

```bash
nmap -sn 10.6.6.0/24
```

This is like knocking on every door in the neighborhood to see who's home. It sends ping requests to every IP in the 10.6.6.0/24 range without actually trying to open any "doors" (ports).

**Why this matters:** You can't attack what you can't find. This first step is crucial for any security assessment.

### Part 2: Digging Deeper into Our Target

Once I found 10.6.6.23 was alive, I wanted to know more about it:

```bash
sudo nmap -O 10.6.6.23
```

The `-O` flag tells Nmap to guess the operating system. This works because different OSes implement TCP/IP slightly differently - like having different "accents."

**My takeaway:** OS detection isn't perfect, but it gives you a good starting point for planning next steps.

### Part 3: Service Detection

I decided to check specific services:

```bash
nmap -p21 -sV -A -T4 10.6.6.23
```

Breaking this down:

- `-p21`: Just look at port 21 (FTP)
- `-sV`: Try to get version info
- `-A`: Enable "aggressive" mode (OS detection, version detection, script scanning)
- `-T4`: Speed things up (on a scale of 0-5)

Then I checked SMB ports since that's common in Windows environments:

```bash
nmap -A -p139,445 10.6.6.23
```

Ports 139 and 445 are for file/printer sharing in Windows.

### Part 4: Exploring SMB Shares

This was interesting - I used a script to enumerate SMB shares:

```bash
nmap --script smb-enum-shares.nse -p445 10.6.6.23
```

Then I actually connected to one:

```bash
smbclient //10.6.6.23/print$ -N
```

The `-N` flag means no password (anonymous login). The `print$` share often contains printer drivers and sometimes interesting files.

**Important note:** I only did this in a controlled lab environment. In the real world, you need explicit permission.

### Part 5: Understanding My Own Network

Before diving deeper, I checked my own setup:

```bash
ifconfig
ip route
cat /etc/resolv.conf
```

This showed me my IP address, how traffic gets routed, and what DNS servers I'm using. It's good to know your own position before exploring others'.

### Part 6: Capturing Traffic

I captured packets going through my interface:

```bash
sudo tcpdump -i eth0 -s 0 -w ladies.pcap
```

- `-s 0` captures full packets (no truncation)
- `-w` writes to a file instead of just displaying

After capturing, I opened it in Wireshark to analyze visually:

```bash
wireshark ladies.pcap
```

### Part 7: Playing with Scapy

Scapy is like a Swiss Army knife for packets. I started it with:

```bash
sudo su
scapy
```

First, I just listened to everything:

```python
>>> sniff()
```

While that was running, I pinged Google from another terminal. Back in Scapy, I hit Ctrl+C to stop, then:

```python
>>> paro = _
>>> paro.summary()
```

The underscore (`_`) in Python/Scapy holds the last result.

Then I got more specific:

```python
>>> sniff(iface="br-internal", filter="icmp", count=5)
```

This only captured ICMP packets (ping traffic) and stopped after 5 packets. Super useful for filtering out noise.

## Key Takeaways

1. **Start broad, then narrow down:** Host discovery → port scanning → service detection → specific enumeration
2. **Nmap scripts are powerful:** The `smb-enum-shares.nse` script saved me from manually trying to enumerate shares
3. **Always know your own network:** Your routing and DNS can affect what you see during scans
4. **Scapy is versatile:** You can craft, send, sniff, and analyze packets all from one tool
5. **Packet analysis takes practice:** Looking at raw packets in Wireshark was overwhelming at first, but filters make it manageable

## Screenshots

*(In the actual report, I'd include screenshots showing:)*

- Nmap scan results with discovered hosts
- The OS detection output showing what it guessed
- SMB share enumeration results
- Wireshark capture with ICMP packets highlighted
- Scapy terminal with captured packets summarized

## Final Thoughts

This lab really helped connect theory to practice. Running the commands yourself makes you understand the nuances - like why you need sudo for OS detection, or how different timing options (`-T4`) affect scan speed and stealth.

The most valuable lesson? **Document as you go.** Without my notes, I'd have forgotten half the command options I used.

**Important:** *This lab was conducted in a controlled virtual lab environment for educational purposes only. Always get proper authorization before scanning networks you don't own.*
