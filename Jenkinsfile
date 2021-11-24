pipeline {
    agent {
        label 'ubuntucompose'
    }
    tools {
        maven "M3"
        terraform "terraform"
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
        stage('Run terraform') {
            steps {
                dir('infrastructure/terraform') { 
                withCredentials([file(credentialsId: 'panda_ansible', variable: 'panda_ansible')]) {
                    sh "cp \$panda_ansible ../panda_ansible.pem"
                }
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred']]) {
                    sh 'terraform init && terraform apply -auto-approve -var-file panda_ansible.tfvars'
                }
            }
        }
        }
        stage('Copy Ansible role') {
            steps {
                sh 'sleep 180'
                sh 'cp -r infrastructure/ansible/panda/ /etc/ansible/roles/'
            }
        }
        stage('Run Ansible') {
            steps {
                dir('infrastructure/ansible') { 
                    sh 'chmod 600 ../panda_ansible.pem'
                    sh 'ansible-playbook -i ./inventory playbook.yml -e ansible_python_interpreter=/usr/bin/python3'
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