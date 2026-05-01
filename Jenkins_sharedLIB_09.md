#  Jenkins Shared Libraries

##  Definition

A Jenkins Shared Library is a collection of reusable pipeline scripts, functions, and other code components that can be used across multiple Jenkins pipelines, 

 allowing you to maintain and update code in one place instead of duplicating it across multiple Jenkinsfiles.

##  Shared Library Structure

```text
my-shared-library/
├── vars/
│   └── myUtils.groovy
├── src/
│   └── com/example/
│       └── MyClass.groovy
└── resources/
```

### Explanation

* **vars/**       → contains Groovy scripts that define global variables or functions, which can be accessed directly in the Jenkinsfile.
* **src/**        → Contains Groovy classes for complex functionality and encapsulated logic used in your pipeline.
* **resources/**  → stores non-Groovy files, like configuration files, for use in your library. (JSON, YAML, etc.)

## ❗ Important Rule

* File name in `vars/` = function name used in Jenkinsfile
* Must contain `def call()` method

## Why Do We Use Shared Library?

* To avoid repeating the same code in every Jenkinsfile.
* To make pipelines clean and short.
* To manage all common logic in one place.
* Easy to update — change in one place reflects everywhere

##  Sample Contents

### vars/myUtils.groovy

```groovy
def call(String name) {
    echo "Hello, ${name}!"
}
```

### src/com/example/MyClass.groovy

```groovy
package com.example

class MyClass {
    static void greet(script, String name) {
        script.echo "Hello, ${name}!"
    }
}
```

### resources/config/myConfig.json

```json
{
  "key": "value"
}
```

##  How to Set It Up

### Step 1: Create GitHub Repository

Example:

```text
https://github.com/MvvsnmDevops4122/10_JENKINS_SHARED_LIBRARY_2026.git
```

Repository structure:

```text
vars/
src/
resources/
```

### Step 2: Configure in Jenkins

1. Jenkins → Manage Jenkins
2. Configure System
3. Global Pipeline Libraries

Fill details:

* Name: `JENKINS_SHARED_LIBRARY`
* Default version: `main`
* Retrieval method: Git
* Project Repository: https://github.com/MvvsnmDevops4122/10_JENKINS_SHARED_LIBRARY_2026.git  #shareliberay repository url
* Credentials: Add if private

Save configuration

##  Sample Jenkinsfile Using Shared Library

```groovy
@Library('JENKINS_SHARED_LIBRARY') _

pipeline {
    agent any

    tools {
        maven "mvn_3.9.15"
    }

    stages {

        stage('Checkout') {
            steps {
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
    }

    post {
        success {
            sendSlackNotifications('SUCCESS')
        }

        failure {
            sendSlackNotifications('FAILED')
        }
    }
}
```

##  Important Note

The below file must exist:

```text
vars/sendSlackNotifications.groovy
```

```groovy
def call(String status) {
    echo "Build Status: ${status}"
}
```

##  Importing Multiple Shared Libraries

### Individual Import

```groovy
@Library('my-shared-library') _
@Library('my-shared-lib-2') _
@Library('my-shared-lib-3') _
```

### Combined Import

```groovy
@Library(['my-shared-library', 'otherlib@abc1234', 'thirdlibName']) _
```
