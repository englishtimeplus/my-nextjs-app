pipeline {
    agent any

    // 환경 변수 설정
    environment {
        GIT_REPO = "https://github.com/englishtimeplus/my-nextjs-app.git"
        DOCKER_REGISTRY = 'mossei822' // 본인 Docker Hub ID
        DOCKER_IMAGE_NAME = 'my-nextjs-app'
        DOCKER_CREDENTIALS_ID = 'dckr_pat_lMilEYYSVy_bOxKa2RyOiDCX8LA' // Jenkins에 등록한 Docker Hub 인증 정보 ID
        GITHUB_CREDENTIALS_ID = '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' // GitHub Token 인증 정보 ID
        // KUBE_CONFIG_CREDENTIALS_ID = 'kubeconfig-credentials' // Jenkins에 등록한 Kubeconfig 인증 정보 ID
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git url: "${env.GIT_REPO}",
                    branch: 'master',
                    credentialsId: "${env.GITHUB_CREDENTIALS_ID}"
            }
        }

        // 'Build Docker Image' 스테이지는 중복되고 오류를 유발하므로 삭제했습니다.
        // stage('Build Docker Image') { ... }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Git 커밋 해시를 이용해 고유한 이미지 태그 생성
                    def imageTag = env.GIT_COMMIT.take(7)
                    def fullImageName = "${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}"
                    
                    // Jenkins의 Docker 인증 정보 사용
                    docker.withRegistry("https://index.docker.io/v1/", env.DOCKER_CREDENTIALS_ID) {
                        // Docker 이미지 빌드 (고유 태그 사용)
                        def customImage = docker.build("${fullImageName}:${imageTag}", '.')
                        
                        // Docker 이미지 푸시
                        customImage.push()
                        
                        // latest 태그도 추가로 푸시 (선택사항)
                        customImage.push('latest')

                        // 다음 스테이지에서 사용할 수 있도록 이미지 정보를 환경 변수로 저장
                        env.BUILT_IMAGE_NAME = "${fullImageName}:${imageTag}"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Build & Push 단계에서 생성된 정확한 이미지 태그를 사용하여 배포
                    // --namespace 에는 실제 사용하는 네임스페이스를 입력해야 합니다.
                    sh "kubectl set image deployment/nextjs-deployment nextjs-container=${env.BUILT_IMAGE_NAME} --namespace=default"
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
