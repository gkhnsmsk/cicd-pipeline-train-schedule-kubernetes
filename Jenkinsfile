//5th try
pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "gsimsek/train-schedule-kubernetes"
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
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    //app = docker.build(docker_image_name)
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
                        //app.push("${env.build_number}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('echo') {
            steps {
            sh "echo $DOCKER_IMAGE_NAME:$BUILD_NUMBER"    
            }
        }
        stage('Update Kube Config') {
            when {
                branch 'master'
            }
            steps {
                withAWS(region:'eu-central-1',credentials:'aws') {
                    sh 'aws eks --region eu-central-1 update-kubeconfig --name eksworkshop-eksctl'       
                }
            }
        }
        stage('Deploy Updated Image to Cluster'){
            steps {
                sh '''
                    export IMAGE="$DOCKER_IMAGE_NAME:$BUILD_NUMBER"
                    kubectl apply -f train-schedule-kube.yml
                    '''
            }
        }        
    }
}

//                input 'Deploy to Production?'
//                milestone(1)
//                withKubeConfig([credentialsId: 'kubeconfig',serverUrl: 'https://E5624AA4631FF2516F793E316BA8889E.gr7.eu-central-1.eks.amazonaws.com']) {
//                sh 'kubectl get pods'

//                sshagent(['kubemaster_username_privatekey']){
//                    sh "scp -o StrictHostKeyChecking=no train-schedule-kube.yml ubuntu@35.158.92.60:/home/ubuntu"
//                    script{
//                        try{
//                            sh "ssh ubuntu@35.158.92.60 kubectl apply -f train-schedule-kube.yml"
//                        }catch(error){
//                            sh "ssh ubuntu@35.158.92.60 kubectl create -f train-schedule-kube.yml"
//                        }
