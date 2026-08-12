# Lab Report: Secure Isolation & Multi-Tenancy

**Course:** IKB42603 Cloud Computing Security Essentials[cite: 2]
**Student Name:** Eikmal
**Lab Focus:** Compute, network, and storage isolation utilizing Docker and Kubernetes[cite: 2]

---

## Setup: Cluster with Policy Enforcement

To ensure that isolation rules strictly take effect, a Kubernetes cluster was provisioned with the default Container Network Interface (CNI) disabled[cite: 2]. The Calico CNI was subsequently installed to enforce network policies[cite: 2].

<img width="537" height="441" alt="Screenshot 2026-08-10 194132" src="https://github.com/user-attachments/assets/8188c025-957a-464c-babe-018ad107f5d3" />
<img width="891" height="548" alt="Screenshot 2026-08-10 194138" src="https://github.com/user-attachments/assets/f8d652b2-72d6-4b99-bd6b-c62e72fe3896" />
<img width="745" height="367" alt="Screenshot 2026-08-10 194147" src="https://github.com/user-attachments/assets/4b430b9a-d476-4b7d-b77e-ee6a04732e33" />


---

## Session A: Compute Isolation & the Default-Open Risk[cite: 2]

### Task 1: Two Tenants on One Cluster
Two distinct customers were modeled by creating two separate namespaces (`tenant-a` and `tenant-b`) sharing the same physical cluster[cite: 2]. A basic Nginx web server deployment was created and exposed on port 80 for each respective tenant[cite: 2].

<img width="495" height="300" alt="Screenshot 2026-08-10 194235" src="https://github.com/user-attachments/assets/4fe72c2c-e07a-4b6f-ad07-13f98c79ea92" />
<img width="620" height="127" alt="Screenshot 2026-08-10 194256" src="https://github.com/user-attachments/assets/7acb4bda-d6d9-4842-bc01-b103de27f23c" />


### Task 2: Observe the Default-Open Risk
A temporary test probe was executed from `tenant-a` to attempt a connection to the service IP of `tenant-b`[cite: 2]. The probe returned an `HTTP 200` status, proving that isolation is not automatic on shared infrastructure and must be actively configured[cite: 2].

<img width="600" height="65" alt="Screenshot 2026-08-10 194718" src="https://github.com/user-attachments/assets/7318efa6-0264-43fb-a2ac-606bfda9d1d7" />
<img width="1291" height="85" alt="Screenshot 2026-08-10 194700" src="https://github.com/user-attachments/assets/948ff333-a9f0-4223-8bf5-4eb3c5e52ce9" />

### Task 3: Contain the Noisy Neighbour (Resource Quotas)
To demonstrate resource isolation, a `ResourceQuota` was applied to `tenant-a`[cite: 2]. This quota restricted CPU requests, memory requests, and the total number of pods to prevent a single tenant from exhausting the shared node's capacity[cite: 2].

<img width="507" height="350" alt="Screenshot 2026-08-10 194810" src="https://github.com/user-attachments/assets/45858b4e-665c-4f0d-a3b9-209a20db5737" />


---

## Session B: Network & Storage Isolation[cite: 2]

### Task 4: Default-Deny Network Isolation
A default-deny ingress `NetworkPolicy` was applied directly to `tenant-b` to implement the segmentation principle[cite: 2]. Rerunning the exact same cross-tenant probe from Task 2 resulted in a connection timeout, successfully proving that cross-tenant traffic was blocked[cite: 2].

<img width="519" height="204" alt="image" src="https://github.com/user-attachments/assets/44050225-3c6f-4e48-a5c6-48a728744b35" />
<img width="930" height="77" alt="image" src="https://github.com/user-attachments/assets/f6a86887-80fc-48e7-b7b4-a09adf1c2d98" />


### Task 5: Storage & Secret Isolation
A unique generic secret was stored in each tenant's namespace[cite: 2]. Role-Based Access Control (RBAC) was utilized to create a service account specifically scoped to `tenant-a`[cite: 2]. Authorization checks (`auth can-i`) successfully proved that `tenant-a` could read its own secrets but was denied access to `tenant-b`'s secrets[cite: 2].

<img width="714" height="298" alt="image" src="https://github.com/user-attachments/assets/e3b770cb-04f5-4777-930d-406c6dbade90" />


### Task 6: Data Remanence & Secure Deletion
A container volume was mounted to demonstrate data remanence[cite: 2]. A file containing sensitive strings was created and deleted normally via `rm`, and a subsequent scan revealed that the underlying bytes may still persist[cite: 2]. A secure wipe was then demonstrated by actively overwriting the file with zeroes (`dd`) prior to deletion[cite: 2].

<img width="1477" height="180" alt="image" src="https://github.com/user-attachments/assets/789dbb2c-334f-4ca4-8edc-6092b2467744" />



---

## Short-Answer Questions

**Q1. Why can containers in different namespaces reach each other by default, and why is that dangerous in multi-tenant cloud?**
* Kubernetes is designed with a default-open network model where pods in one namespace can freely route traffic to pods in another namespace without restrictions[cite: 2].
* This default behavior presents a critical security risk in multi-tenant cloud environments because shared infrastructure isolation is not automatic; without configuration, a compromised or malicious tenant could easily access the workloads and data of a neighboring tenant[cite: 2].

**Q2. Explain the default-deny principle and how your Network Policy implements it.**
* The default-deny principle dictates that all network traffic should be blocked by default, and only specifically required traffic is permitted by exception[cite: 2].
* The implemented `NetworkPolicy` achieved this by targeting `tenant-b` with a `podSelector: {}` and enforcing an `Ingress` policy type without defining any explicit `allow` rules, thereby denying all inbound connections[cite: 2].

**Q3. How do virtual machines and containers differ in isolation strength? When would you add a VM boundary?**
* Virtual machines offer stronger hardware-level isolation because they run entirely separate operating systems managed by a hypervisor. Containers are lighter but inherently share the host operating system's kernel.
* A VM boundary should be introduced in strict multi-tenant environments where the risk of a shared-kernel exploit or privilege escalation is unacceptably high.

**Q4. What is data remanence, and why is cryptographic erasure the preferred cloud solution?**
* Data remanence is the residual data that remains on storage media even after standard deletion commands have been executed[cite: 2].
* Because cloud consumers rarely have administrative control over the physical storage blocks to execute physical secure wipes, cryptographic erasure is the preferred solution; securely destroying the encryption key instantly renders all residual data completely unrecoverable[cite: 2].

**Q5. Which of the three isolation dimensions (compute, network, storage) did each task exercise?**
* **Tasks 1, 2, and 3 (Session A):** Exercised **compute isolation** by utilizing namespaces, containers, and resource quotas[cite: 2].
* **Task 4:** Exercised **network isolation** through the enforcement of a default-deny Network Policy[cite: 2].
* **Tasks 5 and 6:** Exercised **storage isolation** by managing per-tenant secrets and addressing data remanence[cite: 2].

---

## Cleanup & Teardown
To finalize the lab and release resources, the cluster and local Docker volume were completely removed[cite: 2]:
```bash
kind delete cluster --name ccse-lab2
docker volume rm ccse-vol
