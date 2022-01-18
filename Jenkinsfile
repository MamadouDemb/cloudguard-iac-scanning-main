pipeline {
    agent any
    stages {
        stage('IaC-Syntax-Validation') {
            steps {
                dir('iac-code') {
                    git branch: 'main',
                    url: 'https://github.com/gbrembati/terraform-iac-scanning.git'
                }
                sh '''
                    cd iac-code/aws

                    terraform --version
                    terraform init
                    terraform validate
                '''
            }
        }

        stage('IaC-Shiftleft-Code-Scan') {
            environment {
               CHKP_CLOUDGUARD_CREDS = credentials('CloudGuard_Credentials')
            }
            steps {
                sh '''
                    export SHIFTLEFT_REGION=us
                    export CHKP_CLOUDGUARD_ID=$CHKP_CLOUDGUARD_CREDS_USR
                    export CHKP_CLOUDGUARD_SECRET=$CHKP_CLOUDGUARD_CREDS_PSW
                    shiftleft iac-assessment --Infrastructure-Type terraform --path iac-code/aws --ruleset -64 --severity-level Critical --Findings-row --environmentId <SHIFTLEFT-ENVIRONMENT-ID>
                '''
            }
        }

        stage('IaC-Shiftleft-Execution-Plan') {
            environment {
               AWS_CREDS = credentials('AWS_Credentials')
               CHKP_CLOUDGUARD_CREDS = credentials('CloudGuard_Credentials')
            }
            steps {
                sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_CREDS_USR
                    export AWS_SECRET_ACCESS_KEY=$AWS_CREDS_PSW
                    export AWS_DEFAULT_REGION="eu-west-1"

                    terraform --version
                    terraform plan --out=tf-plan-file
                    terraform show --json tf-plan-file > plan-file.json

                    export SHIFTLEFT_REGION=us
                    export CHKP_CLOUDGUARD_ID=$CHKP_CLOUDGUARD_CREDS_USR
                    export CHKP_CLOUDGUARD_SECRET=$CHKP_CLOUDGUARD_CREDS_PSW
                    shiftleft iac-assessment --Infrastructure-Type terraform --path ./plan-file.json --ruleset -64 --severity-level Critical --Findings-row --environmentId <SHIFTLEFT-ENVIRONMENT-ID>
                '''
            }
        }

        stage('IaC-Provider-Cleanup') {
            steps {
                sh '''
                    du -sh
                    rm plan-file.json
                    rm -r .terraform
                    rm -r .terraform.lock.hcl
                    du -sh
                '''
            }
        }
    }
}