def COLOR_MAP=[
    'SUCESS':'good',
     'FAILURE': 'danger',
    ]
    pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    stages {
        stage('Fetch code') {
            steps {
                git branch: 'vp-rem', url: 'https://github.com/devopshydclub/vprofile-repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
                echo "Running SonarQube analysis..."
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage("UploadArtifact") {
    steps {
        echo "Uploading artifact to Nexus..."
        nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: '172.31.83.184:8081',
            groupId: 'QA',
            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}".replaceAll(" ", "-"),
            repository: 'vprofile-repo',
            credentialsId: 'nexuslogin',
            artifacts: [
                [artifactId: 'vproapp',
                 classifier: '',
                 file: 'target/vprofile-v2.war',
                 type: 'war']
                ]
            )
        }
    }
 }
 post{
     always{
         echo 'Slack Notifications.'
         slackSend channel: 'jenkinscicd',
         color: COLOR_MAP[currentBuild.currentResult],
         message: "*${currentBuild.currentResult}:*Job ${env.JOB_NAME} build ${env.BUUILD_NUMBER} \n More info at:${env.BUILD_URL} "
     }
 }
}