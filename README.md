# Security Writeups

My personal writeups from practising offensive security on **intentionally vulnerable
training machines** (VulnHub / PentesterLab style boot-to-root boxes) and **Capture
the Flag (CTF) competitions**. They document my own learning process: enumeration,
exploitation and privilege escalation on lab targets, and challenge solutions across
crypto, web, forensics, reversing and general skills.

> The writeups themselves are written in **Traditional Chinese** (my study notes). This
> README is an English index so the scope and the tools/techniques are clear at a glance.
> Screenshots are referenced from my HackMD uploads.

## Contents

```
machines/   boot-to-root practice machines (enumeration → foothold → privilege escalation)
ctf/        writeups from CTF competitions, grouped by event
picoctf/    picoCTF challenges, grouped by category
```

## Lab machines — `machines/`

Boot-to-root exercises on deliberately vulnerable VMs. Each writeup walks from network
and service enumeration, through gaining an initial foothold, to privilege escalation to
root. Recurring areas: Linux privilege escalation (SUID binaries, file capabilities,
writable system files, misconfigured `sudo`), web-app footholds, and credential recovery.

| Machine | Focus area |
|---|---|
| Alpine | NFS/SMB enumeration, SSH, Linux privilege escalation |
| Breakout | Service enumeration, Linux capabilities privilege escalation |
| DC-1 | Vulnerable CMS (Drupal), SUID-based privilege escalation |
| Easy v3 | WordPress enumeration (wpscan), database & `sudo` privilege escalation |
| Empire | Web foothold and Linux privilege escalation |
| FristiLeaks 1.3 | Web exploitation and privilege escalation |
| Matrix | Enumeration, foothold and privilege escalation |
| Metasploitable | Classic vulnerable-service enumeration |
| Napping | Web-based foothold and `sudo` privilege escalation |
| PentesterLab – After reporting | Extended web-exploitation exercise (most detailed writeup) |
| PumpkinFestival | Multi-stage boot-to-root (Mission-Pumpkin series) |

## CTF competitions — `ctf/`

Writeups from CTF events I played. Challenges span web exploitation, cryptography,
reversing, forensics and misc.

| Event | Categories covered |
|---|---|
| AIS3 2025 | Web, misc, reversing |
| CRHCCTF | Misc, cryptography |
| HITCON CTF 2025 | Cryptography (RSA) |
| idek CTF | Cryptography, scripting |
| Jingle Shells CTF | Web exploitation, steganography |
| radioactive | Reversing, beginner challenges |
| THJCC 2025 | Cryptography (RSA), misc |

## picoCTF — `picoctf/`

picoCTF challenges organised by category — a good overview of fundamentals across the
main CTF disciplines.

| Category | Notes |
|---|---|
| General Skills | Linux/CLI fundamentals, scripting, Rust fix-me tasks (most detailed) |
| Web Exploitation | Common web vulnerabilities (robots.txt, SSTI, auth bypass, etc.) |
| Reverse Engineering | Static/dynamic analysis of binaries |
| Cryptography | Classical ciphers, hashing, encoding |
| Forensics | File/metadata analysis, steganography, packet captures |
| picoCTF 2025 | Mixed challenges from the 2025 event |

## Tools used across these writeups

Nmap, netdiscover, enum4linux, Burp Suite, Wireshark, wpscan, Hydra, John the Ripper,
Hashcat, Metasploit, linpeas, CyberChef / dcode-style decoders, and standard Linux
command-line tooling.

---

*These are educational writeups against intentionally vulnerable lab targets and public
CTF challenges, kept as a personal study record.*
