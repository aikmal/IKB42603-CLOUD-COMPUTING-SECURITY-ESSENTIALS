# Lab 1 Report: Cloud Account Security, Identity & Access Management

This report documents the implementation of local cloud identity infrastructure and platform-level access controls, matching each task with its corresponding verification proof.

## One-Time Environment Setup
- Verified Docker installation using `docker --version`.
- Started the LocalStack container and verified its health status via `curl http://localhost:4566/_localstack/health`.
- Configured dummy credentials to securely direct the AWS CLI tool at the LocalStack container endpoint.
- Executed `sts get-caller-identity` to verify initial operations under the simulated root profile.

<img width="1897" height="440" alt="lab 1 session A" src="https://github.com/user-attachments/assets/87045933-a53e-4c01-bfe6-3c0b9864fe43" />
<img width="568" height="278" alt="Screenshot 2026-07-29 200346" src="https://github.com/user-attachments/assets/bf958413-1846-464f-b68b-00ded3d19272" />


## Task 1: Mapping the Cloud Identity Landscape
- Completed the mapping table defining the foundational building blocks of cloud identity.

| Concept | AWS Term | Purpose |
| :--- | :--- | :--- |
| All-powerful owner | **Root user** | The initial, unrestricted identity created with full administrative access over all resources. It represents a significant security liability and should not be used for daily administrative operations. |
| Human/app identity | **IAM User** | A specific identity created within the cloud account representing a single person or service, with unique credentials and tailored permissions. |
| Permission bundle | **IAM Policy** | A formal JSON document defining explicit allowed or denied actions against specific cloud resources. |
| Collection of users | **IAM Group** | A logical management boundary used to aggregate multiple IAM users so permissions can be granted collectively, ensuring standard management at scale. |
| Temporary identity | **IAM Role** | An identity meant to be assumed temporarily by trusted entities or applications, providing short-lived security credentials instead of hardcoded long-lived keys. |

## Task 2: Create a Least-Privilege Admin
- Configured dummy credentials to securely direct the AWS CLI tool at the LocalStack container endpoint.
- Executed `sts get-caller-identity` to verify initial operations under the root profile.
- Created an administrative group `Admins`, attached the `AdministratorAccess` policy, and provisioned `CloudAdmin_eikmal` into it.
- Verified active group membership using the `get-group` query.


<img width="800" height="400" alt="Screenshot 2026-07-29 get-group Admins" src="PASTE_YOUR_GITHUB_IMAGE_URL_HERE" />
<img width="597" height="380" alt="lab1 task 2 (2)" src="https://github.com/user-attachments/assets/9d368785-efec-4391-8e58-1ace915e05be" />


## Task 3: Enforce Least Privilege with a Scoped Policy
- Created a restricted identity named `Analyst_aikmal` to enforce functional access separation.
- Attached a scoped policy granting exclusive `AmazonS3ReadOnlyAccess` privileges.
- Verified restrictions via `list-attached-user-policies` to prove zero structural write or delete access exists.

<img width="597" height="431" alt="Screenshot 2026-07-29 203338" src="https://github.com/user-attachments/assets/9cae5b42-7efd-41e2-a655-7b302559b6d6" />


### Short-Answer Evaluation: Blast-Radius Reduction
> **Q: If the Analyst account were stolen, why is the damage limited compared to a stolen admin account? Connect your answer to blast-radius reduction.**
>
> **A:** The blast radius represents the maximum potential damage an attacker can inflict upon compromising a specific asset or identity. If the Analyst account is compromised, the attacker's operational capabilities are strictly confined by the attached `AmazonS3ReadOnlyAccess` policy. They can only read data out of S3 buckets; they are fundamentally blocked from deleting objects, modifying network configurations, manipulating other IAM accounts, or spinning up unauthorized infrastructure. In contrast, compromising an Admin account exposes the entire organization to catastrophic structural damage, resource encryption, or a complete tenant takeover.

## Task 4: Credential Hygiene & Access Keys
- Generated programmatic authentication keys for `Analyst_aikmal` to model secure cloud integration keys.
- Audited the active state using the `list-access-keys` command.
- Demonstrated key rotation and compromise mitigation by successfully toggling the key status to `Inactive`.

<img width="578" height="508" alt="image" src="https://github.com/user-attachments/assets/f251546c-4f36-4b9c-b99d-acb018ea6e7c" />

## One-Time Environment Setup
- Create a Local Kubernetes Cluster


<img width="847" height="448" alt="lab1 session b" src="https://github.com/user-attachments/assets/9a07cfe0-9daf-4a7a-ab3f-9cb36a537168" />


## Tasks 5 & 6: Kubernetes Namespace Isolation & RBAC Definition
- Built a local Kubernetes engine utilizing `kind` to handle platform-level enforcement testing.
- Created `dev` and `prod` logical namespaces to establish clear tenant barriers.
- Defined a namespaced `Role` restricting resource capability strictly to pod operations (`get`, `list`, `watch`).
- Linked the permissions explicitly to a targeted `dev-user` service account via a local `RoleBinding`.


<img width="293" height="309" alt="Screenshot 2026-07-30 114509" src="https://github.com/user-attachments/assets/e08e2161-3ad7-47c2-a47b-481e6f2266e5" />

<img width="836" height="199" alt="Screenshot 2026-07-30 114548" src="https://github.com/user-attachments/assets/6bf20e1f-d6a9-461a-8492-e9ad52d25612" />


## Task 7: Access Control Validation
- Impersonated the service account identity profile using `kubectl auth can-i` flags.
- Validated authorization limits across alternating commands and namespaces, generating a perfect verification matrix: `YES` (list pods in dev), `NO` (delete pods in dev), and `NO` (list pods in prod).



<img width="432" height="240" alt="image" src="https://github.com/user-attachments/assets/a7a319ce-8f4f-4a7a-ad87-1f1e8f5b4631" />



## Lab Report Short-Answer Questions

### Q1. Why is attaching policies to groups better than attaching them directly to users?
Attaching policies directly to individual users leads to "permission creep" and becomes impossible to audit at scale. Managing authorization at the group level ensures administrative consistency: modifying a single group policy instantly adjusts access rights for every user assigned to that group, ensuring compliance and preventing loose, forgotten permissions.

### Q2. What is the difference between an IAM User and an IAM Role?
An **IAM User** represents a persistent, long-lived identity associated with a permanent set of credentials (passwords or access keys) mapping to a specific human or static software integration. An **IAM Role** is a dynamic, temporary identity that does not possess permanent credentials; instead, authorized entities (like EC2 instances or external users) temporarily *assume* the role to receive short-lived tokens, heavily minimizing exposure windows.

### Q3. Explain least privilege using the Analyst account, and how it reduces blast radius if compromised.
The principle of least privilege dictates that an identity should only hold the exact minimum permissions required to execute its immediate operational duties. By restricting the Analyst account strictly to `AmazonS3ReadOnlyAccess`, the identity is blocked from executing harmful operations. If the keys are leaked, the blast radius is restricted entirely to data reading within S3, protecting the remaining enterprise control plane from compromise.

### Q4. In Kubernetes, what is the difference between a Role and a RoleBinding?
A **Role** is a namespaced configuration template that explicitly defines *what permissions are allowed* (verbs like `get`, `list` matched against resources like `pods`). A **RoleBinding** acts as the glue or enforcement link—it maps that defined Role to a specific entity or subject (such as a User, Group, or ServiceAccount), granting them those permissions within that exact namespace.

### Q5. Why did the developer service account fail to access prod, and which security principle does that demonstrate?
The developer service account failed to view the `prod` namespace because the cluster operates under a **Default Deny** architecture; permissions must be explicitly declared to exist. Because the underlying `RoleBinding` was scoped exclusively within the boundaries of the `dev` namespace, the authorization context did not extend out to `prod`. This directly demonstrates the core security principles of **Least Privilege** and **Compartmentalization**.

## Final Verification Deliverable
- Exported the complete YAML structure configuration mapping of the active role binding to verify the deployment state matches specifications.

<img width="498" height="301" alt="image" src="https://github.com/user-attachments/assets/271fbc8c-4574-462e-a9b6-e50e93df4d3b" />

# Conclusion

Every required task within Session A and Session B has been executed successfully. Access controls have been mapped across identity boundaries in LocalStack, and strict role enforcement limits have been verified inside the Kubernetes engine. Environments were safely torn down following compilation.
  
