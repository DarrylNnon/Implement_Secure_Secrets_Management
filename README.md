# Implement Secure Secrets Management
Secure secrets management is critical for safeguarding sensitive information such as API Keys, credentials and configuration secrest. This project involves setting up a secrets management system, integrating it with applications, configuring robust access controls, and ensuring secrets are rotated and monitored.

### Steps to Complete the Project:
1. **Set up HashiCorp Vault or AWS Secrets Manager**.
2. **Integrate secret management with applications**.
3. **Configure access controls and audit logging**.
4. **Rotate and monitor secrets usage**.

---

### Prerequisites
- A Linux system (or a cloud instance) with **sudo** privileges.
- Basic understanding of secrets management and APIs.
- Installed tools: HashiCorp Vault (for Vault option) or AWS CLI (for AWS Secrets Manager).

---

## Step 1: Set up HashiCorp Vault or AWS Secrets Manager

We’ll use **HashiCorp Vault** for this guide. If you prefer AWS Secrets Manager, I can provide steps for that as well.

---

### **Option A: Using HashiCorp Vault**

#### Step 1.1: Install Vault

1. Download and install Vault on your system:

   ```bash
   sudo apt update
   wget https://releases.hashicorp.com/vault/1.14.0/vault_1.14.0_linux_amd64.zip
   sudo unzip vault_1.14.0_linux_amd64.zip -d /usr/local/bin/
   ```

2. Verify installation:

   ```bash
   vault --version
   ```

#### Step 1.2: Start Vault Server

1. Start a development instance (for learning purposes):

   ```bash
   vault server -dev
   ```

   This will print a **root token** and start Vault on `http://127.0.0.1:8200`. Save the token.

2. Set up the Vault CLI environment:

   ```bash
   export VAULT_ADDR='http://127.0.0.1:8200'
   export VAULT_TOKEN=<your-root-token>
   ```

#### Step 1.3: Initialize and Unseal Vault

For production, initialize and unseal Vault manually or via an automated process.

```bash
vault operator init
vault operator unseal
```

### **Option B: Using AWS Secrets Manager**

1. Install the AWS CLI:

   ```bash
   sudo apt install awscli -y
   aws configure
   ```

2. Create a secret in AWS Secrets Manager:

   ```bash
   aws secretsmanager create-secret --name my-secret --secret-string '{"username":"admin","password":"password123"}'
   ```

3. Verify the secret:

   ```bash
   aws secretsmanager get-secret-value --secret-id my-secret
   ```

---

## Step 2: Integrate Secret Management with Applications

Secrets must be accessed securely by applications without hardcoding sensitive data.

---

### **Step 2.1: Integrate HashiCorp Vault**

1. Store a secret in Vault:

   ```bash
   vault kv put secret/db username=admin password=password123
   ```

2. Retrieve the secret using the CLI:

   ```bash
   vault kv get secret/db
   ```

3. Integrate Vault with your application (Python example):

   Install the Vault client library:

   ```bash
   pip install hvac
   ```

   Sample script to retrieve secrets:

   ```python
   import hvac

   client = hvac.Client(url='http://127.0.0.1:8200', token='<VAULT_TOKEN>')
   secret = client.secrets.kv.v2.read_secret_version(path='db')
   print(secret['data']['data'])
   ```

---

### **Step 2.2: Integrate AWS Secrets Manager**

1. Retrieve a secret using AWS CLI:

   ```bash
   aws secretsmanager get-secret-value --secret-id my-secret
   ```

2. Integrate with a Python application:

   Install the Boto3 library:

   ```bash
   pip install boto3
   ```

   Sample script to retrieve secrets:

   ```python
   import boto3
   import json

   client = boto3.client('secretsmanager')
   response = client.get_secret_value(SecretId='my-secret')
   secret = json.loads(response['SecretString'])
   print(secret)
   ```

---

## Step 3: Configure Access Controls and Audit Logging

Restrict who and what can access secrets and monitor all interactions.

---

### **Step 3.1: Access Control in Vault**

1. Enable authentication methods (e.g., Token, AppRole):

   Enable AppRole for application authentication:

   ```bash
   vault auth enable approle
   ```

2. Create a policy to restrict access:

   Create a file `db-policy.hcl`:

   ```hcl
   path "secret/data/db" {
       capabilities = ["read"]
   }
   ```

   Apply the policy:

   ```bash
   vault policy write db-policy db-policy.hcl
   ```

3. Assign the policy to an AppRole:

   ```bash
   vault write auth/approle/role/app-role policies="db-policy"
   ```

4. Retrieve the Role ID and Secret ID:

   ```bash
   vault read auth/approle/role/app-role/role-id
   vault write -f auth/approle/role/app-role/secret-id
   ```

### **Step 3.2: Audit Logging in Vault**

1. Enable audit logging:

   ```bash
   vault audit enable file file_path=/var/log/vault_audit.log
   ```

2. View logs:

   ```bash
   sudo tail -f /var/log/vault_audit.log
   ```

---

### **Step 3.3: Access Control in AWS Secrets Manager**

1. Use IAM policies to restrict access to secrets:

   Example policy for read-only access:

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": "secretsmanager:GetSecretValue",
               "Resource": "arn:aws:secretsmanager:region:account-id:secret:my-secret"
           }
       ]
   }
   ```

2. Enable AWS CloudTrail for audit logging:

   ```bash
   aws cloudtrail create-trail --name secrets-manager-trail --s3-bucket-name my-cloudtrail-bucket
   aws cloudtrail start-logging --name secrets-manager-trail
   ```

---

## Step 4: Rotate and Monitor Secrets Usage

Automated secrets rotation reduces the risk of leaked credentials.

### **Step 4.1: Rotate Secrets in Vault**

Enable dynamic secrets for a database:

```bash
vault secrets enable database
vault write database/config/mydb plugin_name=mysql-database-plugin \
    connection_url="{{username}}:{{password}}@tcp(127.0.0.1:3306)/" \
    allowed_roles="readonly"

vault write database/roles/readonly db_name=mydb \
    creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; \
    GRANT SELECT ON *.* TO '{{name}}'@'%';" \
    default_ttl="1h" max_ttl="24h"
```

### **Step 4.2: Rotate Secrets in AWS Secrets Manager**

Enable automatic rotation for a secret:

```bash
aws secretsmanager rotate-secret --secret-id my-secret --rotation-lambda-arn arn:aws:lambda:region:account-id:function:rotation-function
```

Monitor secret usage with CloudTrail logs and IAM access analyze

## Final Step: Document Secrets Management Setup

Create detailed documentation for your team:

### **Documentation Outline**

1. **Overview**  
   - Purpose and importance of secure secrets management.
2. **Setup Steps**  
   - Detailed installation and integration steps for Vault/AWS Secrets Manager.
3. **Access Control**  
   - Example policies and access management procedures.
4. **Rotation and Monitoring**  
   - How to rotate secrets and monitor access logs.
5. **Sample Code**  
   - Application integration scripts.


By following these steps, you’ll have implemented a robust and secure secrets management system that can be scaled and maintained effectively.
