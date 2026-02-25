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
	}
     }
