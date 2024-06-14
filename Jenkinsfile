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
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBackend -Dsonar.projectBaseDir=. -Dsonar.host.url=http://localhost:9000 -Dsonar.login=sqp_21387d7c2cf25893f95cf5058dc5c87f30967895 -Dsonar.java.binaries=target -Dsonar.sources=src/main/java -Dsonar.tests=src/test/java -Dsonar.test.inclusions=src/test/java/br/ce/wcaquino/taskbackend/utils/DateUtilsTest.java -Dsonar.test.inclusions=src/test/java/br/ce/wcaquino/taskbackend/controller/TaskControllerTest.java -Dsonar.java.test.libraries=target/tasks-backend/WEB-INF/lib -Dsonar.java.coveragePlugin=jacoco -Dsonar.coverage.jacoco.reportPath=target/jacoco.exec -Dsonar.java.libraries=target/tasks-backend/WEB-INF/lib -Dsonar.qualitygate.wait=true -Dsonar.coverage.exclusions=**TaskBackendApplication.java**,**/model/**"
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
    }
}