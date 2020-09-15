pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=db5ffceeae98840ce1f26e43e64db0c7021eddd9 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**/Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(35)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8081/')], contextPath: 'target/tasks', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps{
                dir('api-tes') {
                    git credentialsId: 'github_login', url: 'https://github.com/febrito/tasks-api-test'
                    sh 'mvn test'
                
                }
            }
        }

        stage ('Deploy Frontend') {
            steps {
                dir('frontend') 
                git credentialsId: 'github_login', url: 'https://github.com/febrito/tasks-frontend'
                 sh 'mvn clean package'
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8081/')], contextPath: 'tasks', war: 'target/tasks.war'
            }
        }
    }
}


