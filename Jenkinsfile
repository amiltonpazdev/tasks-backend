pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                bat 'mvn test' // não usar o clean novamente para não apagar o binário gerado anteriormente
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SonarQubeLocal') {
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBackend -Dsonar.projectBaseDir=. -Dsonar.host.url=http://localhost:9000 -Dsonar.login=sqp_be8d26129472c1ae4a252e28e1b46af5154b6526 -Dsonar.java.binaries=target -Dsonar.sources=src/main/java -Dsonar.tests=src/test/java -Dsonar.test.inclusions=src/test/java/br/ce/wcaquino/taskbackend/utils/DateUtilsTest.java -Dsonar.test.inclusions=src/test/java/br/ce/wcaquino/taskbackend/controller/TaskControllerTest.java -Dsonar.java.test.libraries=target/tasks-backend/WEB-INF/lib -Dsonar.java.coveragePlugin=jacoco -Dsonar.coverage.jacoco.xmlReportPaths=target/jacoco.exec -Dsonar.java.libraries=target/tasks-backend/WEB-INF/lib -Dsonar.qualitygate.wait=true -Dsonar.coverage.exclusions=**TaskBackendApplication.java**,**/model/**"
                }                
            }
        }
        stage ('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarQubeCredencial'
                }
                
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'tomcat-credencial', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
         stage ('API Test') {
            steps {
                dir('api-test') {
                    git credentialsId: 'github-credencial', url: 'https://github.com/amiltonpazdev/tasks-apitest'
                    bat 'mvn test'
                }
            }
        }
        stage ('Build Fontend') {
             steps {
                dir('frontend') {
                    git credentialsId: 'github-credencial', url: 'https://github.com/amiltonpazdev/tasks-frontend'
                    bat 'mvn clean package -DskipTests=true'
                    deploy adapters: [tomcat8(credentialsId: 'tomcat-credencial', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }                
            }
        }
        stage ('Functional Test') {
            steps {
                dir('functional-test') {
                    git credentialsId: 'github-credencial', url: 'https://github.com/amiltonpazdev/tasks-functional-tests'
                    bat 'mvn test'
                }
            }
        }
        stage( 'Deploy Prod') {
            steps {
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }
        stage ('Health Check') {
            steps {
                sleep(5)
                dir('functional-test') {                    
                    bat 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, stdioRetention: '', testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, target/failsafe-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war', onlyIfSuccessful: true
        }
    }
}