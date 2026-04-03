pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION    = 'us-west-1'
        TF_IN_AUTOMATION      = 'true'
        
        // AWS_ACCESS_KEY_ID & AWS_SECRET_ACCESS_KEY VALUE PAIR NEEDED WHEN USING DOCKER & HOMELAB
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id') 
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }
            // AWS EC2 has IAM Role attached. Homelab doesnt have IMDS so terraform can pull automatically.
            // This is why AWS Access and Secret keyvalue pair must be hardcoded.


    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init -reconfigure'
            }
        }

        stage('Terraform Apply') {
            steps {
                sh '''
                    terraform plan -out=tfplan
                    terraform apply -auto-approve tfplan
                '''
            }
        }

        stage('Optional Destroy') {
            steps {
                script {
                    def destroyChoice = input(
                        message: 'Do you want to run terraform destroy?',
                        ok: 'Submit', 
                        parameters: [
                            choice(
                                name: 'DESTROY',
                                choices: ['no', 'yes'],
                                description: 'Select yes to destroy resources'
                            )
                        ]
                    )
                    if (destroyChoice == 'yes') {
                        sh 'terraform destroy -auto-approve'
                    } else {
                        echo "Skipping destroy"
                    }
                }
            }
        }
    }
}