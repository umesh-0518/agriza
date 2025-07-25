pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/umesh-0518/agriza.git'
        GIT_BRANCH = 'Development'  
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

        stage('Deploy to Development Server') {
            steps {
                sshagent(['ec2-user']) {
                    sh '''
                       ssh -o StrictHostKeyChecking=no ec2-user@54.169.151.86 '
                       cd /home/ec2-user/frontend/backend &&
                       git pull origin Development &&
                       npm install --legacy-peer-deps &&
                       pm2 describe app || pm2 start app.js --name app &&
                       pm2 restart app
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Code pushed and deployed successfully!"
        }
        failure {
            echo "❌ Build failed. Please check logs."
        }
    }
}

## To set up and configure an automated CI/CD pipeline for two GitHub repositories (frontend and backend) based on a Git branching strategy. The pipeline should automatically deploy code to:

Development Environment when pushed to the Development branch

Testing Environment when pushed to the Testing branch

✅ What Has Been Implemented
1. Jenkins Setup
Installed and configured Jenkins on an EC2 Amazon Linux instance.

Installed required plugins: Git, SSH Agent, NodeJS, and Pipeline.

2. GitHub Webhook Integration
Created webhook in GitHub to trigger Jenkins job on push events to the Development branch.

Webhook URL: http://<jenkins-public-ip>:8080/github-webhook/

3. Jenkins Pipeline (frontend-dev)
Created a scripted pipeline with the following stages:

Pull Code from the Development branch

Make Change (append a timestamp comment)

Push Code back to GitHub

Deploy to EC2 via SSH:

Pull latest code

Run npm install

Start/restart app using pm2

4. SSH Deployment to EC2
Configured SSH access using Jenkins credentials (SSH key).

Deployment path: /home/ec2-user/frontend/backend/

App starts via PM2 (index.js or your main app file).

🧪 How to Test
Make a change and push to Development branch:

bash
Copy
Edit
git checkout Development
git add .
git commit -m "Test commit"
git push origin Development
Jenkins will automatically:

Pull latest code

Deploy to EC2

Restart app with pm2

Visit the public EC2 IP or domain to verify deployment.


