# Portfolio Deployment Using Jenkins CI/CD Pipeline

> This project automates the **deployment of my personal  Portfolio Website** using **Jenkins CI/CD** pipeline and **Apache2 web server** on a remote ubuntu server .

## Project Overview :

* **Goal :**  Automate build , test , and deployment of Portfolio Website  .

* **Pipeline :** Uses `Jenkinsfile` with SSH-based deployment .

* **Tools Used :** Jenkins , Github , Apache2 , SSh .

* **Trigger :** Github Webhook ( Auto-trigger build on push )

* **Notification :** Logs and console output in jenkins help track build process .

## Teck Stack :

- **Frontend**         : HTML , CSS JS (Portfolio Website)

- **CI/CD**            : Jenkins Pipeline (Pipeline-as-code) 

- **Webserver**        : Apache2 

- **Deployment**       : SSH + SCP to remote server  

- **Version Control**  : Git & Github 
 

## Environment Steup :

* ### Setup Jenkins Server 
 
  * Java installation :

  bash
 
      sudo apt update
      sudo apt install openjdk-17-jdk -y
      curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
      /usr/share/keyrings/jenkins-keyring.asc > /dev/null
      echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
      https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
      /etc/apt/sources.list.d/jenkins.list > /dev/null
      sudo apt update
      sudo apt install jenkins -y
      sudo systemctl enable jenkins
      sudo systemctl start jenkins

  
* ### Setup Remote Server 
    
    * Install Apache2 On Remote Server 

  bash 
        
      sudo apt update 

      sudo apt install apache2 -y  
       
      sudo systemctl start apache2

      sudo systemctl enable apache2 


## Setting Up Jenkins Credentials 

### Step 1 : Copy your pem key on jenkins server 

#### From your local machine : 
bash 

      scp -i your-key your-key ubuntu@<jenkins-server-public-ip>:/home/ubuntu


### Step 2 : Add Pem Key To Jenkins Credentials 

1. Go to settings --> credentials --> Under stored scoped to jenkins -> Click (global) --> Add credentials 

2. #### Fill In : 
   *  **Kind** : SSH Username with private key 
   
   *  **ID**   : `Portfolio-key-credentials` 
   
   *  **Username** : `Ubuntu`
   
   *  **private key** : Choose **Enter directly** and paste the contents of `key which store on jenkins server`

   *  **Click Create** 


## Create jenkins Pipeline Job

1. **Create Job** : Portfolio-deployment 

2. **Job type**  : Pipeline 

3. Under **Pipeline script from scm** : 
      * Choose : Git 

      * Connect to your github repo 

4. Add a jenkinsfile in your repo 
---
### Jenkinsfile for Remote Deployment

  bash


    pipeline {
    agent any

    environment {
        SSH_CRED =  Portfolio-key-credentials
        SERVER_IP = '54.196.240.222'
        REMOTE_USER = 'ubuntu'
        WEB_DIR = '/var/www/html'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/Nikhil-patil678/Nikhil-Portfolio.git', branch: 'main'
            }
        }

        stage('Deploy to Apache Server') {
            steps {
                sshagent(credentials: ["${SSH_CRED}"]) {
                    sh '''
                        echo "Cleaning old files on server..."
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} "sudo rm -rf ${WEB_DIR}/*"

                        echo "Uploading project files to /tmp..."
                        scp -r index.html assets ${REMOTE_USER}@${SERVER_IP}:/tmp/

                        echo "Verifying /tmp content..."
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} "ls -l /tmp"

                        echo "Deploying to Apache directory..."
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} "sudo cp -r /tmp/index.html /tmp/assets ${WEB_DIR}/"

                        echo "Restarting Apache to clear cache..."
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} "sudo systemctl restart apache2"

                        echo "✅ Deployment completed!"
                    '''
                }
            }
        }
    }
}

---

5. Build the job :

#### Console Output Showing Succeesful Deployment 

![my image](./Images/Screenshot%202025-12-05%20141856.png)

* ### Access URL : 

bash 

    http://<public-ip of remote-server>:3000


## Setup Github Hooks 

### Step 1 : Enable Github hook Trigger In Jenkins Job 

1. Open your Jenkins Job --> Configure 

2. Under Triggers Check : 
     *  `GitHub hook trigger for GITScm polling`

1. Go to github repo -->  **settings** -> **Webhooks** --> **Add Webhooks** --> 

2. Under Payload URL Enter  :

    * `http://< jenkins-server-ip >:8080/github-webhook/`

3.  Content Type : application/x-www-form-urlencoded 

4. Trigger : Just the push event

**Now every time you push changes to GitHub, Jenkins will automatically build and deploy your site.**

---
#### Once you run the Jenkins job (or push new code to trigger automatically), open a browser and visit:
bash
    
    http://54.221.183.57/
    
---


### You should see your portfolio website live. 🎉

![my image](./Images/Screenshot%202025-12-05%20142541.png)

![myimage](./Images/Screenshot%202025-12-05%20142613.png)

![my image](./Images/Screenshot%202025-12-05%20142624.png)

## Author : 

**NIKHIL PATIL**

📧 np3250072@gmail.com

🔗 **[Linkdein]** : https://www.linkedin.com/in/nikhil-patil-259132306/ 

🔗 **[Github]**  : https://github.com/Nikhil-patil678 

---









         