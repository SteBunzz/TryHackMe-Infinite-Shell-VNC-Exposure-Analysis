# Potential Unintended Attack Surface – Infinite Shell (TryHackMe)

## Summary
During analysis of the **Infinite Shell** room VM, an **unauthenticated WebSocket service on TCP port 80** was identified.  
This service proxies connections directly to a **live VNC session**, effectively exposing a **persistent graphical shell** over HTTP/WebSocket.

This behavior appears **unrelated to the intended challenge path**, but may represent an **unintended attack surface** in the VM environment.

---

## Environment
- Platform: TryHackMe
- Room: Infinite Shell
- Target: Linux VM provided by the room
- Access Level: User (ubuntu)

---

## Discovery Process

# 1️⃣. Service Enumeration

- TCP port **80** identified as open
- Standard HTTP methods (`GET`, `POST`, `OPTIONS`) rejected
- Server response header:
Server: WebSockify Python/3.x

# 2️⃣. Process Inspection

Running processes revealed:


```
python3 -m websockify 80 localhost:5901 -D
```

## This indicates:

WebSockify listening on port 80

Traffic forwarded to VNC (5901)

Daemonized and persistent

---
# 3️⃣ Protocol Verification

A manual WebSocket handshake was performed:

```
curl -i \
  -H "Connection: Upgrade" \
  -H "Upgrade: websocket" \
  http://<VM-IP>
```

## Response:

- 101 Switching Protocols
- Payload contained RFB 003.008

## This confirms:

- Successful WebSocket upgrade
- Exposure of VNC (RFB protocol) via HTTP/WebSocket

---

# 4️⃣ VNC Exposure Evidence

- Active TigerVNC session observed:
	- Listening on 0.0.0.0:5901
	
- VNC runtime artifacts present:
	- ~/.vnc/tryhackme:1.log
	- Persistent desktop session

---

# 5️⃣VNC Credential Usage and Session Access

## -VNC Configuration Directory

Path: /home/ubuntu/.vnc/
Contents:
- `passwd` — VNC password file (hashed, 8 bytes)
- `tryhackme:1.log` — VNC session log
- `tryhackme:1.pid` — PID file for the active VNC server
- `xstartup` — startup script launching a full MATE desktop session

---

# 6️⃣⚠️VNC Authentication Files

- The VNC service required authentication when accessed:

```
vncviewer localhost:5901
```
- Authentication failed when no password was provided

- The running VNC server explicitly referenced a password file:
		Tried to load this.
```
/tmp/tigervnc.<random>/passwd
```

- A successful connection was established using:
```
vncviewer localhost:5901 -passwd /tmp/tigervnc.Pk3ha1/passwd
```
## ⚠️Session Result

- A graphical session initialization was triggered
- Multiple GUI windows were rendered incompletely and repeatedly
- Severe graphical artifacts (pixelation, RGB distortions) were observed
- The system became unstable and temporarily unresponsive

##  Evidence of Active Usage
- tryhackme:1.log confirms:
	- recent session start
	- active connections
	- session resets
- Timestamp confirms activity on:
```
Jan 28
```
## ⚠️ Technical Impact
- Password-based GUI access available locally without additional restrictions
- Direct reuse of server-side credential files enabled session access
- Uncontrolled GUI session spawning led to system instability
# 7️⃣Impact Assessment
- Port 80 allows unauthenticated access to a WebSocket endpoint
- WebSocket directly bridges to a live graphical VNC shell
- Potential implications in a real environment:
	- Remote desktop access
	- Persistent attacker foothold
	- Full interactive control of the system

---

# 8️⃣Stability Observation
- Attempting full interaction via a VNC/WebSocket client caused VM instability
- Suggests exposure may be unintentional or infrastructure-related

---
# ⚠️ SPOILER ALERT
Intended Challenge Path (Context)
- The official solution path involved:
	- Apache log analysis
	- Detection of Base64-encoded payloads
- The VNC/WebSocket exposure appears independent of the exercise objective

# ℹ️Responsible Disclosure Note
This report is shared for:
- Awareness
- Validation
- Possible hardening or clarification

No exploitation beyond protocol verification was performed.

--- 
# Conclusion
An exposed WebSocket-to-VNC bridge on port 80 was identified during normal enumeration.
While likely part of the VM infrastructure, it represents a realistic persistence mechanism that could be misinterpreted as part of the challenge or abused in other contexts.

---
# References
- WebSockify
- VNC / RFB Protocol
- WebSocket Upgrade Mechanism
