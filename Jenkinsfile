pipeline { 
    agent any

    environment {
        GIT_REPO = "https://github.com/englishtimeplus/my-nextjs-app.git"
        DOCKER_REGISTRY = 'mossei822'
        DOCKER_IMAGE_NAME = 'my-nextjs-app'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        GITHUB_CREDENTIALS_ID = 'github-credentials'
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git url: "${env.GIT_REPO}",
                    branch: 'master',
                    credentialsId: "${env.GITHUB_CREDENTIALS_ID}"
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    def imageTag = env.GIT_COMMIT.take(7)
                    def fullImageName = "${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}"
                    
                    docker.withRegistry("https://index.docker.io/v1/", env.DOCKER_CREDENTIALS_ID) {
                        def customImage = docker.build("${fullImageName}:${imageTag}", '.')
                        customImage.push()
                        customImage.push('latest')
                        env.BUILT_IMAGE_NAME = "${fullImageName}:${imageTag}"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "kubectl set image deployment/nextjs-deployment nextjs-container=${env.BUILT_IMAGE_NAME} --namespace=default"
                }
            }
        }
    }

    post {
        always {
            echo '파이프라인이 종료되었습니다.'
        }
    }
}
