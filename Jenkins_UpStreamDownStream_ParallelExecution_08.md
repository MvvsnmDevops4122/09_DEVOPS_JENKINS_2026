#  Jenkins Upstream–Downstream Triggers

* In Jenkins, upstream/downstream triggering is used to connect multiple jobs in a sequence.  
* When one job completes successfully (upstream), Jenkins can automatically start the next job (downstream),  with no need to trigger each one manually.

** Mainly used for low environments.**

---

##  Understanding the Jenkins Job Chain

**Job Flow Overview:**  
**bsnl-development ➝ bsnl-QA ➝ bsnl-UAT**  
(Build → Test → Validate)

🔼 **Upstream Project:** bsnl-development  
➝ This means bsnl-QA is triggered after bsnl-development  

🔽 **Downstream Project:** bsnl-UAT  
➝ This means bsnl-UAT will be triggered after bsnl-QA  

*  So, if we start only the first job, the other two will run automatically.

---

##  Example

In a real software project, teams work in stages:

- **bsnl-development:** Developers build the code  
- **bsnl-QA:** QA team tests the build  
- **bsnl-UAT:** Business/users validate the product  

✅ All stages run in order — without manual clicks.

---

## 1️ Jenkins Freestyle Job – Upstream & Downstream Configuration

**Create 3 freestyle jobs:**

- bsnl-development → First job  
- bsnl-QA → Second job  
- bsnl-UAT → Third job  

---

###  Set bsnl-development as Upstream for bsnl-QA

1. Open the bsnl-QA job in Jenkins  
2. Click **Configure**  
3. Scroll to **Build Triggers**  
4. Enable: ✅ “Build after other projects are built”  
5. Enter **bsnl-development**  
6. Click **Save**  

* Now, bsnl-QA will auto-trigger after bsnl-development completes.

---

###  Set bsnl-QA as Upstream for bsnl-UAT

1. Open the bsnl-UAT job  
2. Click **Configure**  
3. Scroll to **Build Triggers**  
4. Enable: ✅ “Build after other projects are built”  
5. Enter **bsnl-QA**  
6. Click **Save**

---

###  Verify the Chain

- Open **bsnl-QA job**  
👉 You’ll see upstream & downstream relationships  

- Click **Build Now** on bsnl-development  

👉 Execution flow:  
**bsnl-development ➝ bsnl-QA ➝ bsnl-UAT**

---

##  Jenkins Declarative Pipeline Job – Upstream & Downstream

1. Create **three declarative pipeline jobs**

- bsnl-development  
- bsnl-QA  
- bsnl-UAT  

👉 To auto-trigger QA job, add below stage in master job:

```groovy
stage('idea-declarative-qa-pl')
{
    steps{
         build job : 'idea-QA'
    }
}
````

2. Verify branches (**dev/master/qa**)

3. Final script (add condition for previous job success)

---

### Declarative Pipeline Script

```groovy
pipeline {
    agent any

    tools {
        maven "mvn_3.9.15"
    }

    stages {

        stage('Checkout') {
            steps {
                notifyBuild('STARTED')
                git 'https://github.com/MvvsnmDevops4122/05_DEVOPS_MAVEN-WEBAPPLICATION-PROJECT_2026.git'
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
                curl -u admin:1234 \
                --upload-file target/maven-web-application.war \
                "http://52.91.157.198:8080/manager/text/deploy?path=/maven-web-application&update=true"
                '''
            }
        }

        stage('Master_DownStream_Job')
        {
            steps{
                build job : 'Declarative_UpStream_DownStream_Development_Pipeline_Job'
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

###  Slack Notification Function

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

#  Parallel Job Execution in Jenkins

* In Jenkins, parallel execution means running multiple independent stages or tasks at the same time instead of waiting 
  for one stage to finish before starting the next.

* Normally, Jenkins runs stages one after another (sequential execution). But with parallel execution, independent tasks 
  can run simultaneously, which helps save time and improves overall pipeline performance, especially in large pipelines.

---

##  Parallel Job Execution (Maven + Sonar and Nexus + Tomcat)

```groovy
pipeline {
    agent any

    tools {
        maven "mvn_3.9.15"
    }

    stages {

        stage('Git Checkout') {
            steps {
                snotifyBuild('STARTED')
                git 'https://github.com/MvvsnmDevops4122/05_DEVOPS_MAVEN-WEBAPPLICATION-PROJECT_2026.git'
            }
        }

        stage('Build and SQ Report') {
            parallel {
                stage('BUILD') {
                    steps {
                        sh 'mvn clean package'
                    }
                }
                stage('Sonar') {
                    steps {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Nexus and Tomcat') {
            parallel {
                stage('Nexus') {
                    steps {
                        sh 'mvn clean deploy'
                    }
                }
                stage('Tomcat') {
                    steps {
                        sh '''
                        curl -u admin:1234 \
                        --upload-file target/maven-web-application.war \
                        "http://52.91.157.198:8080/manager/text/deploy?path=/maven-web-application&update=true"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                notifyBuild(currentBuild.result)
            }
        }
        failure {
            script {
                notifyBuild(currentBuild.result)
            }
        }
    }
}
```

---

###  Notification Function

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

    slackSend(color: colorCode, message: summary, channel: '#declarative_pipeline_channel')
}
```
