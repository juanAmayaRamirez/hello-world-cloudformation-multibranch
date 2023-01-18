pipeline{
	agent any
    parameters {
        choice(
            name: 'ENV',
            choices: 'dev\nqa\nuat\nprod',
            description: 'environment'
        )
        string(
            name: 'PROJECT_NAME',
            defaultValue: 'cfn-jenkins',
            description: 'name of the project to be deployed'
        )
        string(
            name: 'S3_ARTIFACTS_BUCKET',
            defaultValue: '229413076389-sam-artifacts-dev',
            description: 'name of the artifacts bucket'
        )
    }
	environment{
		AWS_DEFAULT_REGION="us-east-1"
	}
	stages{
		stage("Validate"){
			steps{
                sh "aws --version"
				withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'my-creds', secretKeyVariable:'AWS_SECRET_ACCESS_KEY')]){
					sh "aws sts get-caller-identity"
				}
			}
		}
		stage("Package"){
			steps{
				withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'my-creds', secretKeyVariable:'AWS_SECRET_ACCESS_KEY')]){
				    sh "aws cloudformation package --template-file template.yml --s3-bucket ${params.S3_ARTIFACTS_BUCKET} --s3-prefix ${params.PROJECT_NAME} --output-template-file packaged-template.yml"
				}
			}
		}
		stage("Deploy"){
			steps{
				withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'my-creds', secretKeyVariable:'AWS_SECRET_ACCESS_KEY')]){
                    sh "aws cloudformation deploy --template-file packaged-template.yml --stack-name ${params.PROJECT_NAME}-${params.ENV} --parameter-overrides Env=${params.ENV} ProjectName=${params.PROJECT_NAME} --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM --tags Key1=Value1"
			    }
		    }
	    }
    }
    post{
        always{
            withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'my-creds', secretKeyVariable:'AWS_SECRET_ACCESS_KEY')]){
                sh "aws cloudformation describe-stack-events --stack-name ${params.PROJECT_NAME}-${params.ENV}"
            }
        }
        failure{
            withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'my-creds', secretKeyVariable:'AWS_SECRET_ACCESS_KEY')]){
                script {
                    def stackStatus = sh(returnStdout: true, script: "aws cloudformation describe-stacks --stack-name ${params.PROJECT_NAME}-${params.ENV} --query 'Stacks[0].StackStatus'").trim()
                    if (stackStatus == '"ROLLBACK_COMPLETE"') {
                        sh "aws cloudformation delete-stack --stack-name ${params.PROJECT_NAME}-${params.ENV}"
                    }
                }
            }
        }
    }
}