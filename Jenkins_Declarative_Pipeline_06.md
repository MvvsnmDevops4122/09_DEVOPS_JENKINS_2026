#  Jenkins Declarative Pipeline 

---
# Definition

*  Declarative Pipeline is a simple and structured way to create CI/CD workflows in Jenkins.
*  It uses predefined syntax, so it is easy to read, write, and maintain.
---

## Declarative Pipeline Syntax


pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello, Declarative Pipeline!'
            }
        }
    }
}

---

##  Declarative Pipeline Example

```groovy
pipeline {
    agent any

    tools {
        maven "mvn_3.9.15"
    }

    parameters {
        choice(
            name: 'BRANCH_NAME',
            choices: ['main', 'master', 'development', 'feature'],
            description: 'Select Git Branch'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                notifyBuild('STARTED')

                echo "Selected branch: ${params.BRANCH_NAME}"
                git branch: params.BRANCH_NAME,
                    url: 'https://github.com/MvvsnmDevops4122/05_DEVOPS_MAVEN-WEBAPPLICATION-PROJECT_2026.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('SonarQube Report') {
            steps {
                sh "mvn sonar:sonar"
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh "mvn deploy"
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                curl -u satya:password \
                --upload-file target/maven-web-application.war \
                "http://54.91.76.99:8080/manager/text/deploy?path=/maven-web-application&update=true"
                '''
            }
        }
    }

    post {

        success {
            notifyBuild('SUCCESS')
        }

        failure {
            notifyBuild('FAILED')
        }
    }
}
```

---

##  Slack Notification Function

```groovy
def notifyBuild(String buildStatus = 'STARTED') {

    buildStatus = buildStatus ?: 'SUCCESS'

    def colorCode = '#FF0000'
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    if (buildStatus == 'STARTED') {
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESS') {
        colorCode = '#00FF00'
    } else {
        colorCode = '#FF0000'
    }

    slackSend(
        color: colorCode,
        message: summary,
        channel: '#declarative_pipeline_channel'
    )
}
```

---

#  Jenkins Triggers

##  Poll SCM

```groovy
triggers {
    pollSCM('* * * * *')
}
```

* Checks repository **every 1 minute**
* Triggers build if changes detected

---

##  Build Periodically

```groovy
triggers {
    cron('* * * * *')
}
```

*  Runs build **every 1 minute (fixed schedule)**
*  Even if no code change

---

##  Webhooks (Recommended)

###  Step-by-Step Setup

###  Step 1: Install Plugin

* GitHub Integration Plugin

---

###  Step 2: Configure Jenkins URL

 Go to:
`Manage Jenkins → Configure System`

 Set:

```
Jenkins URL = http://your-jenkins-ip:8080/
```

---

###  Step 3: Enable in Job

```groovy
triggers {
    githubPush()
}
```

---

###  Step 4: Configure in GitHub

 Repository → Settings → Webhooks → Add webhook

**Fill:**

* Payload URL:

```
http://your-jenkins-ip:8080/github-webhook/
```

* Content type:

```
application/json
```

* Events:

```
Just push event
```

---

###  Step 5: Test

 Push code → Jenkins build triggers automatically 

---

#  Jenkins Parameters

---

##  Step 1: Open Job

 Go to Jenkins Job
 Click **Configure**

---

##  Step 2: Add Pipeline Code

 Select:

```
Pipeline script
```

 Add:

```groovy
pipeline {
    agent any

    parameters {
        choice(
            name: 'BRANCH_NAME',
            choices: ['main', 'master', 'development', 'feature'],
            description: 'Select Git Branch'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Selected branch: ${params.BRANCH_NAME}"
                git branch: params.BRANCH_NAME,
                    url: 'https://github.com/MvvsnmDevops4122/05_DEVOPS_MAVEN-WEBAPPLICATION-PROJECT_2026.git'
            }
        }
    }
}
```

 Click **Save**

---

##  Step 3: Run with Parameters

 Click:

```
Build with Parameters
```

 Select:

```
development
```

Click **Build**

---

##  Step 4: Verify Output

 In console output:

```
Selected branch: development
```

 Confirms correct branch selection ✅
 
 ---

---

---
