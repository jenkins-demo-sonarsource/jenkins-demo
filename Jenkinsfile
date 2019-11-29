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

        stage('SonarCloud branch analysis') {
            agent any
            when {
                not {
                    changeRequest()
                }
            }
            steps {
                withSonarQubeEnv(installationName: 'SonarCloud') {
                    sh "mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:RELEASE:sonar \
                        -Dsonar.projectKey=jenkins-demo-sonarsource_jenkins-demo \
                        -Dsonar.branch.name=${env.BRANCH_NAME}"
                }
            }
        }

        stage('SonarCloud PR analysis') {
            agent any
            when {
                changeRequest()
            }
            steps {
                withSonarQubeEnv(installationName: 'SonarCloud') {
                    sh "mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:RELEASE:sonar \
                        -Dsonar.projectKey=jenkins-demo-sonarsource_jenkins-demo \
                        -Dsonar.pullrequest.key=${env.CHANGE_ID} \
                        -Dsonar.pullrequest.base=${env.CHANGE_TARGET} \
                        -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH}"
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
