# Reskilling — Linux and Cybersecurity

Portfolio repository with the technical documentation of the practical labs from the
**Reskilling — Linux and Cybersecurity** track, promoted by Cabo Verde's Ministry of
Digital Economy, in partnership with Skodji Digital and the World Bank Group.

Each session corresponds to a folder with a technical report (`README.md`) and real
evidence collected in a practical environment (TryHackMe / KillerCoda Ubuntu
Playground).

**Student:** Arilson do Rosário

---

## Session Index

| Session | Title | Learning Objective | Report |
|---|---|---|---|
| 01 | Introduction to Linux for Security and Network Commands | LO1 — Analyze | [sessao-01/README.md](sessao-01/README.md) |
| 02 | Linux System Auditing and Advanced Log Analysis | LO2 — Evaluate | [sessao-02/README.md](sessao-02/README.md) |
| 03 | Linux Network Hardening and Firewall Configuration | LO3 — Apply | [sessao-03/README.md](sessao-03/README.md) |
| 04 | Secure Management of Remote SSH Access on Linux | LO4 — Apply | [sessao-04/README.md](sessao-04/README.md) |

---

## Session Summaries

### Session 01 — Network Reconnaissance
Nmap scan against a remote Windows Server target, identifying 5 open ports
(FTP, DNS, HTTP, RPC, RDP) and analyzing the local network environment
(`ip a`, `ss -tuln`). Anonymous FTP login and risky HTTP methods were flagged as
key findings.

### Session 02 — SSH Log Forensics
Investigation of an SSH brute-force attack through analysis of `/var/log/auth.log`,
using `grep`, `awk`, and `journalctl`, culminating in the identification of the
attacker's IP, the compromise timestamp, and the affected user.

### Session 03 — Firewall Hardening
Configuration of a "deny everything by default, open only what is essential"
security policy using UFW and iptables, including the distinction between DROP and
REJECT and blocking a simulated malicious IP.

### Session 04 — Secure SSH Access
Elimination of password authentication in favor of Ed25519 cryptographic keys,
blocking direct root login, and changing the default SSH port, always validating
the configuration before restarting the service.

---

## Practical Environments Used

- [TryHackMe](https://tryhackme.com/)
- [KillerCoda Ubuntu Playground](https://killercoda.com/playgrounds/scenario/ubuntu)

## Author

Reskilling — Linux and Cybersecurity track
Skodji Digital / Ministry of Digital Economy — Government of Cabo Verde
