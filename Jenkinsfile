pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                git credentialsId: 'github-pat', branch: 'main', url: 'https://github.com/cyse7125-su24-team10/helm-k8s-operator.git'
            }
        }
        stage('helm lint') {
            steps {
                script {
                    sh "helm lint ./"
                    sh "helm template ./"
                }
            }
        }
        stage('release') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-pat', usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_TOKEN')]) {
                    script {
                        sh "npm install @semantic-release/changelog"
                        sh "npm install @semantic-release/git"
                        sh "npm install @semantic-release/github@10.1.1"
                        sh "npm install @semantic-release/commit-analyzer"
                        sh "npm install @semantic-release/release-notes-generator"
                        sh "npm install semantic-release-helm"
                        sh "npm install @semantic-release/npm"
                        sh "npx semantic-release"
                    }
                }
            }
        }
    }
}