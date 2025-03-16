pipeline {
    environment {
        DOCKER_ID = "aiberber"
        DOCKER_MOVIE_IMAGE = "movie_service"
        DOCKER_CAST_IMAGE = "cast_service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any
    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    sh '''
                    cd movie-service
                    docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG .
                    cd ../cast-service
                    docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG .
                    '''
                }
            }
        }
        stage('Push Docker Images') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy to Dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    export PATH=$PATH:/usr/local/bin
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp fastapiapp/values.yaml values.yml
                    sed -i "s+repository: sajjadhz/fastapiapp+repository: $DOCKER_ID/$DOCKER_MOVIE_IMAGE+g" values.yml
                    sed -i "s+tag: \\"latest\\"+tag: $DOCKER_TAG+g" values.yml
                    helm upgrade --install movie-service fastapiapp --values=values.yml --namespace dev
                    cp fastapiapp/values.yaml values.yml
                    sed -i "s+repository: sajjadhz/fastapiapp+repository: $DOCKER_ID/$DOCKER_CAST_IMAGE+g" values.yml
                    sed -i "s+tag: \\"latest\\"+tag: $DOCKER_TAG+g" values.yml
                    helm upgrade --install cast-service fastapiapp --values=values.yml --namespace dev
                    '''
                }
            }
        }
        stage('Deploy to QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    export PATH=$PATH:/usr/local/bin
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp fastapiapp/values.yaml values.yml
                    sed -i "s+repository: sajjadhz/fastapiapp+repository: $DOCKER_ID/$DOCKER_MOVIE_IMAGE+g" values.yml
                    sed -i "s+tag: \\"latest\\"+tag: $DOCKER_TAG+g" values.yml
                    helm upgrade --install movie-service fastapiapp --values=values.yml --namespace qa
                    cp fastapiapp/values.yaml values.yml
                    sed -i "s+repository: sajjadhz/fastapiapp+repository: $DOCKER_ID/$DOCKER_CAST_IMAGE+g" values.yml
                    sed -i "s+tag: \\"latest\\"+tag: $DOCKER_TAG+g" values.yml
                    helm upgrade --install cast-service fastapiapp --values=values.yml --namespace qa
                    '''
                }
            }
        }
        stage('Deploy to Staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    export PATH=$PATH:/usr/local/bin
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp fastapiapp/values.yaml values.yml
                    sed -i "s+repository: sajjadhz/fastapiapp+repository: $DOCKER_ID/$DOCKER_MOVIE_IMAGE+g" values.yml
                    sed -i "s+tag: \\"latest\\"+tag: $DOCKER_TAG+g" values.yml
                    helm upgrade --install movie-service fastapiapp --values=values.yml --namespace staging
                    cp fastapiapp/values.yaml values.yml
                    sed -i "s+repository: sajjadhz/fastapiapp+repository: $DOCKER_ID/$DOCKER_CAST_IMAGE+g" values.yml
                    sed -i "s+tag: \\"latest\\"+tag: $DOCKER_TAG+g" values.yml
                    helm upgrade --install cast-service fastapiapp --values=values.yml --namespace staging
                    '''
                }
            }
        }
        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Deploy to Production?', ok: 'Yes'
                }
                script {
                    sh '''
                    export PATH=$PATH:/usr/local/bin
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp fastapiapp/values.yaml values.yml
                    sed -i "s+repository: sajjadhz/fastapiapp+repository: $DOCKER_ID/$DOCKER_MOVIE_IMAGE+g" values.yml
                    sed -i "s+tag: \\"latest\\"+tag: $DOCKER_TAG+g" values.yml
                    helm upgrade --install movie-service fastapiapp --values=values.yml --namespace prod
                    cp fastapiapp/values.yaml values.yml
                    sed -i "s+repository: sajjadhz/fastapiapp+repository: $DOCKER_ID/$DOCKER_CAST_IMAGE+g" values.yml
                    sed -i "s+tag: \\"latest\\"+tag: $DOCKER_TAG+g" values.yml
                    helm upgrade --install cast-service fastapiapp --values=values.yml --namespace prod
                    '''
                }
            }
        }
    }
}
