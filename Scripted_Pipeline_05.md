#  Jenkins Scripted Pipeline 

Jenkins Scripted Pipeline is a powerful feature of Jenkins that allows you to define and automate your **CI/CD (Continuous Integration and Continuous Delivery)** workflows using **Groovy scripting language**.

##  Step-by-Step Setup of a Jenkins Scripted Pipeline

### Step 1: Create a Jenkins Pipeline Job
1. Go to Jenkins Dashboard → Click on **“New Item”**  
2. Enter a name like **scriptedPL-ci-cd-job**  
3. Choose **Pipeline → Click OK**  

#### Enable Stage View (if not visible)
1. After creating the job and opening it, you may not see the Stage View section.  
   This happens if the **Pipeline: Stage View Plugin** is not yet installed.  

####  Install the Stage View Plugin
1. Go to: **Manage Jenkins → Manage Plugins → search “Pipeline: Stage View Plugin” → install**  
   Once installed and Jenkins is restarted, the Stage View will be visible after a pipeline run that includes stage blocks.  

---

##  Step 2: Scripted Pipeline Syntax
```groovy id="s1"
node {
    stage('Example') {
        // commands go here
    }
}
````

---

##  Step 3: Checkout Code from GitHub

### Use Snippet Generator

1. Go to your pipeline job → **Configure → Pipeline Syntax**
2. In Snippet Generator:

   * Select **git**
   * Enter repository URL, branch, credentials
   * Click **Generate Pipeline Script**
   * Copy code
3. Paste inside **node {}**

```groovy id="s2"
node {
    stage('checkout') {
        git 'https://github.com/MvvsnmDevops4122/05_DEVOPS_MAVEN-WEBAPPLICATION-PROJECT_2026.git'
    }
}
```

---

##  Step 4: Configure Maven in Jenkins

1. Go to **Manage Jenkins → Global Tool Configuration → Maven → Add Maven**
2. Name it **Maven-3.9.15**
3. Jenkins installs automatically

---

## ⚙️ Step 5: Build Stage – Fixing mvn Error

Jenkins needs Maven path using **tool command**
// /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/mvn_3.9.15/bin

```groovy id="s3"
def mavenHome = tool name: 'mvn_3.9.15'

stage('Build') {
    sh "${mavenHome}/bin/mvn clean package"
}
```

###  Explanation

* **tool** → Gets Maven path
* **def** → Declares variable
* **${mavenHome}/bin/mvn** → Full Maven command

---

##  Step 6: SonarQube Analysis

###  Prerequisites

* Sonar server running
* properties configured in **pom.xml**

```groovy id="s4"
def mavenHome = tool name: 'Maven-3.9.15'

stage('checkout') {
        git 'https://github.com/MvvsnmDevops4122/05_DEVOPS_MAVEN-WEBAPPLICATION-PROJECT_2026.git'
    }

stage('Build') {
    sh "${mavenHome}/bin/mvn clean package"
    }

stage('SonarQube  Report') {
    sh "${mavenHome}/bin/mvn sonar:sonar"
    }
}
```

---

## 📦 Step 7: Upload Artifact to Nexus

### ✅ Prerequisites

* Nexus server running
* distributionManagement in **pom.xml**
* Credentials stored in Jenkins

```groovy id="s5"
stage('Upload to Nexus') {
    sh "${mavenHome}/bin/mvn deploy"
}
```

---

## 🚀 Step 8: Deploy to Tomcat Server

### ✅ Prerequisites

* Tomcat Manager enabled (**manager/text**)
* WAR file built (**target/*.war**)
* User with deploy privileges

```groovy id="s6"
stage('Deploy to Tomcat') {
    sh '''
    curl -u satya:password \
    --upload-file target/maven-web-application.war \
    "http://54.91.76.99:8080/manager/text/deploy?path=/maven-web-application&update=true"
    '''
}
```

### ✅ What this does

* **curl** → HTTP request tool
* **-u tomcat:tomcat** → Authentication
* **--upload-file** → Upload WAR
* **path=/maven-web-application** → App path
* **update=true** → Redeploy

---

## ✅ Final Full Pipeline Script

```groovy id="s7"
node {
    def mavenHome = tool name: 'mvn_3.9.15'

    stage('checkout') {
        git 'https://github.com/MvvsnmDevops4122/05_DEVOPS_MAVEN-WEBAPPLICATION-PROJECT_2026.git'
    }

    stage('Build') {
        sh "${mavenHome}/bin/mvn clean package"
    }

    stage('SonarQube Report') {
        sh "${mavenHome}/bin/mvn sonar:sonar"
    }

    stage('Upload to Nexus') {
        sh "${mavenHome}/bin/mvn deploy"
    }

    stage('Deploy to Tomcat') {
        sh '''
        curl -u satya:password \
        --upload-file target/maven-web-application.war \
        "http://54.91.76.99:8080/manager/text/deploy?path=/maven-web-application&update=true"
        '''
    }
}
```

---

## 🔔 Slack Notification

```groovy id="s8"
node {

    echo "git branch name: ${env.BRANCH_NAME}"
    echo "build number is: ${env.BUILD_NUMBER}"
    echo "node name is: ${env.NODE_NAME}"

    def mavenHome = tool name: 'mvn_3.9.15'

    try {
         // START message (must be first)
        notifyBuild('STARTED')
        stage('checkout') {
            git 'https://github.com/MvvsnmDevops4122/05_DEVOPS_MAVEN-WEBAPPLICATION-PROJECT_2026.git'
        }

        stage('Build') {
            sh "${mavenHome}/bin/mvn clean package"
        }

        stage('SonarQube Report') {
            sh "${mavenHome}/bin/mvn sonar:sonar"
        }

        stage('Upload to Nexus') {
            sh "${mavenHome}/bin/mvn deploy"
        }

        stage('Deploy to Tomcat') {
            sh '''
                curl -u satya:password \
                --upload-file target/maven-web-application.war \
                "http://54.91.76.99:8080/manager/text/deploy?path=/maven-web-application&update=true"
            '''
        }

    } catch (e) {
        currentBuild.result = "FAILED"
    } finally {
        notifyBuild(currentBuild.result)
    }
}

def notifyBuild(String buildStatus = 'STARTED') {

    buildStatus = buildStatus ?: 'SUCCESS'

    def colorName = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    if (buildStatus == 'STARTED') {
        colorName = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESS') {
        colorName = 'GREEN'
        colorCode = '#00FF00'
    } else {
        colorName = 'RED'
        colorCode = '#FF0000'
    }

    slackSend(
        color: colorCode,
        message: summary,
        channel: '#scripted_pipeline_channel'
    )
}
```

```
```
