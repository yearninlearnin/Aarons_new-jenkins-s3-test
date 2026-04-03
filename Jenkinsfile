pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION    = 'us-west-1'
        TF_IN_AUTOMATION      = 'true'
        
    }
            // AWS EC2 has IAM Role attached. Homelab doesnt have IMDS so terraform can pull automatically.
            // This is why AWS credentials must be injected from Jenkins credential store.


    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'JenkinsTest']]) {
                    sh 'terraform init -reconfigure'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'JenkinsTest']]) {
                    sh '''
                        terraform plan -out=tfplan
                        terraform apply -auto-approve tfplan
                    '''
                }
            }
        }

        stage('Optional Destroy') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'JenkinsTest']]) {
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
}