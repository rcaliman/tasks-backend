pipeline {
    agent any

    stages {

        stage('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonarqube_scanner'
            }
            steps {
                withSonarQubeEnv('sonarqube_installations') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=sqp_acc20052e372e36d6fcd72bbf1b65595bacffa28 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                sleep(5)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: 'tomcat_credentials', path: '', url: 'http://localhost:8001')], contextPath: 'tasks-backend', war: 'target/*.war'
            }
        }

    }
}
