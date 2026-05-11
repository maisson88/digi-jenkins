# Jenkins Lab 1 – CI Foundation: Unit Tests, Webhooks & Secrets

## Overview

This lab demonstrates a complete Continuous Integration (CI) workflow using Jenkins with a PHP application hosted on GitHub.

The pipeline automatically:

* Pulls code from GitHub
* Runs PHPUnit tests
* Fails builds when tests fail
* Triggers builds automatically using GitHub Webhooks
* Uses Jenkins Credentials to securely store secrets

---

# Technologies Used

* Jenkins
* GitHub
* PHP
* PHPUnit
* Docker
* GitHub Webhooks
* zrok/ngrok

---

# Lab Objectives

* Create a Declarative Jenkins Pipeline
* Configure GitHub Webhooks
* Run automated PHPUnit tests
* Store GitHub PAT securely in Jenkins Credentials
* Understand Jenkins stages and console output
* Trigger builds automatically on every push

---

# Project Structure

```bash
digi-jenkins/
│
├── Jenkinsfile
├── index.php
├── tests/
│   └── SampleTest.php
└── README.md
```

---

# Step 1 – Clone Repository

```bash
git clone https://github.com/maisson88/digi-jenkins.git
cd digi-jenkins
```

---

# Step 2 – Access Jenkins Container

```bash
docker exec -u root -it jenkins bash
```

---

# Step 3 – Fix Debian Repository Issue

Inside the container:

```bash
rm -f /etc/apt/sources.list.d/*
```

Create a new sources list:

```bash
cat > /etc/apt/sources.list << 'EOF'
deb https://deb.debian.org/debian bookworm main
deb https://deb.debian.org/debian bookworm-updates main
deb https://security.debian.org/debian-security bookworm-security main
EOF
```

Update packages:

```bash
apt clean
rm -rf /var/lib/apt/lists/*
apt update
```

---

# Step 4 – Install PHP & PHPUnit

Install required packages:

```bash
apt install -y php-cli php-mbstring php-xml php-curl unzip wget
```

Download PHPUnit:

```bash
wget -O /usr/local/bin/phpunit https://phar.phpunit.de/phpunit-9.phar
chmod +x /usr/local/bin/phpunit
```

Verify installation:

```bash
phpunit --version
```

Expected output:

```bash
PHPUnit 9.6.34
```

---

# Step 5 – Configure Jenkins Credentials

In Jenkins:

```text
Manage Jenkins → Credentials → Global → Add Credentials
```

Choose:

* Kind: Secret text
* Scope: Global

Fill in:

| Field       | Value                              |
| ----------- | ---------------------------------- |
| Secret      | GitHub Personal Access Token       |
| ID          | github-token                       |
| Description | GitHub PAT for Jenkins CI pipeline |

---

# Step 6 – Configure zrok Tunnel

Enable zrok:

```bash
zrok2 enable YOUR_TOKEN
```

Create public share:

```bash
zrok2 share public http://192.168.148.130:8081
```

Example public URL:

```text
https://zkwgqq2hz4ux.shares.zrok.io
```

---

# Step 7 – Configure GitHub Webhook

In GitHub repository:

```text
Settings → Webhooks → Add webhook
```

Payload URL:

```text
https://YOUR-ZROK-URL/github-webhook/
```

Content type:

```text
application/json
```

Events:

* Just the push event

---

# Step 8 – Jenkins Pipeline Configuration

Create a new Pipeline Job in Jenkins.

Enable:

```text
GitHub hook trigger for GITScm polling
```

---

# Jenkinsfile

```groovy
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/maisson88/digi-jenkins.git',
                    branch: 'main'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh '/usr/local/bin/phpunit --log-junit results.xml tests/'
            }
        }

        stage('Display Results') {
            steps {
                junit 'results.xml'
            }
        }
    }

    post {
        always {
            echo 'Pipeline job finished.'
        }

        success {
            echo 'All tests passed successfully.'
        }

        failure {
            echo 'The code failed the tests.'
        }
    }
}
```

<img width="974" height="266" alt="Screenshot from 2026-05-11 17-02-12" src="https://github.com/user-attachments/assets/2cf495d7-0ecf-4bf9-9bc0-5ab022325ad1" />

<img width="1280" height="800" alt="Screenshot from 2026-05-11 16-54-22" src="https://github.com/user-attachments/assets/f67c8060-014b-49dc-b731-a38ee657be19" />

<img width="1280" height="800" alt="Screenshot from 2026-05-11 16-20-45" src="https://github.com/user-attachments/assets/82569c1c-ab29-4dd1-a91e-28ed166c342b" />

---

# Step 9 – Trigger Automatic Build

Push any change:

```bash
git add .
git commit -m "test webhook"
git push
```

Jenkins automatically:

* Receives webhook
* Starts pipeline
* Runs PHPUnit tests
* Shows build result

---

# Example Failed Build

If a unit test fails:

```bash
FAILURES!
Tests: 1, Assertions: 1, Failures: 1.
```

Jenkins build status becomes:

```text
FAILURE
```

---

# Example Successful Build

```bash
OK (1 test, 1 assertion)
```

Jenkins build status becomes:

```text
SUCCESS
```

---

# Learning Outcomes

By completing this lab, I learned how to:

* Build CI pipelines using Jenkins
* Integrate GitHub with Jenkins using Webhooks
* Run automated unit tests
* Secure secrets using Jenkins Credentials
* Debug Jenkins pipeline errors
* Understand Jenkins console logs and build stages

---

# Author

Maisson Ahmed
