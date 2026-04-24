# Handling Jenkins Slowness: The Public IP Problem

## Problem

Jenkins is a web application that depends on its configured IP address or DNS name.

When Jenkins is hosted on servers like AWS EC2, Azure VM, etc., a server restart may change the public IP.

If the IP changes, Jenkins may behave unexpectedly:

- The web dashboard loads slowly or may break
- Jobs that use webhooks (like GitHub) may fail
- Email notifications may not work

---

## Solution

### Step 1: Connect to Jenkins Server

```bash
ssh <your-server-ip>
````

### Step 2: Navigate to Jenkins Directory

```bash
cd /var/lib/jenkins/
```

### Step 3: Open Jenkins Configuration File

```bash
vi jenkins.model.JenkinsLocationConfiguration.xml
```

### Step 4: Update Jenkins URL

Find the old IP:

```xml
<jenkinsUrl>http://OLD_IP:8080/</jenkinsUrl>
```

Replace with new public IP:

```xml
<jenkinsUrl>http://NEW_IP:8080/</jenkinsUrl>
```

### Step 5: Restart Jenkins

```bash
sudo systemctl restart jenkins
```

---

## Best Practice

Assign a Static IP or Elastic IP to your server.

---

2. # Enable & Disable Jenkins project(jobs)

* Prevent New Builds During Maintenance

* Temporarily Disable a Jenkins Job

If you need to pause builds during server maintenance or updates:

* Navigate to the specific Jenkins job you want to disable.

---

## Steps

### Step 1: Open the Job

Go to your Jenkins dashboard and select the required job.

---

### Step 2: Disable the Job

* Click on Disable Project (usually near the top of the job page).

---

### Step 3: Verify Status

* The job will show as [DISABLED] in the job list

---

### Step 3:Enable Job

* To enable again, click “Enable Project”.

## When a job is disabled:

* Even if new commits happen (for example, pushed to GitHub), webhooks will still fire, but Jenkins will ignore them.
* No new builds will be triggered manually or automatically.Note:
* You must have the correct permissions (Job Configure or Admin rights) to disable or enable a project.
* Existing running builds will not be affected — only new builds are blocked.
* Missed commits during the disabled period will not automatically build when the job is re-enabled.

### Jenkins Plugins (Detailed Use)

1. # Maven Integration Plugin in Jenkins

* The Maven Integration Plugin is specially designed for Java-based projects and helps Jenkins automatically build, test, and package Java applications using Maven,
  directly integrating with the project’s pom.xml.

---

## Difference Between Freestyle Project and Maven Project in Jenkins

### Freestyle Project:

* Does not recognize pom.xml automatically
* You have to manually add a build step → “Invoke top-level Maven targets”
* Does not auto-detect Maven version you must select or configure it each time.
* Needs more manual configuration not ideal for pure Java/Maven projects.

---

### Maven Project:

* Automatically detects pom.xml no need to specify it manually.
* No need to add build steps it’s already built-in for Maven jobs.
* You just select the Maven version once from the dropdown (added in Global Tool Config).
* Much cleaner, faster, and smoother setup for Java applications.

---

## How to Install

1. Go to Dashboard → Manage Jenkins → Manage Plugins.
2. Click on the Available tab.
3. Search for Maven Integration Plugin.

---

## How to Create Maven Jobs

When you create a new job:

* Click New Item → Enter job name.

* Select “Maven Project”.

* In the job configuration:

* Just select the Maven version you added earlier (example: Maven-3.8.1).

* Set Goals and options like clean package.

* Set Root POM as pom.xml (default).

2. # Safe Restart Plugin

Purpose: Restart Jenkins safely without interrupting running jobs.

Safe Restart Command: http://<your-jenkins-ip>:8080/safeRestart

Regular Restart: http://<your-jenkins-ip>:8080/restart

systemctl restart jenkins ---> It is in linux server, But we dont have permission to access it , Then how to restart it.

# [IQ] what is the difference between restart and safeRestart ?

restart: It will stop the running jobs and then restarted

safeRestart: It will wait for complition of all the jobs and restarted

NOTE: for safeRestart we have plugin called as a safeRestart, Install and try it

3. # Next Build Number Plugin

Purpose: Manually set the next build number.

# Steps:

```
 1. Install the plugin.  

 2. Jenkins Dashboard → Job → Next Build Number.

 3. Enter the new build number (must be higher).

 Linux file path (not always accessible): /var/lib/jenkins/jobs/<job-name>/nextBuildNumber
```

4. # Audit Trail Plugin

Purpose: Tracks Jenkins actions like job creation, deletion, etc.

## Steps:

1. Install plugin.

2. Manage Jenkins → Configure System → Audit Trail

3. Add logger:

   Log file: /var/lib/jenkins/audit-trail.log

   Rotation:

   ```
       Size: 20 MB

       Count: 5 files
   ```

## Log file names:

audit-trail.log.0

audit-trail.log.1 to .5

Command: tail -f audit-trail.log.0  --> go and trigger the build

5. # JaCoCo Plugin

Purpose: Measure code coverage using tests.

## Use cases:

```
-> Fail build if coverage < threshold.

-> Stop deployment if code not fully tested.
```

## Steps:

-> Install plugin.

-> Configure in Post-build Actions.

-> Add coverage thresholds.

6. # Deploy to Container Plugin

Purpose: Deploy your .war file directly to a Tomcat (or similar) server.

## Steps:

```
 1. Install the plugin: Manage Jenkins → Manage Plugins → Available → Search Deploy to container → Install.

 2. Configure Jenkins job: Post-build action → Deploy war/ear to a container.

 3. Provide the WAR path and Tomcat server credentials + URL.
```

7. # SSH Agent Plugin

Purpose: Provides SSH credentials in pipeline jobs (used in Git/remote tasks).

We'll cover this more in Jenkins pipelines.

8. # Blue Ocean Plugin

Purpose: New, modern UI for Jenkins.

Easier pipeline visualization.

Optional interface.

9. # Build Name and Description Setter Plugin

Purpose: Customize the build name dynamically.

Steps:

```
   1. Install plugin.

   2. Job → Configure → Build Environment → Check Set Build Name

      Example: jio-dev-#${BUILD_NUMBER}
```

10. # Thin Backup Plugin

Purpose: Backs up Jenkins configurations.

Will cover in Jenkins backup section.

11. # Role-Based Authentication Plugin

Purpose: Assign specific roles (admin, read-only, etc.)

Will cover in Jenkins security section.

12. # Build With Parameters

The Build With Parameters feature in Jenkins allows you to pass dynamic data (like branch names, version numbers, or environment settings) to your builds.

1. Steps: Dashboard → Job → Configure.

2. Check “This project is parameterized”.

3. Add Parameter → Choice Parameter:

   Name: BranchNames

   Choices:

   main
   dev
   qa

4. In Git branch section (Source Code Mgmt):

   */${BranchNames}

##  11. Managing Views in Jenkins

##  What is a View in Jenkins?

y

A View is like a virtual folder or category that helps organize your Jenkins jobs.

Instead of keeping all jobs on a single dashboard, you can create custom views to group related jobs.

Example Views: Airtel Jobs, Jio Jobs, Testing Jobs, Deployment Jobs.

```

---

Now this is **100% your original content — nothing removed, nothing modified**.

If you want next:
- I can **fix grammar without changing meaning**
- Or make **interview-ready version** 👍
```
