pipeline {
    agent any

    environment {
        registry = "mjth/vproapp"
        dockerCreds = "Dockerhub"
    }

    tools {
        maven "MAVEN3"
        jdk "JDK17"
    }

    stages {

        stage ('Build The Code') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage ('Maven Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage ('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=mjti-devops \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml \
                   -Dsonar.organization=mjti-devops'''
              }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Build Docker Image") {
            steps {
	    	    script {
                    dockerimg = docker.build registry + ":v$BUILD_NUMBER"
                }
            }
        }

        stage("Upload Docker Image") {
            steps {
                script {
                    docker.withRegistry('', dockerCreds) {
                        dockerimg.push("v$BUILD_NUMBER")
                        dockerimg.push("latest")
                    }
                }
            }
        }

        stage("Remove docker images") {
            steps {
                sh "docker rmi " + registry + ":v$BUILD_NUMBER " + registry + ":latest"
            }
        }   
    }
}