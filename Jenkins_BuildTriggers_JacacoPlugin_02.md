#  DevOps Notes – Crontab, Jenkins, GitHub Webhook & More

---

##  Crontab: Scheduling Tasks

###  Syntax

```bash
* * * * * /path/to/command
- - - - -
| | | | |
| | | | +---- Day of the Week (0 - 6) (Sunday = 0 or 7)
| | | +------ Month (1 - 12)
| | +-------- Day of the Month (1 - 31)
| +---------- Hour (0 - 23)
+------------ Minute (0 - 59)
````

###  Commands

| Command                | Description                                   |
| ---------------------- | --------------------------------------------- |
| `crontab -l`           | List all scheduled cron jobs for current user |
| `crontab -e`           | Edit crontab                                  |
| `crontab -r`           | Remove crontab                                |
| `crontab -l -u <user>` | View another user's crontab (root only)       |

---

###  Control Cron Access

```bash
 sudo touch /etc/cron.allow       # Create allow file
 sudo vi /etc/cron.allow          # Add allowed usernames (one per line)
```

---

###  Sample Shell Script – hello.sh

```bash
#!/bin/bash
echo "Welcome to KK FUNDA"
echo "Today date is:"
date
uptime
pwd
```

###  Run the Script

```bash
sh hello.sh
./hello.sh
chmod u+x hello.sh
```

---

###  Schedule Script in Cron

```bash
*/1 * * * * /home/ec2-user/hello.sh >> /home/ec2-user/hello.log
```

 **Use `>>` to append output and preserve logs.**

---

#  Jenkins Build Triggers

Build triggers allow Jenkins to automatically start a build when something changes — like a code update — without manually clicking **"Build Now"**.

---

##  SCM Polling (Source Code Management)

###  What It Does

Jenkins checks the Git repository at regular intervals (you define the time using a cron schedule).

If it finds any new commits or code changes, it triggers a build.

---

###  Benefits

* You don’t need to manually run the build
* Saves time and automates continuous integration

---

###  Example Cron Schedule

```bash
* * * * *
```

> This will check the repository every 1 minute for any new commits.
> If new commits are found, Jenkins triggers a build.

---

###  When should you use Poll SCM?

* ✅ When GitHub webhooks are not working or unavailable
* ✅ If your project is hosted on an internal Git server

---

##  Periodic Builds

Build Periodically means Jenkins does not wait for code changes. Instead, it triggers builds based on a fixed time schedule.

---

###  Key Points

* ✅ Jenkins triggers a build on schedule, even if nothing has changed
* ✅ Good for nightly builds, tests, or scheduled deployments

---

###  When to Use

* ✅ Nightly Builds

Example: Run the job every night at **1 AM**

---

## 📗 GitHub Webhook

Whenever a developer makes a change (push/update) in the GitHub repository, GitHub immediately notifies Jenkins to start the build.

---

### 🔔 Configuration

1. In Jenkins Job → Enable **GitHub hook trigger for GITScm polling**

2. In GitHub:

   * Go to **Repository → Settings → Webhooks → Add Webhook**
   * Payload URL: `http://65.1.95.162:8080/github-webhook/`
   * Content type: `application/json`
   * Trigger: **Just the push event**
   * Click **Add Webhook**

---

#  5. JaCoCo – Code Coverage Tool

JaCoCo is a code coverage tool that checks:

* ✅ How much of your code is covered by tests
  *(unit tests, integration tests, etc.)*

---

### ✅ Why It’s Important

* Ensures important parts of the code are properly tested
* Helps maintain high code quality

---

###  When to Use JaCoCo

*  When your project requires **high code quality**
*  If you want to block deployments for **low test coverage**
*  When your team wants Jenkins to fail the build automatically if test coverage is poor.
---

###  Install JaCoCo Plugin in Jenkins

* Go to **Manage Jenkins → Plugin Manager**
* Search: **JaCoCo → Install**

#### In Job:

* Go to **Post-build Actions**
* Click **Add Post-build Action → Record JaCoCo coverage report**

---

#  Discard Old Builds in Jenkins

Jenkins can automatically delete older builds after a certain number of builds or days.

👉 This helps **save disk space** by removing old build data.

---

###  Steps

* Go to **Job → Configure**
* Enable ✅ **Discard Old Builds**

---

###  Set Values

* **Max number of builds to keep**
* **Max days to keep builds**

---

### 🎯 Example

* **Max # of builds to keep:** `5` → keeps latest 5 builds
* **Max # of days to keep builds:** `7` → keeps builds for 7 days

```

---
