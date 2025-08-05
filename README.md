# TERRAFORM & AWS EKS
## 📦 Demo 3
This exercise is part of **Module 12**: **Terraform** in the Nana DevOps Bootcamp. This This demo project shows how to configure a shared remote storage for Terraform using Amazon S3. By centralizing state, teams can safely collaborate on infrastructure code and avoid conflicts caused by local .tfstate files.
## 📌 Objective
- Configure amazon S3 as remote storage for terraform state.


## 🚀 Technologies Used
- **Terraform**: Infrastructure as Code Tool for managing cloud resources.
- **AWS**: Cloud Provider
- **EC2**: Instance on AWS
- **VPC**: Virtual Private Cloud for networking.
- **EKS**: Manage Kubernetes cluster.
- **Docker**: Container
- **Git**: Version Control
- **S3**: AWS storage service.
  
  
   
## 📋 Prerequisites
- Ensure you have an AWS Account.
- You have done the previous Terraform demo.
- You have done module 8 demos.
- Ensure you have the Java Maven App from Module 8
- SSH agent plugin is installed
  
## 🎯 Features
- Creating a New Branch for Java-Maven-App project
- Creating SSH-Key pair.
- Installing Terraform inside Jenkins container.
- Create the provision server stage.
- Adjust Jenkinsfile
  
       
## 🏗 Project Architecture



## ⚙️ Project Configuration
### Creating  New Branch
1. Open the Java-Maven-App application from Module 8 used to build the CI/CD pipeline.
2. Create a New branch called jenkinsfile-sshagent

### Creating SSH Key-Pair
1. Go to AWS console and create a new Key-Pair, named myapp-key-pair
2. Select the Key pair type RSA
3. Select the .pem format and create Key pair.
4. Go To Jenkins and create a new credentials in the multibranch pipeline.
5. Select kind as SSH Username with private key
   * Select ID: server-ssh-key
   * Username: ec2-user
   * Select enter directly and Copy and Paste the content of the PEM file in the key section.
     
### Install Terraform Inside the Jenkins Container
1. SSH the DigitalOcean droplet.
   ```bash
     ssh root@198.199.70.18
   ```
3. Check the container ID
   ```bash
   docker ps
   ```
5. Access the jenkins container as Root user.
   ```bash
     docker exec -it -u 0 <> bash
   ```
 6. Check the OS
    ```bash
      cat /etc/os-release    
    ```
7. Install terraform according your OS and Terraform guidelines
   [Terraform Instal](https://developer.hashicorp.com/terraform/install)
   ```bash
     wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
     echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
     sudo apt update && sudo apt install terraform
   ```
   
### Terraform Configuration Files
1. Create a Terraform folder in the java-maven-app
2. Inside the terraform folder create main.tf file
3. Use the baseline configuration for creating a VPC from demo1
4. Replace the previous key pair by the new pair created.
   ```bash
   key_name = "myapp-key-pair"
   ```
6. Add a new file in the terraform folder named entry-script.sh. Copy the script into this file which installs docker and docker compose in the EC2 instance.
7. Add default values to the variables
8. 

### Creating a S3 Bucket on AWS
