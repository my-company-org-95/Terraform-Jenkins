pipeline {
    agent any

    parameters {
        booleanParam(
            name: 'autoDestroy', 
            defaultValue: true, 
            description: 'Automatically destroy Terraform resources without manual approval?'
        )
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    tools {
        terraform 'terraform-1.13.3'  // Must match the Jenkins Tool name
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

        stage('Destroy') {
            when {
                expression { return params.autoDestroy == true }
            }
            steps {
                dir("terraform") {
                    // Destroy without manual input
                    sh 'terraform destroy -auto-approve'
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
