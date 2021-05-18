properties([pipelineTriggers([githubPush()])])

pipeline {
    agent any
    environment {    
        // Git Checkout SCM stage
        CLASS           = "GitSCM"
        BRANCH          = "main"
        GIT_CREDENTIALS = "GitHub_SSH"
        GIT_URL         = "git@github.com:AleksandrTGH/Project_Infrastructure.git"

        // Terraform stages
        REGION          = "us-east-1"
        PLAN_FILE        = "tf_plan"
        PLAN_OUTPUT_FILE = "plan_output.txt"
        TF_OUTPUT_FILE   = "tf_output.txt"
        

        // Notification to Slack stages
        CHANNEL                  = "test"
        SLACK_CREDENTIALS        = "Slack"
        PLAN_OUTPUT_FILE_COMMENT = "${JOB_NAME}_#${BUILD_NUMBER}\n<terraform plan> command output file"
        TF_OUTPUT_FILE_COMMENT   = "${JOB_NAME}_#${BUILD_NUMBER}\n<terraform output> command output file"

        // blocks SlackSend
        MESSAGE_TEXT = "${JOB_NAME}_#${BUILD_NUMBER}. Apply changes?"
        YES_URL      = "${BUILD_URL}input/${STEP_ID}/proceedEmpty?"
        NO_URL       = "${BUILD_URL}input/${STEP_ID}/abort?"
        
        // Input step
        INPUT_MESSAGE = "Apply changes?"
        STEP_ID       = "Choisee"
    }

    stages {                  
        stage('Git Infrastructure checkout SCM') {
            steps {
                checkout([
                    $class: "${CLASS}",
                    branches: [[name: "${BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "${GIT_URL}",
                        credentialsId: "${GIT_CREDENTIALS}",
                    ]]
                ])
            }
        }

        stage('Terraform init') {
            steps {
                withAWS(region:"${REGION}") {
                    sh "terraform init -input=false"
                }
            }
        }

        stage('Terraform plan') {
            steps {
                withAWS(region:"${REGION}") {
                    sh "terraform plan -out=${PLAN_FILE} -input=false > ${PLAN_OUTPUT_FILE}"
                }
            }
        }
        
        stage('Notification to Slack') {
            steps {
                slackUploadFile (
                    filePath: "${PLAN_OUTPUT_FILE}",
                    channel: "${CHANNEL}",
                    credentialId: "${SLACK_CREDENTIALS}",
                    initialComment: "${PLAN_OUTPUT_FILE_COMMENT}"
                )
                slackSend (
                    botUser: true, 
                    channel: "${CHANNEL}", 
                    blocks: [
                                [
			                        "type": "section",
			                        "text": [
				                        "type": "plain_text",
				                        "text": "${MESSAGE_TEXT}"
                                    ]
                                ],
                                [
			                        "type": "actions",
			                        "elements": [
                                        [
					                        "type": "button",
					                        "text": [
                                                "type": "plain_text",
						                        "text": "Yes"
                                            ],
					                        "style": "primary",
					                        "url": "${YES_URL}"
                                        ],
                                        [
					                        "type": "button",
					                        "text": [
						                        "type": "plain_text",
						                        "text": "No"
                                            ],
                                            "style": "danger",
                                            "url": "${NO_URL}"
                                        ]
                                    ]
			                    ]
                            ], 
                    tokenCredentialId: "${SLACK_CREDENTIALS}"
                )
            }
        }

        stage('Input step') {
            steps {
                input message: "${INPUT_MESSAGE}", id: "${STEP_ID}"
            }
        }    

        stage('Terraform apply') {
            steps {
                withAWS(region:"${REGION}") {
                    sh "terraform apply -input=false ${PLAN_FILE}"
		            sh "sleep 10"
                    sh "terraform output > ${TF_OUTPUT_FILE}"
                    slackUploadFile (
                        filePath: "${TF_OUTPUT_FILE}",
                        channel: "${CHANNEL}",
                        credentialId: "${SLACK_CREDENTIALS}",
                        initialComment: "${TF_OUTPUT_FILE_COMMENT}"
                    )
                }
            }
        }
	    
        //stage('Terraform destroy') {
        //    steps {
        //        withAWS(region:"${REGION}") {
        //            sh "terraform apply -destroy -auto-approve"
        //        }
        //    }
        //}
    }
}
