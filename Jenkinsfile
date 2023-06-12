def backenndDockerTag=""
def frontendDockerTag=""
def frontendImage="magaj/frontend"
def backendImage="magaj/backend"
def dockerRegistry=""
def registryCredentials="docker_hub"

pipeline {
    agent {
        lablel 'agent'
    }
    tools {
        terraform 'Terraform'
    }
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
            script{
                backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                
                currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
            }
        }
    }
    stage('deploy app') {
        steps {
            script{
                withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        sh "docker-compose up -d"
                    }
                }
            }
        }
    }
}
