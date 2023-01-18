pipeline{
	agent any
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
				    sh "aws cloudformation package --template-file template.yml --s3-bucket 229413076389-sam-artifacts-dev --s3-prefix project-cfn-jenkins --output-template-file packaged-template.yml"
				}
			}
		}
		stage("Deploy"){
			steps{
				withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'my-creds', secretKeyVariable:'AWS_SECRET_ACCESS_KEY')]){
					sh "aws cloudformation deploy --template-file packaged-template.yml --stack-name cfn-jenkins --parameter-overrides Env=dev ProjectName=cfn-jenkins --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM --tags Key1=Value1"
                    script {
                        def stackStatus = sh(returnStdout: true, script: 'aws cloudformation describe-stacks --stack-name cfn-jenkins --query "Stacks[0].StackStatus"').trim()
                        if (stackStatus == 'ROLLBACK_COMPLETE') {
                            sh 'aws cloudformation delete-stack --stack-name cfn-jenkins' 
                        }
                    }
			    }
		    }
	    }
    }
}