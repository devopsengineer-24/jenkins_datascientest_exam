pipeline {
    environment {
        DOCKER_ID = "devops24engineer"
        DOCKER_IMAGE_CASTS = "cast-service"
        DOCKER_IMAGE_MOVIES = "movie-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any
    post {
        failure {
            echo "JOB HAS FAILED"
            mail to: "jean.hublart@gmail.com",
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
    }
    stages {
        stage('Docker Build'){
            steps {
                script {
                    sh '''
                    docker rm -f jenkins-castsapi
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_CASTS:$DOCKER_TAG .
                    sleep 6
                    '''
                }
                script {
                    sh '''
                    docker rm -f jenkins-moviesapi
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIES:$DOCKER_TAG .
                    sleep 6
                    '''
                }
            }
        }
        stage('Docker run'){
            steps {
                script {
                    sh '''
                    docker run -d -p 81:80 --name jenkins-castsapi $DOCKER_ID/$DOCKER_IMAGE_CASTS:$DOCKER_TAG
                    sleep 10
                    '''
                }
                script {
                    sh '''
                    docker run -d -p 82:80 --name jenkins-moviesapi $DOCKER_ID/$DOCKER_IMAGE_MOVIES:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
        }
        stage('Test Acceptance'){
            steps {
                script {
                    sh '''
                    curl localhost:81
                    '''
                }
                script {
                    sh '''
                    curl localhost:82
                    '''
                }
            }
        }
        stage('Docker Push'){
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CASTS:$DOCKER_TAG
                    '''
                }
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIES:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy - Dev'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    // Add Kubernetes config
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    '''
                }
                script {
                    // Deploy Casts DB
                    sh '''
                    helm upgrade --install app castsdb --values=helmcharts/casts/castsdb/dev-values.yaml --namespace dev --create-namespace
                    '''
                }
                script {
                    // Deploy Casts API
                    sh '''
                    cp helmcharts/casts/castsapi/dev-values.yaml dev-values.yaml
                    cat dev-values.yaml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" dev-values.yaml
                    helm upgrade --install app castsapi --values=dev-values.yaml --namespace dev --create-namespace
                    '''
                }
                script {
                    // Deploy Movies DB
                    sh '''
                    helm upgrade --install app moviesdb --values=helmcharts/movies/moviesdb/dev-values.yaml --namespace dev --create-namespace
                    '''
                }
                script {
                    // Deploy Movies API
                    sh '''
                    rm dev-values.yaml
                    cp helmcharts/movies/moviesapi/dev-values.yaml dev-values.yaml
                    cat dev-values.yaml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" dev-values.yaml
                    helm upgrade --install app moviesapi --values=dev-values.yaml --namespace dev --create-namespace
                    '''
                }
                script {
                    // Deploy NGINX
                    sh '''
                    helm upgrade --install app nginx --values=helmcharts/nginx/dev-values.yaml --namespace dev --create-namespace
                    '''
                }
            }
        }
        // stage('Deploy - QA'){
        //     environment {
        //         KUBECONFIG = credentials("config")
        //     }
        //     steps {
        //         script {
        //             // Add Kubernetes config
        //             sh '''
        //             rm -Rf .kube
        //             mkdir .kube
        //             ls
        //             cat $KUBECONFIG > .kube/config
        //             '''
        //         }
        //     }
        //     steps {
        //         script {
        //             // Deploy Casts DB
        //             sh '''
        //             helm upgrade --install app castsdb --values=helmcharts/casts/castsdb/qa-values.yaml --namespace qa --create-namespace
        //             '''
        //         }
        //     }
        //     steps {
        //         script {
        //             // Deploy Casts API
        //             sh '''
        //             cp helmcharts/casts/castsapi/qa-values.yaml qa-values.yaml
        //             cat qa-values.yaml
        //             sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" qa-values.yaml
        //             helm upgrade --install app castsapi --values=qa-values.yaml --namespace qa --create-namespace
        //             '''
        //         }
        //     }
        //     steps {
        //         script {
        //             // Deploy Movies DB
        //             sh '''
        //             helm upgrade --install app moviesdb --values=helmcharts/movies/moviesdb/qa-values.yaml --namespace qa --create-namespace
        //             '''
        //         }
        //     }
        //     steps {
        //         script {
        //             // Deploy Movies API
        //             sh '''
        //             rm qa-values.yaml
        //             cp helmcharts/movies/moviesapi/qa-values.yaml qa-values.yaml
        //             cat qa-values.yaml
        //             sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" qa-values.yaml
        //             helm upgrade --install app moviesapi --values=qa-values.yaml --namespace qa --create-namespace
        //             '''
        //         }
        //     }
        //     steps {
        //         script {
        //             // Deploy NGINX
        //             sh '''
        //             helm upgrade --install app nginx --values=helmcharts/nginx/qa-values.yaml --namespace qa --create-namespace
        //             '''
        //         }
        //     }
        // }
        // stage('Deploy - Staging'){
        //     environment {
        //         KUBECONFIG = credentials("config")
        //     }
        //     steps {
        //         script {
        //             // Add Kubernetes config
        //             sh '''
        //             rm -Rf .kube
        //             mkdir .kube
        //             ls
        //             cat $KUBECONFIG > .kube/config
        //             '''
        //         }
        //     }
        //     steps {
        //         script {
        //             // Deploy Casts DB
        //             sh '''
        //             helm upgrade --install app castsdb --values=helmcharts/casts/castsdb/stag-values.yaml --namespace staging --create-namespace
        //             '''
        //         }
        //     }
        //     steps {
        //         script {
        //             // Deploy Casts API
        //             sh '''
        //             cp helmcharts/casts/castsapi/stag-values.yaml stag-values.yaml
        //             cat stag-values.yaml
        //             sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" stag-values.yaml
        //             helm upgrade --install app castsapi --values=stag-values.yaml --namespace staging --create-namespace
        //             '''
        //         }
        //     }
        //     steps {
        //         script {
        //             // Deploy Movies DB
        //             sh '''
        //             helm upgrade --install app moviesdb --values=helmcharts/movies/moviesdb/stag-values.yaml --namespace staging --create-namespace
        //             '''
        //         }
        //     }
        //     steps {
        //         script {
        //             // Deploy Movies API
        //             sh '''
        //             rm stag-values.yaml
        //             cp helmcharts/movies/moviesapi/stag-values.yaml stag-values.yaml
        //             cat stag-values.yaml
        //             sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" stag-values.yaml
        //             helm upgrade --install app moviesapi --values=stag-values.yaml --namespace staging --create-namespace
        //             '''
        //         }
        //     }
        //     steps {
        //         script {
        //             // Deploy NGINX
        //             sh '''
        //             helm upgrade --install app nginx --values=helmcharts/nginx/stag-values.yaml --namespace staging --create-namespace
        //             '''
        //         }
        //     }
        // }
    }
}