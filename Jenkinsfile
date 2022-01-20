Skip to content
Search or jump to…
Pull requests
Issues
Marketplace
Explore
 
@radleap 
radleap
/
cicd-pipeline-train-schedule-autodeploy
Public
forked from bhavukm/cicd-pipeline-train-schedule-autodeploy
Code
Pull requests
Actions
Projects
Wiki
Security
Insights
Settings
cicd-pipeline-train-schedule-autodeploy/Jenkinsfile
@radleap
radleap adding docker tag :latest
Latest commit 39ffa74 19 minutes ago
 History
 3 contributors
@radleap@whboyd@bhavukm
87 lines (87 sloc)  2.6 KB
   
pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "radleap/train-schedule:latest"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                expression {
                    return env.GIT_BRANCH == "origin/master"
                }
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                expression {
                    return env.GIT_BRANCH == "origin/master"
                }
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                expression {
                    return env.GIT_BRANCH == "origin/master"
                }
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            when {
                expression {
                    return env.GIT_BRANCH == "origin/master"
                }
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}