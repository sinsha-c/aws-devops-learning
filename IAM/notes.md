# 🔐 AWS IAM (Identity & Access Management)

## 📌 What is IAM?
IAM is an AWS service used to manage access to AWS resources securely.

It helps you control:
- Who can access AWS (users)
- What they can access (permissions)
- How they access it (roles, policies)

---

## 🧩 Key Components

### 1. Users
- Individual accounts in AWS
- Example: Developer, Admin

### 2. Groups
- Collection of users
- Easier permission management

### 3. Roles
- Temporary access to AWS services
- Used by EC2, Lambda, etc.

### 4. Policies
- JSON rules that define permissions
- Example: Allow S3 read access

---

## 🔐 Important IAM Best Practices
- Never use root account for daily work
- Enable MFA (Multi-Factor Authentication)
- Follow least privilege principle
- Use roles instead of hardcoded credentials

---

## 🚀 What I Learned
- How AWS controls security access
- Difference between user, role, and policy
- How permissions are assigned

---

## 📸 Hands-on Practice
- Created IAM user
- Attached policy (S3 ReadOnlyAccess)
- Logged in using IAM user

---

## 🧠 Real-world use
IAM is used in companies to ensure:
- Secure cloud access
- Controlled permissions
- Audit and compliance safety
