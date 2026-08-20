# Network Security Practical — Traffic Analysis & Firewall Hardening

---

## What's in here

This started as a look at network security fundamentals — the usual suspects: viruses, worms, Trojans, phishing — and the core defenses (firewalls, encryption, defence-in-depth). But the more interesting part is the practical section, where I:

- Pulled my network configuration with `ipconfig`
- Captured live traffic with Wireshark across DNS, HTTP, TCP, and UDP
- Filtered down to something that looked *wrong*
- Confirmed it was an active TCP SYN scan targeting my host
- Responded by hardening the firewall against the exposed service and other insecure protocols

## Background: Threats Covered

| Threat | Key Trait |
|---|---|
| **Viruses** | Attach to legitimate files, need user action to spread |
| **Worms** | Self-replicate with zero human interaction — fast and dangerous |
| **Trojans** | Rely on deception, not replication — usually deliver a backdoor |
| **Phishing** | Still the most common and costliest attack vector out there |

And the core defensive concepts: firewalls (especially NGFWs), TLS/SSL encryption, network segmentation, patch management, and zero-trust access control.

## Tools Used

- **Wireshark** — packet capture and traffic analysis
- **Windows Defender Firewall (Advanced Security)** — rule creation and hardening
- **Command Prompt (`ipconfig`)** — network configuration review

## The Interesting Bit: Catching a Live Port Scan

While filtering traffic, I noticed something odd around a specific point in the capture: an internal host started firing off a rapid burst of SYN packets across multiple ports (80, 445, 62078) in the space of milliseconds, then pivoted hard into repeatedly hammering **port 5357** — the port used by **WS-Discovery**.

A second host on a different subnet did the exact same thing shortly after, targeting the same port.

Using a filter like:

```
ip.addr == <scanner_ip> && tcp.flags.syn == 1 && tcp.flags.ack == 1
```

I confirmed my host replied with a SYN-ACK (port 5357 was open and listening) — but the scanner never completed the handshake. It just moved on, leaving a trail of TCP retransmissions in its wake.

That pattern — SYN sent, SYN-ACK received, connection abandoned, repeat — is a textbook signature of a **TCP SYN stealth scan**. In other words: someone (or something automated) was actively probing my host for open services.

## Response: Firewall Hardening

Two rules went in as a direct result of what the traffic revealed:

1. **Block Telnet (port 23)** — proactive hardening. Telnet sends credentials in plaintext, so there's no good reason to leave it open.
2. **Block WS-Discovery (port 5357)** — evidence-based hardening. This one came straight out of the investigation above: the port was actively being probed, so it got closed.

This is really the core idea behind the whole exercise — don't just configure a firewall based on best-practice checklists, configure it based on what your own traffic is actually telling you.

## Repo Structure

```
├── README.md
├── report/              # Full writeup (viruses, worms, trojans, phishing, firewalls, encryption)
├── screenshots/          # Wireshark captures + firewall rule configs
└── references/           # Full source list (APA-style)
```

## 📚 References

Full citation list is included in the report — sources include CISA, NSA, Check Point, Fortinet, Norton, Malwarebytes, and peer-reviewed papers from journals like *Computers & Security* and *Frontiers in Computer Science*.

---

**Author:** Lulamela Maxakatho

*If you spot something I should tighten up or explain better, feel free to open an issue — always happy to compare notes on this stuff.*
