# Chapter 23: Cloud & IaC - Infinite Scale
> "Friends don't let friends click in the console."

## The Goal: Infrastructure as Code (IaC)
You are done with manual server setup.
You want to code your infrastructure.
1.  **Cloud:** AWS/GCP/Azure via SDKs.
2.  **Config:** Ansible/Terraform via Wrappers.

Python is the native language of the Cloud.

---

## 23.1 The AWS SDK (`boto3`)

`boto3` is the standard Python library for Amazon Web Services.
It allows you to create S3 buckets, EC2 instances, and IAM users.

### Installation
```bash
pip install boto3
```
(Requires `~/.aws/credentials` configuration).

### Example: S3 Bucket Lister
```python
import boto3

# Create S3 Client
s3 = boto3.client('s3')

# List Buckets
response = s3.list_buckets()

print("My Buckets:")
for bucket in response['Buckets']:
    print(f"- {bucket['Name']}")
```

---

## 23.2 Uploading Files to the Cloud

The DevOps classic: **The Backup Script**.

```python
import boto3
from botocore.exceptions import NoCredentialsError

def upload_to_aws(local_file, bucket, s3_file):
    s3 = boto3.client('s3')

    try:
        s3.upload_file(local_file, bucket, s3_file)
        print("Upload Successful")
        return True
    except FileNotFoundError:
        print("The file was not found")
    except NoCredentialsError:
        print("Credentials not available")
    return False

# Usage
upload_to_aws('backup.zip', 'my-infra-backups', '2023-10-27-backup.zip')
```

---

## 23.3 Python & Ansible

Ansible is written in Python.
You can write your own **Ansible Modules** in Python if the built-in ones aren't enough.

### An Ansible Module Structure (Simplified)
An Ansible module is just a script that prints JSON to stdout.

```python
#!/usr/bin/python
import json
import sys

def main():
    # 1. Read Arguments (from Ansible)
    # 2. Do Work (e.g. Check if a user exists)
    
    result = {
        "changed": True,
        "msg": "User created successfully"
    }

    # 3. Print JSON
    print(json.dumps(result))

if __name__ == '__main__':
    main()
```

### Running Ansible from Python
Sometimes you want Python to trigger a Playbook.

```python
import subprocess

def run_playbook(playbook_path, inventory):
    cmd = ["ansible-playbook", "-i", inventory, playbook_path]
    subprocess.run(cmd, check=True)
```

---

## 23.4 DevOps Use Case: The EC2 Manager

A script to find and stop expensive "Development" servers at night.

```python
import boto3

ec2 = boto3.resource('ec2')

def stop_dev_instances():
    # Filter: Tag "Env=Dev" and State="running"
    filters = [
        {'Name': 'tag:Env', 'Values': ['Dev']},
        {'Name': 'instance-state-name', 'Values': ['running']}
    ]
    
    instances = ec2.instances.filter(Filters=filters)
    
    for instance in instances:
        print(f"Stopping {instance.id}...")
        instance.stop()

if __name__ == "__main__":
    stop_dev_instances()
```

---

## 23.5 The Laws of Cloud Scripts

1.  **The Law of Regions:** Always specify your region (`us-east-1`). Defaults kill you.
2.  **The Law of Tags:** Only touch resources with specific tags. Don't stop Prod by accident!
3.  **The Law of Cost:** Remember that API calls (and resources) cost money. optimize your loops.
4.  **The Law of Secrets:** Never commit your `AWS_SECRET_ACCESS_KEY`. Use Roles or Environment Variables.

---

## [!] Common Mistakes

### Error #1: Hardcoding Keys
```python
boto3.client('s3', aws_access_key_id='AKIA...') # FIREABLE OFFENSE!
```
**Fix:** Let `boto3` find them in `~/.aws/credentials` or Environment Variables automatically.

### Error #2: Pagination
AWS returns only 1000 items (e.g., S3 objects or EC2 instances).
If you have 5000, your script only sees the first 1000.
**Fix:** Use `boto3` **Paginators**.

---

## Chapter Summary

-   **Cloud SDKs:** Python controls the cloud (`boto3`).
-   **Automation:** Script backups, cleanups, and scaling events.
-   **Ansible:** Python powers configuration management logic.
-   **Risk:** With great power (`ec2.terminate_instances`) comes great responsibility.
