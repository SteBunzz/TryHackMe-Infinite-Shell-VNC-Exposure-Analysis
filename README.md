# Infinite Shell – Potential Unintended Attack Surface (TryHackMe)

This repository documents the identification of a **potential unintended attack surface**
during analysis of the **Infinite Shell** room on the TryHackMe platform.

An **unauthenticated WebSocket service exposed on TCP port 80** was found to proxy
connections directly to a **live VNC session**, effectively exposing a graphical shell
over HTTP/WebSocket.

The behavior appears **independent from the intended challenge path** and may be related
to VM infrastructure or misconfiguration.

---

## Repository Contents

- `THM - Infinite Shell - Bug Repo.md`  
  Full technical analysis, discovery process, and impact assessment.

- `LICENSE`  
  Project license.

---

## Scope

- Platform: TryHackMe  
- Room: Infinite Shell  
- Target: Linux VM  
- Access level: User (`ubuntu`)

All observations were made during **normal enumeration and protocol verification**.
No persistence or destructive actions were performed.

---

## Purpose

This repository is shared for:

- Educational purposes
- Security research documentation
- Awareness of realistic infrastructure-level attack surfaces

It is **not intended as an exploit** and does not provide weaponized tooling.

---

## Disclaimer

This project is **not affiliated with TryHackMe**.  
The findings may be the result of intentional lab setup or underlying infrastructure
and should not be interpreted as a vulnerability disclosure unless confirmed by the platform.

---

## License

This project is released under the **MIT License**.
See the `LICENSE` file for details.
