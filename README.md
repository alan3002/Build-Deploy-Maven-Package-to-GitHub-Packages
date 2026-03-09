Build & Deploy Maven Package to GitHub Packages using Jenkins

Compile. Package. Authenticate securely. Deploy to GitHub Packages — fully automated using a Jenkins Pipeline.

Table of Contents

Lab Overview

Prerequisites

Step 1 — Create GitHub PAT Credential in Jenkins

Step 2 — Add Maven settings.xml via Config File Management

Step 3 — Set Up the Java Application

Step 4 — Configure pom.xml for GitHub Packages

Step 5 — Create the Jenkins Pipeline Job

Step 6 — Write the Jenkinsfile

Step 7 — Run and Verify

Lab Summary

Lab Overview

In this lab you will create a complete CI/CD pipeline using Jenkins that:

Pulls a Java Maven project from GitHub

Builds and packages it into a JAR file

Authenticates securely using Jenkins credentials

Uses Config File Provider to manage settings.xml

Deploys the artifact to GitHub Packages (Maven Registry)

This demonstrates a real-world DevOps CI/CD workflow.

Prerequisites
Jenkins Setup
Requirement	Configuration
Jenkins Running	Accessible via browser
JDK Installed	Manage Jenkins → Global Tool Configuration
Maven Installed	Manage Jenkins → Global Tool Configuration
Required Jenkins Plugins

Install the following plugins:

Plugin	Purpose
Pipeline	Run Jenkinsfile pipelines
Git	Pull code from GitHub
Config File Provider	Manage settings.xml
GitHub Integration	GitHub SCM integration
Maven Integration	Execute Maven builds

Install via:

Manage Jenkins → Manage Plugins → Available
GitHub Requirements

You need:

A GitHub repository

A Personal Access Token (PAT) with permissions:

Scope	Purpose
write:packages	Upload artifacts
read:packages	Download artifacts
repo	Access repository

Create the token:

GitHub → Settings → Developer Settings → Personal Access Tokens
Step 1 — Create GitHub PAT Credential in Jenkins

Store credentials securely in Jenkins.

Navigate to:

Manage Jenkins → Credentials → System → Global Credentials

Click Add Credentials and fill:

Field	Value
Kind	Username with password
Username	Your GitHub username
Password	GitHub PAT
ID	github-packages-cred
Description	GitHub Packages Deploy Credential

Click Save.

Jenkins encrypts credentials and they are referenced only by ID.

Step 2 — Add Maven settings.xml via Config File Management

Instead of storing settings.xml in your repository, Jenkins can manage it securely.

Navigate:

Manage Jenkins → Managed Files

Click Add a new Config

Select:

Global Maven Settings File

Configuration:

Field	Value
Name	maven-github-settings
ID	maven-github-settings

Paste the following configuration:

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0">
  <servers>
    <server>
      <id>github</id>
      <username>${env.GH_USER}</username>
      <password>${env.GH_TOKEN}</password>
    </server>
  </servers>
</settings>

Save the configuration.

At runtime Jenkins injects credentials using environment variables.

Step 3 — Set Up the Java Application

Your repository should look like this:

your-repo
│
├── src
│   └── main
│       └── java
│           └── com/example
│               └── App.java
│
├── pom.xml
└── Jenkinsfile
App.java
package com.example;

public class App {

    public static void main(String[] args) {
        System.out.println("Hello Jenkins CI/CD!");
    }

}
Step 4 — Configure pom.xml for GitHub Packages

The distributionManagement section tells Maven where to deploy the artifact.

<project xmlns="http://maven.apache.org/POM/4.0.0">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>jenkins-demo</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <name>Jenkins GitHub Packages Demo</name>

    <distributionManagement>
        <repository>
            <id>github</id>
            <name>GitHub Packages</name>
            <url>https://maven.pkg.github.com/YOUR_USERNAME/YOUR_REPO</url>
        </repository>
    </distributionManagement>

</project>

Replace:

YOUR_USERNAME
YOUR_REPO

with your GitHub username and repository name.

Step 5 — Create the Jenkins Pipeline Job

Go to:

Jenkins Dashboard → New Item

Create a Pipeline Job.

Configuration:

Setting	Value
Name	maven-deploy-pipeline
Pipeline Type	Pipeline script from SCM
SCM	Git
Repository URL	Your GitHub repo
Branch	*/main
Script Path	Jenkinsfile

Click Save.

Step 6 — Jenkinsfile

Create a Jenkinsfile in your repository root.

pipeline {

    agent any

    environment {
        GITHUB_CREDS = credentials('github-packages-cred')
        JAVA_HOME = tool name: 'jdk11'
        MAVEN_HOME = tool name: 'maven3'
        PATH = "${JAVA_HOME}/bin:${PATH}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Deploy') {

            steps {

                configFileProvider([
                    configFile(
                        fileId: 'maven-github-settings',
                        variable: 'MAVEN_SETTINGS'
                    )
                ]) {

                    sh '''
                        export GH_USER=${GITHUB_CREDS_USR}
                        export GH_TOKEN=${GITHUB_CREDS_PSW}

                        ${MAVEN_HOME}/bin/mvn -s $MAVEN_SETTINGS -B clean package
                        ${MAVEN_HOME}/bin/mvn -s $MAVEN_SETTINGS -B deploy
                    '''

                }

            }

        }

    }

    post {

        success {
            echo "Build and deployment to GitHub Packages completed successfully."
        }

        failure {
            echo "Pipeline failed. Check console logs."
        }

    }
    
    }


Step 7 — Run and Verify
Trigger the Build

Go to your Jenkins pipeline.

Click:

Build Now

Jenkins will:

Clone the repository

Build the Maven project

Inject credentials

Authenticate with GitHub Packages

Deploy the artifact

Verify Build Logs

Open:

Build → Console Output

Look for:

BUILD SUCCESS
Verify Package on GitHub

Navigate to:

GitHub → Repository → Packages

You should see:

Field	Value
Package	jenkins-demo
Version	1.0.0
Files	jenkins-demo-1.0.0.jar
Registry	GitHub Maven Registry
Lab Summary
Step	Description
Credentials	Stored GitHub PAT securely in Jenkins
settings.xml	Managed via Config File Provider
App Code	Created simple Java Maven project
pom.xml	Configured GitHub Packages repository
Pipeline Job	Created Jenkins pipeline
Jenkinsfile	Implemented CI/CD pipeline
Verification	Confirmed artifact in GitHub Packages

If you want, I can also give you a GitHub README that looks more professional (with diagrams, badges, and DevOps architecture) — which makes your DevOps portfolio much stronger for interviews.
