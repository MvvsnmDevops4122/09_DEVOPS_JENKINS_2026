#  Jenkins Complete Notes

##  What is Jenkins?

Jenkins is an open-source automation tool used for:

* Continuous Integration (CI)
* Continuous Delivery/Deployment (CD)

It automates the process of building, testing, and deploying applications, making development faster, more reliable, and error-free.

---

##  Key Features of Jenkins

*  Continuous Integration (CI) – Automates testing & code integration
*  Continuous Delivery (CD) – Prepares code for deployment
*  Continuous Deployment – Fully automates production release
*  Built With: Java
*  Originally Called: Hudson
*  Created By: Kohsuke Kawaguchi

---

##  Jenkins Workflow

```
GitHub → Jenkins → Maven → SonarQube → Nexus → Tomcat → Notifications
```
<img width="1095" height="554" alt="image" src="https://github.com/user-attachments/assets/773a193d-78b8-4a24-b30b-b160601c2dd8" />

### Steps:

1. Developer pushes code to GitHub
2. Jenkins detects code change automatically
3. Builds the project using Maven
4. Packages artifact (.war/.jar)
5. Checks code quality using SonarQube
6. Uploads artifact to Nexus Repository
7. Deploys artifact to a Tomcat server
8. Sends build notifications (Email, Slack, etc.)

---

##  Why Developers Love Jenkins

*  Free & Open Source
*  1800+ Plugins
*  Integrates with Git, Docker, Kubernetes
*  Strong community support
*  Customizable pipelines(Declarative & Scripted)
*  Cross-platform (Windows, Linux, macOS)

---

##  Advantages of Jenkins for CI/CD

*  Faster development
*  Early bug detection
*  Better code quality
*  Easy rollback
*  Improved collaboration
*  Consistent releases

---

##  Jenkins vs Other CI/CD Tools

| Tool           | Feature                |
| -------------- | ---------------------- |
| GitLab CI/CD   | Built-in pipelines     |
| GitHub Actions | GitHub automation      |
| CircleCI       | Cloud-based            |
| Travis CI      | Open-source friendly   |
| TeamCity       | Enterprise (JetBrains) |
| Bamboo         | Jira integration       |

---

##  Continuous Integration (CI)

When a developer updates code in GitHub (SCM), Jenkins immediately detects the change, then:

 * Builds the artifact using Maven
 * Generates code quality reports using SonarQube
 * Uploads the artifact to the Nexus repository
   
This entire automated process is called Continuous Integration (CI).

### Advantages:

*  Immediate bug detection
*  Less merge conflicts
*  Improved Code Quality [Consistent Testing, Code Reviews]
*  Ready-to-Deploy Build Artifacts
*  Developers receive immediate feedback on their code changes, allowing them to address issues promptly.
---

##  Continuous Delivery (CD)

* Continuous Delivery (CD) means code is always ready for production, but the final deployment is done manually — typically 
  with just one click after proper verification.

NOTE: Once we get the approval from the client and the sign-off mail from the QA team then we are going to deploy

### Key Points :

* Last step is manual 

* Safer method –

* Best for client projects 

* Faster feedback 

---

##  Continuous Deployment

* Continuous Deployment means automatically deploying every successful build directly to production 

👉 with no manual intervention, no clicks, and no approval required.

### Use Case:

* Best for home-based projects, personal tools, or internal company tools

---

##  CD vs Deployment

<img width="1138" height="623" alt="image" src="https://github.com/user-attachments/assets/2f423481-5680-4b87-9146-12f9169ef290" />

```
Delivery → Manual approval → Production
Deployment → Auto deploy → Production
```

---

##  When to Use a Freestyle Job

* For **simple projects** with fewer stages
* When doing **manual integrations** with external tools
* When you **don’t need complex logic**, like what Pipelines provide
* Ideal for **learning, quick setups, and POCs (proof of concept)**

---

##  Create Jenkins Job (Freestyle)

 1. Go to Jenkins Dashboard → New Item

 2. Enter Job Name: e.g., jio-dev

 3. Select Freestyle Project → Click OK

 4. In General Tab:

    Add a description

 5. In Source Code Management:

    Select Git --> Add your repository URL --> If Git is not installed, run: sudo yum install git -y

 6. In Branches to Build:

    Use: */development ---> Please ensure which branch your selected.

 7. In Build:

    Add build step: Invoke top-level Maven targets

    Goals: clean package --> Click Save --> Click Build Now

 8. If it fails, check Console Output – likely Maven is not installed.


```
clean package
```

---

##  Maven Installation

 1. Jenkins Dashboard → Manage Jenkins

 2. Click Tools → Scroll to Maven

 3. Add Maven → Name: maven_3.9.8

 4. Select version 3.9.8 (or latest) --> Save

 5. Go back to the job → Configure → Build step → Select installed Maven version

 6. Save and Build Now

👉 WAR file location (after successful build): /var/lib/jenkins/workspace/jio-dev/target/maven-web-application.war

---

##  SonarQube Integration

 1. Get the SonarQube server URL, e.g., http://3.110.105.157:9000/

 2. Go to GitHub Repository → switch to development branch

 3. Edit pom.xml → Inside <properties> tag, add:

    <sonar.host.url>http://3.110.105.157:9000/</sonar.host.url>

 4. In Jenkins job:

   Configure → Build → update goal: 
   ```
   clean package sonar:sonar
   ```

 5. Save and Build Now

    Check SonarQube Dashboard → Projects → for analysis results

## 📦 Nexus Integration
---
# 1. Update pom.xml in Maven project:


<distributionManagement>
  <repository>
    <id>nexus</id>
    <name>KK FUNDA Releases Nexus Repository</name>
    <url>http://44.201.165.168:8081/repository/jio-release/</url>
  </repository>
  <snapshotRepository>
    <id>nexus</id>
    <name>KK FUNDA Snapshot Nexus Repository</name>
    <url>http://44.201.165.168:8081/repository/jio-snapshot/</url>
  </snapshotRepository>
</distributionManagement>
```

## 2. Where to store Nexus credentials?

 1. Inside Maven’s settings.xml

 2. Find the file: find / -name settings.xml

 3. Typically in: /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.9.9/conf/

 4. Add credentials:

  <server>
    <id>nexus</id>
    <username>admin</username>
    <password>****</password>
  </server>

### Jenkins Goal:

  Configure → Build → Goal: 
  ```
  clean deploy sonar:sonar
  ```
  Save and Build Now


http://44.201.165.168:8081/repository/jio-release/
http://44.201.165.168:8081/repository/jio-snapshot/

---

## Tomcat Integration (WAR Deployment)

 1. Install plugin:
    

    Manage Jenkins → Plugins → Install Deploy to container plugin


 2. Go to Jenkins Job → Configure

    Scroll to Post-build Actions

    Select: Deploy war/ear to a container

    WAR/EAR file path:
```
    **/maven-web-application.war
```
    Add container: Tomcat 9

    Provide:

    Tomcat URL (e.g., http://<ip>:8080)

    Jenkins credentials (admin/*****)

 4. Save and Build Now
---




##  Jenkins Slowness Fix

 Update ip addree:  vi /var/lib/jenkins/jenkins.model.JenkinsLocationConfiguration.xml

---
