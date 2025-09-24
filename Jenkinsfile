pipeline {
    agent any

    parameters {
        booleanParam(
            name: 'autoApprove', 
            defaultValue: false, 
            description: 'Automatically run apply after generating plan?'
        )
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    tools {
        terraform 'terraform-1.13.3'  // 👈 updated version; must match Jenkins Tool name
    }

    stages {
        stage('Checkout') {
            steps {
                dir("terraform") {
                    git branch: 'main', url: "https://github.com/my-company-org-95/Terraform-Jenkins.git"
                }
            }
        }

        stage('Terraform Init & Plan') {
            steps {
                dir("terraform") {
                    sh 'terraform init -input=false'
                    sh 'terraform plan -out=tfplan -input=false'
                    sh 'terraform show -no-color tfplan > tfplan.txt'
                }
            }
        }

        stage('Approval') {
            when {
                not { equals expected: true, actual: params.autoApprove }
            }
            steps {
                script {
                    def planText = readFile 'terraform/tfplan.txt'
                    input message: "Do you want to apply the plan?",
                          parameters: [
                              text(name: 'Plan', description: 'Please review the plan', defaultValue: planText)
                          ]
                }
            }
        }

        stage('Apply') {
            steps {
                dir("terraform") {
                    sh 'terraform destroy -input=false tfplan'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Cleaning workspace."
            cleanWs()
        }
    }
}
