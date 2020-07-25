pipeline{
    agent any
    stages{
        stage('Build Backend'){
            steps{
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests'){
            steps{
                bat 'mvn test'
            }
        }
        stage('Sonar Analysis'){
            environment{
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps{
                withSonarQubeEnv('SONAR_LOCAL'){
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://192.168.0.104:9000 -Dsonar.login=07015b39f1bafb6f7f079173c857e05a9f3d2a08 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage('Quality Gate'){
            steps{
                sleep(45)
                timeout(time: 1 , unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomCatLogin', path: '', url: 'http://18.212.115.92:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
                dir('api-test') {
                    git credentialsId: 'GitHub_Login', url: 'https://github.com/romelo2/tasks-api-test'        
                    bat 'mvn test'    
                }
            }
        }
        stage ('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git credentialsId: 'GitHub_Login', url: 'https://github.com/romelo2/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomCatLogin', path: '', url: 'http://18.212.115.92:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Test') {
            steps {
                dir('functional-test') {
                    git credentialsId: 'GitHub_Login', url: 'https://github.com/romelo2/tasks-functional-tests'        
                    bat 'mvn test'    
                }
            }
        }
        stage('Deploy Prod') {
            steps {
                bat 'docker-compose build'
                bat 'docler-compose up -d'
            }
        }
    }
}