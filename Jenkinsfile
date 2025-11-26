// pipeline {
//     agent any

//     environment {
//         // SonarQube
//         SONAR_HOST = "http://sonarqube.imcc.com/"
//         SONAR_TOKEN = credentials('sonar-token-2401072')

//         // Nexus
//         NEXUS_URL = "http://nexus.imcc.com/"
//         IMAGE_NAME = "nextjs-project"

//         // Next.js ENV Variables from Jenkins Credentials
//         NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = credentials('clerk-pub-2401072')
//         CLERK_SECRET_KEY                  = credentials('clerk-secret-2401072')
//         CONVEX_DEPLOYMENT                 = credentials('convex-deploy-2401072')
//         NEXT_PUBLIC_CONVEX_URL            = credentials('convex-url-2401072')
//         NEXT_PUBLIC_STREAM_API_KEY        = credentials('stream-pub-2401072')
//         STREAM_SECRET_KEY                 = credentials('stream-secret-2401072')
//     }

//     stages {

//         stage('Checkout Code') {
//             steps {
//                 git branch: 'main',
//                     url: 'https://github.com/sam160203/interview.git'
//             }
//         }

//         stage('SonarQube Analysis') {
//             steps {
//                 withSonarQubeEnv('sonarqube-server') {
//                     sh """
//                         sonar-scanner \
//                         -Dsonar.projectKey=2401072_interview-stream \
//                         -Dsonar.sources=. \
//                         -Dsonar.host.url=${SONAR_HOST} \
//                         -Dsonar.login=${SONAR_TOKEN}
//                     """
//                 }
//             }
//         }

//         stage('Install & Build Next.js') {
//             steps {
//                 sh """
//                     echo "Creating .env file..."

//                     echo "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY" > .env
//                     echo "CLERK_SECRET_KEY=$CLERK_SECRET_KEY" >> .env
//                     echo "CONVEX_DEPLOYMENT=$CONVEX_DEPLOYMENT" >> .env
//                     echo "NEXT_PUBLIC_CONVEX_URL=$NEXT_PUBLIC_CONVEX_URL" >> .env
//                     echo "NEXT_PUBLIC_STREAM_API_KEY=$NEXT_PUBLIC_STREAM_API_KEY" >> .env
//                     echo "STREAM_SECRET_KEY=$STREAM_SECRET_KEY" >> .env

//                     yarn install
//                     yarn build
//                 """
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 sh "docker build -t ${IMAGE_NAME}:latest ."
//             }
//         }

//         stage('Push to Nexus') {
//             steps {
//                 withCredentials([usernamePassword(
//                     credentialsId: 'nexus-creds-2401072',
//                     usernameVariable: 'NEXUS_USER',
//                     passwordVariable: 'NEXUS_PASS'
//                 )]) {
//                     sh """
//                         docker login ${NEXUS_URL} -u $NEXUS_USER -p $NEXUS_PASS
//                         docker tag ${IMAGE_NAME}:latest nexus.imcc.com/repository/docker-hosted/${IMAGE_NAME}:latest
//                         docker push nexus.imcc.com/repository/docker-hosted/${IMAGE_NAME}:latest
//                     """
//                 }
//             }
//         }

//         stage('Deploy to Kubernetes') {
//             steps {
//                 sh """
//                     kubectl apply -f k8s/

//                     kubectl set env deployment/nextjs-deployment \
//                     NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY \
//                     CLERK_SECRET_KEY=$CLERK_SECRET_KEY \
//                     CONVEX_DEPLOYMENT=$CONVEX_DEPLOYMENT \
//                     NEXT_PUBLIC_CONVEX_URL=$NEXT_PUBLIC_CONVEX_URL \
//                     NEXT_PUBLIC_STREAM_API_KEY=$NEXT_PUBLIC_STREAM_API_KEY \
//                     STREAM_SECRET_KEY=$STREAM_SECRET_KEY
//                 """
//             }
//         }
//     }
// }




pipeline {
    // REPLACED 'agent any' with the specialized Kubernetes Pod agent
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
        // SonarQube
        SONAR_HOST = "http://sonarqube.imcc.com/"
        SONAR_TOKEN = credentials('sonar-token-2401072')

        // Nexus
        NEXUS_URL = "http://nexus.imcc.com/"
        IMAGE_NAME = "nextjs-project"

        // Next.js ENV Variables from Jenkins Credentials
        NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = credentials('clerk-pub-2401072')
        CLERK_SECRET_KEY                 = credentials('clerk-secret-2401072')
        CONVEX_DEPLOYMENT                = credentials('convex-deploy-2401072')
        NEXT_PUBLIC_CONVEX_URL           = credentials('convex-url-2401072')
        NEXT_PUBLIC_STREAM_API_KEY       = credentials('stream-pub-2401072')
        STREAM_SECRET_KEY                = credentials('stream-secret-2401072')
    }

    stages {

        stage('Checkout Code') {
            steps {
                // Checkout runs on the main container (usually the first one, or 'node')
                container('node') {
                    git branch: 'main',
                        url: 'https://github.com/sam160203/interview.git'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Run in the specialized sonar-scanner container
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

        stage('Install & Build Next.js') {
            steps {
                // Run in the node container as it has yarn/npm/node
                container('node') {
                    sh """
                        echo "Creating .env file..."

                        // Create .env file for the build process
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

        stage('Build Docker Image') {
            steps {
                // Run in the dind container to execute Docker commands
                container('dind') {
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push to Nexus') {
            steps {
                // Run in the dind container
                container('dind') {
                    withCredentials([usernamePassword(
                        credentialsId: 'nexus-creds-2401072',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )]) {
                        sh """
                        docker login ${NEXUS_URL} -u $NEXUS_USER -p $NEXUS_PASS
                        docker tag ${IMAGE_NAME}:latest ${NEXUS_URL}/repository/docker-hosted/${IMAGE_NAME}:latest
                        docker push ${NEXUS_URL}/repository/docker-hosted/${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Run in the kubectl container which is configured with the kubeconfig secret
                container('kubectl') {
                    sh """
                        echo "Applying Kubernetes manifests from k8s/..."
                        kubectl apply -f k8s/

                        echo "Setting environment variables on deployment/nextjs-deployment..."
                        kubectl set env deployment/nextjs-deployment \\
                        NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY \\
                        CLERK_SECRET_KEY=$CLERK_SECRET_KEY \\
                        CONVEX_DEPLOYMENT=$CONVEX_DEPLOYMENT \\
                        NEXT_PUBLIC_CONVEX_URL=$NEXT_PUBLIC_CONVEX_URL \\
                        NEXT_PUBLIC_STREAM_API_KEY=$NEXT_PUBLIC_STREAM_API_KEY \\
                        STREAM_SECRET_KEY=$STREAM_SECRET_KEY
                    """
                }
            }
        }
    }
}

