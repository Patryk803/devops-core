pipeline {
    agent {
        label 'ubuntucompose'
    }
    tools {
        maven "M3"
    }
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
    stages {
        stage('Clear running apps'){
            steps {
            sh """
            docker rm -f ${NAZWA_KONTENERA} || {
              echo "This container is not existing"
            } 
            """
            }
        }
        stage('Get Code'){
            steps {
                git branch: 'test_selenium', url: 'https://github.com/Patryk803/devops-core.git'
            }
        }
        stage('Build and Junit'){
            steps {
                sh "mvn clean install"
            }
        }
        stage('Build Docker image'){
            steps {
                sh "mvn package -Pdocker"
            }
        }
        stage('Run Docker app'){
            steps{
                sh "docker run -d -p 0.0.0.0:8080:8080 --name ${NAZWA_KONTENERA} ${IMAGE}:${VERSION}"
            }
        }
        stage('Test Selemium'){
            steps{
                sh "mvn test -Pselenium"
            }
        }
        stage('Deploy jar to artifactory'){
            steps{
                configFileProvider([configFile(fileId: '8f59e6da-6a5b-4473-a402-3e09e49b7c19', variable: 'maven_settings')]) {
                    sh "mvn -s $maven_settings deploy"
                }
            }
        }
    }
    post {
        always {
            sh "docker stop ${NAZWA_KONTENERA}"
            deleteDir()
        }
    }
}