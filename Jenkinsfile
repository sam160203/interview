// pipeline {
//     agent {
//         kubernetes {
//             yaml '''
// apiVersion: v1
// kind: Pod
// spec:
//   containers:

//   - name: node
//     image: node:18
//     command: ['cat']
//     tty: true

//   - name: sonar-scanner
//     image: sonarsource/sonar-scanner-cli
//     command: ['cat']
//     tty: true

//   - name: kubectl
//     image: bitnami/kubectl:latest
//     command: ['cat']
//     tty: true
//     securityContext:
//       runAsUser: 0
//       readOnlyRootFilesystem: false
//     env:
//       - name: KUBECONFIG
//         value: /kube/config
//     volumeMounts:
//       - name: kubeconfig-secret
//         mountPath: /kube/config
//         subPath: kubeconfig

//   - name: dind
//     image: docker:dind
//     args: ["--storage-driver=overlay2", "--insecure-registry=nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085"]
//     securityContext:
//       privileged: true
//     env:
//       - name: DOCKER_TLS_CERTDIR
//         value: ""

//   volumes:
//     - name: kubeconfig-secret
//       secret:
//         secretName: kubeconfig-secret
// '''
//         }
//     }

//     environment {
//         NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = "pk_test_bWlnaHR5LWhvdW5kLTI0LmNsZXJrLmFjY291bnRzLmRldiQ"
//         NEXT_PUBLIC_CONVEX_URL           = "https://flexible-bobcat-761.convex.cloud"
//         NEXT_PUBLIC_STREAM_API_KEY       = "dcwx5mqvzn93"
//     }

//     stages {

//         // =========================
//         // Stage 1: Checkout Code
//         // =========================
//         stage('Checkout Code') {
//             steps {
//                 git branch: 'main', 
//                     credentialsId: 'github-credentials-sam',
//                     url: 'https://github.com/sam160203/interview.git'
//             }
//         }

//         // =========================
//         // Stage 2: Install + Build Frontend
//         // =========================
//         stage('Install + Build Frontend') {
//             steps {
//                 container('node') {
//                     withCredentials([
//                         string(credentialsId: 'clerk-secret', variable: 'CLERK_SECRET_KEY'),
//                         string(credentialsId: 'stream-secret', variable: 'STREAM_SECRET_KEY')
//                     ]) {
//                         sh '''
//                             # Create .env.local with all keys
//                             echo "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY" > .env.local
//                             echo "CLERK_SECRET_KEY=$CLERK_SECRET_KEY" >> .env.local
//                             echo "NEXT_PUBLIC_CONVEX_URL=$NEXT_PUBLIC_CONVEX_URL" >> .env.local
//                             echo "CONVEX_DEPLOYMENT=dev:flexible-bobcat-761" >> .env.local
//                             echo "NEXT_PUBLIC_STREAM_API_KEY=$NEXT_PUBLIC_STREAM_API_KEY" >> .env.local
//                             echo "STREAM_SECRET_KEY=$STREAM_SECRET_KEY" >> .env.local

//                             npm install
//                             npm run build
//                         '''
//                     }
//                 }
//             }
//         }

//         // =========================
//         // Stage 3: Build Docker Image
//         // =========================
//         stage('Build Docker Image') {
//             steps {
//                 container('dind') {
//                     sh '''
//                         docker build \
//                             --build-arg NEXT_PUBLIC_CONVEX_URL="$NEXT_PUBLIC_CONVEX_URL" \
//                             --build-arg NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY" \
//                             --build-arg NEXT_PUBLIC_STREAM_API_KEY="$NEXT_PUBLIC_STREAM_API_KEY" \
//                             -t interviewhub-app:latest .
//                     '''
//                 }
//             }
//         }

//         // =========================
// // Stage 4: SonarQube Analysis
// // =========================
// stage('SonarQube Analysis') {
//     steps {
//         container('sonar-scanner') {
//             withCredentials([string(credentialsId: 'sonarqube-token-imcc', variable: 'SONAR_TOKEN')]) {
//                 sh '''
//                     sonar-scanner \
//                         -Dsonar.projectKey=interviewhub-app \
//                         -Dsonar.sources=. \
//                         -Dsonar.host.url=http://my-sonarqube-sonarqube.sonarqube.svc.cluster.local:9000 \
//                         -Dsonar.login=$SONAR_TOKEN
//                 '''
//             }
//         }
//     }
// }

//         // =========================
//         // Stage 5: Login to Nexus
//         // =========================
//         stage('Login to Nexus') {
//             steps {
//                 container('dind') {
//                     withCredentials([usernamePassword(
//                         credentialsId: 'nexus-credentials-imcc',
//                         usernameVariable: 'NEXUS_USER',
//                         passwordVariable: 'NEXUS_PASS'
//                     )]) {
//                         sh '''
//                             docker login nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085 \
//                                 -u $NEXUS_USER -p $NEXUS_PASS
//                         '''
//                     }
//                 }
//             }
//         }

//         // =========================
//         // Stage 6: Push Docker Image
//         // =========================
//         stage('Push to Nexus') {
//             steps {
//                 container('dind') {
//                     sh '''
//                         docker tag interviewhub-app:latest \
//                             nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085/2401072/interviewhub:v1

//                         docker push \
//                             nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085/2401072/interviewhub:v1
//                     '''
//                 }
//             }
//         }

//         // =========================
//         // Stage 7: Deploy to Kubernetes
//         // =========================
//         stage('Deploy to Kubernetes') {
//             steps {
//                 container('kubectl') {
//                     sh '''
//                         set -x
//                         kubectl apply -f k8s/deployment.yaml -n 2401072
//                         kubectl apply -f k8s/service.yaml -n 2401072
//                         kubectl rollout status deployment/interviewhub-deployment -n 2401072
//                     '''
//                 }
//             }
//         }

//         // =========================
//         // Stage 8: Debug Pods
//         // =========================
//         stage('Debug Pods') {
//             steps {
//                 container('kubectl') {
//                     sh '''
//                         kubectl get pods -n 2401072
//                         kubectl describe pods -n 2401072 | head -n 200
//                     '''
//                 }
//             }
//         }
//     }
// }




pipeline {
    agent any

    environment {
        # SonarQube
        SONAR_HOST = "http://sonarqube.imcc.com/"
        SONAR_TOKEN = credentials('sonar-token-2401072')

        # Nexus
        NEXUS_URL = "http://nexus.imcc.com/"
        IMAGE_NAME = "nextjs-project"

        # Next.js ENV Variables from Jenkins Credentials
        NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = credentials('clerk-pub-2401072')
        CLERK_SECRET_KEY                  = credentials('clerk-secret-2401072')
        CONVEX_DEPLOYMENT                 = credentials('convex-deploy-2401072')
        NEXT_PUBLIC_CONVEX_URL            = credentials('convex-url-2401072')
        NEXT_PUBLIC_STREAM_API_KEY        = credentials('stream-pub-2401072')
        STREAM_SECRET_KEY                 = credentials('stream-secret-2401072')
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sam160203/interview'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=nextjs-project \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST} \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Install & Build Next.js') {
            steps {
                sh """
                    echo "Creating .env file..."

                    echo "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY" > .env
                    echo "CLERK_SECRET_KEY=$CLERK_SECRET_KEY" >> .env
                    echo "CONVEX_DEPLOYMENT=$CONVEX_DEPLOYMENT" >> .env
                    echo "NEXT_PUBLIC_CONVEX_URL=$NEXT_PUBLIC_CONVEX_URL" >> .env
                    echo "NEXT_PUBLIC_STREAM_API_KEY=$NEXT_PUBLIC_STREAM_API_KEY" >> .env
                    echo "STREAM_SECRET_KEY=$STREAM_SECRET_KEY" >> .env

                    yarn install
                    yarn build
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-creds-2401072',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh """
                        docker login ${NEXUS_URL} -u $NEXUS_USER -p $NEXUS_PASS
                        docker tag ${IMAGE_NAME}:latest nexus.imcc.com/repository/docker-hosted/${IMAGE_NAME}:latest
                        docker push nexus.imcc.com/repository/docker-hosted/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl apply -f k8s/

                    kubectl set env deployment/nextjs-deployment \
                    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY \
                    CLERK_SECRET_KEY=$CLERK_SECRET_KEY \
                    CONVEX_DEPLOYMENT=$CONVEX_DEPLOYMENT \
                    NEXT_PUBLIC_CONVEX_URL=$NEXT_PUBLIC_CONVEX_URL \
                    NEXT_PUBLIC_STREAM_API_KEY=$NEXT_PUBLIC_STREAM_API_KEY \
                    STREAM_SECRET_KEY=$STREAM_SECRET_KEY
                """
            }
        }
    }
}


