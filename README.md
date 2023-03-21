# CICD_Jenkins_JavaSpringBoot_Docker_Kubernetes_AWS_EKS


[ CI/CD, Jenkins, Kubernetes, Docker, EKS, ECR ] CI/CD Jenkins pipeline to deploy a Dockerized Java SpringBoot app  to EKS cluster

<br>

## Result:

<br>

<img width="1456" alt="Screenshot 2023-03-21 at 15 33 11" src="https://user-images.githubusercontent.com/104728608/226737690-be74ad69-6157-4f69-864b-cb31cce467b4.png">

<img width="1346" alt="Screenshot 2023-03-21 at 15 22 02" src="https://user-images.githubusercontent.com/104728608/226742394-82127049-ee83-40eb-bd56-3910e2d0b4ea.png">

<br><br>


<details markdown=1><summary markdown="span">Jenkins file</summary>

``` yml
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

---

<br><br>

## 1. Install Jenkins on an EC2 instance

```
sudo hostnamectl set-hostname Jenkins
sudo apt update
sudo apt install default-jdk -y  #install Java 11

sudo apt install maven -y #instal Maven
```
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt update
sudo apt install jenkins -y
```
<br>

Now go to browser. enter public dns name or public IP address with port no 8080.<br>
http://EC2_public_dns_name:8080

<br><br>

## 2. Install Docker on an EC2 instance

```
sudo apt-get update
sudo apt install gnupg2 pass -y
sudo apt install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

<br><br>
install Docker plugins to Jenkins

<img width="1281" alt="Screenshot 2023-03-21 at 21 42 16" src="https://user-images.githubusercontent.com/104728608/226748027-f5bff7fa-84fa-41ed-a3b6-b87acf998dfb.png">

<img width="1267" alt="Screenshot 2023-03-21 at 21 45 41" src="https://user-images.githubusercontent.com/104728608/226748774-e182dafa-a5a7-40d0-a1cc-1c940f9cce81.png">

restart services
```
sudo usermod -a -G docker jenkins
sudo service jenkins restart
sudo systemctl daemon-reload
sudo service docker restart
```
