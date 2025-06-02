pipeline {
    agent any

    tools {
        maven 'Maven 3'
        jdk 'JDK17'
    }

    environment {
        SONARQUBE = 'MySonarQube'
        NEXUS_URL = 'http://localhost:8081/repository/maven-releases/'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Sagarbandal11/spring-petclinic.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
                sh 'cp target/spring-petclinic-3.4.0-SNAPSHOT.jar target/petclinic.jar'
            }
        }
       
        stage('Check JAR') {
    steps {
        sh 'ls -lh target'
     }
   }

        stage('Code Quality') {
            steps {
                withSonarQubeEnv('MySonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'localhost:8081',
                    groupId: 'com.spring',
                    version: '1.0',
                    repository: 'maven-releases',
                    credentialsId: 'nexus-creds',
                    artifacts: [
                        [artifactId: 'petclinic', classifier: '', file: 'target/petclinic.jar', type: 'jar']
                    ]
                )
            }
        }

        stage('Dockerize') {
            steps {
                sh '''
                DOCKER_BUILDKIT=0 docker build -t petclinic:1.0 .
                docker tag petclinic:1.0 localhost:5000/petclinic:1.0
                docker push localhost:5000/petclinic:1.0
                '''
            }
        }

        stage('Deploy to K8s') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
}
