//5th try
pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "gitlab.lrz.de:5005/shortcut/tools/shortcut.lab/app_image"
        //REGISTRY = "gitlab.lrz.de:5005"
        //registryCredential = "gitlab_token_for_EKS_pull"
        //docker_image_name = "gsimsek/train-schedule-kubernetes"
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
                branch 'gokhan_test'
            }
            steps {
                script {
                    // app = docker.build(DOCKER_IMAGE_NAME)
                    app = docker.build("$DOCKER_IMAGE_NAME:${env.BUILD_ID}")
                    sh 'echo $app'
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'gokhan_test'
            }
            steps {
                script {
                    //docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                    docker.withRegistry('https://gitlab.lrz.de:5005/shortcut/tools/shortcut.lab', 'gitlab_token_for_EKS_pull') {
                        //app.push("${env.BUILD_NUMBER}")
                        app.push()
                        //app.push("latest")
                    }
                }
            }
        }
        stage('echo') {
            steps {
            sh "echo $DOCKER_IMAGE_NAME:$BUILD_NUMBER"    
            }
        }
        /*
        stage('Update Kube Config') {
            when {
                branch 'gokhan_test'
            }
            steps {
                withAWS(region:'eu-central-1',credentials:'aws_credentials') {
                    sh 'aws eks --region eu-central-1 update-kubeconfig --name eksworkshop-eksctl'       
                }
            }
        }
        */
        stage('Deploy Updated Image to Cluster'){
            steps {
                sh '''
                    kubectl apply -f ./train-schedule-kube.yml
                    kubectl get pods
                    '''
            }
        }        
    }
}
