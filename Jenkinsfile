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
//         SONAR_HOST    = "http://my-sonarqube-sonarqube.sonarqube.svc.cluster.local:9000"
        
//         // Nexus Settings (Screenshot ke hisaab se v2 prefix use kiya hai)
//         NEXUS_URL     = "nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085"
//         IMAGE_NAME    = "2401072_nextjs-project"
//         K8S_NAMESPACE = "2401072"

//         // Jenkins Credentials
//         SONAR_TOKEN                       = credentials('sonar-token-2401072')
//         NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = credentials('clerk-pub-2401072')
//         CLERK_SECRET_KEY                  = credentials('clerk-secret-2401072')
//         CONVEX_DEPLOYMENT                 = credentials('convex-deploy-2401072')
//         NEXT_PUBLIC_CONVEX_URL            = credentials('convex-url-2401072')
//         NEXT_PUBLIC_STREAM_API_KEY        = credentials('stream-pub-2401072')
//         STREAM_SECRET_KEY                 = credentials('stream-secret-2401072')
//     }

//     stages {
//         stage('Clean & Checkout') {
//             steps {
//                 container('node') {
//                     sh 'rm -rf * || true'
//                     git url: 'https://github.com/sam160203/interview.git', branch: 'main'
//                 }
//             }
//         }

//         stage('Install & Build') {
//             steps {
//                 container('node') {
//                     sh """
//                         echo "Generating .env file..."
//                         echo "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY}" > .env
//                         echo "CLERK_SECRET_KEY=${CLERK_SECRET_KEY}" >> .env
//                         echo "CONVEX_DEPLOYMENT=${CONVEX_DEPLOYMENT}" >> .env
//                         echo "NEXT_PUBLIC_CONVEX_URL=${NEXT_PUBLIC_CONVEX_URL}" >> .env
//                         echo "NEXT_PUBLIC_STREAM_API_KEY=${NEXT_PUBLIC_STREAM_API_KEY}" >> .env
//                         echo "STREAM_SECRET_KEY=${STREAM_SECRET_KEY}" >> .env
//                         yarn install && yarn build
//                     """
//                 }
//             }
//         }

//         stage('SonarQube Analysis') {
//             steps {
//                 container('sonar-scanner') {
//                     sh """
//                         sonar-scanner \\
//                         -Dsonar.projectKey=2401072_interview-stream \\
//                         -Dsonar.sources=. \\
//                         -Dsonar.host.url=${SONAR_HOST} \\
//                         -Dsonar.login=${SONAR_TOKEN}
//                     """
//                 }
//             }
//         }

//         stage('Docker Build & Push') {
//             steps {
//                 container('dind') {
//                     sh """
//                         sleep 10
//                         echo "Logging into Nexus..."
//                         docker login ${NEXUS_URL} -u student -p Imcc@2025
                        
//                         echo "Building Image..."
//                         docker build -t ${IMAGE_NAME}:latest .
                        
//                         echo "Tagging and Pushing to v2 folder (as per Nexus screenshot)..."
//                         docker tag ${IMAGE_NAME}:latest ${NEXUS_URL}/v2/${IMAGE_NAME}:v1
//                         docker push ${NEXUS_URL}/v2/${IMAGE_NAME}:v1
//                     """
//                 }
//             }
//         }

//         stage('Kubernetes Deploy') {
//             steps {
//                 container('kubectl') {
//                     script {
//                         try {
//                             sh """
//                                 # 1. Namespace ensure karo
//                                 kubectl create namespace ${K8S_NAMESPACE} || true
                                
//                                 # 2. PULL SECRET FIX: Isme port 8085 aur student credentials check karo
//                                 kubectl delete secret nexus-pull-secret -n ${K8S_NAMESPACE} || true
//                                 kubectl create secret docker-registry nexus-pull-secret \\
//                                     --docker-server=${NEXUS_URL} \\
//                                     --docker-username=student \\
//                                     --docker-password=Imcc@2025 \\
//                                     --namespace=${K8S_NAMESPACE}
                                
//                                 # 3. Manifests apply karo
//                                 kubectl apply -f k8s/ -n ${K8S_NAMESPACE}

//                                 # 4. Image Update (v2 prefix ke sath)
//                                 kubectl set image deployment/nextjs-deployment nextjs-container=${NEXUS_URL}/v2/${IMAGE_NAME}:v1 -n ${K8S_NAMESPACE}
                                
//                                 # 5. Env Vars setup
//                                 kubectl set env deployment/nextjs-deployment \\
//                                     NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY} \\
//                                     CLERK_SECRET_KEY=${CLERK_SECRET_KEY} \\
//                                     CONVEX_DEPLOYMENT=${CONVEX_DEPLOYMENT} \\
//                                     NEXT_PUBLIC_CONVEX_URL=${NEXT_PUBLIC_CONVEX_URL} \\
//                                     NEXT_PUBLIC_STREAM_API_KEY=${NEXT_PUBLIC_STREAM_API_KEY} \\
//                                     STREAM_SECRET_KEY=${STREAM_SECRET_KEY} -n ${K8S_NAMESPACE}
                                
//                                 echo "Waiting for rollout..."
//                                 kubectl rollout status deployment/nextjs-deployment -n ${K8S_NAMESPACE} --timeout=300s
//                             """
//                         } catch (Exception e) {
//                             sh """
//                                 echo "DEPLOYMENT FAILED - Check Pull Secret permissions"
//                                 kubectl get events -n ${K8S_NAMESPACE} --sort-by='.lastTimestamp' | tail -n 10
//                                 exit 1
//                             """
//                         }
//                     }
//                 }
//             }
//         }
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
    command: ["cat"]
    tty: true
  - name: sonar-scanner
    image: sonarsource/sonar-scanner-cli
    command: ["cat"]
    tty: true
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["cat"]
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
        // Naye placeholders ke hisaab se values
        NAMESPACE         = "2401072"             // <NAMESPACE>
        APP_NAME          = "nextjs-app"          // <APP_NAME>
        APP_PORT          = "3000"                // <APP_PORT>
        PROJECT_NAMESPACE = "v2"                  // Nexus folder
        TAG               = "v1"                  // <TAG>
        
        REGISTRY_URL      = "nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085"
        SONAR_HOST_URL    = "http://my-sonarqube-sonarqube.sonarqube.svc.cluster.local:9000"
        SONAR_PROJECT     = "2401072_interview-stream"

        // Credentials
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
                    sh "sonar-scanner -Dsonar.projectKey=${SONAR_PROJECT} -Dsonar.sources=. -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        stage('Docker Build - Tag - Push') {
            steps {
                container('dind') {
                    sh """
                        while (! docker stats --no-stream ); do sleep 1; done
                        docker login ${REGISTRY_URL} -u student -p Imcc@2025
                        
                        docker build -t ${APP_NAME}:${TAG} .
                        
                        # Tagging as per Nexus screenshot structure
                        docker tag ${APP_NAME}:${TAG} ${REGISTRY_URL}/${PROJECT_NAMESPACE}/${NAMESPACE}_nextjs-project:${TAG}
                        docker push ${REGISTRY_URL}/${PROJECT_NAMESPACE}/${NAMESPACE}_nextjs-project:${TAG}
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
                                kubectl create namespace ${NAMESPACE} || true
                                
                                # Pull Secret name is 'nexus-secret' (Do not change)
                                kubectl delete secret nexus-secret -n ${NAMESPACE} || true
                                kubectl create secret docker-registry nexus-secret \\
                                    --docker-server=${REGISTRY_URL} \\
                                    --docker-username=student \\
                                    --docker-password=Imcc@2025 -n ${NAMESPACE}
                                
                                kubectl apply -f k8s/ -n ${NAMESPACE}

                                # Update Image path in deployment
                                kubectl set image deployment/${APP_NAME}-deployment ${APP_NAME}=${REGISTRY_URL}/${PROJECT_NAMESPACE}/${NAMESPACE}_nextjs-project:${TAG} -n ${NAMESPACE}
                                
                                # Set Env Secrets
                                kubectl set env deployment/${APP_NAME}-deployment \\
                                    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY} \\
                                    CLERK_SECRET_KEY=${CLERK_SECRET_KEY} \\
                                    CONVEX_DEPLOYMENT=${CONVEX_DEPLOYMENT} \\
                                    NEXT_PUBLIC_CONVEX_URL=${NEXT_PUBLIC_CONVEX_URL} \\
                                    NEXT_PUBLIC_STREAM_API_KEY=${NEXT_PUBLIC_STREAM_API_KEY} \\
                                    STREAM_SECRET_KEY=${STREAM_SECRET_KEY} -n ${NAMESPACE}
                                
                                echo "Waiting for rollout..."
                                kubectl rollout status deployment/${APP_NAME}-deployment -n ${NAMESPACE} --timeout=300s
                            """
                        } catch (Exception e) {
                            sh "kubectl get pods -n ${NAMESPACE} && exit 1"
                        }
                    }
                }
            }
        }
    }
}