# Lab Report: Monitoring, Logging & Incident Detection

**Course:** IKB42603 Cloud Computing Security Essentials  
**Student:** Eikmal   
**Lecturer:** Madam Adani  

---

## Session A (Week 9) - Logging & Centralisation

### Task 1 & 2: Generate & Centralise Logs (CloudWatch)
**Objective:** Create a local application log of authentication events and securely transmit them to a centralized logging service (AWS CloudWatch via LocalStack) to ensure visibility and prevent local log tampering.

**Walkthrough & Steps Taken:**
1. Initialized a LocalStack container (`localstack/localstack:3.0.0`) mapping port 4566.
2. Configured AWS CLI to use the LocalStack endpoint and created a new log group (`/ccse/app`) and log stream (`auth`).
3. Generated a simulated local `auth.log` file containing standard logins, failed brute-force attempts, and a data export event.
4. Used a bash `while` loop to read the local log line-by-line, append a millisecond timestamp, and push each event to the CloudWatch stream using `put-log-events`.
5. Read the centralized logs back from CloudWatch using `get-log-events` to verify successful transmission.

<img width="671" height="520" alt="Screenshot 2026-09-02 182238" src="https://github.com/user-attachments/assets/b242cb55-4217-4eec-802a-02a8efee38f2" />
<img width="1906" height="223" alt="Screenshot 2026-09-02 182332" src="https://github.com/user-attachments/assets/61068501-3662-4e35-abff-9ffd9f2d81d3" />

---

### Task 3: Query for Security-Relevant Activity
**Objective:** Parse raw log data to extract actionable security intelligence, specifically identifying the source of a brute-force attack.

**Walkthrough & Steps Taken:**
1. Used `grep` to filter the local `auth.log` for `LOGIN_FAIL` events.
2. Piped the output into `awk` to extract the IP address column, then used `sort` and `uniq -c` to count the occurrences.
3. Successfully identified that the IP address `203.0.113.9` failed to authenticate 4 times.

<img width="571" height="64" alt="Screenshot 2026-09-02 182347" src="https://github.com/user-attachments/assets/e41b2184-528f-40d4-b4c5-997d06a79ae0" />


---

## Session B (Week 10) - Tamper-Proofing, Detection & Response

### Task 4: Tamper-Proof (Hash-Chained) Logs
**Objective:** Implement cryptographic hash-chaining to ensure log integrity, allowing the system to instantly detect if an attacker attempts to alter or delete historical records to cover their tracks.

**Walkthrough & Steps Taken:**
1. Created an `auth.chain` file by iterating through `auth.log`. For each line, the script concatenated the previous line's hash with the current line's text and generated a new SHA-256 hash.
2. Simulated a log tampering event by using `sed` to silently change the `EXPORT_DATA` size from `500MB` to `5MB` in a new file (`auth.tampered`).
3. Recalculated the hash chain on the tampered file. The final hash (`72f1d5...`) completely diverged from the original final hash (`ababa7...`), successfully proving the chain was broken and tampering was detected.

<img width="1108" height="437" alt="Screenshot 2026-09-02 182519" src="https://github.com/user-attachments/assets/22694be5-4ca1-4cd3-a0c9-1f4e28886da1" />


---

### Task 5: Detect the Incident (Correlation)
**Objective:** Simulate a Security Information and Event Management (SIEM) system by correlating multiple distinct, low-level events into a single high-confidence security alert.

**Walkthrough & Steps Taken:**
1. Executed a correlation script targeting the suspicious IP (`203.0.113.9`).
2. The script counted the occurrences of `LOGIN_FAIL`, `LOGIN_OK`, and `EXPORT_DATA` originating from that specific IP.
3. The script's logic (`FAILS >= 3 AND SUCCESS >= 1 AND EXPORT >= 1`) was triggered, generating a critical alert: `probable brute-force -> compromise -> data exfiltration`.

<img width="611" height="230" alt="Screenshot 2026-09-02 182541" src="https://github.com/user-attachments/assets/f1fe40bb-7ea5-4380-99c8-0a3468f1710f" />

---

### Task 6: Incident Response (Contain & Collect)
**Objective:** Execute the standard incident response lifecycle to stop the active threat and preserve immutable forensic evidence.

**Walkthrough & Steps Taken:**
1. **Containment:** Deployed an Alpine container with `NET_ADMIN` capabilities and used `iptables` to create a `DROP` rule specifically targeting all inbound traffic from the attacker's IP (`203.0.113.9`).
2. **Collection:** Copied the raw `auth.log` to a timestamped evidence file (`evidence_20260902.log`).
3. **Integrity:** Generated a SHA-256 hash of the evidence file (`evidence.sha256`) to ensure it can be cryptographically proven in the future that the forensic copy was not altered post-collection.

<img width="771" height="195" alt="Screenshot 2026-09-02 182601" src="https://github.com/user-attachments/assets/bd72bd1b-e861-437f-bb27-3f25c4d29a4a" />


---

## Deliverables: Incident Report

### Incident Report: Data Exfiltration via Brute-Force Compromise
**Detection:** The incident was detected on March 1, 2025, via automated log correlation. While individual `LOGIN_FAIL` events did not trigger a critical alarm, a SIEM correlation script identified a rapid succession of events originating from IP `203.0.113.9`: four failed administrative logins, immediately followed by a successful login, and culminating in a 500MB data export. 

**Analysis:** The attacker successfully executed a brute-force or credential-stuffing attack against the `admin` account. Upon gaining access, they immediately pivoted to exfiltrate a large volume of data. Post-incident analysis of the hash-chained logs revealed the attacker also attempted to tamper with the audit trail by modifying the exported data size from `500MB` to `5MB`, but the cryptographic chain successfully detected the alteration.

**Containment:** The active threat was neutralized by applying a host-level network firewall rule (`iptables -A INPUT -s 203.0.113.9 -j DROP`), immediately severing the attacker's connection and preventing further lateral movement or exfiltration.

**Evidence & Integrity:** A point-in-time forensic copy of the application logs was collected and saved as `evidence_20260902.log`. To ensure the chain of custody and guarantee the file's immutability for compliance and investigation purposes, a SHA-256 hash (`0adc5d2ac06...`) was generated and securely stored in `evidence.sha256`.

**Lesson Learned:** Relying solely on passwords for administrative accounts is insufficient. To prevent future brute-force compromises, Multi-Factor Authentication (MFA) must be enforced for all logins, and rate-limiting or automated IP banning (e.g., fail2ban) should be implemented after three failed authentication attempts.

---

## Deliverables: Short-Answer Questions

### Q1. What is the difference between a log and an event? Give an example of each from this lab.
* A **log** is a durable, historical, and immutable text record of something that happened in the past. An example from this lab is the line `2025-03-01T09:01:10 LOGIN_FAIL user=admin ip=203.0.113.9` stored statically inside the `auth.log` file.
* An **event** is an active, near-real-time trigger or alert generated when a specific condition is met, often requiring immediate action. An example from this lab is the SIEM correlation script outputting `ALERT: probable brute-force -> compromise -> data exfiltration` to the terminal.

### Q2. Why must audit logs be tamper-proof, and how does a hash chain achieve this?
Audit logs must be tamper-proof because an attacker's first post-exploitation objective is often to alter or delete logs to erase their tracks and maintain persistence. A hash chain prevents this by cryptographically linking each log entry to the one before it. The hash of Line 2 is calculated using the text of Line 2 *plus* the hash of Line 1. If an attacker modifies even a single character in Line 1, the hash for Line 1 changes, which breaks the hash calculation for Line 2, causing a cascading failure that instantly exposes the tampering.

### Q3. How did correlation detect an incident that no single log line revealed?
Correlation detected the incident by analyzing the *context and sequence* of events rather than looking at them in isolation. A single `LOGIN_FAIL` is common (e.g., a typo). A single `LOGIN_OK` is normal behavior. A single `EXPORT_DATA` might be a routine administrative backup. None of these individual lines justify a critical alert. However, by correlating them across time and linking them to a single IP address, the SIEM identified the malicious pattern: the failure attempts proved intent, the success proved compromise, and the export proved data theft.

### Q4. List the incident-response steps you performed and the goal of each.
1. **Containment:** Applied an iptables `DROP` rule to block the attacker's IP. The goal is to stop the bleeding, halt active exfiltration, and prevent the attacker from doing further damage.
2. **Collect Evidence:** Copied the raw logs to a timestamped file and generated a SHA-256 hash. The goal is to preserve the state of the system for forensic analysis and ensure the evidence is immutable so it holds up in an audit or court.
3. **Document:** Wrote an incident report outlining the timeline and lessons learned. The goal is to inform stakeholders, fulfill compliance requirements, and improve future security posture.

### Q5. How do the same logs serve both security monitoring and compliance evidence (Weeks 6, 11)?
* For **security monitoring (Week 6)**, logs are ingested in real-time by SIEM tools to trigger immediate alerts, identify active threats, and allow security teams to respond to incidents as they happen (e.g., blocking an IP during a brute-force attack).
* For **compliance evidence (Week 11)**, those exact same logs are archived in a secure, tamper-proof, append-only centralized store (like CloudWatch) to prove to external auditors that security controls (like access control and monitoring) were actively enforced and operating effectively over a specific historical period.
