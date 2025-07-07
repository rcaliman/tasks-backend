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
                    /* groovylint-disable-next-line LineLength */
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
                /* groovylint-disable-next-line LineLength */
                deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: 'tomcat_credentials', path: '', url: 'http://localhost:8001')], contextPath: 'tasks-backend', war: 'target/*.war'
            }
        }

        stage('API Test') {
            steps {
                dir('api-test') {
                    git branch: 'main', credentialsId: 'github_credentials', url: 'https://github.com/rcaliman/tasks-api-test'
                    sh 'mvn test'
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git branch: 'master', credentialsId: 'github_credentials', url: 'https://github.com/rcaliman/tasks-frontend'
                    sh 'mvn clean package'
                    /* groovylint-disable-next-line LineLength */
                    deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: 'tomcat_credentials', path: '', url: 'http://localhost:8001')], contextPath: 'tasks', war: 'target/*.war'
                }
                    
            }
        }

        stage('Funcional Test') {
            steps {
                dir('funcional-test') {
                    git branch: 'main', credentialsId: 'github_credentials', url: 'https://github.com/rcaliman/tasks-funcional-tests'
                    sh 'mvn test'
                }
            }
        }
        stage('Deploy Prod') {
            steps {
                sh 'docker-compose build'
                sh 'docker-compose up'
            }
        }
    }
}
