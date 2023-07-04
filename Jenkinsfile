pipeline {
    agent any
    environment {
      PATH="$PATH:/opt/apache-maven-3.9.2/bin"
    }
    stages {
        stage("GetCode") {
            steps {
                git 'https://github.com/thierno953/docker-project.git'
            }
        }
        stage("mvn package") {
            steps {
                sh 'mvn clean install'
            }
        }
        stage ('SonarQubeAnalysis') {
            steps {
                withSonarQubeEnv('sonarqube-9.9.0') {
                    sh "mvn sonar:sonar"
                }
            }
        }
        stage ('BuildDockerImage') {
            steps {
                sh 'docker image build -t thiernos/sonar-image:latest .'
            }
        } 
        stage ('PUSH Image') {
            steps {
                withCredentials([string(credentialsId: 'HubPwd', variable: 'Hub')]) {
                sh "docker login -u thiernos - p ${Hub}"
                }
                sh 'docker push thiernos/sonar-image:latest'
            }
        }
        stage ('CreateContainer') {
            steps {
                sshagent(['ssh-conn']) {
                    ssh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.178 docker run -p 8000:80 -d --name docker-container thiernos/sonar-image:latest'
                }
            }
        }
    }
}