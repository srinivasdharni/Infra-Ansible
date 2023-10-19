	pipeline {
	
	  agent any
	
	  options {
	    ansiColor('xterm')
	  }
	
	  parameters {
	    choice(name: 'ENV', choices: ['dev', 'prod'], description: 'Choose Environment')
	    choice(name: 'APP_NAME', choices: ['backend', 'frontend'], description: 'Choose AppName')
	    string(name: 'VERSION', defaultValue: '', description: 'Version to Deploy')
	  }
	
	  stages {
	
	    stage('Update Parameter Store') {
	      steps {
	        sh 'aws ssm put-parameter --name "${ENV}.${APP_NAME}.app_version" --type "String" --value "${VERSION}" --overwrite'
	      }
	    }
	
	    stage('Deployment') {
	      steps {
	        script {
	          env.ssh_user = sh (script: 'aws ssm get-parameter --name "ssh_username" --query "Parameter.Value" --output text', returnStdout: true).trim()
	          env.ssh_pass = sh (script: 'aws ssm get-parameter --name "ssh_password" --query "Parameter.Value" --output text --with-decryption --output text', returnStdout: true).trim()
	        }
	        wrap([$class: "MaskPasswordsBuildWrapper", varPasswordPairs: [[password: ssh_pass]]]) {
	
	          sh '''
	            aws ec2 describe-instances --filters "Name=tag:Name,Values=${ENV}-${APP_NAME}" --query "Reservations[*].Instances[*].PrivateIpAddress" --output text  >hosts
	            ansible-playbook -i hosts main.yml -e role_name=${APP_NAME} -e env=${ENV} -e ansible_user=${ssh_user} -e ansible_password=${ssh_pass}
	          '''
	        }
	
	      }
	    }
	
	  }
	
	}