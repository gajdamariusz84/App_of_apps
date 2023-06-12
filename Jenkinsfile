def backenndDockerTag=""
def frontendDockerTag=""
def frontendImage="magaj/frontend"
def backendImage="magaj/backend"
def dockerRegistry=""
def registryCredentials="docker_hub"

pipeline {
    agent {
        label 'agent'
    }
    tools {
        terraform 'Terraform'
    }

    stages {
        stage('Get Code') {
            steps {
            checkout scm
            }
        }
        stage('clean running containers') {
            steps {
            sh "docker rm -f frontend backend"
            }
        }
        stage('adjust version') {
            steps {
                script {
                    backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                    frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                    
                    currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
                }
            }
        }
        stage('deploy app') {
            steps {
                script {
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                        docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                            sh "docker-compose up -d"
                        }
                    }
                }
            }
        }
        stage('selenium tests') {
            steps {
                sh "pip3 install -r test/selenium/requirements.txt"
                sh "python3 -m pytest test/selenium/FrontendTest.py"
            }
        }
        stage('run terraform') {
            steps {
                dir('Terraform') {
                    git branch: 'master', url: 'https://github.com/gajdamariusz84/Terraform'
                    withAWS(credentials: 'AWS', region: 'us-east-1') {
                        sh 'terraform init && terraform apply -auto-approve -var-file="terraform.tfvars"'
                    }
                }
            }
        }
        stage('run ansible') {
            steps {
                script {
                    sh "ansible-galaxy install -r requirements.yml"
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                        ansiblePlaybook inventory: 'inventory', playbook: 'playbook.yml'
                    }
                }
            }
        }
    }

    post {
        always {
            withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                sh "docker-compose down"
                cleanWs()                        
            }
            
        }
    }
}
