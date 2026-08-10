pipeline {

    agent any

    options {
        timestamps()
    }

    stages {

        stage('Deploy to Azure VM') {
            steps {

                withCredentials([
                    string(credentialsId: 'VM_HOST', variable: 'VM_HOST'),
                    string(credentialsId: 'VM_USER', variable: 'VM_USER'),
                    sshUserPrivateKey(
                        credentialsId: 'VM_SSH_KEY',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {

                    sh """
                        chmod 600 \$SSH_KEY

                        ssh -o StrictHostKeyChecking=no \
                            -i \$SSH_KEY \
                            \$VM_USER@\$VM_HOST \
                            "bash -lc '
                                set -e

                                echo \\"========== Connected to Azure VM ==========\\" 

                                cd /home/azureuser/tech_store

                                echo \\"Checking out UAT branch...\\" 
                                git checkout uat

                                echo \\"Pulling latest source code...\\" 
                                git pull origin uat

                                echo \\"Installing backend dependencies...\\" 
                                cd backend
                                npm install

                                echo \\"Installing frontend dependencies...\\" 
                                cd ..
                                npm install

                                echo \\"Restarting Backend...\\" 
                                pm2 restart backend

                                echo \\"Restarting Frontend...\\" 
                                pm2 restart frontend

                                echo \\"Reloading Nginx...\\" 
                                sudo systemctl reload nginx

                                echo \\"========== Deployment Completed Successfully ==========\\" 
                            '"
                    """
                }
            }
        }
    }

    post {

        success {
            echo '======================================='
            echo 'Deployment Successful.'
            echo '======================================='
        }

        failure {
            echo '======================================='
            echo 'Deployment Failed.'
            echo '======================================='
        }

        always {
            cleanWs()
        }
    }
}
