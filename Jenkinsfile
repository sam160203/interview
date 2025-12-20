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
//     command: ["cat"]
//     tty: true
//   - name: sonar-scanner
//     image: sonarsource/sonar-scanner-cli
//     command: ["cat"]
//     tty: true
//   - name: kubectl
//     image: bitnami/kubectl:latest
//     command: ["cat"]
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
//         NAMESPACE         = "2401072"
//         APP_NAME          = "nextjs-app"
//         PROJECT_NAMESPACE = "v2"
//         TAG               = "v1"
//         REGISTRY_URL      = "nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085"
//         SONAR_HOST_URL    = "http://my-sonarqube-sonarqube.sonarqube.svc.cluster.local:9000"
//         SONAR_PROJECT     = "2401072_interview-stream"

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
//                     sh "sonar-scanner -Dsonar.projectKey=${SONAR_PROJECT} -Dsonar.sources=. -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_TOKEN}"
//                 }
//             }
//         }

//         stage('Docker Build - Tag - Push') {
//             steps {
//                 container('dind') {
//                     sh """
//                         while (! docker stats --no-stream ); do sleep 1; done
//                         docker login ${REGISTRY_URL} -u student -p Imcc@2025
//                         docker build -t ${APP_NAME}:${TAG} .
//                         docker tag ${APP_NAME}:${TAG} ${REGISTRY_URL}/${PROJECT_NAMESPACE}/${NAMESPACE}_nextjs-project:${TAG}
//                         docker push ${REGISTRY_URL}/${PROJECT_NAMESPACE}/${NAMESPACE}_nextjs-project:${TAG}
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
//                                 kubectl create namespace ${NAMESPACE} || true
                                
//                                 # Purane deployments ko clean karo (Corrected naming)
//                                 kubectl delete deployment nextjs-app-deployment -n ${NAMESPACE} || true
//                                 kubectl delete deployment nextjs-deployment -n ${NAMESPACE} || true
                                
//                                 # Pull Secret for Nexus
//                                 kubectl delete secret nexus-secret -n ${NAMESPACE} || true
//                                 kubectl create secret docker-registry nexus-secret --docker-server=${REGISTRY_URL} --docker-username=student --docker-password=Imcc@2025 -n ${NAMESPACE}
                                
//                                 # Apply manifests from k8s folder
//                                 kubectl apply -f k8s/ -n ${NAMESPACE}

//                                 # Force update the image path (Strictly aligned with Tag/Push stage)
//                                 kubectl set image deployment/nextjs-app-deployment nextjs-app=${REGISTRY_URL}/${PROJECT_NAMESPACE}/${NAMESPACE}_nextjs-project:${TAG} -n ${NAMESPACE}
                                
//                                 # Update Env Variables
//                                 kubectl set env deployment/nextjs-app-deployment \\
//                                     NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY} \\
//                                     CLERK_SECRET_KEY=${CLERK_SECRET_KEY} \\
//                                     CONVEX_DEPLOYMENT=${CONVEX_DEPLOYMENT} \\
//                                     NEXT_PUBLIC_CONVEX_URL=${NEXT_PUBLIC_CONVEX_URL} \\
//                                     NEXT_PUBLIC_STREAM_API_KEY=${NEXT_PUBLIC_STREAM_API_KEY} \\
//                                     STREAM_SECRET_KEY=${STREAM_SECRET_KEY} -n ${NAMESPACE}
                                
//                                 echo "Waiting for rollout..."
//                                 kubectl rollout status deployment/nextjs-app-deployment -n ${NAMESPACE} --timeout=300s
//                             """
//                         } catch (Exception e) {
//                             sh """
//                                 echo "Deployment failed! Printing pods status..."
//                                 kubectl get pods -n ${NAMESPACE}
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
        NAMESPACE         = "2401072"
        APP_NAME          = "nextjs-app"
        TAG               = "v1"
        REGISTRY_URL      = "nexus-service-for-docker-hosted-registry.nexus.svc.cluster.local:8085"
        SONAR_HOST_URL    = "http://my-sonarqube-sonarqube.sonarqube.svc.cluster.local:9000"
        SONAR_PROJECT     = "2401072_interview-stream"

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
                        yarn install --network-timeout 1000000 && yarn build
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
                        
                        # Fix: Direct tagging without extra PROJECT_NAMESPACE
                        docker tag ${APP_NAME}:${TAG} ${REGISTRY_URL}/${NAMESPACE}_nextjs-project:${TAG}
                        docker push ${REGISTRY_URL}/${NAMESPACE}_nextjs-project:${TAG}
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
                                
                                # Clean start
                                kubectl delete deployment nextjs-app-deployment -n ${NAMESPACE} --wait=false || true
                                
                                # Refresh Secret
                                kubectl delete secret nexus-secret -n ${NAMESPACE} || true
                                kubectl create secret docker-registry nexus-secret --docker-server=${REGISTRY_URL} --docker-username=student --docker-password=Imcc@2025 -n ${NAMESPACE}
                                
                                # Apply manifests from k8s folder
                                kubectl apply -f k8s/ -n ${NAMESPACE}

                                # Fix: Image path matches exactly what was pushed in stage 4
                                # Fetch Service ClusterIP to bypass Node DNS issues
                                REGISTRY_IP=\$(kubectl get svc nexus-service-for-docker-hosted-registry -n nexus -o jsonpath='{.spec.clusterIP}')
                                kubectl set image deployment/nextjs-app-deployment nextjs-app=\${REGISTRY_IP}:8085/${NAMESPACE}_nextjs-project:${TAG} -n ${NAMESPACE}
                                
                                # Update Env Variables
                                kubectl set env deployment/nextjs-app-deployment \\
                                    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY} \\
                                    CLERK_SECRET_KEY=${CLERK_SECRET_KEY} \\
                                    CONVEX_DEPLOYMENT=${CONVEX_DEPLOYMENT} \\
                                    NEXT_PUBLIC_CONVEX_URL=${NEXT_PUBLIC_CONVEX_URL} \\
                                    NEXT_PUBLIC_STREAM_API_KEY=${NEXT_PUBLIC_STREAM_API_KEY} \\
                                    STREAM_SECRET_KEY=${STREAM_SECRET_KEY} -n ${NAMESPACE}
                                
                                echo "Waiting for rollout..."
                                kubectl rollout status deployment/nextjs-app-deployment -n ${NAMESPACE} --timeout=180s
                            """
                        } catch (Exception e) {
                            sh """
                                echo "Rollout failed. Printing Pod Events for Debugging:"
                                kubectl describe pods -l app=nextjs-app -n ${NAMESPACE} | grep -A 30 Events
                                exit 1
                            """
                        }
                    }
                }
            }
        }
    }
}