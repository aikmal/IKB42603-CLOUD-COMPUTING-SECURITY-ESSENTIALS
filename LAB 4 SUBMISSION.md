# Lab Report: Access Control & Network Security

**Course:** IKB42603 Cloud Computing Security Essentials  
**Student:** Eikmal  
**Date:** September 2, 2026  
**Lecturer:** Madam Adani 

---

## Session A (Week 7) - Authentication & Authorization

### Task 1: Authentication - A Password-Protected Service
**Objective:** Implement HTTP Basic authentication on a web service to enforce the boundary of identity (who you are).

**Walkthrough & Steps Taken:**
1. Created an Apache htpasswd file storing hashed credentials for user `student` with password `P@ssword!`.
2. Configured Nginx (`default.conf`) with `auth_basic "Restricted"` and mapped the credentials file.
3. Launched the `authsvc` container forwarding port 8080.
4. Tested access without credentials (received `401 Unauthorized`) and with valid credentials (received `200 OK` and `Authenticated OK`).

<img width="627" height="436" alt="Screenshot 2026-09-02 180059" src="https://github.com/user-attachments/assets/eb3d5193-b369-44af-8029-f46bb174213a" />


---

### Task 2: Add a Second Factor (MFA / TOTP)
**Objective:** Augment authentication with a time-based one-time password (TOTP) to protect against compromised credentials.

**Walkthrough & Steps Taken:**
1. Generated a random base32 shared secret key from `/dev/urandom`.
2. Calculated the current 6-digit TOTP code using `oathtool --totp -b "$SECRET"`.
3. Simulated user entry by prompting for the code and comparing it against the dynamically generated expected code within the 30-second window, yielding `MFA OK`.


<img width="695" height="196" alt="Screenshot 2026-09-02 180402" src="https://github.com/user-attachments/assets/31103b22-904f-4cd0-8b9e-e7be4b25cba8" />


---

### Task 3: Authorization - RBAC Roles
**Objective:** Configure Kubernetes Role-Based Access Control (RBAC) to enforce least privilege (what an authenticated identity is permitted to do).

**Walkthrough & Steps Taken:**
1. Spun up a local single-node cluster using `kind create cluster --name ccse-lab4`.
2. Created a dedicated namespace `app` and a service account `dev`.
3. Created a role `dev-role` granting only read verbs (`get`, `list`) on `pods`, and bound it to `dev` using `dev-rb`.
4. Verified permissions using `kubectl auth can-i`:
   - `list pods`: **yes**
   - `create deploy`: **no**
   - `delete pods`: **no**


<img width="663" height="602" alt="Screenshot 2026-09-02 180545" src="https://github.com/user-attachments/assets/80ccc2ff-8ba8-46a8-b7ad-69e6f486c958" />


---

## Session B (Week 8) - Network Security & Hardening

### Task 4: Network Segmentation (Three-Tier Architecture)
**Objective:** Isolate tiers using distinct bridge networks to enforce defense-in-depth and prevent lateral movement.

**Walkthrough & Steps Taken:**
1. Created two isolated bridge networks: `frontend-net` and `backend-net`.
2. Deployed three containers:
   - `db` (Redis) attached strictly to `backend-net`.
   - `web` (Nginx) attached strictly to `frontend-net`.
   - `app` (Nginx) attached to both networks as a reverse-proxy mediator.
3. Tested connectivity:
   - `web -> db`: Failed (`BLOCKED`, host unresolvable).
   - `app -> db`: Succeeded (`REACHABLE` on port 6379).


<img width="654" height="714" alt="Screenshot 2026-09-02 180705" src="https://github.com/user-attachments/assets/5c4e4fe2-1867-4a11-8afb-6c3a54d060de" />


---

### Task 5: Host Firewall Rules (Default-Deny)
**Objective:** Model cloud security group behavior at the host level using an iptables default-deny stance.

**Walkthrough & Steps Taken:**
1. Spun up an Alpine container equipped with `NET_ADMIN` capability and installed `iptables`.
2. Configured the default INPUT policy to `DROP`.
3. Added explicit allow rules only for HTTPS (`tcp dport 443`) and local loopback traffic (`lo`).
4. Listed the ruleset with `iptables -L INPUT -n` to verify the restricted attack surface.


<img width="644" height="300" alt="Screenshot 2026-09-02 180756" src="https://github.com/user-attachments/assets/9553ce74-0606-4e5b-81b0-40055d9c43cc" />


---

### Task 6: Container & Host Hardening
**Objective:** Minimize container privileges and scan images for known Common Vulnerabilities and Exposures (CVEs).

**Walkthrough & Steps Taken:**
1. Launched an unprivileged Nginx container with restrictive flags:
   - Non-root user (`--user 1000:1000`).
   - Read-only root filesystem (`--read-only`) with a writable `/tmp` scratchpad (`--tmpfs /tmp`).
   - Dropped kernel capabilities (`--cap-drop ALL`).
   - Disabled privilege escalation (`--security-opt no-new-privileges`).
2. Verified the running container settings via `docker inspect` (`User=1000:1000 ReadOnly=true`).
3. Scanned the `nginx:alpine` base image using Trivy, identifying 4 HIGH-severity vulnerabilities.

<img width="784" height="398" alt="Screenshot 2026-09-02 181016" src="https://github.com/user-attachments/assets/dad7f59d-e306-4890-8cd8-a674cab86d35" />
<img width="1227" height="571" alt="Screenshot 2026-09-02 181035" src="https://github.com/user-attachments/assets/0c49a6fd-284b-421f-b4e6-8493fffe90f9" />


---

## Deliverables: Short-Answer Questions

### Q1. Explain the difference between authentication and authorization using Tasks 1 and 3.
* **Authentication (AuthN)** verifies **who you are**. In Task 1, the user proved their identity by providing a known username and password; invalid requests were rejected with `401 Unauthorized`, while valid credentials were accepted with `200 OK`.
* **Authorization (AuthZ)** determines **what you are allowed to do** once identity has been established. In Task 3, the service account identity `system:serviceaccount:app:dev` was already authenticated, but Kubernetes RBAC evaluated its permissions against the cluster API. It was authorized to read pods (`yes`), but forbidden from creating deployments or deleting pods (`no`).

### Q2. Why is MFA so effective, and which attacks does it defeat?
MFA combines multiple independent authentication categories—specifically *something you know* (password) and *something you have* (a time-synchronized authenticator device). It is effective because compromising a single credential is no longer sufficient to breach an account. It defeats:
* **Credential stuffing & brute force attacks:** Stolen or guessed passwords cannot generate the dynamic 30-second TOTP token.
* **Phishing & data breach leaks:** Static passwords dumped from third-party breaches cannot be reused without physical access to the seed generator.
* **Keylogging:** Captured keystrokes only expose an expired one-time code.

### Q3. How does network segmentation limit the damage of a compromised web server?
Network segmentation divides infrastructure into isolated zones. By placing the public-facing `web` service on `frontend-net` and the database on `backend-net`, direct network routing between them does not exist. If an attacker achieves remote code execution (RCE) on the web tier, they cannot scan, probe, or query the database directly. This limits blast radius and halts lateral movement, forcing attackers to attempt pivoting through heavily monitored intermediate application servers.

### Q4. What does a default-deny firewall policy achieve, and how does it relate to cloud security groups?
A default-deny policy sets the base posture to `DROP` all inbound packets unless an explicit `ACCEPT` rule matches. This implements network least privilege by ensuring unmanaged or newly opened ports are closed to external scans. This mirrors cloud security groups (e.g., AWS Security Groups), which are stateful firewalls that deny all inbound traffic by default and require administrators to define explicit ingress rules (such as allowing TCP 443).

### Q5. List the hardening measures you applied and the attack surface each one removes.
1. **Non-root user (`--user 1000:1000`):** Prevents container breakout processes from possessing UID 0 (root) capabilities on the host kernel if container isolation fails.
2. **Read-only root filesystem (`--read-only`):** Prevents attackers from downloading persistence scripts, replacing binaries, or dropping malware into system directories (e.g., `/bin`, `/usr`).
3. **Capability drop (`--cap-drop ALL`):** Strips default Linux kernel privileges (such as `CAP_NET_ADMIN` or `CAP_SYS_ADMIN`), preventing raw network packet manipulation, module loading, or host mounting attempts.
4. **No New Privileges (`--security-opt no-new-privileges`):** Prevents binaries with the SUID or SGID bit enabled (like `sudo` or `passwd`) from elevating privileges within the container execution path.
