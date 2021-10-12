pipeline {
    agent any
    environment {
        APP_GIT_URL = "https://github.com/pornpasok/demo-app-k8s.git"
        APP_BRANCH = "main"
        APP_TAG = "latest"
        APP_NAME = "tono"
        APP_PORT = "3000"
        DEV_PROJECT = "workshop"
        SQ_SERVER = "https://sq.7-11.io"
        NEXUS_SERVER = "repository.7-11.io"
        NEXUS_SERVER_PULL_PORT = "5000"
        NEXUS_SERVER_PUSH_PORT = "5001"
        DOMAIN_NAME = "7-11.tech"

    }
    
    stages {
        stage('Clean') {
            steps {
                echo 'Clean Workspace'
                sh '''
                    rm -rf *
                '''
                echo 'Clean App'
                sh '''
                    if kubectl get deployment ${APP_NAME} -n ${DEV_PROJECT}; then echo exists && kubectl delete deployment ${APP_NAME} -n ${DEV_PROJECT} && kubectl delete svc ${APP_NAME} -n ${DEV_PROJECT}; else echo no deployment; fi
                '''
            }
        }

        stage('SCM') {
            steps {
                echo 'Pull code from SCM'
                git(
                    url: "${APP_GIT_URL}",
                    //credentialsId: 'github-cicd',
                    branch: "${APP_BRANCH}"
                )
            }
        }

        stage('Source Code Scan') {
            steps {
                echo 'Source Code Scan SonarQube'
                withCredentials([string(credentialsId: 'sq-token', variable: 'SQ_TOKEN')]) {
                    sh '''
                        /sonar-scanner-4.6.2.2472-linux/bin/sonar-scanner \
                        -Dsonar.projectKey="${APP_NAME}" \
                        -Dsonar.sources=. \
                        -Dsonar.host.url="${SQ_SERVER}" \
                        -Dsonar.login="$SQ_TOKEN"
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Build Docker Images'
                sh '''
                    docker build -t ${NEXUS_SERVER}:${NEXUS_SERVER_PULL_PORT}/${APP_NAME} .
                '''
            }
        }

        stage('Push Images to Artifactory') {
            steps {
                echo 'Push Images to Artifactory'
                withCredentials([usernamePassword(credentialsId: 'sds-nexus', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        # Docker Login Pull
                        echo docker login to ${NEXUS_SERVER}:${NEXUS_SERVER_PULL_PORT}
                        docker login ${NEXUS_SERVER}:${NEXUS_SERVER_PULL_PORT} -u $DOCKER_USER -p $DOCKER_PASSWORD 
                        # Docker Login Push
                        echo docker login to ${NEXUS_SERVER}:${NEXUS_SERVER_PUSH_PORT}
                        docker login ${NEXUS_SERVER}:${NEXUS_SERVER_PUSH_PORT} -u $DOCKER_USER -p $DOCKER_PASSWORD 
            
                        # Docker Tag
                        docker tag ${NEXUS_SERVER}:${NEXUS_SERVER_PULL_PORT}/${APP_NAME}:latest ${NEXUS_SERVER}:${NEXUS_SERVER_PUSH_PORT}/${APP_NAME}:latest

                        # Docker Push
                        docker push ${NEXUS_SERVER}:${NEXUS_SERVER_PUSH_PORT}/${APP_NAME}:latest
                    '''
                }
            }
        }

        stage('Deploy to Dev ENV') {
            steps {
                echo 'Deploy to Dev ENV'
                sh '''
                    kubectl create deployment ${APP_NAME} -n ${DEV_PROJECT} --image=${NEXUS_SERVER}:${NEXUS_SERVER_PULL_PORT}/${APP_NAME}
                '''

            }
        }

        stage('Expose Service to Dev ENV') {
            steps {
                echo 'Expose Service to Dev ENV'
                sh '''
                    kubectl expose deployment ${APP_NAME} -n ${DEV_PROJECT} --port=80 --target-port=${APP_PORT}
                '''

            }
        }

        stage('Create Ingress') {
            steps {
                echo 'Create Ingress'
                sh '''
                    cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ${APP_NAME}
  namespace: ${DEV_PROJECT}
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: ${APP_NAME}-${DEV_PROJECT}.${DOMAIN_NAME}
      http:
        paths:
          - backend:
              serviceName: ${APP_NAME}
              servicePort: 80
EOF
                '''

            }
        }


        stage('Scale App') {
            steps {
                echo 'Scale App'
                sh '''
                    kubectl scale deployment ${APP_NAME} -n ${DEV_PROJECT} --replicas=2
                '''
            }
        }

        // stage('Get App Endpoint') {
        //     steps {
        //         echo 'Get App Endpoint'
        //         sh '''
        //             sleep 30
        //             kubectl get service ${APP_NAME} -n ${DEV_PROJECT} | tail -n +2 | awk '{print $4}'
        //             ELB_ENDPOINT=$(kubectl get service ${APP_NAME} -n ${DEV_PROJECT} | tail -n +2 | awk '{print $4}')
        //             echo "ELB_ENDPOINT: http://$ELB_ENDPOINT"
        //         '''
        //     }
        // }

        stage('Check App') {
            steps {
                echo 'Check App'
                sh '''
                    sleep 30
                    # AWS ELB 
                    #STATUSCODE=$(curl -s -o /dev/null -I -w "%{http_code}" http://$ELB_ENDPOINT)

                    # Service
                    STATUSCODE=$(curl -s -o /dev/null -I -w "%{http_code}" http://${APP_NAME}-${DEV_PROJECT}.${DOMAIN_NAME})
                    if test $STATUSCODE -ne 200; then echo ERROR:$STATUSCODE && exit 1; else echo SUCCESS; fi;
                '''
            }
        }

    }
}