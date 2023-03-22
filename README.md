# CICD_Jenkins_JavaSpringBoot_Docker_Kubernetes_AWS_EKS


[ CI/CD, Jenkins, Kubernetes, Docker, EKS, ECR ] CI/CD Jenkins pipeline to deploy a Dockerized Java SpringBoot app  to EKS cluster

(the page is under development)
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

![unlock jenkins](https://user-images.githubusercontent.com/104728608/226752568-5b413ec3-9b8a-4e3a-bbd5-3e765cca0f45.png)

<br>

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword #unlock Jenkins
```
<br><br>

![customize jenkins](https://user-images.githubusercontent.com/104728608/226751554-53623217-49f9-4bd0-93e4-c69c11511e64.png)

![image](https://user-images.githubusercontent.com/104728608/226751493-9cddbd80-0545-4f68-9e67-6ee22a43a71d.png)

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

<img width="1267" alt="Screenshot 2023-03-21 at 21 45 41" src="https://user-images.githubusercontent.com/104728608/226751995-746547c8-d515-42d7-9b47-cd2f5a275a3d.png">

restart services
```
sudo usermod -a -G docker jenkins
sudo service jenkins restart
sudo systemctl daemon-reload
sudo service docker restart
```

<br><br>

## 2. Create EKS cluster

Install the AWS CLI version 2 on EC2

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 

sudo apt install unzip

sudo unzip awscliv2.zip  

sudo ./aws/install

aws --version
```

Install eksctl on EC2 Instance

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin

eksctl version
```

Install kubectl on EC2 Instance

```
sudo curl --silent --location -o /usr/local/bin/kubectl   https://s3.us-west-2.amazonaws.com/amazon-eks/1.22.6/2022-03-09/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl 

kubectl version --short --client
```
Create IAM Role with Administrator Access 

<br>
<img width="700" alt="Screenshot 2023-03-22 at 08 30 45" src="https://user-images.githubusercontent.com/104728608/226843952-8aa6336f-4dfa-4d1b-9f00-caa9fac1e53d.png">

<img width="700" alt="Screenshot 2023-03-22 at 08 40 16" src="https://user-images.githubusercontent.com/104728608/226847236-76b88323-f6f2-45b0-b360-bab57292de6c.png">


<br>

```
sudo su - jenkins
```
```
eksctl create cluster --name demo-eks --region us-east-1 --nodegroup-name my-nodes --node-type t3.small --managed --nodes 2 

eksctl get cluster --name demo-eks --region us-east-1

aws eks update-kubeconfig --name demo-eks --region us-east-1

cat  /var/lib/jenkins/.kube/config
```
