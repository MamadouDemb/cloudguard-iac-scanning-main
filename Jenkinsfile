pipeline {
    agent any
    stages {
        stage('IaC-Syntax-Validation') {
            steps {
                dir('aws') {
                    git branch: 'main',
                    url: 'https://github.com/MamadouDemb/cloudguard-iac-scanning-main.git'
                }
                sh '''
                    cd aws

                    terraform --version
                    terraform init
                    terraform validate
                '''
            }
        }

        stage('IaC-Shiftleft-Code-Scan') {
            environment {
               CHKP_CLOUDGUARD_ID = credentials("CHKP_CLOUDGUARD_ID")
               CHKP_CLOUDGUARD_SECRET = credentials("CHKP_CLOUDGUARD_SECRET")
               SHIFTLEFT_REGION =  credentials("SHIFTLEFT_REGION")
            }
            steps {
                sh '''
                    export SHIFTLEFT_REGION=eu1
                    export CHKP_CLOUDGUARD_ID=$CHKP_CLOUDGUARD_ID
                    export CHKP_CLOUDGUARD_SECRET=$CHKP_CLOUDGUARD_SECRET
                    shiftleft --directory ~/shiftleft iac-assessment --Infrastructure-Type terraform --path aws --ruleset -64 --severity-level High  --environmentId ec00ab44-b2a5-4d4d-9746-ffaa110dd3b4

                '''
                sh '''
                   if ["$?" = "6" ]; then exit 0; fi
                '''
            }
        }

        stage('IaC-Shiftleft-Execution-Plan') {
            environment {
               AWS_CREDS = credentials('AWS_Credentials')
               AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
               AWS_SECRET_ACCESS_KEY = credentials('AWS_ACCESS_KEY_ID')
               CHKP_CLOUDGUARD_ID = credentials("AWS_SECRET_ACCESS_KEY")
               CHKP_CLOUDGUARD_SECRET = credentials("CHKP_CLOUDGUARD_SECRET")
               SHIFTLEFT_REGION =  credentials("SHIFTLEFT_REGION")
            }
            steps {
                sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    export AWS_DEFAULT_REGION="eu-west-1"

                    terraform --version
                    terraform plan --out=tf-plan-file
                    terraform show --json tf-plan-file > plan-file.json

                    export SHIFTLEFT_REGION=eu1
                    export CHKP_CLOUDGUARD_ID=$CHKP_CLOUDGUARD_ID
                    export CHKP_CLOUDGUARD_SECRET=$CHKP_CLOUDGUARD_SECRET
                    shiftleft iac-assessment --Infrastructure-Type terraform --path ./plan-file.json --ruleset -64 --severity-level  High --Findings-row --environmentId ec00ab44-b2a5-4d4d-9746-ffaa110dd3b4
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
