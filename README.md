# TERRAFORM & AWS EKS
## 📦 Demo 3
This exercise is part of **Module 12**: **Terraform** in the Nana DevOps Bootcamp. This demo shows how to set up a complete CI/CD pipeline to automate Terraform infrastructure provisioning using Jenkins. The pipeline runs Terraform commands to plan and apply changes whenever there is an update in the infrastructure code repository.




## 📦 Demo 4

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
   [Terraform Install](https://developer.hashicorp.com/terraform/install)
   
   ```bash
         wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
         echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
         sudo apt update && sudo apt install terraform
   ```
   
### Terraform Configuration Files
1. Create a Terraform folder in the java-maven-app
2. Inside the terraform folder create main.tf file
3. Use the baseline configuration from demo1 to create the VPC.
4. Replace the previous key pair by the new pair created.
   ```bash
   key_name = "myapp-key-pair"
   ```
6. Add a new file in the terraform folder named entry-script.sh. Copy the script into this file which installs docker and docker compose in the EC2 instance.
7. Create a variables.tf file and add the variables
8. Add the default values to the variables.
9. Crate the output.tf file , and copy the outputs in this section.

### Modifying Jenkins File to Provision Server
1. Add a new stage called "provision server"
2. Define the steps and script
3. Add the terraform commands in the script section.
   <details><summary><strong>Terraform Commands</strong></summary>
       The terraform commands must be executed where the terraform files are located. In this casee, we first need to switch directories to run the commands.
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
4. Set the Environment Variables to enable Terraform to connect to AWS
   ```bash
       environment{
                  AWS_ACCESS_KEY_ID = credentials("jenkins_aws_access_key_id")
                  AWS_SECRET_ACCESS_KEY = credentials("jenkins-aws_secret_access_key")
                  TF_VAR_env_prefix = "test"
              }
   ```
   <details><summary><strong>AWS Credentials</strong></summary>
       Ensure the crendetials exist in Jenkins. In this case, the credentials already existed in Jenkins because we created them in a previous demo
    </details>
    <details><summary><strong>Terraform ENV Variables</strong></summary>
       the prefix TF_VAR_xxx allows terraform to recongnize ENV Varibles. In this case, we are overwriting the value of the env prefix variable from dev to test.
    </details>

### Modifying Jenkinsfile Deploy Stage

1. Add theSSH section to SSH to the EC2 instance: In this case, the IP address to access the EC2 is unknown until the instance is created using Terraform. Therfore, we must save the output of the public IP from terraform in a ENV varible so Jenkins can use it to SSH the EC2 instance dynamically.
2. In the provision server stage we must obtain the Publiic IP addreess of the EC2:
   Add the terraform output command to the provision server stage, where ec2_public_ip is the name of the terraform output.
   ```bash
   terraform output ec2_public_ip
   ```
3. Print the value to Stdout and \save it into an ENV variable in Jenkins
   ```bash
       EC2_PUBLIC_IP = sh(
             script: "terraform output ec2_public_ip",
             returnStdout: true
       ).trim()
   ```

   4. Now we can reference the Public ip inside the deploy stage
  
      ```bash
      
             stage("deploy") {
      
                  steps{
      
                      script{
      
                          //Adding waiting period to give time to the EC2 and script to be ready to execute commands, when the server is created the 1st time      
                          echo "Waiting for EC2 to initialize..."
                          sleep(time: 120, unit: "SECONDS")
      
                          echo "deploying docker image to EC2"
                          echo "${EC2_PUBLIC_IP}"
                          
                          def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME} ${DOCKER_CREDENTIALS_USR} ${DOCKER_CREDENTIALS_PSW}"
                          def ec2Instance = "ec2-user@${EC2_PUBLIC_IP}"
      
                          sshagent(['server-ssh-key']){
                              //StrictHostKeyChecking=no turnoff strict host checking when connecting to ec2
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

      ### Allowing Jenkins to SSH AWS
      1.  Add the jenkins ip to the terraform variables files.
      ```bash
            variable jenkins_ip {
                default = "198.199.70.18/32"
            }  
      ```
      6. Modify the Ingress rules in the main.tf file to allow Jenkins to SSH AWS
         ```bash
             ingress {
                from_port = 22
                to_port = 22
                protocol = "tcp"
                cidr_blocks = [var.my_ip, var.jenkins_ip, var.my_ip_home]
            }
         ```
         7. Using environment block to read values:
            ```bash
                environment {
                    DOCKER_CREDENTIALS = credentials("docker-hub-repo")
                    //This docker crendetials (username & password) contains the whole object username and password, but it automatically creates the password and username reference out of the box
                    //DOCKER_CREDENTIALS_USR  --> username
                    //DOCKER_CREDENTIALS_PSW --> password
                }
            
            ```

       ### Modifyng the Server-cmds.sh
      1. Adding docker login to the server-cmds script. The EC2 needs to authenticate 
         ```bash
             #Env Variables
              export IMAGE=$1
              export DOCKER_USER=$2
              export DOCKER_PWD=$3

             #Passing values to docker login 
              echo $DOCKER_PWD | docker login -u $DOCKER_USER --password-stdin
              echo "Docker Image-2: $IMAGE"
              docker-compose -f docker-compose.yaml up --detach
              echo "success"
         ```


      ### Git
   1. commit changes to the ssh-agent branch
      ```bash
        git add .
        git commit -m "CI/CD pipeline"
        git push
      ```
   3. Run the pipeline
   4. SSH to the EC2 using the PEM file and the public ip address
      ```bash
      ssh -i ec2-user@
   6. Verify that the java-maven-app and the postgres containers are running in the EC2
   
## 📦 Demo 4
This This demo project shows how to configure a shared remote storage for Terraform using Amazon S3. By centralizing state, teams can safely collaborate on infrastructure code and avoid conflicts caused by local .tfstate files.
## 📌 Objective
- Configure amazon S3 as remote storage for terraform state.

## Project Configuration
In the previous demo, the tfstate was saved in the jenkins serer howver, this does not allow to work collaborate among different team memebers. To share the state, the best way is to confiigure a remote terraform state file where it will be stored.
1. Inside the main.tf add the terraform block which allows to configure metadata about terraform. Define the required version and the remote backed.
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
2. Go to S3 service in your AWS console
3. Create the S3 Bucket in your AWS console. Using the same name previosly defined, the name must be unique fo the bucket
4. Enable versioning
5. Select the encyption type SSE-S3
6. Disanle Buckey key
7. Commit the changes and push
8. Execute pipeline
   <details><summary><strong>Migrating local state</strong></summary>
       If there is already a local state, then you should do terraform init and confirm the new backend manually. For demo purposed, the infrasturture here was deleted and recreate it.
    </details>

9. Check the new state in the bucket
10. Check the resoures created using terraform state list
   
