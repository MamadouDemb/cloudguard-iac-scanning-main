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
                    shiftleft --version
                   if ["$?" != "6" ]; then exit 1; fi
                '''
            }
        }


    }
}
