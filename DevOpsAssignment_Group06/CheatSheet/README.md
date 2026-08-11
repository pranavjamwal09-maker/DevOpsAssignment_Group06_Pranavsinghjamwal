# DevOps & Windows Terminal Cheat Sheet

A comprehensive quick-reference guide for Windows command-line environments (Command Prompt / PowerShell) and essential Git commands used in DevOps labs.

---

##  1. Windows Command Prompt (CMD) Essentials

| Action | CMD Command | Example / Usage |
| :--- | :--- | :--- |
| **List files & folders** | `dir` | `dir` |
| **Change directory** | `cd` | `cd C:\Users\Projects` |
| **Go up one folder** | `cd ..` | `cd ..` |
| **Create new folder** | `mkdir` / `md` | `mkdir DevOpsAssignment` |
| **Create empty file** | `type nul > filename` | `type nul > notes.txt` |
| **View file contents** | `type` | `type notes.txt` |
| **Delete file** | `del` | `del notes.txt` |
| **Delete folder** | `rmdir /s` | `rmdir /s OldFolder` |
| **Clear screen** | `cls` | `cls` |

---

##  2. Windows PowerShell Commands

| Action | PowerShell Command | Example / Usage |
| :--- | :--- | :--- |
| **List files** | `ls` or `Get-ChildItem` | `ls` |
| **Create new file** | `New-Item` | `New-Item index.html -ItemType File` |
| **Create new folder** | `New-Item` | `New-Item Assignment -ItemType Directory` |
| **Check IP Configuration** | `Get-NetIPAddress` | `Get-NetIPAddress` |
| **Check System Info** | `Get-ComputerInfo` | `Get-ComputerInfo` |

---

##  3. Essential Git Commands (Windows)

### **Repository Initial Setup**
```cmd
git init
git clone [https://github.com/username/repository.git](https://github.com/username/repository.git)

Checking Status & Logs

DOS
git status
git log --oneline


Staging & Committing Changes
DOS
git add .
git commit -m "Add DevOps lab assignment files"


Branching & Merging
DOS
git branch                       :: List all local branches
git branch feature-branch         :: Create new branch
git checkout feature-branch       :: Switch to branch
git merge feature-branch          :: Merge into active branch


Remote Synchronization
DOS
git remote -v                    :: View connected remote repository
git push origin main             :: Push changes to GitHub
git pull origin main             :: Pull latest changes from GitHub
