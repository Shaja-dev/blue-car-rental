pipeline {
    agent any 

    parameters {
        choice(name: 'TEAM', choices: ['default', 'test', 'dev', 'staging', 'prod'], description: 'Select the target environment')
        string(name: 'INSTANCE_TYPE', defaultValue: 't2.micro', description: 'AWS EC2 instance type')
        string(name: 'INSTANCE_COUNT', defaultValue: '1', description: 'Number of instances to deploy, must be a positive number')

    }

    environment {
        AWS_REGION= "us-east-1"
        INSTANCE_COUNT_INT = "${params.INSTANCE_COUNT.toInteger()}"

    }

    stages {
        stage('Create Key Pair for AWS instance') {
            steps {
                echo "Creating Key Pair "
                sh """
                    aws ec2 create-key-pair --region ${AWS_REGION} --key-name ${ENVIRONMENT} --query KeyMaterial --output text > ${TEAM}
                    chmod 400 ${TEAM}
                """
            }
        }
        stage('Create AWS Resources') {
            steps {
                sh """
                    cd Task-1-Solution/terraform
                    terraform workspace select ${params.ENVIRONMENT} || terraform workspace new ${params.ENVIRONMENT}
                    terraform init
                    terraform apply -var='ec2_type=${params.INSTANCE_TYPE}' \
                                    -var='num_of_instance=${INSTANCE_COUNT_INT}' \
                                    -var='ec2_key=${ENVIRONMENT}' \
                                    -auto-approve
                """
            }
        }

    }
    post {
        always {
            echo 'Post Always block'
        }
        success {
            echo 'Delete the Key Pair'
                timeout(time:5, unit:'DAYS'){
                input message:'Approve terminate'
                }
           sh """
                aws ec2 delete-key-pair --region ${AWS_REGION} --key-name ${TEAM}
                rm -rf ${TEAM}
                """
            echo 'Delete AWS Resources'            
                sh """
                cd Task-1-Solution/terraform
                terraform destroy --auto-approve
                """
        }
        failure {

            echo 'Delete the Key Pair'
            sh """
                aws ec2 delete-key-pair --region ${AWS_REGION} --key-name ${TEAM}
                rm -rf ${TEAM}
                """
            echo 'Delete AWS Resources'            
                sh """
                cd Task-1-Solution/terraform
                terraform destroy --auto-approve
                """
        }
    }
}