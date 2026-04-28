## Multibranch Pipeline(MBPL) Jenkins

* A **Multibranch Pipeline** in Jenkins automatically detects branches in a repository and creates separate pipeline jobs for each branch.

* Each branch executes its own **Jenkinsfile**, allowing you to define different CI/CD workflows for different branches.

---

# Setup Steps

###  1. Install Plugins

- Pipeline Plugin  
- Multibranch Pipeline Plugin  
- Git or GitHub Plugin  

(Install via **Manage Jenkins → Plugins**)

---

###  2. Create a Multibranch Pipeline

- Go to **Jenkins Dashboard → New Item**  
- Name your job → choose **Multibranch Pipeline** → Click OK  

---

###  3. Configure Repository Source

- Under **Branch Sources → Click Add Source**  
- Choose Git  

**Fill in:**
- Repo URL (e.g. https://github.com/user/repo.git)  
- Credentials (if private)  

---

###  4. Add Jenkinsfile to Each Branch

- Place a Jenkinsfile at the root of each branch.  
- It can be shared or unique per branch.  
- Use scriptPath if file isn’t at root (e.g., devops/Jenkinsfile)  

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
````

---

###  5. Trigger a Scan

Click **Scan Repository Now** to create pipeline jobs for all valid branches.

---

#  Step-by-Step Guide to Jenkins Backup and Restore

## 1.  What is Jenkins Backup?

* A Jenkins backup means saving Jenkins' configuration, job data, plugins, credentials, and other important files.

* This helps prevent data loss and ensures you can recover if there is a system failure, misconfiguration, or crash.

---

## 2. 📦 What Should Be Backed Up?

Jenkins stores all its important data in the JENKINS_HOME directory. You should back up the following:

*  Jenkins Home Directory ($JENKINS_HOME or /var/lib/jenkins/)
*  Job Configurations ($JENKINS_HOME/jobs/)
*  User Profiles ($JENKINS_HOME/users/)
*  Plugins ($JENKINS_HOME/plugins/)
*  Secrets & Credentials ($JENKINS_HOME/secrets/)
*  Build History ($JENKINS_HOME/jobs/*/builds/)
*  Global Configuration ($JENKINS_HOME/config.xml)
*  Node Configuration ($JENKINS_HOME/nodes/)

---

#  Backup Methods in Jenkins

## 🔹 Method 1: Manual Backup (Copy Jenkins Home Directory)

### Steps:

1. Stop Jenkins to avoid inconsistencies.
   `sudo systemctl stop jenkins`

2. Copy Jenkins Home Directory to a backup location.
   `sudo cp -r /var/lib/jenkins /backup/jenkins_backup_$(date +%F)`

3. Start Jenkins again after backup.

4. Optional: Compress the backup for storage efficiency.
   `tar -czvf /backup/jenkins_backup_$(date +%F_%H_%M).tar.gz /backup/jenkins_backup_$(date +%F_%H_%M)`

---

## 🔹 Method 2: Using Jenkins ThinBackup Plugin

### What is ThinBackup?

ThinBackup is a plugin used in Jenkins to take **backups of your entire Jenkins setup**.

It will save your **jobs, users, plugins, build history, credentials, and more** so that if something goes wrong, you can **restore everything easily**.

---

### Why Do We Use ThinBackup?

We use ThinBackup mainly to **avoid data loss**.

Imagine your Jenkins server crashes, or someone deletes a job by mistake — you don’t want to reconfigure everything from scratch, right?

ThinBackup helps you to **take safe backups automatically**, and even restore them easily.

It **saves time**, avoids tension, and is very useful especially when working in teams or handling big projects.

---

### Steps:

1. Install the ThinBackup Plugin

   * Go to Manage Jenkins → Manage Plugins → Available
   * Search for ThinBackup → install it.
   * Restart Jenkins.

2. Configure ThinBackup

   * Go to Manage Jenkins → ThinBackup
   * Set Backup Directory (e.g., /backup/jenkins-thinbackup/)
   * Set Schedule for automatic backups
   * Click Save

3. Run a Backup

   * Click Backup Now to manually trigger a backup

---

##  Method 3: Using Jenkins Job to Automate Backup

### Steps:

1. Create a new job (Freestyle Project)
2. Add a build step → Execute Shell
3. Paste this script

```bash
#!/bin/bash
BACKUP_DIR="/backup/jenkins_auto_backup_$(date +%F_%H-%M)"
sudo mkdir -p $BACKUP_DIR
sudo cp -r /var/lib/jenkins/* $BACKUP_DIR
sudo tar -czvf $BACKUP_DIR.tar.gz $BACKUP_DIR
sudo rm -rf $BACKUP_DIR
```

4. Schedule it under Build Triggers using Cron syntax
   `H 2 * * *`

5. sudo visudo

   `jenkins ALL=(ALL) NOPASSWD: /bin/mkdir, /bin/cp, /bin/tar, /bin/rm`

---

#  Restoring a Backup

##  Manual Restore

1. Stop Jenkins
   `sudo systemctl stop jenkins`

2. Restore Jenkins Home Directory
   `sudo cp -r /backup/jenkins_backup_DATE /var/lib/jenkins`

3. Set Correct Permissions
   `sudo chown -R jenkins:jenkins /var/lib/jenkins`

4. Start Jenkins
   `sudo systemctl start jenkins`

---

##  ThinBackup Restore

1. Go to Manage Jenkins → ThinBackup
2. Select Restore Backup
3. Choose the backup file and restore
4. Restart the jenkins

---

#  Best Practices for Jenkins Backup

* Automate backups using jobs or cron jobs
* Store backups remotely (AWS S3, Google Drive, etc.)
* Test restores regularly to ensure backups are working
* Exclude logs and unnecessary files to reduce backup size

```
