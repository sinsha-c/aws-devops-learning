# Task 0: Git Installation and Setup (Windows Git Bash)

## Objective

Install Git and configure Git Bash before starting repository activities.

---

## Step 1: Install Git

Download Git for Windows:

https://git-scm.com/download/win

During installation:

- Select Git Bash
- Keep default settings (recommended)
- Complete installation

---

## Step 2: Verify Git Installation

Open Git Bash and run:

```bash
git --version
```

✔ Expected Output:
```text
git version 2.xx.x
```

## Step 3: Configure Git (One-time setup)

### Set Username
```bash
git config --global user.name "Your Name"
```

### Set Email
```bash
git config --global user.email "your-email@example.com"
```

---

## Step 4: Verify Configuration
```bash
git config --global --list
```

✔ You should see:
```text
user.name=Your Name
user.email=your-email@example.com
```

---

## Step 5: Create Project Directory
```bash
mkdir my-project
cd my-project
```

---

## Step 6: Initialize Git Repository
```bash
git init
```

✔ Output:
```text
Initialized empty Git repository in ...
```

---

## Step 7: Check Git Status
```bash
git status
```

✔ Output:
```text
On branch main (or master)
No commits yet
```

---
