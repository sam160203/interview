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
        // SonarQube Details (Internal Cluster DNS)
        SONAR_HOST = "http://my-sonarqube-sonarqube.sonarqube.svc.cluster.local:9000" 
        SONAR_TOKEN = credentials('sonar-token-2401072')

        // Nexus Registry Details
        NEXUS_URL = "nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085"
        NEXUS_REPO = "repository/2401072"
        IMAGE_NAME = "nextjs-project" 
        NEXUS_CREDS_ID = 'nexus-creds-2401072' 

        // Kubernetes Namespace ID
        K8S_NAMESPACE = "2401072"
        
        // Next.js Secrets
        NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = credentials('clerk-pub-2401072')
        CLERK_SECRET_KEY                  = credentials('clerk-secret-2401072')
        CONVEX_DEPLOYMENT                 = credentials('convex-deploy-2401072')
        NEXT_PUBLIC_CONVEX_URL            = credentials('convex-url-2401072')
        NEXT_PUBLIC_STREAM_API_KEY        = credentials('stream-pub-2401072')
        STREAM_SECRET_KEY                 = credentials('stream-secret-2401072')
    }

    stages {

        stage('Clean Workspace') {
            steps {
                container('node') {
                    sh 'rm -rf * || true'
                }
            }
        }

        stage('Checkout Code') {
            steps {
                container('node') {
                    git url:'https://github.com/sam160203/interview.git', branch:'main'
                }
            }
        }

        stage('Install & Build Next.js') {
            steps {
                container('node') {
                    sh """
                        echo "Creating .env file for Next.js build..."

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
        }

        stage('SonarQube Analysis') {
            steps {
                container('sonar-scanner') {
                    withSonarQubeEnv('sonarqube-server') { 
                        sh """
                        sonar-scanner \\
                        -Dsonar.projectKey=2401072_interview-stream \\
                        -Dsonar.sources=. \\
                        -Dsonar.host.url=${SONAR_HOST} \\
                        -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                container('dind') {
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

       stage('Push to Nexus') {
            steps {
                container('dind') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: NEXUS_CREDS_ID,
                            usernameVariable: 'NEXUS_USER',
                            passwordVariable: 'NEXUS_PASS'
                        )
                    ]) {
                        // FIX: Script block to safely define Groovy variables (imageTag)
                        script {
                            def imageTag = "${NEXUS_URL}/${NEXUS_REPO}/${IMAGE_NAME}:${BUILD_NUMBER}"
                            def latestTag = "${NEXUS_URL}/${NEXUS_REPO}/${IMAGE_NAME}:latest"

                            sh """
                                echo "Logging in to Nexus Docker Registry: ${NEXUS_URL}"
                                docker login ${NEXUS_URL} -u $NEXUS_USER -p $NEXUS_PASS

                                # Update latest tag (FIX: Changed // to #)
                                
                                echo "Tagging image: ${imageTag}"
                                docker tag ${IMAGE_NAME}:latest ${imageTag}

                                echo "Pushing image to Nexus..."
                                docker push ${imageTag}

                                # Push the latest tag as well
                                docker tag ${IMAGE_NAME}:latest ${latestTag}
                                docker push ${latestTag}
                            """
                        }
                    }
                }
            }
        }

        stage('Create Namespace') {
            steps {
                container('kubectl') {
                    sh """
                        kubectl create namespace ${K8S_NAMESPACE} || echo "Namespace already exists"
                        kubectl get ns
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh """
                        kubectl apply -f k8s/ -n ${K8S_NAMESPACE}

                        # Injecting sensitive environment variables directly into the K8s deployment (FIX: Changed // to #)
                        kubectl set env deployment/nextjs-deployment \\
                        NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY \\
                        CLERK_SECRET_KEY=$CLERK_SECRET_KEY \\
                        CONVEX_DEPLOYMENT=$CONVEX_DEPLOYMENT \\
                        NEXT_PUBLIC_CONVEX_URL=$NEXT_PUBLIC_CONVEX_URL \\
                        NEXT_PUBLIC_STREAM_API_KEY=$NEXT_PUBLIC_STREAM_API_KEY \\
                        STREAM_SECRET_KEY=$STREAM_SECRET_KEY \\
                        -n ${K8S_NAMESPACE}
                        
                        kubectl rollout status deployment/nextjs-deployment -n ${K8S_NAMESPACE} --timeout=120s
                    """
                }
            }
        }

        stage('Debug Deployment') {
            steps {
                container('kubectl') {
                    sh """
                        kubectl get all -n ${K8S_NAMESPACE}

                        kubectl describe deployment nextjs-deployment -n ${K8S_NAMESPACE} | tail -n 10
                    """
                }
            }
        }
    }
}