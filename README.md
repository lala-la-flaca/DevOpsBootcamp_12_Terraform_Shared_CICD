# TERRAFORM & AWS EKS
## 📦 Demo 3
This exercise is part of **Module 12**: **Terraform** in the Nana DevOps Bootcamp. This demo shows how to set up a complete CI/CD pipeline to automate Terraform infrastructure provisioning using Jenkins. The pipeline runs Terraform commands to plan and apply changes whenever there is an update in the infrastructure code repository.

Git Repo:
[Demo3_&_4_Repo](https://gitlab.com/devopsbootcamp4095512/devopsbootcamp_8_jenkins_pipeline/-/tree/jenkinsfile-sshagent/java-maven-app?ref_type=heads)

## 📌 Objective
* Integrate Terraform workflows into a CI/CD pipeline.
* Use a remote backend to store Terraform state.

## 🚀 Technologies Used
- **Terraform**: Infrastructure as Code Tool for managing cloud resources.
- **AWS**: Cloud Provider
- **EC2**: Instance on AWS
- **VPC**: Virtual Private Cloud for networking.
- **EKS**: Manage Kubernetes cluster.
- **Docker**: Container
- **Git**: Version Control
- **S3**: AWS storage service.
- **Jenkins**: CI/CD automation server to run pipelines
  
  
   
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
1. Open the Java-Maven-App application from Module 8, used to build the CI/CD pipeline.
2. Create a New branch called jenkinsfile-sshagent
   ```bash
     git checkout -b jenkinsfile-sshagent
   ```

### Creating SSH Key-Pair
1. In the AWS Management Console, create a new key pair named myapp-key-pair
   
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/2%20ccreating%20key%20pair%20my%20app.png" width=800/>
   
3. Select Key pair type as RSA
4. Select .pem format and create the key pair.
5. In Jenkins, create a new credential in the Multibranch Pipeline configuration.
6. Set the credential type to SSH Username with private key, then configure as follows:
   * Select ID: server-ssh-key
   * Username: ec2-user
   * Private key: Select enter directly and paste the content of the PEM file.

   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/3%20add%20private%20key%20to%20jenkins%20creentials.png" width=800/>
     
### Install Terraform Inside the Jenkins Container
1. SSH into DigitalOcean droplet.
   ```bash
     ssh root@198.199.70.18
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/4%20ssh%20to%20jenkins%20server%20DO%20droplet.png" width=800/>
   
2. Check the container ID
   ```bash
   docker ps
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/5%20insise%20jenkins%20docker%20container.png" width=800/>
   
3. Access the Jenkins container as the root user.
   ```bash
     docker exec -it -u 0 <container_id> bash
   ```
4. Verify the operating system.
    ```bash
      cat /etc/os-release    
    ```
    <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/6%20checking%20what%20os%20we%20have%20%20availabel.png" width=800/>
    
5. Install Terraform:
   Follow the Terraform installation guide for your OS.
   [Terraform Install](https://developer.hashicorp.com/terraform/install)
   For Ubuntu/Debian-based systems:
   ```bash
         wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
         echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
         sudo apt update && sudo apt install terraform
   ```
6. Check that terrform is installed
   ```bash
   terraform -v
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/7%20terraform%20instlled%20i%20the%20container.png" width=800/>
   
### Terraform Configuration Files
1. In the java-maven-app directory, create a folder named terraform. 
2. Inside the Terraform folder, create a main.tf file.
3. Use the baseline configuration from Demo 1 to create the VPC.
4. Replace the previous key pair with the new one:
   ```bash
   key_name = "myapp-key-pair"
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/8%20copy%20the%20basic%20ec2%20configuration%20from%20the%20previous%20demo%20and%20update%20the%20keypair%20section%20to%20use%20the%20nrw%20key%20created.png" width=800/>
   
6. Create a file named entry-script.sh in the terraform folder and add the script that installs Docker and Docker Compose on the EC2 instance.
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/8%20inside%20terraform%20foldeer%20created%20the%20user%20data%20scriot.png" width=800/>
   
8. Create a variables.tf file and define the required variables.
9. Add default values for the variables in variables.tf
10. Create an outputs.tf file and add the required output definitions.

### Modifying Jenkins File to Provision Server
1. Add a new stage called "provision server"
   
2. Define the steps and scripts for this stage
   
3. Add Terraform commands inside the script section.
   
   <details><summary><strong>Terraform Commands</strong></summary>
       Terraform commands must be executed from the directory containing the Terraform configuration files
    </details>
    
    ```bash
            stage("provision server"){
                    //Terraform provision server
                    steps {
                        script {
                            dir("java-maven-app/terraform"){   
                                    sh '''
                                        terraform init 
                                        terraform apply --auto-approve
                                    '''
                                
                                
                            }
                        }
                    }            
                }
            
    ```
6. Set environment variables to allow Terraform to connect to AWS
   Add the environment block inside the provision server stage:
   ```bash
       environment{
                  AWS_ACCESS_KEY_ID = credentials("jenkins_aws_access_key_id")
                  AWS_SECRET_ACCESS_KEY = credentials("jenkins-aws_secret_access_key")
                  TF_VAR_env_prefix = "test"
              }
   ```
   <details><summary><strong>AWS Credentials</strong></summary>
       Ensure the credentials exist in Jenkins. In this example, the credentials already exist because they were created in a previous demo
    </details>
    <details><summary><strong>Terraform ENV Variables</strong></summary>
       The prefix `TF_VAR_` allows Terraform to recognize environment variables. In this example, the `env_prefix` variable is overwritten from `dev` to `test`
    </details>

  <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/10%20provision%20server%20env%20variable%20to%20overwrite%20dev.png" width=800/>
  
### Modifying Jenkinsfile Deploy Stage

1. Enabling dynamic SSH access to the EC2 instance:
   When the EC2 instance is created with Terraform, its public IP address is not known in advance.To enable Jenkins to connect via SSH dynamically, store the Terraform output for the public IP in an environment variable.
  
2. Retrieving  the EC2 public IP in the provision server stage
   Run the following command to get the public IP. In this example, ec2_public_ip is the name of the Terraform output:
   ```bash
     terraform output ec2_public_ip
   ```
3. Saving the EC2 public IP in a Jenkins environment variable.
   Store the output so it can be used later in the pipeline:
   ```bash
       EC2_PUBLIC_IP = sh(
             script: "terraform output ec2_public_ip",
             returnStdout: true
       ).trim()
   ```
  <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/11%20gettig%20the%20public%20ic%20to%20save%20it%20as%20a%20env%20variable%20and%20use%20it%20in%20the%20deploy%20stage.png" width=800/>
5. Referencing the public IP in the deploy stage
  
      ```bash      
             stage("deploy") {      
                  steps{      
                      script{      
                          // Wait for EC2 initialization and script readiness
                          echo "Waiting for EC2 to initialize..."
                          sleep(time: 120, unit: "SECONDS")
      
                          echo "deploying docker image to EC2"
                          echo "${EC2_PUBLIC_IP}"
                          
                          def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME} ${DOCKER_CREDENTIALS_USR} ${DOCKER_CREDENTIALS_PSW}"
                          def ec2Instance = "ec2-user@${EC2_PUBLIC_IP}"
      
                          sshagent(['server-ssh-key']){
                              //StrictHostKeyChecking=no --> Disable strict host key checking when connecting to EC2
                              dir("java-maven-app"){
                                  echo "Inside java-maven-app directory to copy files to ec2:"
                                  sh "ls"
      
                                  echo "Copying files to EC2:"
                                  sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                                  sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"

                                  echo "SSH & Running Script:"
                                  sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                              }
                          }                     
                      }
                  }
              }
### Allowing Jenkins to SSH into AWS
1.  Add the Jenkins IP address to Terraform variables
      ```bash
            variable jenkins_ip {
                default = "198.199.70.18/32"
            }  
      ```
    
2. Update the security group ingress rules in main.tf
   ```bash
             ingress {
                from_port = 22
                to_port = 22
                protocol = "tcp"
                cidr_blocks = [var.my_ip, var.jenkins_ip, var.my_ip_home]
            }
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/12%20adding%20jenkins%20ip%20address%20to%20be%20able%20to%20ssh%20ec2%20security%20group.png" width=800/>
   
3. Use the environment block to store Docker Hub credentials
   ```bash
                environment {
                    DOCKER_CREDENTIALS = credentials("docker-hub-repo")
                    //// Automatically provides:
                    //DOCKER_CREDENTIALS_USR  --> username
                    //DOCKER_CREDENTIALS_PSW --> password
                }
            
    ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/13%20adding%20credentials%20docker.png" width=800/>

  ### Modifiyng the Server-cmds.sh Script
  1. Adding Docker login commands to the script so the EC2 instance can authenticate with Docker Hub.
     
  2. Passing the image name, Docker username, and password as script arguments.
     ```bash
         #Env Variables
         export IMAGE=$1
         export DOCKER_USER=$2
         export DOCKER_PWD=$3
    
         #Authenticate with Docker Hub
         echo $DOCKER_PWD | docker login -u $DOCKER_USER --password-stdin
    
         # Deploy the application 
         echo "Docker Image-2: $IMAGE"
         docker-compose -f docker-compose.yaml up --detach
         echo "Deployment successful"
     ```
### Git Operations
1. Commit changes to the ssh-agent branch:
   ```bash
        git add .
        git commit -m "CI/CD pipeline"
        git push
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/14%20pushing%20chnages%20to%20git.png" width=800/>
   
2. Run the Jenkins pipeline.
   
3. SSH into the EC2 instance using the .pem file and public IP address:
   
    ```bash
        ssh -i myapp-key-pair.pem ec2-user@<EC2_PUBLIC_IP>
    ```
    
4. Verify that the java-maven-app and postgres containers are running:
    ```bash
      docker ps
    ```
    <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/15%20docker%20containers%20up%20and%20running.png" width=800/>

  
## 📦 Demo 4
This This demo project shows how to configure a shared remote storage for Terraform using Amazon S3. By centralizing state, teams can safely collaborate on infrastructure code and avoid conflicts caused by local .tfstate files.
## 📌 Objective
- Configure amazon S3 as remote storage for terraform state.

## Remote State Using an S3 Backend
In the previous demo, the tfstate file was stored on the Jenkins server. This approach does not support collaboration among multiple team members. To share the state securely and enable collaboration, configure a remote Terraform state backend using Amazon S3.

1. Configure the S3 backend in main.tfInside the main.tf:
   
   Add the terraform block to define the required Terraform version and S3 backend configuration:
   
   ```bash
     terraform {
        required_version = ">= 0.12"
        //backend config
        backend "s3" {
           bucket = "myapp-tf-s3-bucket-aws"
           key = "myapp/state.tfstate"
           region = "us-east-2"

         }
      }
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/16%20backend%20remote%20state%20config.png" width=800/>
   
2. Create the S3 bucket in AWS:
    In the AWS Management Console, go to the S3 service.
    Create a new bucket with the same name defined in the backend configuration (myapp-tf-s3-bucket-aws).
    Note: Bucket names must be unique across all AWS accounts.
    Enable Versioning.
    Set the encryption type to SSE-S3.
    Disable Bucket Key.

   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/18%20create%20the%20s3%20bucket.png" width=800/>

4. Commit and push changes to git repository.
    ```bash
      git add .
      git commit -m "Configure remote state with S3 backend"
      git push
    ```
5. Execute pipeline
   <details><summary><strong>Migrating local state</strong></summary>
        If a local state file already exists, run `terraform init` and confirm the new backend when prompted. For this demo, the local infrastructure was deleted and recreated for simplicity.
    </details>

6. Verify the remote state:
    * In the S3 console, confirm that the state.tfstate file exists in the bucket.
      <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/21%20file%20in%20s3%20bucket.png" width=800/>
    * List resources managed by Terraform:
      <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_12_Terraform_Shared_CICD/blob/main/Img/terraform%20list%20comming%20from%20s3%20bucket.png" width=800/>

   
