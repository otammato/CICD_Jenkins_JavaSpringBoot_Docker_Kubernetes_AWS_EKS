# CICD_Jenkins_JavaSpringBoot_Docker_Kubernetes_AWS_EKS


[ CI/CD, Jenkins, Kubernetes, Docker, EKS, ECR ] CI/CD Jenkins pipeline to deploy a Dockerized Java SpringBoot app  to EKS cluster

<br><br>

## Result:

<br>

<img width="1456" alt="Screenshot 2023-03-21 at 15 33 11" src="https://user-images.githubusercontent.com/104728608/226737690-be74ad69-6157-4f69-864b-cb31cce467b4.png">

<br><br><br>


<details markdown=1><summary markdown="span">Jenkins file</summary>

```
pipeline {
    agent any
    
    tools {
        maven 'Maven3'
    }
    
    environment {
        registry = "public.ecr.aws/y2b7b1f3/springboot_app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/otammato/CICD_Jenkins_JavaSpringBoot_Docker_Kubernetes_AWS_EKS.git']])
            }
        }
        
        stage('Build Jar') {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage('Build Image') {
            steps {
                script {
                    docker.build registry
                }
            }
        }
        
        stage('Pushing to ECR') {
            steps{  
                script {
                    sh 'aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/y2b7b1f3'
                    sh 'docker push public.ecr.aws/y2b7b1f3/springboot_app:latest'
                }
            }
        }
        
        stage('K8s Deploy') {
            steps{
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                    sh "kubectl apply -f eks-deploy-k8s.yaml"
                }
            }
        }
    }
}

```

</details>
