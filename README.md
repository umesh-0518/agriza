pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/umesh-0518/agriza.git'
        GIT_BRANCH = 'Testing'  
        GIT_CREDENTIALS = 'github-access'
    }

    stages {
        stage('Pull Code') {
            steps {
                git branch: "${GIT_BRANCH}", credentialsId: "${GIT_CREDENTIALS}", url: "${GIT_REPO}"
            }
        }

        stage('Make Change') {
            steps {
                sh 'echo "// Updated by Jenkins on $(date)" >> README.md'
            }
        }

        stage('Push Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS}", usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh """
                        git config user.name "umesh-0518"
                        git config user.email "uraghuwanshi25@gmail.com"
                        git add README.md
                        git commit -m "Auto update from Jenkins" || echo "No changes to commit"
                        git remote set-url origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/umesh-0518/agriza.git
                        git push origin ${GIT_BRANCH}
                    """
                }
            }
        }

        stage('Deploy to Testing Server') {
            steps {
                sshagent(['ec2-user']) {
                    sh '''
                       ssh -o StrictHostKeyChecking=no ec2-user@54.169.151.86 '
                       cd /home/ec2-user/frontend/backend &&
                       git pull origin Testing &&
                       npm install --legacy-peer-deps &&
                       pm2 describe app || pm2 start app.js --name app &&  ##[Dev & Test on different servers	pm2 start app.js --name app]
                                                                              [Dev & Test on same server	pm2 start app.js --name dev-app/test-app]
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Code pushed and deployed successfully!"
        }
        failure {
            echo " Build failed. Please check logs."
        }
    }
}


##  This Jenkins pipeline automates the CI/CD process for the `Testing` environment.  
It performs the following:

1. Pulls code from the `Testing` branch of GitHub.
2. Makes an automatic change to simulate a CI update.
3. Pushes the change back to the `Testing` branch.
4. Deploys the application on an AWS EC2 instance using SSH and PM2.
##  Technologies Used
- Jenkins
- GitHub
- AWS EC2 (Amazon Linux)
- Node.js / npm
- PM2
- SSH Agent Plugin
- Shell scripting
## Jenkins Pipeline Stages
###  Pull Code from GitHub
```groovy
git branch: "${GIT_BRANCH}", credentialsId: "${GIT_CREDENTIALS}", url: "${GIT_REPO}"
Pulls the latest code from the Testing branch of the frontend repository.
2️⃣ Make Change in Code
bash
Copy
Edit
echo "// Updated by Jenkins on $(date)" >> README.md
Appends a line to the README.md file to simulate a change.
3️⃣ Push Code to GitHub
bash
Copy
Edit
git config user.name "umesh-0518"
git config user.email "uraghuwanshi25@gmail.com"
git add README.md
git commit -m "Auto update from Jenkins" || echo "No changes to commit"
git remote set-url origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/umesh-0518/agriza.git
git push origin ${GIT_BRANCH}
Pushes the change to the remote Testing branch using stored credentials.
4️⃣ Deploy to EC2 Server
bash
Copy
Edit
ssh -o StrictHostKeyChecking=no ec2-user@<EC2-IP> '
cd /home/ec2-user/frontend/backend &&
git pull origin Testing &&
npm install --legacy-peer-deps &&
pm2 describe app || pm2 start app.js --name app &&
pm2 restart app
'
Remotely deploys the latest code on the EC2 instance and restarts the application using PM2.
# Jenkins Credentials Used
ID	Description
github-access	GitHub username & token
ec2-user	SSH key to access EC2
# EC2 Configuration
OS: Amazon Linux
Ports Opened: 3000 (for Node.js app), 22 (for SSH)
Directory path: /home/ec2-user/frontend/backend
📎 Notes
## Webhook is configured in GitHub to automatically trigger Jenkins when a push is made to the Testing branch.
 ## PM2 is used to keep the Node.js app running persistently.

Pipeline will show a ✅ message on successful execution.


