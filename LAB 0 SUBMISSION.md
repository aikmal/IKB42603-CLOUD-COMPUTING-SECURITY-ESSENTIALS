# Lab 0 Report

This report summarizes the environment setup guide and places each available screenshot beside the corresponding step.

## Step 1: Install Docker
- Verified Docker installation using `docker --version`.
- Confirmed Docker runtime with `docker run --rm hello-world`.

<img width="624" height="69" alt="Screenshot 2026-07-28 223422" src="https://github.com/user-attachments/assets/7c4a16df-584d-4250-b23f-9c0dff0191ea" />


## Step 2: Install AWS CLI
- Verified AWS CLI installation with `aws --version`.
- Confirmed AWS CLI readiness for AWS and LocalStack use.

<img width="609" height="66" alt="Screenshot 2026-07-28 223650" src="https://github.com/user-attachments/assets/19d6d7a3-447c-45f5-8235-d71e89c3a1d6" />


## Step 3: Install kind and kubectl
- Verified installation of `kind` and `kubectl`.
- Confirmed cluster tooling readiness for Kubernetes local clusters.

<img width="200" height="66" alt="Screenshot 2026-07-29 183214" src="https://github.com/user-attachments/assets/645f82fb-ff93-4600-8d8b-31c458243baf" />
<img width="262" height="75" alt="Screenshot 2026-07-28 223839" src="https://github.com/user-attachments/assets/ea3beb5c-94e3-4842-b527-95be7f06cf15" />




## Step 4: Install Helper Tools
- Verified installation of helper tools including OpenSSL, `oathtool`, and Trivy.
- These tools support certificate handling, one-time password generation, and container/image security scanning.

<img width="262" height="75" alt="Screenshot 2026-07-28 223839" src="https://github.com/user-attachments/assets/606552e8-3600-4bb3-9b9d-6971ebf8ee5b" />
<img width="642" height="161" alt="Screenshot 2026-07-28 223927" src="https://github.com/user-attachments/assets/fd554c23-7f11-4627-9fdb-d4a966416e72" />



## Step 5: LocalStack Health Check
- Checked LocalStack service status via `http://localhost:4566/_localstack/health`.
- Verified that LocalStack services such as `iam`, `s3`, `lambda`, `kinesis`, and more are available.

<img width="1898" height="636" alt="Screenshot 2026-07-28 225126" src="https://github.com/user-attachments/assets/5fe3bb89-f8fd-4ee5-a3d1-774797c1d547" />


## Step 6: One-Time AWS CLI Configuration
- Configured the AWS CLI with `aws_access_key_id`, `aws_secret_access_key`, and default region.
- Verified the LocalStack endpoint by using `aws sts get-caller-identity --endpoint-url=http://localhost:4566`.

<img width="516" height="318" alt="Screenshot 2026-07-28 232242" src="https://github.com/user-attachments/assets/4c46ca77-7583-4bf1-aa5b-eb978cae4874" />

## Conclusion

The required command-line tools for the lab environment are installed and their versions have been recorded. Docker, AWS CLI, kind, kubectl and OpenSSL are available for subsequent cloud-computing security labs.
