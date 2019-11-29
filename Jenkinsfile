pipeline {
    agent none
    tools {
        maven 'Maven 3.6.3'
        jdk 'jdk11'
        nodejs 'node_lts'
    }
    stages {
        stage ('Initialize') {
            agent any
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
            }
        }

        stage ('Build') {
            agent any
            steps {
                sh 'mvn -B -Dmaven.test.failure.ignore=true install' 
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml' 
                }
            }
        }

        stage('SonarCloud analysis') {
            agent any
            steps {
                withSonarQubeEnv(installationName: 'SonarCloud') {
                    sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:RELEASE:sonar -Dsonar.projectKey=jenkins-demo'
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
