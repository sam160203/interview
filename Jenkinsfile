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
//     - name: KUBECONFIG
//       value: /kube/config
//     volumeMounts:
//     - name: kubeconfig-secret
//       mountPath: /kube/config
//       subPath: kubeconfig

//   - name: dind
//     image: docker:dind
//     args: ["--storage-driver=overlay2", "--insecure-registry=nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085"]
//     securityContext:
//       privileged: true
//     env:
//     - name: DOCKER_TLS_CERTDIR
//       value: ""

//   volumes:
//   - name: kubeconfig-secret
//     secret:
//       secretName: kubeconfig-secret
// '''
//         }
//     }

//     environment {
//         // SonarQube Details (Internal Cluster DNS)
//         SONAR_HOST = "http://my-sonarqube-sonarqube.sonarqube.svc.cluster.local:9000" 
//         SONAR_TOKEN = credentials('sonar-token-2401072')

//         // Nexus Registry Details
//         NEXUS_URL = "nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085"
//         NEXUS_REPO = "repository/2401072"
//         IMAGE_NAME = "nextjs-project" 
//         NEXUS_CREDS_ID = 'nexus-creds-2401072' 

//         // Kubernetes Namespace ID
//         K8S_NAMESPACE = "2401072"
        
//         // Next.js Secrets
//         NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = credentials('clerk-pub-2401072')
//         CLERK_SECRET_KEY                  = credentials('clerk-secret-2401072')
//         CONVEX_DEPLOYMENT                 = credentials('convex-deploy-2401072')
//         NEXT_PUBLIC_CONVEX_URL            = credentials('convex-url-2401072')
//         NEXT_PUBLIC_STREAM_API_KEY        = credentials('stream-pub-2401072')
//         STREAM_SECRET_KEY                 = credentials('stream-secret-2401072')
//     }

//     stages {

//         stage('Clean Workspace') {
//             steps {
//                 container('node') {
//                     sh 'rm -rf * || true'
//                 }
//             }
//         }

//         stage('Checkout Code') {
//             steps {
//                 container('node') {
//                     git url:'https://github.com/sam160203/interview.git', branch:'main'
//                 }
//             }
//         }

//         stage('Install & Build Next.js') {
//             steps {
//                 container('node') {
//                     sh """
//                         echo "Creating .env file for Next.js build..."

//                         echo "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY" > .env
//                         echo "CLERK_SECRET_KEY=$CLERK_SECRET_KEY" >> .env
//                         echo "CONVEX_DEPLOYMENT=$CONVEX_DEPLOYMENT" >> .env
//                         echo "NEXT_PUBLIC_CONVEX_URL=$NEXT_PUBLIC_CONVEX_URL" >> .env
//                         echo "NEXT_PUBLIC_STREAM_API_KEY=$NEXT_PUBLIC_STREAM_API_KEY" >> .env
//                         echo "STREAM_SECRET_KEY=$STREAM_SECRET_KEY" >> .env

//                         yarn install
//                         yarn build
//                     """
//                 }
//             }
//         }

//         stage('SonarQube Analysis') {
//             steps {
//                 container('sonar-scanner') {
//                     withSonarQubeEnv('sonarqube-server') { 
//                         sh """
//                         sonar-scanner \\
//                         -Dsonar.projectKey=2401072_interview-stream \\
//                         -Dsonar.sources=. \\
//                         -Dsonar.host.url=${SONAR_HOST} \\
//                         -Dsonar.login=${SONAR_TOKEN}
//                         """
//                     }
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 container('dind') {
//                     sh "docker build -t ${IMAGE_NAME}:latest ."
//                 }
//             }
//         }

//        stage('Push to Nexus') {
//             steps {
//                 container('dind') {
//                     withCredentials([
//                         usernamePassword(
//                             credentialsId: NEXUS_CREDS_ID,
//                             usernameVariable: 'NEXUS_USER',
//                             passwordVariable: 'NEXUS_PASS'
//                         )
//                     ]) {
//                         // FIX: Script block to safely define Groovy variables (imageTag)
//                         script {
//                             def imageTag = "${NEXUS_URL}/${NEXUS_REPO}/${IMAGE_NAME}:${BUILD_NUMBER}"
//                             def latestTag = "${NEXUS_URL}/${NEXUS_REPO}/${IMAGE_NAME}:latest"

//                             sh """
//                                 echo "Logging in to Nexus Docker Registry: ${NEXUS_URL}"
//                                 docker login ${NEXUS_URL} -u $NEXUS_USER -p $NEXUS_PASS

//                                 # Update latest tag (FIX: Changed // to #)
                                
//                                 echo "Tagging image: ${imageTag}"
//                                 docker tag ${IMAGE_NAME}:latest ${imageTag}

//                                 echo "Pushing image to Nexus..."
//                                 docker push ${imageTag}

//                                 # Push the latest tag as well
//                                 docker tag ${IMAGE_NAME}:latest ${latestTag}
//                                 docker push ${latestTag}
//                             """
//                         }
//                     }
//                 }
//             }
//         }

//         stage('Create Namespace') {
//             steps {
//                 container('kubectl') {
//                     sh """
//                         kubectl create namespace ${K8S_NAMESPACE} || echo "Namespace already exists"
//                         kubectl get ns
//                     """
//                 }
//             }
//         }

//         // --- NEW STAGE: Create Nexus Pull Secret ---
//         stage('Create Nexus Pull Secret') {
//             steps {
//                 container('kubectl') {
//                     withCredentials([
//                         usernamePassword(
//                             credentialsId: NEXUS_CREDS_ID, // Using the Nexus Credential ID
//                             usernameVariable: 'NEXUS_USER',
//                             passwordVariable: 'NEXUS_PASS'
//                         )
//                     ]) {
//                         sh """
//                             echo "Attempting to create/update nexus-pull-secret..."
//                             # Note: --dry-run and apply are used to handle creation/update without failing if it exists (FIXED COMMENT)
//                             kubectl create secret docker-registry nexus-pull-secret \\
//                                 --docker-server=${NEXUS_URL} \\
//                                 --docker-username=$NEXUS_USER \\
//                                 --docker-password=$NEXUS_PASS \
//                                 -n ${K8S_NAMESPACE} || echo "Secret already exists or creation failed, proceeding..."
//                         """
//                     }
//                 }
//             }
//         }

//         stage('Deploy to Kubernetes') {
//             steps {
//                 container('kubectl') {
//                     sh """
//                         echo "🚀 Applying Kubernetes manifests in namespace ${K8S_NAMESPACE}..."
//                         kubectl apply -f k8s/ -n ${K8S_NAMESPACE}

//                         # Injecting sensitive environment variables directly into the K8s deployment
//                         echo "🔑 Injecting application secrets into deployment..."
//                         kubectl set env deployment/nextjs-deployment \\
//                         NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY \\
//                         CLERK_SECRET_KEY=$CLERK_SECRET_KEY \\
//                         CONVEX_DEPLOYMENT=$CONVEX_DEPLOYMENT \\
//                         NEXT_PUBLIC_CONVEX_URL=$NEXT_PUBLIC_CONVEX_URL \\
//                         NEXT_PUBLIC_STREAM_API_KEY=$NEXT_PUBLIC_STREAM_API_KEY \\
//                         STREAM_SECRET_KEY=$STREAM_SECRET_KEY \\
//                         -n ${K8S_NAMESPACE}
                        
//                         echo "🔍 Waiting for deployment rollout..."
                        
//                         kubectl rollout status deployment/nextjs-deployment -n ${K8S_NAMESPACE} --timeout=120s || true
//                     """
//                 }
//             }
//         }

//         stage('Debug Deployment') {
//             steps {
//                 container('kubectl') {
//                     sh """
//                         echo "--- Kubernetes Status (Debug) ---"
//                         kubectl get all -n ${K8S_NAMESPACE}

//                         # Show Pod details to find the exact error (ImagePullBackOff/CrashLoopBackOff)
//                         POD_NAME=\$(kubectl get pods -n ${K8S_NAMESPACE} -l app=nextjs-app -o jsonpath='{.items[0].metadata.name}')
//                         echo "--- Pod Description (\$POD_NAME) ---"
//                         kubectl describe pod \$POD_NAME -n ${K8S_NAMESPACE} | tail -n 25
//                     """
//                 }
//             }
// }
//     }
// }


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
        SONAR_HOST    = "http://my-sonarqube-sonarqube.sonarqube.svc.cluster.local:9000"
        
        // Nexus Settings (Screenshot ke hisaab se v2 prefix use kiya hai)
        NEXUS_URL     = "nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085"
        IMAGE_NAME    = "2401072_nextjs-project"
        K8S_NAMESPACE = "2401072"

        // Jenkins Credentials
        SONAR_TOKEN                       = credentials('sonar-token-2401072')
        NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = credentials('clerk-pub-2401072')
        CLERK_SECRET_KEY                  = credentials('clerk-secret-2401072')
        CONVEX_DEPLOYMENT                 = credentials('convex-deploy-2401072')
        NEXT_PUBLIC_CONVEX_URL            = credentials('convex-url-2401072')
        NEXT_PUBLIC_STREAM_API_KEY        = credentials('stream-pub-2401072')
        STREAM_SECRET_KEY                 = credentials('stream-secret-2401072')
    }

    stages {
        stage('Clean & Checkout') {
            steps {
                container('node') {
                    sh 'rm -rf * || true'
                    git url: 'https://github.com/sam160203/interview.git', branch: 'main'
                }
            }
        }

        stage('Install & Build') {
            steps {
                container('node') {
                    sh """
                        echo "Generating .env file..."
                        echo "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY}" > .env
                        echo "CLERK_SECRET_KEY=${CLERK_SECRET_KEY}" >> .env
                        echo "CONVEX_DEPLOYMENT=${CONVEX_DEPLOYMENT}" >> .env
                        echo "NEXT_PUBLIC_CONVEX_URL=${NEXT_PUBLIC_CONVEX_URL}" >> .env
                        echo "NEXT_PUBLIC_STREAM_API_KEY=${NEXT_PUBLIC_STREAM_API_KEY}" >> .env
                        echo "STREAM_SECRET_KEY=${STREAM_SECRET_KEY}" >> .env
                        yarn install && yarn build
                    """
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                container('sonar-scanner') {
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

        stage('Docker Build & Push') {
            steps {
                container('dind') {
                    sh """
                        sleep 10
                        echo "Logging into Nexus..."
                        docker login ${NEXUS_URL} -u student -p Imcc@2025
                        
                        echo "Building Image..."
                        docker build -t ${IMAGE_NAME}:latest .
                        
                        echo "Tagging and Pushing to v2 folder (as per Nexus screenshot)..."
                        docker tag ${IMAGE_NAME}:latest ${NEXUS_URL}/v2/${IMAGE_NAME}:v1
                        docker push ${NEXUS_URL}/v2/${IMAGE_NAME}:v1
                    """
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                container('kubectl') {
                    script {
                        try {
                            sh """
                                # 1. Namespace ensure karo
                                kubectl create namespace ${K8S_NAMESPACE} || true
                                
                                # 2. PULL SECRET FIX: Isme port 8085 aur student credentials check karo
                                kubectl delete secret nexus-pull-secret -n ${K8S_NAMESPACE} || true
                                kubectl create secret docker-registry nexus-pull-secret \\
                                    --docker-server=${NEXUS_URL} \\
                                    --docker-username=student \\
                                    --docker-password=Imcc@2025 \\
                                    --namespace=${K8S_NAMESPACE}
                                
                                # 3. Manifests apply karo
                                kubectl apply -f k8s/ -n ${K8S_NAMESPACE}

                                # 4. Image Update (v2 prefix ke sath)
                                kubectl set image deployment/nextjs-deployment nextjs-container=${NEXUS_URL}/v2/${IMAGE_NAME}:v1 -n ${K8S_NAMESPACE}
                                
                                # 5. Env Vars setup
                                kubectl set env deployment/nextjs-deployment \\
                                    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY} \\
                                    CLERK_SECRET_KEY=${CLERK_SECRET_KEY} \\
                                    CONVEX_DEPLOYMENT=${CONVEX_DEPLOYMENT} \\
                                    NEXT_PUBLIC_CONVEX_URL=${NEXT_PUBLIC_CONVEX_URL} \\
                                    NEXT_PUBLIC_STREAM_API_KEY=${NEXT_PUBLIC_STREAM_API_KEY} \\
                                    STREAM_SECRET_KEY=${STREAM_SECRET_KEY} -n ${K8S_NAMESPACE}
                                
                                echo "Waiting for rollout..."
                                kubectl rollout status deployment/nextjs-deployment -n ${K8S_NAMESPACE} --timeout=300s
                            """
                        } catch (Exception e) {
                            sh """
                                echo "DEPLOYMENT FAILED - Check Pull Secret permissions"
                                kubectl get events -n ${K8S_NAMESPACE} --sort-by='.lastTimestamp' | tail -n 10
                                exit 1
                            """
                        }
                    }
                }
            }
        }
    }
}