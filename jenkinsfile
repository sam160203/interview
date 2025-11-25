pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:

  - name: node
    image: node:18
    command: ['cat']
    tty: true

  - name: sonar-scanner
    image: sonarsource/sonar-scanner-cli
    command: ['cat']
    tty: true

  - name: kubectl
    image: bitnami/kubectl:latest
    command: ['cat']
    tty: true
    securityContext:
      runAsUser: 0
      readOnlyRootFilesystem: false
    env:
      - name: KUBECONFIG
        value: /kube/config
    volumeMounts:
      - name: kubeconfig-secret
        mountPath: /kube/config
        subPath: kubeconfig

  - name: dind
    image: docker:dind
    args: ["--storage-driver=overlay2", "--insecure-registry=nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085"]
    securityContext:
      privileged: true
    env:
      - name: DOCKER_TLS_CERTDIR
        value: ""

  volumes:
    - name: kubeconfig-secret
      secret:
        secretName: kubeconfig-secret
'''
        }
    }

    environment {
        NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = "pk_test_bWlnaHR5LWhvdW5kLTI0LmNsZXJrLmFjY291bnRzLmRldiQ"
        NEXT_PUBLIC_CONVEX_URL           = "https://flexible-bobcat-761.convex.cloud"
        NEXT_PUBLIC_STREAM_API_KEY       = "dcwx5mqvzn93"
    }

    stages {

        // =========================
        // Stage 1: Checkout Code
        // =========================
        stage('Checkout Code') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github-credentials-sam',
                    url: 'https://github.com/sam160203/interview.git'
            }
        }

        // =========================
        // Stage 2: Install + Build Frontend
        // =========================
        stage('Install + Build Frontend') {
            steps {
                container('node') {
                    withCredentials([
                        string(credentialsId: 'clerk-secret', variable: 'CLERK_SECRET_KEY'),
                        string(credentialsId: 'stream-secret', variable: 'STREAM_SECRET_KEY')
                    ]) {
                        sh '''
                            # Create .env.local with all keys
                            echo "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY" > .env.local
                            echo "CLERK_SECRET_KEY=$CLERK_SECRET_KEY" >> .env.local
                            echo "NEXT_PUBLIC_CONVEX_URL=$NEXT_PUBLIC_CONVEX_URL" >> .env.local
                            echo "CONVEX_DEPLOYMENT=dev:flexible-bobcat-761" >> .env.local
                            echo "NEXT_PUBLIC_STREAM_API_KEY=$NEXT_PUBLIC_STREAM_API_KEY" >> .env.local
                            echo "STREAM_SECRET_KEY=$STREAM_SECRET_KEY" >> .env.local

                            npm install
                            npm run build
                        '''
                    }
                }
            }
        }

        // =========================
        // Stage 3: Build Docker Image
        // =========================
        stage('Build Docker Image') {
            steps {
                container('dind') {
                    sh '''
                        docker build \
                            --build-arg NEXT_PUBLIC_CONVEX_URL="$NEXT_PUBLIC_CONVEX_URL" \
                            --build-arg NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY" \
                            --build-arg NEXT_PUBLIC_STREAM_API_KEY="$NEXT_PUBLIC_STREAM_API_KEY" \
                            -t interviewhub-app:latest .
                    '''
                }
            }
        }

        // =========================
        // Stage 4: SonarQube Analysis
        // =========================
        stage('SonarQube Analysis') {
            steps {
                container('sonar-scanner') {
                    withCredentials([string(credentialsId: 'sonarqube-token-imcc', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            sonar-scanner \
                                -Dsonar.projectKey=interviewhub-app \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://192.168.20.250:9000 \
                                -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        // =========================
        // Stage 5: Login to Nexus
        // =========================
        stage('Login to Nexus') {
            steps {
                container('dind') {
                    withCredentials([usernamePassword(
                        credentialsId: 'nexus-credentials-imcc',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )]) {
                        sh '''
                            docker login nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085 \
                                -u $NEXUS_USER -p $NEXUS_PASS
                        '''
                    }
                }
            }
        }

        // =========================
        // Stage 6: Push Docker Image
        // =========================
        stage('Push to Nexus') {
            steps {
                container('dind') {
                    sh '''
                        docker tag interviewhub-app:latest \
                            nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085/2401072/interviewhub:v1

                        docker push \
                            nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085/2401072/interviewhub:v1
                    '''
                }
            }
        }

        // =========================
        // Stage 7: Deploy to Kubernetes
        // =========================
        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                        set -x
                        kubectl apply -f k8s/deployment.yaml -n 2401072
                        kubectl apply -f k8s/service.yaml -n 2401072
                        kubectl rollout status deployment/interviewhub-deployment -n 2401072
                    '''
                }
            }
        }

        // =========================
        // Stage 8: Debug Pods
        // =========================
        stage('Debug Pods') {
            steps {
                container('kubectl') {
                    sh '''
                        kubectl get pods -n 2401072
                        kubectl describe pods -n 2401072 | head -n 200
                    '''
                }
            }
        }
    }
}
