pipeline {
    agent any

    // environment {
    //     GIT_REPO = "https://github.com/englishtimeplus/my-nextjs-app.git"
    //     DOCKER_IMAGE = "mossei822/my-nextjs-app"
    //     IMAGE_TAG = "latest"
    // } 

    // 환경 변수 설정
    environment {
        GIT_REPO = "https://github.com/englishtimeplus/my-nextjs-app.git"
        DOCKER_REGISTRY = 'mossei822' // 본인 Docker Hub ID로 변경
        DOCKER_IMAGE_NAME = 'my-nextjs-app'
        DOCKER_CREDENTIALS_ID = 'dckr_pat_lMilEYYSVy_bOxKa2RyOiDCX8LA' // Jenkins에 등록한 Docker Hub 인증 정보 ID
        // KUBE_CONFIG_CREDENTIALS_ID = 'kubeconfig-credentials' // Jenkins에 등록한 Kubeconfig 인증 정보 ID
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git url: "${env.GIT_REPO}",
                    branch: 'master',
                    credentialsId: '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' // GitHub Token 기반 인증
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Git 커밋 해시를 이용해 고유한 이미지 태그 생성
                    def imageTag = env.GIT_COMMIT.take(7)
                    def fullImageName = "${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}:${imageTag}"
                    
                    // Jenkins의 Docker 인증 정보 사용
                    docker.withRegistry("https://index.docker.io/v1/", DOCKER_CREDENTIALS_ID) {
                        // Docker 이미지 빌드
                        def customImage = docker.build(fullImageName, '.')
                        
                        // Docker 이미지 푸시
                        customImage.push()
                        
                        // latest 태그도 추가로 푸시 (선택사항)
                        customImage.push('latest')
                    }
                }
            }
        }

         

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    kubectl set image deployment/nextjs-deployment nextjs-container=${DOCKER_IMAGE}:${IMAGE_TAG} --namespace=your-namespace
                    '''
                }
            }
        }
    }

    post {
        always {
            echo '파이프라인이 종료되었습니다.'
            // 예: 슬랙 알림 보내기
        }
    }
}
