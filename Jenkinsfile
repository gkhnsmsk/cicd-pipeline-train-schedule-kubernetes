//5th try
pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "gsimsek/train-schedule-kubernetes"
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
                branch 'master'
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
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                sh "scp -o StrictHostKeyChecking=no kubeconfig.yml ubuntu@35.158.92.60:/home/ubuntu"
                script{
                    try{
                        sh "ssh ubuntu@35.158.92.60 kubectl apply -f ."
                    }catch(error){
                        sh "ssh ubuntu@35.158.92.60 kubectl create -f ."
                    }
//                kubernetesDeploy(
//                    kubeconfigId: 'kubeconfig',
//                    configs: 'train-schedule-kube.yml',
//                    enableConfigSubstitution: true
//                )
                }
            }
        }
    }
}
