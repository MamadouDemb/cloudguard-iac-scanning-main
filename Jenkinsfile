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
                    shiftleft --directory ~/shiftleft iac-assessment --Infrastructure-Type terraform --path aws --ruleset -64 --severity-level High  --environmentId ec00ab44-b2a5-4d4d-9746-ffaa110dd3b4 || f ["$?" == "6" ]; then exit 0 ; fi
                '''
            }
        }

        stage('IaC-Shiftleft-Code-Scan-checkversion') {
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
                    shiftleft --version
                '''
            }
        }


    }
}
