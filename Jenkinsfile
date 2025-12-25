pipeline {
    agent any

    environment {
        // Define the SSH credentials to connect with the EC2 instances
        SSH_CREDENTIALS = 'ubuntu'
        DEV_INSTANCE = '13.213.2.254'      // EC2 IP for Dev Environment
        STAGING_INSTANCE = '13.215.50.79'  // EC2 IP for Staging Environment
        PROD_INSTANCE = '13.213.45.247'    // EC2 IP for Production Environment
    }

    parameters {
        // Environment parameter to trigger the respective branch deployment
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'main'], description: 'Select the Environment to deploy')
    }

    stages {
        stage('Setup Environment') {
            steps {
                script {
                    // Determine which environment's repository to work with based on the parameter
                    def repoBranch = ""
                    def repoDirectory = "/var/www/html/website-demo.git"
                    def INSTANCE_IP = ""

                    // Set the repo branch and instance IP based on the selected environment
                    if (params.ENVIRONMENT == 'dev') {
                        repoBranch = "dev"
                        INSTANCE_IP = DEV_INSTANCE
                    } else if (params.ENVIRONMENT == 'staging') {
                        repoBranch = "staging"
                        INSTANCE_IP = STAGING_INSTANCE
                    } else if (params.ENVIRONMENT == 'main') {
                        repoBranch = "main"
                        INSTANCE_IP = PROD_INSTANCE
                    }

                    // SSH into the respective EC2 instance and check if the directory exists
                    echo "Checking if the repository exists for ${params.ENVIRONMENT} environment..."
                    sshagent(credentials: ['github-ssh-key']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ubuntu@${INSTANCE_IP} <<EOF
                            if [ ! -d "${repoDirectory}" ]; then
                                echo "Repository not found, cloning ${repoBranch} branch..."
                                git clone --branch ${repoBranch} https://github.com/Arif-555/website-demo.git ${repoDirectory}
                            else
                                echo "Repository found, pulling latest changes for ${repoBranch}..."
                                cd ${repoDirectory} && git pull origin ${repoBranch}
                            fi
EOF
                        """
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Deploy the website after pulling the code
                    echo "Deploying updated files to ${params.ENVIRONMENT} environment..."
                    // Add your deployment steps here
                }
            }
        }

        stage('Post Deploy') {
            steps {
                echo "Post Deployment steps can be added here if required."
            }
        }
    }

    post {
        success {
            echo "Deployment to ${params.ENVIRONMENT} environment completed successfully!"
        }
        failure {
            echo "Deployment failed. Please check the logs."
        }
    }
}
