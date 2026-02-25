pipeline {
    agent any

    parameters {
        choice(
            name: 'BRANCH',
            choices: ['main', 'dev', 'release'],
            description: 'Select Git branch'
        )
    }

    environment {
	SONARQUBE_ENV = 'sonarqube-k8s'
        AWS_REGION    = 'ap-south-1'
        ECR_REGISTRY  = '831103387233.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPO      = 'java-sonar-demo'
		IMAGE_TAG     = "${params.BRANCH}-${BUILD_NUMBER}"
		IMAGE_NAME    = "${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}",
                    url: 'https://github.com/shobithgowda7/Java-app-argocd.git'
            }
        }

        stage('Sonar Analysis') {
            steps {
		withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh '''
                      mvn clean verify sonar:sonar \
                      -Dsonar.projectKey=java-sonar-demo \
                      -Dsonar.projectName="Java Sonar K8s Demo"
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build --no-cache -t ${IMAGE_NAME} .
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} \
                | docker login --username AWS --password-stdin ${ECR_REGISTRY}

                    docker push ${IMAGE_NAME}
                    """
                }
            }
	stage('Update K8s Manifests (GitOps for ArgoCD)') {
    		steps {
        		withCredentials([usernamePassword(
            		credentialsId: 'github-creds',
            		usernameVariable: 'GIT_USER',
            		passwordVariable: 'GIT_TOKEN'
       			 )]) {
            		sh '''
            		rm -rf k8s-manifests
            		git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/shobithgowda7/Java-app-argocd.git k8s-manifests

            		cd k8s-manifests/java-app

            		sed -i "s|image:.*|image: ${IMAGE_NAME}|" deployment.yaml

            		git config user.email "jenkins@ci.com"
            		git config user.name "jenkins"

            		git add deployment.yaml
            		git commit -m "Update image to ${IMAGE_TAG}" || echo "No changes to commit"
            		git push origin main
            		'''
       	 		   }
  	  		}
		    }
	  	}

	    }
